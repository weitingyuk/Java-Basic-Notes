## 线程池是如何实现的？简述线程池的任务策略

### 一. 线程池是如何实现的？
创建一个线程池需要输入几个参数：
```
new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, milliseconds,runnableTaskQueue, threadFactory,handler);
```
- corePoolSize（线程池的基本大小）
- runnableTaskQueue（任务队列）：ArrayBlockingQueue，LinkedBlockingQueue，SynchronousQueue，PriorityBlockingQueue
- maximumPoolSize（线程池最大大小）
- ThreadFactory（设置创建线程的工厂）
- keepAliveTime（线程活动保持时间）
- TimeUnit（线程活动保持时间的单位）

### 二. 线程池的主要工作流程
当提交一个新任务到线程池时，线程池的处理流程如下：

1. 首先线程池判断基本线程池是否已满？没满，创建一个工作线程来执行任务。满了，则进入下个流程。
1. 其次线程池判断工作队列是否已满？没满，则将新提交的任务存储在工作队列里。满了，则进入下个流程。
1. 最后线程池判断整个线程池是否已满？没满，则创建一个新的工作线程来执行任务，满了，则交给饱和策略来处理这个任务。

### 三. 线程池的任务策略

#### 四种拒绝策略：
通过reject()拒绝任务有四种策略：
1. CallerRunsPolicy - 只用调用者所在线程来运行任务。。
- 使用场景：一般并发比较小，性能要求不高，不允许失败。但是，由于调用者自己运行任务，如果多次提交任务，可能导致程序阻塞，性能效率上必然的损失较大
2. AbortPolicy - 中止策略，抛出拒绝执行 RejectedExecutionException 异常信息。线程池默认的拒绝策略。我们在调用线程处必须处理好抛出的异常，否则会打断当前的执行流程，影响后续的任务执行。
3. DiscardPolicy - 直接丢弃，不执行其他操作
4. DiscardOldestPolicy - 丢弃队列里最近的一个任务，并执行当前任务。

也可以根据应用场景需要来实现RejectedExecutionHandler接口自定义策略。如记录日志或持久化不能处理的任务。

### 四. 合理的配置线程池
任务的性质：CPU密集型任务，IO密集型任务和混合型任务。
任务的优先级：高，中和低。
任务的执行时间：长，中和短。
任务的依赖性：是否依赖其他系统资源，如数据库连接。

### 五. 线程池的监控
通过线程池提供的参数进行监控。线程池里有一些属性在监控线程池的时候可以使用

1. taskCount：线程池需要执行的任务数量。
1. completedTaskCount：线程池在运行过程中已完成的任务数量。小于或等于taskCount。
1. largestPoolSize：线程池曾经创建过的最大线程数量。通过这个数据可以知道线程池是否满过。如等于线程池的最大大小，则表示线程池曾经满了。
1. getPoolSize:线程池的线程数量。如果线程池不销毁的话，池里的线程不会自动销毁，所以这个大小只增不减。
1. getActiveCount：获取活动的线程数。