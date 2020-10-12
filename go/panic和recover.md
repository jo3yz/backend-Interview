## panic

有点类似于其他语言中的异常机制，没有捕获的异常将导致进程崩溃。除此之外，panic还有一些专有的特点：

- 在程序崩溃之前，会将所在的goroutine中所有defer的函数都运行一次（按照defer的特定规则）
- 由于defer的执行在空return之前，所以甚至可以在defer函数中输出堆栈等运行时信息
- 在defer的处理函数里面可以调用recover，还可以继续抛出panic

## recover

用于在defer函数中帮助panic恢复，recover也有一些特点：

- 只会在defer的函数中生效
- 通过recover函数可以拿到panic抛出的异常类型（可以自定义）

问题在于：

- 如果panic恢复了，之前的现场还存在吗？如果存在的话，下一条语句从哪儿执行？
- 不加区分的恢复所有panic肯定是不可取的，但是什么时候才使用recover来恢复panic？应不应该恢复其他包所引起的panic？
- 能不能从recover的返回值拿到panic的类型信息（类似于异常类型的信息）？

- 什么时候使用error返回值？什么时候使用panic-recover机制？

- panic能传递到函数调用的外层吗？是不是传无可传了才崩溃或者执行defer？