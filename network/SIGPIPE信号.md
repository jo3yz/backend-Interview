## SIGPIPE产生的原因
如果一个 socket 在接收到了 RST packet 之后，程序仍然向这个socket 写入数据，那么就会产生SIGPIPE信号。
## 什么时候会收到RST
- 对端进程崩溃，套接字引用计数变为0，调用close
- 对端直接调用close