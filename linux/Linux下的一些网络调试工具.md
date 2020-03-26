## ping
这好像不用多说了，基于ICMP协议的，用于探查连接的可达性

## netstat
用来显示系统上的所有套接字状态，tcp/udp/unix的，都可以显示。

这些状态包括：
- 连接的四元组（两个IP，两个端口）
- TCP的状态（LISTEN/ESTABLISHED/TIME_WAIT）等，当然UDP没有
- 发送队列和接收队列的长度

netstat默认情况下，只显示ESTABLISHED状态的连接，对于tcp，可以使用-t，udp可以使用-u，```netstat Socket -t -alep```或者```netstat Socket -u -alep```

最后，可以使用```watch -n 1 netstat -ant```每秒刷新一次netstat

## netcat（瑞士军刀）
缩写为nc，可以做到：
- 模拟 TCP 服务端
- 模拟 TCP 客户端
- 模拟 UDP 服务端
- 模拟 UDP 客户端
- 模拟 UNIX socket 服务端
- 模拟 UNIX socket 客户端
- 端口扫描
- 传输文件
- 将服务器 bash 暴露给远程客户端
- 内网穿透，反向获取防火墙后的机器的 bash

具体看这个：https://segmentfault.com/a/1190000016626298

## lsof
用于查询是谁占用了某个端口，例如
```lsof -i 8080```

## tcpdump
**这个特别有用**，用来抓包的。可以把TCP的数据，握手/挥手过程都抓出来。

而且不仅可以抓TCP，UDP也可以抓，在抓UDP的过程（当然是用nc是最方便的）中可以清楚的看到没有握手挥手的过程。