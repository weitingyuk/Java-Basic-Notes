## ArrayDeque 和 LinkedList 的区别
- ArrayDeque 是一个可扩容的数组，LinkedList 是链表结构；
- ArrayDeque 里不可以存 null 值，但是 LinkedList 可以；
- ArrayDeque 在操作头尾端的增删操作时更高效，但是 LinkedList 只有在当要移除中间某个元素且已经找到了这个元素后的移除才是 O(1) 的；
- ArrayDeque 在内存使用方面更高效。

## PriorityBlockingQueue原理
- PriorityBlockingQueue是带优先级的无界阻塞队列，每次出队都返回优先级最高的元素，是二叉树最小堆的实现。
- 优先级：PriorityBlockingQueue始终保证出队的元素是优先级最高的元素而不是在队列里面停留时间最长的原始，并且可以定制优先级的规则，内部通过使用一个二叉树最小堆算法来维护内部数组，这个数组是可扩容的，当当前元素个数>=最大容量时候会通过算法扩容，当队列任务里面的任务由优先级时候本队列比较实用。
- 锁：PriorityBlockingQueue类似于ArrayBlockingQueue内部使用一个独占锁来控制同时只有一个线程可以进行入队和出队，另外前者只使用了一个notEmpty条件变量而没有notFull这是因为前者是无界队列，当put时候永远不会处于await所以也不需要被唤醒,并且take方法由于是阻塞方法，所以是可被中断的，其他方法对中断标志不理会。
- 值得注意的是为了避免在扩容操作时候其他线程不能进行出队操作，实现上使用了先释放锁，然后通过cas保证同时只有一个线程可以扩容成功。