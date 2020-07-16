## 运行期多态虚函数的缺陷
因为虚函数的调用涉及到运行时查询虚表，所以调用成本会较高。而对于普通的成员函数，一般编译器都会将其内联，据测试，虚函数的动态绑定的开销相较于普通成员函数可以达到10倍左右。

## CRTP一般长什么样？
```
template <typename Child> 
class Base 
{
    ...
};
class Derived : public Base<Derived> 
{ // 派生类将自己的类型传给基类模板
    ...
};
```
可以看到，相较于普通的公有继承，这里的基类是通过编译期的Child类型来持有Derived派生类的类型信息的，而派生类在继承自基类的时候，需要将自己的类型信息给它。

## 实现类似于虚函数的功能
函数实现代码如下：
```
template <typename Child>
class Base
{
public:
    Base() = default;
    void f(int i) 
    {
        static_cast<Child*>(this)->f(i);
    }
protected:
    int i_ = 0;
};

class Derived : public Base<Derived>
{
public:
    void f(int i)
    {
        i_ += i;
        std::cout << x_;
    }
private:
    int x_ = 0;
};
```

调用方法和基于虚函数的动态绑定类似，也必须基于指针，如果直接将Derived赋值给Base对象，也会造成对象切割（Object slicing）
```
Base<D> *base = new D();
base->f(); //这里会调用Derived的f

```


## 问问为什么
为什么这里通过将this强转成Child*就可以调用到Derived::f了呢？
在类继承层次中，如果父类和子类都有同名同签名的函数，则基类的函数在派生类中会被**隐藏**，**如果在Base::f中调用this->f，会调用自身，而如果将this强转为Child*，就调用Derived::f了**，CRTP模式就是巧妙的利用了这个编译器的特点，在编译期就完成了类型的静态绑定。
此外这里的多态调用可是**没有额外开销的**，在编译器看来，静态多态的f调用和调用一个普通的成员函数没什么两样，甚至还可能把它内联了……


## 实现纯虚函数呢？
```
template <typename Child>
class Base
{
public:
    Base() = default;
    void f(int i) 
    {
        static_cast<Child*>(this)->f(i);
    }
protected:
    int i_ = 0;
};

class Derived : public Base<Derived>
{

};
```
看起来基于CRTP的编译期多态无法实现运行期检查子类是否实现了f，如果子类没有f，Base::f中就会递归调用自己（因为```static_cast<Child*>(this)->f(i)```解析到了自己身上）。
并且这样的错误不会发生在编译期，只会在运行期爆栈SIGSEGV。这样当然不是很好，一个编译期的错误在运行期才能发现。

可以通过让基类的f有一个默认的实现来阻止运行期错误的发生在，代码如下：
```
template <typename Child>
class Base
{
public:
    Base() = default;
    void f(int i) 
    {
        static_cast<Child*>(this)->f(i);
    }

    void fImpl(int i)
    {
        // default 
    }
protected:
    int i_ = 0;
};

class Derived : public Base<Derived>
{
public:
    void fImpl(int i)
    {
        // derived
    }
};
```
这样虽然不能在编译期给出没有实现纯虚函数的错误，但是至少给了一个默认实现，阻止了运行期错误的发生。

## 如何实现虚析构？
考虑如下代码：
```
template <typename D>
void apply(Base<D>* b) 
{
    // 多态操作
}

{
    std::vector<Derived> v;
    v.push_back(Derived(...))
    ...
    apply(&v[0]);
} // v中的每个D析构，这样的析构是没有问题的
```
但是如果这样呢？
```
B<D>* base = new Derived;
delete b; // 只会析构基类部分，而不会调用Derived的析构函数
```
上述代码甚至连编译错误都不会产生，我们再来看个更糟糕的：
```
template <typename Derived>
class Base {
public:
    ~Base() { static_cast<Derived*>(this)->~Derived(); } // 大错特错
}
```
因为C++对象的析构顺序是先析构派生类再析构基类，所以**在基类的析构函数中，this的真实类型不再是派生类了！**，这样将基类指针转换为派生类指针再调用其成员函数，会造成未定义行为！
幸运的是这个问题并不是无解的：
1. 可以写一个通用的、持有派生类类型的模板函数来完成统一的析构，但是就不能用delete了。
```
template <typename D>
void destroy(B<D>* b) 
{
    delete static_cast<D*>(b);
}
```

2. 妥协，使用虚析构函数，不过相对于将很多成员函数做成virtual的，只对析构函数付出一点成本也是值得的。