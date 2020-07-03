## enable_shared_from_this<T>的作用
shared_ptr<T>是通过对每个内存附加单独的控制块来实现的，控制块上有引用计数。所以在外部可以这么使用：
```
auto ptr = std::make_shared<Foo>(bar);
```
或者
```
std::shared_ptr<Foo> ptr(new Foo(bar));
```
但是shared_ptr的内存的管理权限都是在Foo的外部实现的。如果需要在Foo的成员函数中，返回包裹了this的shared_ptr就意味着要创建一个新的、带有独立的控制块的shared_ptr。这样一来，一块内存被两个控制块管理，凉凉。
enable_shared_from_this<T>就是为了解决这个问题，通过在成员函数内部调用shared_from_this()可以获得一个与外部控制块相同的shared_ptr的副本。


## 为什么不能在构造函数中调用？
因为Foo没有构造完毕的时候，外部的控制块都还没创建呢，所以要创建一个共享外部控制块的shared_ptr更不可能啦

## 为什么继承自enable_shared_from_this<T>的类只能分配在堆上？
废话吗不是，有用shared_ptr<T>管理堆内存的吗？delete栈内存那runtime就崩了

## enable_shared_from_this<T>的实现原理


## 为什么不能私有继承？