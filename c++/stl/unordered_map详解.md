## 主流方法
都使用链表法，动态进行扩容

## 典型实现
### hash容器
```
struct node
{
    node *next;
    Value value;
}
vecotor<node*>
```

### Rehsahing时机
当负载因子达到一定上限后，但是STL的Rehashing好像不是渐进式的

### 扩容
和golang map的二倍扩容不同，stl是扩容到两倍附近的质数上（最开始的容量也是质数，53）

## 使用迭代器迭代的时候会发生什么？