## 保证MQ消息可靠性
MQ要想尽量消息必达，架构上有两个核心设计点：

- （1）消息落地
- （2）消息超时、重传、确认

### MQ消息可靠投递核心流程
MQ将消息投递拆成了上下半场，为了保证消息的可靠投递，上下半场都必须尽量保证消息必达。

##### MQ消息投递
MQ消息投递上半场，MQ-client-sender到MQ-server流程：

- （1）MQ-client将消息发送给MQ-server（此时业务方调用的是API：SendMsg）
- （2）MQ-server将消息落地，落地后即为发送成功
- （3）MQ-server将应答发送给MQ-client（此时回调业务方是API：SendCallback）


MQ消息投递下半场，MQ-server到MQ-client-receiver流程：

- （1）MQ-server将消息发送给MQ-client（此时回调业务方是API：RecvCallback）
- （2）MQ-client回复应答给MQ-server（此时业务方主动调用API：SendAck）
- （3）MQ-server收到ack，将之前已经落地的消息删除，完成消息的可靠投递

##### 如果消息丢了怎么办？
MQ消息投递的上下半场，都可以出现消息丢失，为了降低消息丢失的概率，MQ需要进行超时和重传。

###### 上半场的超时与重传
- 超时重传：如果MQ上半场丢失或者超时，MQ-client-sender内的timer会重发消息，直到期望收到消息，如果重传N次后还未收到，则SendCallback回调发送失败，需要注意的是，这个过程中MQ-server可能会收到同一条消息的多次重发。

###### 下半场的超时与重传
- 指数退避的超时重传：MQ下半场的4或者5或者6如果丢失或者超时，MQ-server内的timer会重发消息，直到收到5并且成功执行6，这个过程可能会重发很多次消息，一般采用指数退避的策略，先隔x秒重发，2x秒重发，4x秒重发，以此类推，需要注意的是，这个过程中MQ-client-receiver也可能会收到同一条消息的多次重发。

