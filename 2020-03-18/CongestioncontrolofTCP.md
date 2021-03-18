## TCP 中常见的拥塞控制算法有哪些？
拥塞控制主要是四个算法：1）慢启动，2）拥塞避免，3）拥塞发生，4）快速恢复。

### 慢启动算法 – Slow Start
所谓慢启动，也就是TCP连接刚建立，一点一点地提速，试探一下网络的承受能力，以免直接扰乱了网络通道的秩序。
-  慢启动算法：
1) 连接建好的开始先初始化拥塞窗口cwnd大小为1，表明可以传一个MSS大小的数据。
2) 每当收到一个ACK，cwnd大小加一，呈线性上升。
3) 每当过了一个往返延迟时间RTT(Round-Trip Time)，cwnd大小直接翻倍，乘以2，呈指数让升。
4) 还有一个ssthresh（slow start threshold），是一个上限，当cwnd >= ssthresh时，就会进入“拥塞避免算法”（后面会说这个算法）

### 拥塞避免算法 – Congestion Avoidance
 如同前边说的，当拥塞窗口大小cwnd大于等于慢启动阈值ssthresh后，就进入拥塞避免算法。算法如下：
1) 收到一个ACK，则cwnd = cwnd + 1 / cwnd
2) 每当过了一个往返延迟时间RTT，cwnd大小加一。

 过了慢启动阈值后，拥塞避免算法可以避免窗口增长过快导致窗口拥塞，而是缓慢的增加调整到网络的最佳值。
 
###  拥塞状态时的算法
TCP拥塞控制默认认为网络丢包是由于网络拥塞导致的，所以一般的TCP拥塞控制算法以丢包为网络进入拥塞状态的信号。对于丢包有两种判定方式，一种是超时重传RTO[Retransmission Timeout]超时，另一个是收到三个重复确认ACK。

- 超时重传RTO[Retransmission Timeout]超时，TCP会重传数据包。TCP认为这种情况比较糟糕，反应也比较强烈：

1. 由于发生丢包，将慢启动阈值ssthresh设置为当前cwnd的一半，即ssthresh = cwnd / 2.
1. cwnd重置为1
1. 进入慢启动过程

- 当收到三个重复确认ACK时，TCP开启快速重传Fast Retransmit算法，而不用等到RTO超时再进行重传：
1. cwnd大小缩小为当前的一半
1. ssthresh设置为缩小后的cwnd大小
1. 然后进入快速恢复算法Fast Recovery。


### 快速恢复算法 – Fast Recovery
在进入快速恢复之前，cwnd和ssthresh已经被更改为原有cwnd的一半。快速恢复算法的逻辑如下：
1. cwnd = cwnd + 3 MSS，加3 MSS的原因是因为收到3个重复的ACK。
1. 重传DACKs指定的数据包。
1. 如果再收到DACKs，那么cwnd大小增加一。
1. 如果收到新的ACK，表明重传的包成功了，那么退出快速恢复算法。将cwnd设置为ssthresh，然后进入拥塞避免算法。