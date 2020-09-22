## 对数组或切片

### 遍历清空元素

比如这样的代码

```go
a := make([]int, 64)
for i := range a {
	a[i] = 0
}

```

会被直接优化为类似于memset的代码，但是最后把会i置为len(a) - 1

### 不关心索引和数据

比如这样的代码

```go
a := make([]int, 64)
for range a {
	// ...
}
```

则直接生成一个最简单的while循环

### 只关心索引

比如这样的代码

```go
a := make([]int, 64)
// 注意由于省略的是后面的参数，所以后面的", _"是多余的
for i := range a {
	// ...
}
```

则生成的是如下类似的代码

```go
ha := a
hv1 := 0
hn := len(ha)
v1 := hv1
for ; hv1 < hn; hv1++ {
    v1 := hv1
    ...
}
```

在for作用域外面拿到的索引就是hv1，所以循环结束后i一定等于len(a)

### 关心索引和数据

比如这样的代码

```go
a := make([]int, 64)
// 注意由于省略的是后面的参数，所以后面的", _"是多余的
for i, tmp := range a {
	// ...
}
```

则生成的是如下类似的代码

```go
ha := a
hv1 := 0
hn := len(ha)
v1 := hv1
for ; hv1 < hn; hv1++ {
    tmp := ha[hv1]
    v1, v2 := hv1, tmp
    ...
}
```

可以看到这里用了个temp来作为数组切片索引的中间结果，所以如下代码中的&v只能拿到这个缓存tmp的地址

```go
func main() {
	arr := []int{1, 2, 3}
	newArr := []*int{}
	for _, v := range arr {
		newArr = append(newArr, &v)
	}
	for _, v := range newArr {
		fmt.Println(*v)
	}
}
```

最后的输出结果是 3 3 3

### 为什么for range不能“永动机”

这样的代码不能像一个while(true)一样无限循环下去，而是只是遍历一次没有append操作前的原始切片

```go
func main() {
	arr := []int{1, 2, 3}
	for _, v := range arr {
		arr = append(arr, v)
	}
}
```

答案也能从生成的代码中找到

```go
ha := arr
hn := len(ha)
for ; hv1 < hn; hv1++ {
	// ...
}
```

因为for range迭代的是一个需要迭代的切片的副本，所以不受影响，但是三段式的for就不一样了。

## 对map（哈希表）

每次对内置map的for range遍历都没有一致的顺序，这是golang的设计故意为之，让使用者不要依赖于哈希表的遍历顺序。

## 对有缓冲channel

会生成类似的代码

```go
ha := a
hv1, hb := <-ha
for ; hb != false; hv1, hb = <-ha {
    v1 := hv1
    hv1 = nil
    ...
}
```

所以如果channel没有close，这个hv1, hb := <-ha语句就会阻塞。所以使用for range遍历channel前一定要将它close掉。