## override的作用
防止在派生类中写错virtual function的签名，导致一些难以察觉的错误

## virtual
在派生类中，实际上virtual可以省略，反正调用的virtual函数的版本都是根据基类指针实际指向的对象类型来确定的

## 在析构函数中
只有基类的析构函数需要写成virtual的，派生类的析构函数写成override就可以了（只有在这里override所修饰的函数可以和父类的不同名）