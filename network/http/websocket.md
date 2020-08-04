## websocket的出现要解决什么问题？
比如一个聊天室web应用，client如果要实时刷新聊天室的聊天记录，如果通过http/https就必须通过轮询的方式获取。因为http/https的连接必须是client主动发起连接，server被动接收连接。websocket下服务器既可以主动向客户端发送消息，客户端也可以主动向服务器发送消息，就像linux socket一样。

## 特点
1. 也是基于tcp的应用层协议
2. 需要借助http来建立连接
3. 真丶全双工
4. 头部数据较小