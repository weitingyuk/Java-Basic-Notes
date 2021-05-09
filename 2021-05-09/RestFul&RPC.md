## RestFul和RPC的区别
### RestFul
REST代表表现层状态转移(representational state transfer):
在设计API时，使用路径定位资源，方法定义操作，通过Content-Type和Accept来协商资源的类型。
REST也有一些限制：

- REST是无状态的，请求之间没有持久的会话信息
- 响应需要声明成可缓存的
- REST关注一致性，如果使用HTTP，需要尽可能使用HTTP的特性，而不是去发明新的公约
这些限制允许REST架构的API更加稳定。
### RPC
RPC代表远程过程调用(remote procedure call):
RPC是跨语言跨平台的服务调用，不仅是C-S或者前后端之间的通信。一个完善的RPC框架还包含代码生成，通信的规范等等，在这里我们只谈API的设计。


### RestFul和RPC的区别

1、从本质区别上看，RPC是基于TCP实现的，REST是基于HTTP来实现的。

2、从传输速度上来看，因为HTTP封装的数据量更多所以数据传输量更大，所以RPC的传输速度是比RESTFUL更快的。

3、因为HTTP协议是各个框架都普遍支持的。在toC情况下，因为不知道情况来源的框架、数据形势是什么样的，所以在网关可以使用Restful利用http来接受。而在微服务内部的各模块之间因为各协议方案是公司内部自己定的，所以知道各种数据方式，可以使用TCP传输以使各模块之间的数据传输更快。所以可以网关和外界的数据传输使用RESTFUL，微服务内部的各模块之间使用RPC。

4、REST的API的设计上是面向资源的，对于同一资源的获取、传输、修改可以使用GET、POST、PUT来对同一个URL进行区别，而RPC通常把动词直接体现在URL上。


### 适用场景
基于RPC的API更加适用行为(也就是命令和过程)，基于REST的API更加适用于构建模型(也就是资源和实体)，处理CRUD。

REST不仅是CRUD(CRUD Boy哈哈)，不过REST也比较适合做CRUD。

#### 使用的方法
- REST使用HTTP的方法，例如：GET,POST,PUT,DELETE,OPTIONS还有比较不常用的PATCH方法。
- RPC通常只会使用GET和POST方法，GET方法通常用来获取信息，POST方法可以用来进行所有的行为。

#### 是否有状态
- RPC：发送一个消息，然后消息会存储到数据库中来保存历史，有可能会有其他的RPC调用，但这个操作对我们不可见
- REST：在用户的消息集合中创建一条消息资源，我们能够通过GET方法来通过相同的URL获取这个历史