# 一种自动反射消息类型的 Google Protobuf 网络传输方案

## 使用protobuf前需要解决的问题

protobuf只提供了对象的序列化和反序列化，但是如果用于tcp的话，不能直接将序列化后的二进制序列送入tcp流，必须解决消息边界的问题。此外接收的对端还需要直接这个字节流代表的是.proto中定义的哪个消息。

- 第一个问题有标准的解决方案：tlv，自定义数据包时在前面加一个固定长度的length header

- 第二个问题，容易想到的方法是在消息头中加一个消息类型的标识符，解析端使用lookup table，接收消息时使用对应的类型进行反序列化。但是比较麻烦，伸缩性不佳。

- 可以使用这里提到的反射方式，在消息头中加一个.proto文件中描述过的typename（不是模板的typename别想多了，比如package elaine中的message query，typename就是elaine.query）

## 怎么个反射法

1. 拿到typename（tcp数据流中提供）

2. 从Descripotr pool中拿到Descriptor*

3. 从Message Factory中拿到 const Message*，注意Message是用原型模式实现的，原型模式的作用在于拿到了一个Base指针，但是它指向一个Derive，也可以通过这个Base指针正确的克隆出一个用Base指针指向的Derive类（通过运行时多态实现）

4. 再通过这个使用原型模式实现的const Message*克隆一个新的类返回，之后可以根据具体的类型再做dynamic_cast

## 传输格式

totalLen -> nameLen -> typeName -> protobuf data -> checksum

