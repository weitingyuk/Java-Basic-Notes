# Java中sleep()和wait()方法的区别

## Java sleep() vs wait() – 流程
1. sleep 是让线程进入阻塞状态，一定时间之后回到非阻塞状态，从而可以重新获得 CPU。
1. wait 调用的时候需要先获得该 Object 的锁，调用 wait 后，会把当前的锁释放掉同时阻塞住；当别的线程调用该 Object 的 notify/notifyAll 之后，有可能得到 CPU，同时重新获得锁。由于有如上描述锁的设计，只要在 notify 的时候首先获得锁，就可以保证 notify 的时候或者处于 wait 线程获得锁之前，或者正在 wait，从而保证不会丢掉这次 notify 信息。

## Java sleep() vs wait() – 总结
1. sleep是线程中的方法，但是wait是Object中的方法。
2. sleep方法不会释放lock，但是wait会释放，而且会加入到等待队列中。
3. sleep方法不依赖于同步器synchronized，但是wait需要依赖synchronized关键字。
4. sleep不需要被唤醒（休眠之后退出阻塞），但是wait需要（不指定时间需要被别人中断）。
