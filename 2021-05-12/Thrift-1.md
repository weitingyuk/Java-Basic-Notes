## Thrift协议学习-1
Thrift是一个轻量级、跨语言的远程服务调用框架，最初由Facebook开发，后面进入Apache开源项目。它通过自身的IDL中间语言, 并借助代码生成引擎生成各种主流语言的RPC服务端/客户端模板代码。
### 一. Thrift软件栈分层
Thrift软件栈分层从下向上分别为：传输层(Transport Layer)、协议层(Protocol Layer)、处理层(Processor Layer)和服务层(Server Layer)。

1. 传输层(Transport Layer)：传输层负责直接从网络中读取和写入数据，它定义了具体的网络传输协议；比如说TCP/IP传输等。
1. 协议层(Protocol Layer)：协议层定义了数据传输格式，负责网络传输数据的序列化和反序列化；比如说JSON、XML、二进制数据等。
1. 处理层(Processor Layer)：处理层是由具体的IDL（接口描述语言）生成的，封装了具体的底层网络传输和序列化方式，并委托给用户实现的Handler进行处理。
1. 服务层(Server Layer)：整合上述组件，提供具体的网络线程/IO服务模型，形成最终的服务。

### 二. Thrift序列化协议
Thrift支持多种序列化协议，常用的有: Binary、Compact、JSON。这里我们主要分析Binary和Compact。

#### 1. Binary序列化
binary序列化是一种二进制的序列化方式。不可读，但传输效率高。

#### 2. Message的序列化
Message的序列化分为两种，strict encoding和old encoding。 在有些实现中，会通过
检查Thrift消息的第一个bit来判断使用了那种encoding：

1 —-> strict encoding
0 —-> old encoding
Message的Binary序列化下面的一张图就够了:

#### 3. Struct的序列化
Struct装的是Thrift通信的实际参数，一个Struct由很多基本类型组合而成，要了解Struct怎么序列化的必须知道这些基本类型的序列化。
定长编码:
上表中的 bool, byte, short, int, long, double采用的都是固定字节数编码，各类型占用的字节数...。

#### 4. Compact序列化
Compact序列化也是一种二进制的序列化，不同于Binary的点主要在于整数类型采用了zigzag 和 varint压缩编码实现，这里简要介绍下zigzag 和 varint整数编码。 

### 三. Thrift的传输层
常用的传输层有以下几种：

- TSocket：使用阻塞式I/O进行传输，是最常见的模式
- TNonblockingTransport：使用非阻塞方式，用于构建异步客户端
- TFramedTransport：使用非阻塞方式，按块的大小进行传输，类似于Java中的NIO

### 四. Thrift的服务端类型

- TSimpleServer：单线程服务器端，使用标准的阻塞式I/O
- TThreadPoolServer：多线程服务器端，使用标准的阻塞式I/O
- TNonblockingServer：单线程服务器端，使用非阻塞式I/O
- THsHaServer：半同步半异步服务器端，基于非阻塞式IO读写和多线程工作任务处理
- TThreadedSelectorServer：多线程选择器服务器端，对THsHaServer在异步IO模型上进行增强
