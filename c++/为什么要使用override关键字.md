## override的作用
防止在派生类中写错virtual function的签名，导致一些难以察觉的错误

## virtual
在派生类中，实际上virtual可以省略，反正调用的virtual函数的版本都是根据基类指针实际指向的对象类型来确定的