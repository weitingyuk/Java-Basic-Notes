## MQ消费幂等
为了防止消息重复消费导致业务处理异常，消息队列RocketMQ版的消费者在接收到消息后，有必要根据业务上的唯一Key对消息做幂等处理。

### 消息重复的场景
1. 发送时消息重复
- 当一条消息已被成功发送到服务端并完成持久化，此时出现了网络闪断或者客户端宕机，导致服务端对客户端应答失败。 如果此时生产者意识到消息发送失败并尝试再次发送消息，消费者后续会收到两条内容相同并且Message ID也相同的消息。
2. 投递时消息重复
- 消息消费的场景下，消息已投递到消费者并完成业务处理，当客户端给服务端反馈应答的时候网络闪断。为了保证消息至少被消费一次，消息队列RocketMQ版的服务端将在网络恢复后再次尝试投递之前已被处理过的消息，消费者后续会收到两条内容相同并且Message ID也相同的消息。
3. 负载均衡时消息重复（包括但不限于网络抖动、Broker重启以及消费者应用重启） 
- 当消息队列RocketMQ版的Broker或客户端重启、扩容或缩容时，会触发Rebalance，此时消费者可能会收到重复消息。

### 解决方案
不同的Message ID对应的消息内容可能相同，有可能出现冲突（重复）的情况，所以真正安全的幂等处理，是以业务唯一标识作为幂等处理的关键依据，而业务的唯一标识可以通过消息Key设置。