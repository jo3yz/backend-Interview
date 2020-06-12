## map是有序还是无序
无序，golang只提供基于hash映射的map

## 结构
```
struct Hmap
{
    uint8   B;    // 可以容纳2^B个项
    uint16  bucketsize;   // 每个桶的大小

    byte    *buckets;     // 2^B个Buckets的数组
    byte    *oldbuckets;  // 前一个buckets，只有当正在扩容时才不为空
};
```
struct Bucket
{
    uint8  tophash[BUCKETSIZE]; // hash值的高8位....低位从bucket的array定位到bucket
    Bucket *overflow;           // 溢出桶链表，如果有
    byte   data[1];             // BUCKETSIZE keys followed by BUCKETSIZE values
};

```
```

## 内部实现
- 通过hash值的低位选择桶（每个桶最多可以存8个pair），高位也就是buckets[]的index
- 再根据hash值的高位在与桶内的tophash[i]进行循环匹配，如果匹配成功，再查看data[i]的key是否与key一样，如果是就返回data[i]的value（这是个小优化，加快比较），如果不是，就再循环匹配tophash[i+1]

## 扩容
- 每次都是二倍扩容
- 和redis一样，是渐进式的，也就是说有byte *buckets和byte *oldbuckets两个指针

## 查找过程（带扩容）
- 先算出hash值，得出对应的bucket index
- 如果有old table，先在old table里找，如果old table中的值已经有了拷贝标记，就去new table找
- 如果old table里找不到，也去new table找
- 如果没有old table，也去new table找