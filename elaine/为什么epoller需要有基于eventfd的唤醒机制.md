
因为epoller的poll方法是必须要epoll_wait返回才会yield出去的，而且被包装成了一个coroutine，会在processor的任务队列空了时执行。

问题在于：

1. 如果processor的任务队列空，processor的线程就会切入到epoll_wait中执行，如果没有新的事件来临，那么poll协程就永远不会切出去，**即使现在任务队列已经由其他线程加入了新的新任务，也得不到及时运行**

2. 如果需要停止processor的主循环，由于processor的主循环在没有任务时会陷入epoll_wait，所以需要在调用stop的时候也唤醒epoller


那如何实现呢？

addTask时，如果队列为空，就唤醒一次epoller