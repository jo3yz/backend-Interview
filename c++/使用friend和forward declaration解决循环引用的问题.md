## friend（友元）可以干什么
### 修饰对象之一：类
假如B是A的友元，B的成员函数可以访问A的**所有成员**，包括protect和private成员变量和成员函数，示例：
class A
{
    friend class B
};
### 修饰对象之二：普通函数
普通函数F是A的友元函数，则这个普通函数可以访问A的**所有成员**，示例：
class A
{
    friend void ::GlobalFunc();
}
## 修饰对象之三：其它类的成员函数
成员函数F是A的友元函数，则这个成员函数可以访问A的**所有成员**，示例：
class A
{
    friend void B::Func();
}
## 循环引用的问题
为C++中包含header file的实质就是把header file**展开**，如果A.h包含了B.h，B.h包含了A.h，当其他源码文件包含A.h或者B.h时，就会**无限展开**，最终耗尽编译器的内存。
但是如果确实两个类互相依赖，比如A的方法需要依赖B的定义，B的方法需要依赖A的定义，考虑Matrix（矩阵）和Vector（向量）的乘法，需要定义如下操作（下面两种都可以）：
```
Vector& Matrix::operator(const Vector& v); // 成员函数形式
Vector& Vector::operator(const Matrix& m); // 成员函数形式

Vector& ::operator(Matrix& m, Vector& v);       // 普通函数形式
Vector& ::operator(Vector& v, const Matrix& m); // 普通函数形式
```
但是Vector的函数需要Matrix的定义，Matrix的函数需要Vector的定义，怎么办？

## forward declaration（前向声明）如何解决循环引用的问题
可以这样解决上面提到的问题：
```
class Matrix; //前向声明Matrix
class Vector
{
public:
    Vector& Vector::operator(const Matrix& m);
    // ...
};
class Matrix
{
public:
    Vector& Matrix::operator(const Vector& v);
    // ...
}
```
在前向声明Matrix后，Vector认为确实有这样的一个Matrix存在，也就是**有了声明，但没有定义**，和在.h中声明普通函数是一个道理，只要有声明，就能知道其**类型**。

## forward declaration（前向声明）需要注意什么
前向声明的类**只有声明，没有定义**，所以无法进行这样的定义：
```
class Matrix;
Matrix m;   // ERROR! Incomplete type!
Matrix *m;  // OK
Matrix &m;  // OK，当然只能作为函数的成员变量或形参类型，因为引用必须在初始化时就和相应的变量绑定，且不能“改嫁”
``