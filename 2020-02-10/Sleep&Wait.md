# Java中sleep()和wait()方法的区别

## Java sleep() vs wait() – 转换流程
### sleep()
1. 调用 sleep 会让当前线程从 Running 进入 Timed Waiting 状态(阻塞)
2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException 
3. 睡眠结束后的线程未必会立刻得到执行
### wait()
1. wait() 让进入 object 监视器的线程到 waitSet 等待变为Waiting/Timed Waiting状态(阻塞)
1. Waiting的线程会在线程调用notify和notifyAll时唤醒，但唤醒后并不一定会获得锁


## Java sleep() vs wait() – 区别总结
1. sleep 是 Thread 方法，而 wait 是 Object 的方法 
1. sleep 不需要强制和 synchronized 配合使用，但 wait 需要 和 synchronized 一起用 
2. sleep 在睡眠的同时，不会释放对象锁的，但 wait 在等待的时候会释放对象锁 



