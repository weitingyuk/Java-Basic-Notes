## 什么是 SYN Flood，如何防止这类攻击？

### 一. 什么是 SYN Floodflood
SYN Flood（半开放攻击）是一种拒绝服务（DDoS）攻击，其目的是通过消耗所有可用的服务器资源使服务器不可用于合法流量。通过重复发送初始连接请求（SYN）数据包，攻击者能够压倒目标服务器机器上的所有可用端口，导致目标设备根本不响应合法流量。

### 二. SYN Flood攻击如何工作？
#### 2.1 TCP连接
通过利用TCP连接的握手过程，SYN Flood攻击工作。在正常情况下，TCP连接显示三个不同的进程以进行连接。

1. 首先，客户端向服务器发送SYN数据包，以便启动连接。
2. 服务器响应该初始包与SYN / ACK包，以确认通信。
3. 最后，客户端返回ACK数据包以确认从服务器接收到的数据包。完成这个数据包发送和接收序列后，TCP连接打开并能发送和接收数据。

#### 2.2 SYN Flood攻击
为了创建拒绝服务，攻击者利用这样的漏洞，即在接收到初始SYN数据包之后，服务器将用一个或多个SYN / ACK数据包进行响应，并等待握手中的最后一步。这是它的工作原理：

- 攻击者向目标服务器发送大量SYN数据包，通常会使用欺骗性的IP地址。
- 然后，服务器响应每个连接请求，并留下开放端口准备好接收响应。
- 当服务器等待从未到达的最终ACK数据包时，攻击者继续发送更多的SYN数据包。每个新的SYN数据包的到达导致服务器暂时维持新的开放端口连接一段时间
- 一旦所有可用端口被使用，服务器就无法正常工作。
#### 2.3 半开攻击
在网络中，当服务器断开连接但连接另一端的机器没有连接时，连接被认为是半开的。在这种类型的DDoS攻击中，目标服务器不断离开打开的连接，等待每个连接超时，然后端口再次可用。结果是这种攻击可以被认为是“半开攻击”。

### 三. 如何减轻SYN Flood攻击？

#### 3.1 增加积压队列
- 目标设备上的每个操作系统都具有一定数量的半开放连接。对大量SYN数据包的一个响应是增加操作系统允许的可能半开连接的最大数量。为了成功增加最大积压，系统必须预留额外的内存资源来处理所有新的请求。如果系统没有足够的内存来处理增加的积压队列大小，系统性能将受到负面影响，但仍然可能优于拒绝服务。
#### 3.2 回收最早的半开TCP连接
- 一旦积压已被填补，另一个缓解策略就是覆盖最早的半开式连接。这种策略要求合法连接可以在比可以填充恶意SYN数据包的积压时间更短的时间内完全建立。当攻击量增加时，或者如果积压量太小而不实际，这种特定的防御就会失败。

#### 3.3 SYN cookie
- 这个策略涉及服务器创建一个cookie。为了避免在积压已经被填满的情况下连接丢失的风险，服务器使用SYN-ACK数据包对每个连接请求进行响应，然后从积压中删除SYN请求，从存储器中删除请求并使端口打开，准备建立新的连接。如果连接是合法请求，并且最终的ACK数据包从客户端计算机发送回服务器，则服务器将重建（有一些限制）SYN积压队列条目。尽管这种缓解措施确实丢失了有关TCP连接的一些信息，但是优于允许合法用户因攻击而发生拒绝服务。