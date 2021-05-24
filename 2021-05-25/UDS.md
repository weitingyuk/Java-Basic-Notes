## Unix域协议 - UDS
IPC（进程间通信）的方式之一，Unix域协议使用套接字API，支持同一台主机的不同进程之间进行通信。

### 优点
Unix域协议比TCP/UDP套接的优点
1. 比起TCP协议通常要更快；
2. 支持在同一台主机上的不同进程之间传递描述符；
3. 支持传递客户端凭证。

uds是客户端和服务端在同一台主机上的IPC方法之一，与其他IPC方法（pipe，共享内存等）相比，uds的优势在于其使用的API几乎等同于网络通讯中使用的API，与客户端和服务端在同一台主机上的TCP相比，unix域字节流套接字的性能要更优。



### UDS和TCP/UDP套接的相同点
套接字API完全一致，即：
- 服务端需要进行bind、listen、accpet等操作才能读写，客户端需要先connect才能进行读写。

### UDS和TCP/UDP套接的不同点
- uds绑定的地址是一个文件系统的绝对路径，比如"/tmp/myuds"
- TCP/UDP套接字使用的地址则包含了地址和端口号
- uds使用的路径并不是普通的文件，需要和uds关联才能对其进行读写

### UDS套接字类型
Unix域套接字有两种类型：
- 字节流套接字（类似TCP）
- 数据报套接字（类似UDP）

### UDS使用场景
1. 本机通信
需要在本机通信时，可以使用uds来代替本地回环接口。
- 不需要经过网络协议栈，省去了各种解析和应答等步骤，而是直接在内核拷贝传递数据。比如最近很热的service mesh，业务进程和sidecar就可以通过uds来通信。
2. 传递描述符
需要传递描述符时，通常可以使用方法有：
- fork调用返回以后，子进程共享父进程的所有描述符
- exec调用执行后，所有的描述符通常保持打开状态
3. 验证发送者的身份
uds传递的另一种辅助数据就是用户凭证。
- 因为用户凭证的数据结构在不同的操作系统中并不一致。

### java中的uds
- java中并不支持直接使用uds，可能是因为java标榜跨平台；uds则只在部分操作系统中才能使用。
- java中使用uds，通常需要使用第三方提供的类库，比如著名的网络通讯组件netty就提供了uds通讯的支持。