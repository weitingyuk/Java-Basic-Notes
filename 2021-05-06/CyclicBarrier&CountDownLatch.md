## CyclicBarrier和CountDownLatch的区别

### 1. 作用不同
CyclicBarrier要等固定数量的线程都到达了栅栏位置才能继续执行,而CountDownLatch只需等待数字到0,也就是说,CountDownLatch用于事件,但是CyclicBarrier是用于线程的。

### 2. 可重用性不同
CountDownLatch在倒数到0并触发门闩打开后,就不能再次使用了,除非新建新的实例;而CyclicBarrier可以重复使用