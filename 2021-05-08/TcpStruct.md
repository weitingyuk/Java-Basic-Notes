## 简述 TCP 的报文头部结构

头部长度：一般为20字节，选项最多40字节，限制60字节。

- 16位源端口号和16位目的端口号。
- 32位序号：一次TCP通信过程中某一个传输方向上的字节流的每个字节的编号，通过这个来确认发送的数据有序，比如现在序列号为1000，发送了1000，下一个序列号就是2000。
- 32位确认号：用来响应TCP报文段，给收到的TCP报文段的序号加1，三握时还要携带自己的序号。
- 4位头部长度：标识该TCP头部有多少个4字节，共表示最长15*4=60字节。同IP头部。
- 6位保留。
- 6位标志。
    - URG（紧急指针是否有效）
    - ACK（表示确认号是否有效）
    - PSH（提示接收端应用程序应该立即从TCP接收缓冲区读走数据）
    - RST（表示要求对方重新建立连接）
    - SYN（表示请求建立一个连接）
    - FIN（表示通知对方本端要关闭连接）
- 16位窗口大小：TCP流量控制的一个手段，用来告诉对端TCP缓冲区还能容纳多少字节。
- 16位校验和：由发送端填充，接收端对报文段执行CRC算法以检验TCP报文段在传输中是否损坏。
- 16位紧急指针：一个正的偏移量，它和序号段的值相加表示最后一个紧急数据的下一字节的序号。