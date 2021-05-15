## Thrift协议学习2 - 网络服务模型

Thrift提供的网络服务模型：单线程、多线程、事件驱动，

Thrift阻塞服务模型、非阻塞服务模型:

- 阻塞服务模型：TSimpleServer、TThreadPoolServer。
- 非阻塞服务模型：TNonblockingServer、THsHaServer和TThreadedSelectorServer。

### 1. TServer
TServer定义了静态内部类Args，Args继承自抽象类AbstractServerArgs。
- TServer的三个方法：serve()、stop()和isServing()。
    - serve()用于启动服务
    - stop()用于关闭服务
    - isServing()用于检测服务的起停状态。
- TServer的不同实现类的启动方式不一样，因此serve()定义为抽象方法。不是所有的服务都需要优雅的退出, 因此stop()方法没有被定义为抽象。

### 2. TSimpleServer
TSimpleServer的工作模式采用最简单的阻塞IO
**优点**：实现方法简洁明了，便于理解
**缺点**：一次只能接收和处理一个socket连接，效率比较低。
**使用**：它主要用于演示Thrift的工作过程，在实际开发过程中很少用到它。

###### serve()方法的操作：


```
设置TServerSocket的listen()方法启动连接监听。
以阻塞的方式接受客户端地连接请求，每进入一个连接即为其创建一个通道TTransport对象。
为客户端创建处理器对象、输入传输通道对象、输出传输通道对象、输入协议对象和输出协议对象。
通过TServerEventHandler对象处理具体的业务请求
```

### 3. TThreadPoolServer

TThreadPoolServer模式采用阻塞socket方式工作，主线程负责阻塞式监听是否有新socket到来，具体的业务处理交由**一个线程**池来处理。

**优点**：ThreadPoolServer解决了TSimpleServer不支持并发和多连接的问题，引入了线程池。实现的模型是One Thread Per Connection。

###### serve()方法的操作：


```
设置TServerSocket的listen()方法启动连接监听。
以阻塞的方式接受客户端的连接请求，每进入一个连接，将通道对象封装成一个WorkerProcess对象(WorkerProcess实现了Runnabel接口)，并提交到线程池。
WorkerProcess的run()方法负责业务处理，为客户端创建了处理器对象、输入传输通道对象、输出传输通道对象、输入协议对象和输出协议对象。
通过TServerEventHandler对象处理具体的业务请求。
```
###### 优点
- 拆分了监听线程(Accept Thread)和处理客户端连接的工作线程(Worker Thread)，数据读取和业务处理都交给线程池处理。因此在并发量较大时新连接也能够被及时接受。
- 线程池模式比较适合服务器端能预知最多有多少个客户端并发的情况，这时每个请求都能被业务线程池及时处理，性能也非常高。

###### 缺点
线程池模式的处理能力受限于线程池的工作能力，当并发请求数大于线程池中的线程数时，新请求也只能排队等待。
### 4. TNonblockingServer
TNonblockingServer模式也是单线程工作，但是采用NIO的模式，借助Channel/Selector机制, 采用IO事件模型来处理。

##### 机制
- 所有的socket都被注册到selector中，在一个线程中通过seletor循环监控所有的socket。
- 每次selector循环结束时，处理所有的处于就绪状态的socket，对于有数据到来的socket进行数据读取操作，对于有数据发送的socket则进行数据发送操作，对于监听socket则产生一个新业务socket并将其注册到selector上。

注意：TNonblockingServer要求底层的传输通道必须使用TFramedTransport。

###### 优点
相比于TSimpleServer效率提升主要体现在IO多路复用上，TNonblockingServer采用非阻塞IO，对accept/read/write等IO事件进行监控和处理，同时监控多个socket的状态变化。
###### 缺点
- TNonblockingServer模式在业务处理上还是采用单线程顺序来完成。
- 在业务处理比较复杂、耗时的时候，例如某些接口函数需要读取数据库执行时间较长，会导致整个服务被阻塞住，此时该模式效率也不高，因为多个调用请求任务依然是顺序一个接一个执行。

### 5. THsHaServer
- THsHaServer继承于TNonblockingServer，引入了线程池提高了任务处理的并发能力。
- THsHaServer是半同步半异步(Half-Sync/Half-Async)的处理模式，Half-Aysnc用于IO事件处理(Accept/Read/Write)，Half-Sync用于业务handler对rpc的同步处理上。


```
注意：THsHaServer和TNonblockingServer一样，要求底层的传输通道必须使用TFramedTransport。
```
###### 优点
THsHaServer与TNonblockingServer模式相比，THsHaServer在完成数据读取之后，将业务处理过程交由一个线程池来完成，主线程直接返回进行下一次循环操作，效率大大提升。
###### 缺点
主线程仍然需要完成所有socket的监听接收、数据读取和数据写入操作。当并发请求数较大时，且发送数据量较多时，监听socket上新连接请求不能被及时接受。


### 6. TThreadedSelectorServer
- TThreadedSelectorServer是对THsHaServer的一种扩充，它将selector中的读写IO事件(read/write)从主线程中分离出来。
- 同时引入worker工作线程池，它也是种Half-Sync/Half-Async的服务模型。

**TThreadedSelectorServer模式是目前Thrift提供的最高级的线程服务模型**，它内部有如果几个部分构成：

1. 一个AcceptThread线程对象，专门用于处理监听socket上的新连接。
1. 若干个SelectorThread对象专门用于处理业务socket的网络I/O读写操作，所有网络数据的读写均是有这些线程来完成。
1. 一个负载均衡器SelectorThreadLoadBalancer对象，主要用于AcceptThread线程接收到一个新socket连接请求时，决定将这个新连接请求分配给哪个SelectorThread线程。
1. 一个ExecutorService类型的工作线程池，在SelectorThread线程中，监听到有业务socket中有调用请求过来，则将请求数据读取之后，交给ExecutorService线程池中的线程完成此次调用的具体执行。主要用于处理每个rpc请求的handler回调处理(这部分是同步的)。


三个组件AcceptThread、SelectorThread和ExecutorService在源码中的定义如下：

```
TThreadedSelectorServer模式中有一个专门的线程AcceptThread用于处理新连接请求，因此能够及时响应大量并发连接请求；
将网络I/O操作分散到多个SelectorThread线程中来完成，因此能够快速对网络I/O进行读写操作，能够很好地应对网络I/O较多的情况。
```
