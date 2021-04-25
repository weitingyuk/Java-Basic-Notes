## AQS实现原理

### 1. 什么是AQS
- AQS的全称是AbstractQueuedSynchronizer，它本质上是一个抽象类
- AQS是Java中几乎所有锁和同步器的一个基础框架


### 2. AQS原理
- 如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。
- 如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。
#### 2.1 CLH队列
- CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。
- AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

#### 2.2 AQS状态
- AQS使用一个int成员变量来表示同步状态，通过内置的FIFO队列来完成获取资源线程的排队工作。
- AQS使用CAS对该同步状态进行原子操作实现对其值的修改。


```
private volatile int state;//共享变量，使用volatile修饰保证线程可见性
```
### 3. AQS实现资源的共享
#### 3.1 AQS定义两种资源共享方式

- Exclusive（独占）：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
    1. 公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
    1. 非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
- Share（共享）：多个线程可同时执行
    - 如Semaphore/CountDownLatch。Semaphore、CountDownLatCh、 CyclicBarrier、ReadWriteLock
    - ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。

#### 3.2 AQS底层使用了模板方法模式
自定义同步器时需要重写下面几个AQS提供的模板方法：


```
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```
默认情况下，每个方法都抛出 UnsupportedOperationException。 

### 4. AQS的应用

###### 4.1 ReentrantLock获取锁流程
1. state初始化为0，表示未锁定状态。
1. A线程lock()时，会调用tryAcquire()独占该锁并将state+1。
1. 此后，其他线程再tryAcquire()时就会失败
1. 直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。
1. 释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。
1. 注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

###### 4.2 CountDownLatch获取锁流程
1. 任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。
1. 这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS(Compare and Swap)减1。
1. 等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。
###### 4.3 Semaphore(信号量)的锁流程
- synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源
- Semaphore(信号量)可以指定多个线程同时访问某个资源。
- Semaphore经常用于限制获取某种资源的线程数量。

Semaphore 有两种模式，公平模式和非公平模式。

- 公平模式： 调用acquire的顺序就是获取许可证的顺序，遵循FIFO；
- 非公平模式： 抢占式的。
###### 4.4 CountDownLatch （倒计时器）的锁流程
- CountDownLatch是一个同步工具类，用来协调多个线程之间的同步。
- 类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。
1. 某一线程在开始运行前等待n个线程执行完毕。将 CountDownLatch 的计数器初始化为n ：
```
new CountDownLatch(n)
```
2. 每当一个任务线程执行完毕，就将计数器减1 
```
countdownlatch.countDown()
```
3. 当计数器的值变为0时，在CountDownLatch上 await() 的线程就会被唤醒。

一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。

- CountDownLatch不足
- CountDownLatch是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用。

###### 4.5 CyclicBarrier(循环栅栏)

#### LockSupport

```
public static void park() : 如果没有可用许可，则挂起当前线程,停车,表示让线程暂停。
public static void unpark(Thread thread)：给thread一个可用的许可，让它得以继续执行，表示让线程继续执行。
```


