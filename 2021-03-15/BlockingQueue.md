## 阻塞队列都有哪几种，有什么区别？
### 未阻塞和阻塞队列的区别
#### 未阻塞:
- 未阻塞的队列在并发想队列添加或者取得数据的时候,必定只会有一个成功,其他都可能添加失败.

#### 阻塞:
- 阻塞的队列会进行线程阻塞操作,让并发的添加或者取得数据进行一定程度的延迟,可以保证大量并发数据的添加. 但是阻塞也是有超时时间的.. 超过一段时间后依然会抛出异常或者抛出false


### 没有实现阻塞接口
1. LinkList
    - 实现java.util.Queue的LinkList, 
1. PriorityQueue
    - 实现java.util.AbstractQueue接口内置的不阻塞队列
1. ConcurrentLinkedQueue
    - 实现java.util.AbstractQueue接口内置的不阻塞队列

 
### 实现阻塞接口的
java.util.concurrent 中加入了 BlockingQueue 接口和五个阻塞队列类。它实质上就是一种带有一点扭曲的 FIFO 数据结构。不是立即从队列中添加或者删除元素，线程执行操作阻塞，直到有空间或者元素可用。
五个队列所提供的各有不同：　　
1. ArrayBlockingQueue ：
    1. 一个由数组支持的有界队列。
1. LinkedBlockingQueue ：
    1. 一个由链接节点支持的可选有界队列。LinkedBlockingQueue的容量（在不指定时容量为Integer.MAX_VALUE），但是也可以选择指定其最大容量，它是基于链表的队列，此队列按 FIFO（先进先出）排序元素。
1. PriorityBlockingQueue ：
    1. 一个由优先级堆支持的无界(没有容量限制)优先级队列。 是一个带优先级的 队列，而不是先进先出队列。元素按优先级顺序被移除.
1. DelayQueue ：
    1. 一个由优先级堆支持的、基于时间的调度队列。（基于PriorityQueue来实现的）是一个存放Delayed 元素的无界阻塞队列，只有在延迟期满时才能从中提取元素。该队列的头部是延迟期满后保存时间最长的 Delayed 元素。如果延迟都还没有期满，则队列没有头部，并且poll将返回null。当一个元素的 getDelay(TimeUnit.NANOSECONDS) 方法返回一个小于或等于零的值时，则出现期满，poll就以移除这个元素了。此队列不允许使用 null 元素。
1. SynchronousQueue ：
    1. 一个利用 BlockingQueue 接口的简单聚集机制，每个 put 必须等待一个 take，反之亦然。同步队列没有任何内部容量，甚至连一个队列的容量都没有。