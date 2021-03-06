## 基于Redis实现消息队列的7种方案

1. 基于List的 LPUSH+RPOP 的实现
1. 基于List的 LPUSH+BRPOP 的实现
1. 基于List的 LPUSH+LRANGE+RPOP 的实现
1. 基于List的 LPUSH+BRPOPLPUSH+LREM 的实现
1. 基于Sorted Set 的实现
1. PUB/SUB，订阅/发布模式
1. 基于Stream类型的实现（在Redis5.0中增加）

### 一、基于List的 LPUSH+RPOP 的实现方案

1. LPUSH在头部（List的左边）添加一个新的元素并返回List长度，充当消息队列中的生成者。
1. RPOP在尾部（List的右边）删除一个元素并返回该元素的值，充当消息队列中的消费者。

基于List类型的插入删除元素操作实现，就是一个典型的先进先出队列的解决方案。

#### 优点：
- List类型是基于链表实现，插入删除元素时间复杂度仅为常量级，有先进先出的特点来保证数据的顺序
- Redis支持消息持久化，在服务端数据是安全的
#### 缺点：
- 消费确认机制实现麻烦加之不能重复消费，一但消费数据就会删除，导致客户端数据是不安全的，当客户端宕机、网络断开会出现数据丢失，也不能实现广播模式
- 没有数据权重的概念，只能先进先出
- 当队列中没有元素时，消费者需要轮询获取数据，会增加Redis的访问压力，增加客户端的cpu占用
不支持分组消费

### 二、基于List的 LPUSH+BRPOP 的实现方案

- 方案二是在方案一上针对队列没有元素时造成服务器资源浪费进行的优化方案，使用了BRPOP做消费者，BRPOP是阻塞的。
- 消费者可以设置数据不存在时的阻塞时间，来减少不必要的轮询。

### 三、基于List的 LPUSH+LRANGE+RPOP 的实现
LRANGE:获取列表指定范围内的元素
- 方案三是在方案一上针对客户端数据安全进行的优化方案，使用LRANGE首先对队列元素只做读取不做消费，在**客户端消费完成后，再使用RPOP对服务端进行消费。**
- 由于LRANGE不是阻塞的就又回到了方案二解决的资源浪费问题上了，无法减少不必要的轮询。
- 重复执行：还存在重复执行的问题，由于先读再消费，在消费者宕机重启后会再次读到没有确认消费的但是已经在消费者处理过的元素，就有了重复消费的风险。

### 四、基于List的 LPUSH+BRPOPLPUSH+LREM 的实现
Redis Lrem 根据参数 COUNT 的值，移除列表中与参数 VALUE 相等的元素。

COUNT 的值可以是以下几种：
```
count > 0 : 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT 。
count < 0 : 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。
count = 0 : 移除表中所有与 VALUE 相等的值。
```

- 方案四是也对客户端数据安全进行的优化方案，是一种安全的队列，虽然也会存在重复消费的风险，但是元素队列的操作都是在服务端进行的，问题发生的概率会大大降低。
- 首先消息队列数量增加了，一个存储待消费消息的队列暂且称为A，一个存储正在消费消息的队列暂且称为B，关键在与BRPOPLPUSH操作，RPOPLPUSH是将A中队尾的消息删除（消费掉）并添加到B队列的队头。
- BRPOPLPUSH是原子性的操作，所以不用担心从A中消费后数据丢失的问题。
- BRPOPLPUSH是阻塞的，不过你也可以使用RPOPLPUSH非阻塞模式。

这个方案当然也不是完美的，还是存在客户端宕机的情况，正在处理中的队列存在长期不消费的消息怎么办？

可以再添加一台客户端来监控长期不消费的消息，重新将消息打回待消费的队列，这个可以使用循环队列的模式来实现。

### 五、基于Sorted Set的实现方案
1. 优先队列的特点，能保证每次取出的元素都是队列中优先级别最高的。这一特点是使用List类型无法满足的，数据只能是先进先出的。
1. 有序集合（Sorted Set）是不允许重复的String类型元素的集合，且每个元素都会关联一个Double类型的分数。有序集合的成员是唯一的，但分数是可以重复。

##### Sorted Set应用 
基于Sorted Set以上的特点在实际开发中有许多的应用，比如做游戏的实时战绩排行榜、博客中文章点赞排行榜等各类排行榜和优先队列的实现。 


```
1. ZADD在集合中添加一个带有分数的元素。
2. ZRANGE返回有序集中，指定区间内的成员，其中成员的位置按分数值递增(从小到大)来排序；我们使用0，0区间来获取处于顶部的元素。
3. ZREVRANGE与ZRANGE不同的是成员的位置按分数值递减(从大到小)来排列。
4. ZREM用来移除有序集中的一个元素，不存在的成员将被忽略。当消费0，0区间元素成功后，移除该元素。
```
##### 优点：
1. Sorted Set类型是基于使用了hash和skiplist两种设计实现；添加和删除都需要修改skiplist，所以复杂度为O(log(n))；查找元素的话可以直接使用hash，其复杂度为O(1) 
1. 可以实现按权值取元素
1. Redis支持消息持久化，在服务端数据是安全的
##### 缺点：
- 消费确认机制需要单独实现
- 不支持阻塞式获取
- 不允许重复消息
- 不支持分组消费

Sorted Set不仅可以实现**优先队列**，还可以将权值当做有序队列的自定义序号，例如**使用时间戳**就可以实现一个有序队列了，但是**如果是做有序队列当然没有List类型来的方便了。**


### 六、基于PUB/SUB，订阅/发布模式的实现方案

- SUBSCRIBE用于订阅一个给定模式信道
- PUBLISH用于将消息发送到指定的信道
- UNSUBSCRIBE用于取消订阅指定信道

#### 优点：
1. 典型的广播模式，一个消息可以发布到多个消费者 
1. 多信道订阅，消费者可以同时订阅多个信道，从而接收多类消息
1. 消息即时发送，消息不用等待消费者读取，消费者会自动接收到信道发布的消息
#### 缺点：
1. 消息一旦发布，不能接收。换句话就是发布时若客户端不在线，则消息丢失，不能寻回
1. 不能保证每个消费者接收的时间是一致的
1. 若消费者客户端出现消息积压，到一定程度，会被强制断开，导致消息意外丢失。通常发生在消息的生产远大于消费速度时

可见，Pub/Sub 模式不适合做消息存储，消息积压类的业务，而是擅长处理广播，即时通讯，即时反馈的业务。



### 七、基于Stream实现的消息队列
主要分为两个类型的消息队列，分别支持广播模式和分组消费模式。

- 基于XADD+XREAD+XDEL实现的有序消息队列
- 基于XADD+XGROUP+XACK+XPENDING+XCLAIM实现的有序消息队列

###### 方案一
基于XADD+XREAD+XDEL实现的有序消息队列， 用来做消息队列并不是一个很好的选择：
1. 第一个原因是XREAD读消息的特点是所有客户端共享队列中的所有消息，是典型的广播模式和发布/订阅模式类型
2. 第二个原因是XREAD是读取消息队列的数据并没有确认操作，需要借助XDEL删除队列消息或者在创建队列是使用MAXLEN来限制队列大小
    - XDEL操作不是作者推荐的方式，是因为stream队列是使用基数树来存储索引的，当删除一个消息时，并不是真正的树中将结点删除，而是标记为删除，只有点宏结点中所有结点都被删除了才会销毁该结点。
    - 当频繁的对队列进行删除操作但没有全部删除时，会出现队列占用的内存空间超出你的预期。

###### 方案二
方案二基于XADD+XGROUP+XACK+XPENDING+XCLAIM实现的有序消息队列，考虑就完善多了，下面会围绕以下6个特点介绍：
###### 1. 消息ID的格式化
- 使用XADD命令可以向队列中添加消息，当被添加的key不存在时创建新的队列。
- XADD拥有子命令MAXLEN，正如上面提到的它可以设置队列的消息上限，你还可以通过~来设置一个不精确的上限，例如MAXLEN~ 1000。
- XADD命令的返回值是消息ID，不管你是自定义的ID或者通过*使用默认的ID，ID都必须满足单向递增且唯一的特点。
- 默认ID是毫秒级的时间戳+当前时间的消息序号组成的，两个数字都是64的，基本上是没有机会重复，还有大型分布式项目都是靠Redis来生唯一ID的，所以最好还是使用默认的ID生成规则。
###### 2. 消息的遍历
- 使用XREADGROUP来获取分组后的消息，读消息时会告知分组名称和当前客户端名称，为后面的消息监控和消息流转做支持。

###### 3. 消息的阻塞/非阻塞读取
- 可以使用XPENDING来查看未确认的消息ID列表，可以查看知道的客户端的PEL，返回值包括消息ID、客户端名称（消息的所有者）、消息传递给客户端到现在的毫秒数、该消息被传递的次数。

###### 4. 消息的分组消费
- 使用XGROUP来对队列设置分组，把客户端与分组进行关联。

- XREADGROUP与XREAD的操作基本上是一致的，都有COUNT子命令来获取返回消息数量；BLOCK选择阻塞时长，单位毫秒，时长为0时永远阻塞，默认不是要BLOCK是非阻塞模式；STREAMS限定接收的消息ID。
###### 5. 消息确认机制
- 使用XACK确认消息处理，使用XREADGROUP读取的消息，服务器将会记住消息传递的客户端名称，消息会被存储在消费者组内的待处理条目列表（PEL）中，即已送达但尚未确认的消息ID列表。
###### 6. 消息队列监控
- 使用XINFO命令来实现对服务器信息的监控。
###### 7. 消息传递
- XCLAIM: 在消息被确认处理前消息的所有者是有可能宕机的，短时间内无法恢复的时候，我们不能让这个消息一直在所有者的PEL，需要使用XCLAIM命令把消息传递给新的客户端，让消息继续被处理。
