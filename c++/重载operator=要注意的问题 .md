## 一定要先记得处理自我赋值的情况
假如Foo是成员带有T* p要管理堆上的资源，如果没有处理自我赋值，可能出现这样的情况：
```
Foo& operator=(const Foo& self)
{
    delete p;
    p = new T(*self.p); //ops
    // other operation...
    return *this;
}
```
代码中的self.p和this.p实际上是一个东西，上面的代码造成了已经delete了的内存的访问，所以一定要处理自我赋值的情况。
## 一个协议：返回reference to this
注意：这仅仅是一个规范，实际上用户返回什么都可以。
返回reference to this便于连续赋值，形如：
```
f1 = f2 = f3 = f4 = 5;
```
实际上，要支持连续赋值，返回函数局部对象的拷贝也可以，但是造成了无谓的构造和析构。
不要以为返回reference to this就没有拷贝了，以f3 = f4 = 5举例，可以看做是
```
rf4 = Foo(5);   // 这里无法避免拷贝
f3.operator=(rf4); // 这里实际上也是拷贝，只不过是行为是用户自定义的
```