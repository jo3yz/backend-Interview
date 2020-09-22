## 多次调用defer的执行顺序

越迟调用的defer语句在作用域结束时越先调用，类似于栈的结构。

## defer和return碰撞出的火花（坑）

- 基础：所有带返回值的函数，在函数起始处就会初始化返回值为对应类型的零值，并且作用域是整个函数。
- **"return xxx"语句的执行并不是原子的**，函数返回的过程是：
  1. 将返回值赋值为xxx
  2. 执行defer
  3. 空return

下面举点个常见的例子（坑）

```go
func f() (result int) {
    defer func() {
        result++
    } ()
    
    return 0
}
```

这里result在函数开始时就初始化为0了，在return预处理时，result被赋值为0，调用了defer后result变为了1，最后空return返回的是result，使得f的返回值是1

```go
func f() (result int) {
    t := 5
    defer func() {
       t = t + 5 
    }()
    return t
}
```

注意这里返回的可不是10，因为空return返回的是result而不是t，所以即使在空return前，对t进行了操作，f仍然返回5

```go
func f() (result int) {
	defer func(r int) {
		r = r + 5
	}(result)
	return 1
}
```

这里返回的是1，因为defer执行的函数只传递了result的值而非地址或引用，所以不会对起造成修改。需要注意的是go中所有的函数调用都是传值的。

## defer的预计算导致结果不达预期

defer后跟一个函数调用时，会预结算出函数所需的参数值，比如如下代码就不达预期

```go
func main() {
	startedAt := time.Now()
	defer fmt.Println(time.Since(startedAt))
	
	time.Sleep(time.Second)
}
```

在Sleep未执行之前，就把time.Since(startedAt)先算出来了。要规避这个问题也有办法

```go
func main() {
	startedAt := time.Now()
    defer func() {
        fmt.Println(time.Since(startedAt))
    }()
	
	time.Sleep(time.Second)
}
```

这样defer后接的函数就没有参数了，就不会预计算了