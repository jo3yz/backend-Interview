## 双向强引用
这是比较常见的情况，会导致生命周期被无限延长

## 与std::bind联用时也可能会意外延长生命周期
考虑这样的语句：
```
auto ptr = make_shared<Foo>();
auto func = std::bind(Foo::Bar, ptr);
```
这里的func会将bind传入的参数拷贝一份，可能会意外延长shared_ptr的周期

## shared_ptr的析构动作在创建时被捕获
- 这需要首先捋清楚基类的虚析构函数到底是干嘛用的：当delete基类指针时，可以正确的调用到派生类的析构函数而不造成资源泄露
- 这意味着即使Base类的析构函数不是virtual，用shared_ptr<Base>管理Derive类的内存，当引用计数为0需要释放内存时，也可以正确的调用派生类的析构函数
- 这个实现和OO范式的方法不太一样，OO的方法猜测是通过虚表找到其实际类型的虚构造函数，而shared_ptr<Base>构造时就记下了实际管理的类型，用于初始化一个deleter<Derive>，在析构时将调用这个deleter，把基类指针直接static_cast为派生类指针，就不需要通过虚表了
- 也就是说，shared_ptr<void>可以安全持有任何对象

## 析构是同步的
在哪个线程中use_count变为0，就在哪个线程析构