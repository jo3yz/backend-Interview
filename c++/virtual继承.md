## 作用
解决菱形继承中，基类将其成员数据实例共享给也从这个基类型直接或间接派生的其它类。比如
```
class D1 : virtual public B {}
class D2 : virtual public B {}
class DD : public D1, D2 {}
```