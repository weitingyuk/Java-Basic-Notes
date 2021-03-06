## 常用的限流算法有哪些？简述令牌桶算法原理
### 一、计数器（固定窗口）算法
1. 计数器算法是使用计数器在周期内累加访问次数，当达到设定的限流值时，触发限流策略。
1. 下一个周期开始时，进行清零，重新计数。
2. 在单机还是分布式环境下实现都非常简单，使用redis的incr原子自增性和线程安全即可轻松实现。

#### 缺点：临界问题
- 假设1min内服务器的负载能力为100，因此一个周期的访问量限制在100
- 然而在第一个周期的最后5秒和下一个周期的开始5秒时间段内，分别涌入100的访问量，虽然没有超过每个周期的限制量，但是整体上10秒内已达到200的访问量，已远远超过服务器的负载能力
- 计数器算法方式限流对于周期比较长的限流，存在很大的弊端。

### 二、滑动窗口算法
滑动窗口算法是将时间周期分为N个小周期，分别记录每个小周期内访问次数，并且根据时间滑动删除过期的小周期。

1. 假设时间周期为1min，将1min再分为2个小周期
1. 统计每个小周期的访问数量，第一个时间周期内，访问数量为75
1. 第二个时间周期内，访问数量为100，超过100的访问则被限流掉了
#### 特点：
当滑动窗口的格子划分的越多，那么滑动窗口的滚动就越平滑，限流的统计就会越精确。
#### 优点：
此算法可以很好的解决固定窗口算法的临界问题。


### 三、漏桶算法
1. 漏桶算法是访问请求到达时直接放入漏桶
1. 如当前容量已达到上限（限流值），则进行丢弃（触发限流策略）。
1. 漏桶以固定的速率进行释放访问请求（即请求通过），直到漏桶为空。
#### 特点
- 因为出水速度是恒定的，那么意味着如果瞬时大流量的话，将有大部分请求被丢弃掉
- 那么当就无法应对短时间的突发流量.

### 四、令牌桶算法
1. 令牌桶算法的原理是系统会以一个恒定的速度往桶里放入令牌
1. 而如果请求需要被处理，则需要先从桶里获取一个令牌
1. 当桶里没有令牌可取时，则拒绝服务。
#### 特点
- 令牌桶算法生成令牌的速度是恒定的，而请求去拿令牌是没有速度限制的。
- 这意味，面对瞬时大流量，该算法可以在短时间内请求拿到大量令牌，而且拿令牌的过程并不是消耗很大的事情。