## SynchronousQueue和LinkedTransferQueue
### 一、SynchronousQueue

SynchronousQueue是一个不存储元素的队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。

它支持公平访问队列。默认情况下线程采用非公平性策略访问队列。

SynchronousQueue负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合传递性场景。

特点：吞吐量高

### 二、LinkedTransferQueue
LinkedTransferQueue是一个由链表结构组成的无界阻塞TransferQueue队列。
相对于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。

1. LinkedTransferQueue采用一种预占模式。
1. 如果队列不为空，则直接取走数据
1. 若队列为空，那就生成一个节点（节点元素为null）入队，然后消费者线程被等待在这个节点上
1. 后面生产者线程入队时发现有一个元素为null的节点，生产者线程就不入队了，直接就将元素填充到该节点
1. 并唤醒该节点等待的线程，被唤醒的消费者线程取走元素，从调用的方法返回。
1. 
我们称这种节点操作为“匹配”方式。