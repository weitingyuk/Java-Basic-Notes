## ZAB协议
Zab（Zookeeper Atomic Broadcast）是为ZooKeeper协设计的崩溃恢复原子广播协议，它保证zookeeper集群数据的一致性和命令的全局有序性。

### 概念介绍
#### 集群角色
1. Leader：同一时间集群总只允许有一个Leader，提供对客户端的读写功能，负责将数据同步至各个节点；
1. Follower：提供对客户端读功能，写请求则转发给Leader处理，当Leader崩溃失联之后参与Leader选举；
1. Observer：与Follower不同的是但不参与Leader选举。
#### 服务状态
1. LOOKING：当节点认为群集中没有Leader，服务器会进入LOOKING状态，目的是为了查找或者选举Leader；
1. FOLLOWING：follower角色；
1. LEADING：leader角色；
1. OBSERVING：observer角色；
#### ZAB状态
1. ELECTION: 集群进入选举状态，此过程会选出一个节点作为leader角色；
1. DISCOVERY：连接上leader，响应leader心跳，并且检测leader的角色是否更改，通过此步骤之后选举出的leader才能执行真正职务；
1. SYNCHRONIZATION：整个集群都确认leader之后，将会把leader的数据同步到各个节点，保证整个集群的数据一致性；
1. BROADCAST：过渡到广播状态，集群开始对外提供服务。

#### ZXID
一个long型（64位）整数，分为两部分：纪元（epoch）部分和计数器（counter）部分，是一个全局有序的数字。
- epoch代表当前集群所属的哪个leader，代表当前命令的有效性
- counter是一个递增的数字。


### 选举
#### 选举发生的时机
1. 服务启动的时候当整个集群都没有leader节点会进入选举状态；
1. 在服务运行中，可能会出现各种情况，服务宕机、断电、网络延迟很高的时候leader都不能再对外提供服务了，所有当其他几点通过心跳检测到leader失联之后，集群也会进入选举状态。
#### 选举规则
应该选择哪个机器当leader呢？
- 1 先选epoch高的
- 2 如果epoch相同，则选择zid高的
- 3 如果epoch和zid都相同，则选择server id高的
#### 选举流程
Leader选举流程：
1. 所有节点第一票先选举自己当leader，将投票信息广播出去；
1. 从队列中接受投票信息；
1. 按照规则判断是否需要更改投票信息，将更改后的投票信息再次广播出去；
1. 判断是否有超过一半的投票选举同一个节点，如果是选举结束根据投票结果设置自己的服务状态，选举结束，否则继续进入投票流程。

#### 广播
- zab在广播状态中保证以下特征

1. 可靠传递:  如果消息m由一台服务器传递，那么它最终将由所有服务器传递。
1. 全局有序: 如果一个消息a在消息b之前被一台服务器交付，那么所有服务器都交付了a和b，并且a先于b。
1. 因果有序: 如果消息a在因果上先于消息b并且二者都被交付，那么a必须排在b之前。
 
当收到客户端的写请求的时候会经历以下几个步骤：
1. Leader收到客户端的写请求，生成一个事务（Proposal），其中包含了zxid；
1. Leader开始广播该事务，需要注意的是所有节点的通讯都是由一个FIFO的队列维护的；
1. Follower接受到事务之后，将事务写入本地磁盘，写入成功之后返回Leader一个ACK；
1. Leader收到过半的ACK之后，开始提交本事务，并广播事务提交信息
1. 从节点开始提交本事务。

zookeeper通过二阶段提交来保证集群中数据的一致性，因为只需要收到过半的ACK就可以提交事务，所以zookeeper的数据并不是强一致性。

zab协议的有序性保证是通过几个方面来体现的：
- 服务之前用TCP协议进行通讯，保证在网络传输中的有序性；
- 节点之前都维护了一个FIFO的队列，保证全局有序性；
- 通过全局递增的zxid保证因果有序性。


#### 状态流转
1. 服务在启动或者和leader失联之后服务状态转为LOOKING；
1. 如果leader不存在选举leader，如果存在直接连接leader，此时zab协议状态为ELECTION；
1. 如果有超过半数的投票选择同一台server，则leader选举结束，被选举为leader的server服务状态为LEADING，其他server服务状态为FOLLOWING/OBSERVING；
1. 所有server连接上leader，此时zab协议状态为DISCOVERY；
1. leader同步数据给learner，使各个从节点数据和leader保持一致，此时zab协议状态为SYNCHRONIZATION；
1. 同步超过一半的server之后，集群对外提供服务，此时zab状态为BROADCAST。