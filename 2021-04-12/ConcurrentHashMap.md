## ConcurrentHashMap原理 - JDK1.7
##### ConcurrentHashMap的Put操作：
- Put方法首先定位到Segment，然后在Segment里进行插入操作。
- 插入操作需要经历两个步骤，第一步判断是否需要对Segment里的HashEntry数组进行扩容，
- 第二步定位添加元素的位置然后放在HashEntry数组里。
- 为了高效ConcurrentHashMap不会对整个容器进行扩容，而只对某个segment进行扩容。


#### 如何统计count：
- 先尝试2次通过不锁住Segment的方式来统计各个Segment大小，
- 如果统计的过程中，容器的count发生了变化，则再采用加锁的方式来统计所有Segment的大小。
- 那么ConcurrentHashMap是如何判断在统计的时候容器是否发生了变化呢？使用modCount变量，在put , remove和clean方法里操作元素前都会将变量modCount进行加1，那么在统计size前后比较modCount是否发生变化，从而得知容器的大小是否发生变化。