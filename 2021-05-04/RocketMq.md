## RocketMQ简介
### 系统部署架构

1. Name Server：是一个几乎无状态节点，可集群部署，在消息队列RocketMQ版中提供命名服务，更新和发现Broker服务。
1. Broker：消息中转角色，负责存储消息，转发消息。分为Master Broker和Slave Broker，一个Master Broker可以对应多个Slave Broker，但是一个Slave Broker只能对应一个Master Broker。Broker启动后需要完成一次将自己注册至Name Server的操作；随后每隔30s定期向Name Server上报Topic路由信息。
1. 生产者：与Name Server集群中的其中一个节点（随机）建立长链接（Keep-alive），定期从Name Server读取Topic路由信息，并向提供Topic服务的Master Broker建立长链接，且定时向Master Broker发送心跳。
1. 消费者：与Name Server集群中的其中一个节点（随机）建立长连接，定期从Name Server拉取Topic路由信息，并向提供Topic服务的Master Broker、Slave Broker建立长连接，且定时向Master Broker、Slave Broker发送心跳。Consumer既可以从Master Broker订阅消息，也可以从Slave Broker订阅消息，订阅规则由Broker配置决定。

### 消息类型

1. 普通消息：消息队列RocketMQ版中无特性的消息，区别于有特性的定时和延时消息、顺序消息和事务消息。
1. 事务消息：实现类似X/Open XA的分布事务功能，以达到事务最终一致性状态。
1. 定时和延时消息：允许消息生产者对指定消息进行定时（延时）投递，最长支持一定时间，例如40天。
1. 顺序消息：允许消息消费者按照消息发送的顺序对消息进行消费。
### 消息特性
1. 消息重试：在消费者返回消息重试的响应后，消息队列RocketMQ版会按照相应的重试规则进行消息重投。
1. 至少投递一次（At-least-once）：消息队列RocketMQ版保证消息成功被消费一次。消息队列RocketMQ版的分布式特点和瞬变的网络条件，或者用户应用重启发布的情况下，可能导致消费者收到重复的消息。开发人员应将其应用程序设计为多次处理一条消息不会产生任何错误或不一致性。


### 集群消费和广播消费
1. 集群消费模式：
- 消息队列RocketMQ版认为任意一条消息只需要被消费者集群内的任意一个消费者处理即可；
2. 广播消费模式：
- 消息队列RocketMQ版会将每条消息推送给消费者集群内所有注册过的消费者，保证消息至少被每台机器消费一次。