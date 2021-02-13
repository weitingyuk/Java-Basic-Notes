## BlockingQueue
ArrayBlockingQueue和LinkedBlockingQueue都是实现了BlockingQueue接口，BlockingQueue接口定义了一种阻塞的FIFO queue，每一个BlockingQueue都有一个容量，让容量满时往BlockingQueue中添加数据时会造成阻塞，当容量为空时取元素操作会阻塞。

BlockingQueue主要提供下面的接口
#### 1. 3个添加元素方法
- add：添加元素到队列里，添加成功返回true，由于容量满了添加失败会抛出IllegalStateException异常
- offer：添加元素到队列里，添加成功返回true，添加失败返回false
- put：添加元素到队列里，如果容量满了会阻塞直到容量不满
#### 2. 3个删除方法
- poll：删除队列头部元素，如果队列为空，返回null。否则返回元素。
- remove：基于对象找到对应的元素，并删除。删除成功返回true，否则返回false
- take：删除队列头部元素，如果队列为空，一直阻塞到队列有元素并删除

## ArrayBlockingQueue和LinkedBlockingQueue的实现原理

### ArrayBlockingQueue
- ArrayBlockingQueue是一个带有长度的阻塞队列，初始化的时候必须要指定队列长度，且指定长度之后不允许进行修改。

- ArrayBlockingQueue的原理就是使用一个可重入锁和这个锁生成的两个条件对象进行并发控制(classic two-condition algorithm: notEmpty, notFull):
    1.  若某线程(线程A)要取数据时，数组正好为空，则该线程会执行notEmpty.await()进行等待；当其它某个线程(线程B)向数组中插入了数据之后，会调用notEmpty.signal()唤醒“notEmpty上的等待线程”。此时，线程A会被唤醒从而得以继续运行。
    2.  若某线程(线程H)要插入数据时，数组已满，则该线程会它执行notFull.await()进行等待；当其它某个线程(线程I)取出数据之后，会调用notFull.signal()唤醒“notFull上的等待线程”。此时，线程H就会被唤醒从而得以继续运行。

### LinkedBlockingQueue
1. LinkedBlockingQueue继承于AbstractQueue，它本质上是一个FIFO(先进先出)的队列。
2. LinkedBlockingQueue实现了BlockingQueue接口，它支持多线程并发。当多线程竞争同一个资源时，某线程获取到该资源之后，其它线程需要阻塞等待。
3. LinkedBlockingQueue是通过单链表实现的。
4. LinkedBlockingQueue在实现“多线程对竞争资源的互斥访问”时，对于“插入”和“取出(删除)”操作分别使用了不同的锁。
- putLock是插入锁，takeLock是取出锁；notEmpty是“非空条件”，notFull是“未满条件”。通过它们对链表进行并发控制。
-  对于插入操作，通过“插入锁putLock”进行同步,插入锁putLock和“条件notFull”相关联；
-  对于取出操作，通过“取出锁takeLock”进行同步, 取出锁takeLock和“条件notEmpty”相关。

### ArrayBlockingQueue和LinkedBlockingQueue的区别

#### 队列中锁的实现不同

    ArrayBlockingQueue实现的队列中的锁是没有分离的，即生产和消费用的是同一个锁；

    LinkedBlockingQueue实现的队列中的锁是分离的，即生产用的是putLock，消费是takeLock

#### 在生产或消费时操作不同

    ArrayBlockingQueue实现的队列中在生产和消费的时候，是直接将枚举对象插入或移除的；

    LinkedBlockingQueue实现的队列中在生产和消费的时候，需要把枚举对象转换为Node<E>进行插入或移除，会影响性能

#### 队列大小初始化方式不同

    ArrayBlockingQueue实现的队列中必须指定队列的大小；

    LinkedBlockingQueue实现的队列中可以不指定队列的大小，但是默认是Integer.MAX_VALUE
    
### ArrayBlockingQueue和LinkedBlockingQueue的使用场景
#### ArrayBlockingQueue
- 分析:
    - 由于基于数组，容量固定所以不容易出现内存占用率过高，但是如果容量太小，取数据比存数据的速度慢，那么会造成过多的线程进入阻塞(也可以使用offer()方法达到不阻塞线程)
    - 由于存取共用一把锁，所以有高并发和吞吐量的要求情况下也不建议使用ArrayBlockingQueue。
- 使用场景 
    - 案例：人事系统中员工离职/变更后，其他依赖应用进行数据同步。
    - 适合变更操作不是非常频繁的场景，这样能有效防止线程阻塞。
    - 如果基本没有并发和吞吐量的要求，所以可以将数据存放到ArrayBlockingQueue中。
#### LinkedBlockingQueue
- 特征: 
    - LinkedBlockingQueue基于链表实现，队列容量默认Integer.MAX_VALUE存/取数据的操作分别拥有独立的锁，可实现存/取并行执行。

- 分析:
    - 基于链表，数据的新增和移除速度比数组快，但是每次存储/取出数据都会有Node对象的新建和移除，所以也存在由于GC影响性能的可能
    - 默认容量非常大，所以存储数据的线程基本不会阻塞，但是如果消费速度过低，内存占用可能会飙升。
    - 读/取操作锁分离，所以适合有并发和吞吐量要求的项目中
- 使用场景:
    - 在项目的一些核心业务且生产和消费速度相似的场景中: 订单完成的邮件/短信提醒。
    - 如果订单的成交量非常大，那么使用ArrayBlockingQueue就会有一些问题，固定数组很容易被使用完，此时调用的线程会进入阻塞，那么可能无法及时将消息推送出去，所以使用LinkedBlockingQueue比较合适，但是要注意消费速度不能太低，不然很容易内存被使用完