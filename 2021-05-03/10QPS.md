## 假如明天是活动高峰？QPS预计会翻10倍，你要怎么做？

### 问题分析
- 和秒杀业务有些类似，不过整个准备时间比较紧张，仅仅只有一天。所有的方案都要考虑在一天内的可行性。
- 架构设计的时候要保证代码的健壮性，这种临时活动尽量不要大量修改代码。
- 为了要保证活动中服务可行，我们可以做的事情：

方案一：
- 临时加一套全新的服务，单独为活动准备，不影响线上的其他功能的正常使用。
- 再走方案二的步骤。

方案二：
- 全量压测：评估现在的服务API的读写比例，再进行全量压测，评估整个服务是否可以支撑10倍的QPS，如果不可以，看看是哪个环节是瓶颈，做特定处理。
- 限流：如果活动库存有限，只有少部分用户才能购买成功，需要限制大部分用户的流量，只有少部分才能进入后端服务。
- 削峰：活动开始的瞬间，有大量用户冲进来，导致了瞬间流量峰值。一般要用MQ中间件来实现流量的削峰填谷。
- 加缓存：可以加缓存的业务尽量加缓存，而且将热点数据的预加载到缓存里面，同时要防止缓存的穿透、击穿、雪崩等问题。可以提高并发效率。
- 降级：如果是其他的非主要业务的请求阻塞时间太长导致的吞吐量下降，可以考虑降级：把一些不是主要业务的功能降级。
- DB：读写分离。