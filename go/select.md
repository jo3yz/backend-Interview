## 作用

用于在goroutine的一个控制流中同时对一个或几个channel的可读可写状态进行监听

## default非阻塞

使用default可以实现非阻塞监听的功能，如果channel一个都没有准备好，也直接跳出select

## 特殊情况

- 空select

  这种情况会直接阻塞当前goroutine

- channel同时就绪

  会随机选一个就绪的channel来进行读或者写操作