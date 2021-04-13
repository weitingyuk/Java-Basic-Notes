## 设计一个阻塞队列

Java实现阻塞队列的几种方式

#### 一、生产者消费者问题
现在用五种方式来实现生产者消费者模型

1. wait()和notify()方法

- 使用synchronized来进行同步；
- 缓冲区满和为空时都调用wait()方法等待；
- 当生产者生产了一个产品或者消费者消费了一个产品之后会唤醒所有线程。

2. 可重入锁ReentrantLock

- 创建一个锁对象ReentrantLock，注意；需要释放锁
- 为这个锁创建两个条件Condition变量，一个为缓冲区notFull，一个为缓冲区notEmpty

3. 阻塞队列BlockingQueue

- 定义一个阻塞队列，其中lockingQueue的put和take时阻塞的方法

4. 信号量Semaphore

- 添加两个信号量notFull和notEmpty作为许可集；
- 可以使用acquire()方法获得一个许可，当许可不足时会被阻塞，release()添加一个许可；
- 加入了另外一个mutex信号量，维护生产者消费者之间的同步关系，保证生产者和消费者之间的交替进行


5. 管道输入输出流PipedInputStream和PipedOutputStream

- 先创建一个管道输入流和管道输出流，然后将输入流和输出流进行连接
- 生产者线程往管道输出流中写入数据，消费者在管道输入流中读取数据，这样就可以实现了不同线程间的相互通讯

注意：这种方式在生产者和生产者、消费者和消费者之间不能保证同步，也就是说在一个生产者和一个消费者的情况下是可以生产者和消费者之间交替运行的，多个生成者和多个消费者者之间则不行
#### 二、采用BlockingQueue实现    
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

