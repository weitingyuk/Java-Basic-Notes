## Java是如何实现线程安全的，哪些数据结构是线程安全的？

## 线程安全的实现
- **为什么需要线程安全**：Java的多线程是一个同时运行多个线程的过程。当多个线程处理同一个数据，并且数据的值在变化时，这种情况就不是线程安全的，会产生数据不一致的结果。
- **线程安全是什么**：当一个线程已经在处理一个对象，并且阻止其他线程处理同一个对象时，这个过程称为线程安全。
- **线程安全的本质**：
    - 多线程之间会共享主线；
    - 线程不能直接读写主内存的共享变量，每个线程都有自己的工作内存
    - 线程需要读写主内存的共享变量时需要先将该变量拷贝一份副本到自己的工作内存，然后在自己的工作内存中对该变量进行操作，完成操作之后需要将结果同步至主内存。
 
### 如何实现线程安全
如果要实现线程安全，就需要保证共享内存的原子性、可见性、有序性
##### 在Java中实现线程安全有四种方法。这些是：
- 使用Synchronization。
- 使用Volatile关键字。
- 使用Atomic变量。
- 使用Final关键字。

### 使用Synchronization
Synchronized 关键字
1. 保证方法或代码块操作的原子性
- Synchronized 保证⽅法内部或代码块内部资源（数据）的互斥访问。即同⼀时间、由同⼀个Monitor监视的代码，最多只能有⼀个线程在访问。
2. 保证监视资源的可见性
- 保证多线程环境下对Monitor资源的数据同步。即任何线程在获取到 Monitor 后的第⼀时间，会先将共享内存中的数据复制到⾃⼰的缓存中；任何线程在释放 Monitor 的第⼀ 时间，会先将缓存中的数据复制到共享内存中。
3. 保证线程间操作的有序性
Synchronized 的原子性保证了由其描述的方法或代码操作具有有序性，同一时间只能由最多只能有一个线程访问，不会触发 JMM 指令重排机制。


### 使用Volatile关键字
保证被 Volatile关键字描述变量的操作具有可见性和有序性（禁止指令重排），但不能保证原子性

### 使用原子变量
- java.util.concurrent.atomic包提供了一系列的 AtomicBoolean、AtomicInteger、AtomicLong 等类。使用这些类来声明变量可以保证对其操作具有原子性来保证线程安全。
- **实现原理**：与Synchronized 使用Monitor保证资源在多线程环境下阻塞互斥访问不同，java.util.concurrent.atomic 包下的各原子类是基于 CAS(CompareAndSwap) 操作原理实现。

### 使用Final关键字
Final变量在java中也是线程安全的，因为一旦分配了某个对象的引用，它就不能指向另一个对象的引用。

### 使用Lock
- java.util.concurrent包下的一个接口，定义了一系列的锁操作方法。主要有ReentrantLock，ReentrantReadWriteLock.ReadLock，ReentrantReadWriteLock.WriteLock 实现类。

## 线程安全的数据结构
1. HashTable
1. ConcurrentHashMap
1. CopyOnWriteArrayList
1. CopyOnWriteArraySet
1. ConcurrentLinkedQueue
1. Vector
1. StringBuffer

### HashTable
- 线程安全的实现方法：HashTable使用synchronized来修饰方法函数来保证线程安全
- 缺点：多线程运行环境下效率表现非常低

### ConcurrentHashMap
- 线程安全的实现方法：用分段锁（JDK1.7）或者是CAS+Synchronized（JDK1.8）来实现的。
- 优点：不仅保证了多线程运行环境下的数据访问安全性，而且性能上有长足的提升。

### CopyOnWriteArrayList
- 线程安全的实现方法：内部有volatile来保持数组，再加ReentrantLock互斥锁来实现的。
- 当add\update\delete操作的时候需要获取锁。当add元素的时候，先获取锁，再使用Arrays.copyOf()来拷贝新建新的数组，在副本上add元素，然后改变原引用指向新副本，最后再释放锁。
- 适用场景：读操作远远多于写操作的应用。

### CopyOnWriteArraySet
- CopyOnWriteArraySet是对CopyOnWriteArrayList使用了装饰模式后的具体实现。

### ConcurrentLinkedQueue
- 基于链表的无界线程安全队列，按FIFO排序。
- 继承AbstractQueue,通过volatile实现多线程对竞争资源的互斥访问。

#### Reference:
- https://www.geeksforgeeks.org/thread-safety-and-how-to-achieve-it-in-java/
- https://dzone.com/articles/7-techniques-for-thread-safe-classes
- https://juejin.cn/post/6844903890224152584#heading-5
