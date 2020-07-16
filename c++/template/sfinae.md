## 简介
- Substitution Failure Is Not An Error（匹配失败不是错误）
- 由于类型推断而引起的替换失败不会立刻引起编译错误，而是会进行一个类似于“编译期重载”的操作
- SFINAE使得C++可以在编译期进行一些方法的重载，这个是运行期0开销的，匹配失败的方法直接就不编译

## 需要注意的点
一旦类型匹配成功，模板就会被实例化，如果在模板函数内部发生了编译错误，**编译会失败！**，也就是说，**模板一旦实例化，SFINAE就无法让错误再被忽略了**

## 写一个is_class
```
template <typename T>
class is_class
{
private:
    template<typename U>
    static char helper(int U::*); // 只能指向int类型的U的成员变量

    template<typename U>
    static int helper(...);


public:
    constexpr static bool value = sizeof(helper<T>(nullptr)) == sizeof(char);
};
```

## enable_if的作用
触发SFINAE机制，可以使用enable_if_t模板使得某些模板在编译期不展开，实现一定的编译期多态的功能，例子如下：
```
// 这样只是保证了有sort名字的函数而已
//对函数的签名并没有要求
template <typename T>
class hasSort {
private:
    template <typename U>
    static char test(decltype(&U::sort));

    template <typename U>
    static int test(...);

public:
    const static bool value = sizeof(test<T>(nullptr)) == sizeof(char);
};


template <typename Container>
class hasReserve {
private:
    struct good { char dummy; };
    struct bad { char dummy[2]; };

    // 用两个模板参数保证了sort(size_t)一定是来自于Container的
    template <typename T, void(T::*)(size_t)>
    class Sfinae;

    // 匹配test时，要先匹配Sfinae，匹配Sfinae就必须要求U有带size_t的reserve函数
    template <typename U>
    static good test(Sfinae<U, &U::reserve>*); // 这里总要传个什么作为形参，就传指针好了

    template <typename U>
    static bad test(...); // 如果第一个匹配失败了，就匹配这个

public:
    const static bool value = sizeof(test<Container>(nullptr)) == sizeof(good);

};

template <typename Container>
std::enable_if_t<hasReserve<Container>::value, void>
BestReserve(Container &c, size_t s) {
    c.reserve(s);
    std::cout << "member reserve" << std::endl;
}

template <typename Container>
std::enable_if_t<!hasReserve<Container>::value, void>
BestReserve(Container& c, size_t s) {
    std::cout << "global reserve" << std::endl;
}


template <typename Container>
std::enable_if_t<hasSort<Container>::value, void>
BestSort(Container& c) {
    c.sort();
    std::cout << "member sort" << std::endl;
}

template <typename Container>
std::enable_if_t<!hasSort<Container>::value, void>
BestSort(Container& c) {
    std::cout << "global sort" << std::endl;
}




struct Foo {
    void sort() {

    }
};

struct Bar {
    void reserve(size_t) {

    }
};

int main()
{
    Foo f;
    Bar b;

    BestSort(f);
    BestSort(b);

    BestReserve(f, 0);
    BestReserve(b, 0);

    return 0;
}

```

1. 对于任意类型，如果它有sort方法，在BestSort里就优先调用它，如果没有再做其他处理，比如调用全局的sort,但是注意这里使用函数签名来匹配成员函数，好像没办法支持对函数参数类型的匹配；
2. 对于任意类型，如果它有reserve方法，在BestReserve里就优先调用它，如果没有再做其他处理，比如调用全局的Reserve，注意这里的reserve是带参数的，所以要做特殊处理（见class Sifnae）；
3. 此外enable_if_t的定义是：
```
template< bool B, class T = void >
using enable_if_t = typename enable_if<B,T>::type;

template<class T>
struct enable_if<true, T> { typedef T type; };

template<bool B, class T = void>
struct enable_if {};
```
实际上就是如果传入的bool值时false的话，这个type就不存在，编译期的模板匹配就会失败，可以借助这个特性达到一定的编译期多态的功能。