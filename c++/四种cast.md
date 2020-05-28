## 不要使用C style的强制类型转换

## const_cast<T>(expression)
用于把const类型的变量去除其const属性

## dynamic_cast<T>(expression)
用于执行安全的“向下转型”，也就是说把基类指针或引用转化为派生类指针或引用。因为“向上转型”是绝对安全的，所以需要这个运算符来保证“向下转型”的安全，如果转型失败（不安全），会返回nullptr，它的开销可能会比较大
### 典型场景
用户已知Derive是派生类，想调用Derive的方法，但只有Base的指针，在保证用户已知没问题的情况下，可以调用dynamic_cast：比如想在一个vecotr<Base *>中使用运行时多态的能力，也即想通过这些指针调用Derive专属的函数。
### 如何避免使用dynamic_cast？
把需要通过vecotr<Base *>调用的Derive的能力想好，做成Base的virtual函数，就不需要转型了。或者使用多个vector保存不同类型的Derive

## reinterpret_cast<T>(expression)
整形数字和指针之间进行强制转换，常用于low-level代码

## static_cast<T>(expression)
和C style的强制转化差不多，比较粗暴。但是无法把const转换为non-const，只能把non-const转换为const