## 可以减少无谓的代码量
- 比如让人头皮发麻的迭代器类型，现在直接auto就完事了
- 也可以用decltype抽取数据类型

## auto和decltype提升代码的泛化性
1. auto 有的时候在泛型代码中，并不知道某个变量的真正类型（比如要获取T实例的某个成员函数的返回值），就可以使用auto了
2. decltype 在代码中，如果拿到T的迭代器类型（但是T可能只有const_iter），可以通过decltype(container.begin())

## auto + decltype = 后置语法返回值
