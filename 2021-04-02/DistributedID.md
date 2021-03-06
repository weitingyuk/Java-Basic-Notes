## 如何实现唯一的分布式 ID

### 业务系统对ID号的要求有哪些呢？

1. 全局唯一性：不能出现重复的ID号，既然是唯一标识，这是最基本的要求。
1. 趋势递增：在MySQL InnoDB引擎中使用的是聚集索引，由于多数RDBMS使用B-tree的数据结构来存储索引数据，在主键的选择上面我们应该尽量使用有序的主键保证写入性能。
1. 单调递增：保证下一个ID一定大于上一个ID，例如事务版本号、IM增量消息、排序等特殊需求。
1. 信息安全：如果ID是连续的，恶意用户的扒取工作就非常容易做了，直接按照顺序下载指定URL即可；如果是订单号就更危险了，竞对可以直接知道我们一天的单量。所以在一些应用场景下，会需要ID无规则、不规则。
- 其中123对应三类不同的场景，3和4需求还是互斥的，很难使用同一个方案满足。不过可以对id进行加密，就可以一定程度上解决3，4的矛盾（内部递增，外部加密或者部分加密）

### 常见方法介绍
#### 1. UUID
UUID(Universally Unique Identifier)的标准型式包含32个16进制数字

##### 优点：

- 性能非常高：本地生成，没有网络消耗。
##### 缺点：

- 不易于存储：UUID太长，16字节128位，通常以36长度的字符串表示，很多场景不适用。
- 信息不安全：基于MAC地址生成UUID的算法可能会造成MAC地址泄露
- UUID做DB主键的时候，UUID就非常不适用：
    - MySQL官方有明确的建议主键要尽量越短越好[4]，36个字符长度的UUID不符合要求。
    - 对MySQL索引不利：UUID的无序性可能会引起数据位置频繁变动，严重影响性能。


##### UUID各版本优缺点
###### 版本1 - 基于时间的UUID：

主要依赖当前的时间戳及机器mac地址，因此可以保证全球唯一性

- 优点：能基本保证全球唯一性
- 缺点：使用了Mac地址，因此会暴露Mac地址和生成时间
###### 版本2 - 分布式安全的UUID：

- 优点：能保证全球唯一性
- 缺点：很少使用，常用库基本没有实现
###### 版本3 - 基于名字空间的UUID（MD5版）：

- 优点：不同名字空间或名字下的UUID是唯一的；相同名字空间及名字下得到的UUID保持重复。
- 缺点：MD5碰撞问题，只用于向后兼容，后续不再使用
###### 版本4 - 基于随机数的UUID：
- 优点：实现简单
- 缺点：重复几率可计算
###### 版本5 - 基于名字空间的UUID（SHA1版）：
- 优点：不同名字空间或名字下的UUID是唯一的；相同名字空间及名字下得到的UUID保持重复。
- 缺点：SHA1计算相对耗时

总得来说：

- 版本 1/2 适用于需要高度唯一性且无需重复的场景；
- 版本 3/5 适用于一定范围内唯一且需要或可能会重复生成UUID的环境下；
- 版本 4 适用于对唯一性要求不太严格且追求简单的场景。
---
#### 2. 类snowflake方案
把64-bit分别划分成多段，分开来标示机器、时间等：
- 1bit不用
- 41-bit表示时间，可以表示（1L<<41）/(1000L*3600*24*365)=69年的时间
- 10-bit表示workID, 即可以分别表示1024台机器。如果我们对IDC划分有需求，还可以将10-bit分5-bit给IDC。
- 12-bit表示自增序列号，可以表示2^12个ID

理论上snowflake方案的QPS约为409.6w/s，这种分配方式可以保证在任何一个IDC的任何一台机器在任意毫秒内生成的ID都是不同的。

##### 优点：
- ID递增：毫秒数在高位，自增序列在低位，整个ID都是趋势递增的。
- 不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也是非常高的。
- 灵活：可以根据自身业务特性分配bit位，非常灵活。
##### 缺点：
- 强依赖机器时钟：强依赖机器时钟，如果机器上时钟回拨，会导致发号重复或者服务会处于不可用状态。
- 在单机上是递增的，但是到分布式环境，每台机器上的时钟不可能完全同步，有时候会出现不是全局递增的情况。
- workerid不好生成：如果不依赖外部服务，workerid其实并不好生成，一般实现还是依赖zk或者db的
- Snowflake会存在并发限制,理论上 Snowflake 方案的 QPS 约为 409.6w/s（1000 * 2^12），**解决方案待续**
##### 应用：
- Mongdb objectID
---

#### 3. 数据库生成
利用给字段设置来保证ID自增，每次业务使用下列SQL读写MySQL得到ID号。
##### 优点：
- 非常简单，利用现有数据库系统的功能实现，成本小。
- ID号单调自增，可以实现一些对ID有特殊要求的业务。
##### 缺点：
- 强依赖DB，当DB异常时整个系统不可用，属于致命问题。
- ID发号性能瓶颈限制在单台MySQL的读写性能。

##### Fix MySQL性能问题:
- 分布式系统中可以多部署几台机器，每台机器设置不同的初始值，且步长和机器数相等。
- 缺点：
    - 系统水平扩展比较困难,扩容复杂
    - ID没有了单调递增的特性，只能趋势递增
    - 数据库压力还是很大，每次获取ID都得读写一次数据库
---
#### 4. Redis生成ID
Redis的所有命令操作都是单线程的，本身提供像 incr 和 increby 这样的自增原子命令，所以能保证生成的 ID 肯定是唯一有序的。

##### 优点：
- 不依赖于数据库，灵活方便，且性能优于数据库；
- 数字ID天然排序，对分页或者需要排序的结果很有帮助。
##### 缺点：
- 如果系统中没有Redis，还需要引入新的组件，增加系统复杂度
- 需要编码和配置的工作量比较大。
---
#### 5. Leaf方案实现
##### 5.1 Leaf-segment数据库方案
在使用数据库的方案上，做了如下改变
- **每次获取一个segment**：原方案每次获取ID都得读写一次数据库，造成数据库压力大。改为利用proxy server批量获取，每次获取一个segment(step决定大小)号段的值。用完之后再去数据库获取新的号段，可以大大的减轻数据库的压力。 
- **各个业务不同的发号需求用biz_tag字段来区分**：每个biz-tag的ID获取相互隔离，互不影响。如果以后有性能需求需要对数据库扩容，不需要上述描述的复杂的扩容操作，只需要对biz_tag分库分表就行。
##### 优点：
- 方便的线性扩展：Leaf服务可以很方便的线性扩展，性能完全能够支撑大多数业务场景。
- 趋势递增：ID号码是趋势递增的8byte的64位数字。
- 容灾性高：Leaf服务内部有号段缓存，即使DB宕机，短时间内Leaf仍能正常对外提供服务。

##### 缺点：
- **ID号码不够随机**：能够泄露发号数量的信息，不太安全。
- TP999**数据波动**大，当号段使用完之后还是会hang在更新数据库的I/O上，tg999数据会出现偶尔的尖刺。
- **DB宕机**会造成整个系统不可用。

##### 5.2 双buffer优化 
对于第二个缺点，Leaf-segment做了一些优化：
- 优化目标：DB取号段的过程能够做到无阻塞。
    - Leaf **取号段的时机**是在号段消耗完的时候进行的，假如取DB的时候网络发生抖动，在这期间进来的请求也会因为DB号段没有取回来，导致线程阻塞。
- 优化方法：提前异步取号段
    - 号段消费到某个点时就异步的把下一个号段加载到内存中。而不需要等到号段用尽的时候才去更新号段。
    
##### 5.3. Leaf-snowflake方案
- 优化目标：解决订单趋势递增的ID
    - Leaf-segment方案可以生成趋势递增的ID，同时ID号是可计算的，不适用于订单ID生成场景，比如竞对在两天中午12点分别下单，通过订单id号相减就能大致计算出公司一天的订单量，这个是不能忍受的。
- 优化方法：
    - Leaf-snowflake方案完全沿用snowflake方案的bit位设计，同时使用Zookeeper持久顺序节点的特性自动对snowflake节点配置wokerID。

##### 5.4. 弱依赖ZooKeeper
除了每次会去ZK拿数据以外，也会在本机文件系统上缓存一个workerID文件。当ZooKeeper出现问题，恰好机器出现问题需要重启时，能保证服务能够正常启动。


#### 6. UidGenerator
UidGenerator是百度开源的分布式ID生成器，基于于snowflake算法的实现,CachedUidGenerator方式主要通过采取如下一些措施和方案规避了时钟回拨问题和增强唯一性：
- 自增列：UidGenerator的workerId在实例每次重启时初始化，且就是数据库的自增ID，从而实现每个实例获取到的workerId不会有任何冲突。
- RingBuffer：UidGenerator不再在每次取ID时都实时计算分布式ID，而是利用RingBuffer数据结构预先生成若干个分布式ID并保存。
- 时间递增：
    - 传统的雪花算法实现都是通过System.currentTimeMillis()来获取时间并与上一次时间进行比较，这样的实现严重依赖服务器的时间。
    - UidGenerator的时间类型是AtomicLong，且通过incrementAndGet()方法获取下一次的时间，从而脱离了对服务器时间的依赖，也就不会有时钟回拨的问题
    - 这种做法的问题：分布式ID中的时间信息可能并不是这个ID真正产生的时间点，例如：获取的某分布式ID的值为3200169789968523265，它的反解析结果为{"timestamp":"2019-05-02 23:26:39","workerId":"21","sequence":"1"}，但是这个ID可能并不是在"2019-05-02 23:26:39"这个时间产生的。