
## synchronized 关键字底层是如何实现的？它与 Lock 相比优缺点分别是什么？

### Synchronized原理
Synchronized在**Jdk 1.6后**通过对象Mark word实现。会在同步块的前后分别生成 monitorenter 和 monitorexit 字节码指令，这两个字节码指令都需要一个引用类型的参数来指明要锁定和解锁的对象。Synchronized实现了偏向锁，轻量级锁，重量级锁。 偏向锁是对轻量级锁的优化，轻量级锁是对重量级锁的优化。
### Synchronized执行流程：
1. 判断Mark word里是否是当前线程，且偏向锁状态为true，如果是，则获得偏向锁。如果不是，那么执行CAS将Mark work修改为当前线程，如果成功，则获得偏向锁，并设置偏向锁为true。否则发生竞争，撤销偏向锁，升级为轻量级锁。
1. 当前线程执行CAS，将mark word替换为锁记录指针。如果成功获得轻量级锁。如果失败，当前线程尝试自旋获取锁，如果获取到，处于轻量级锁。如果一定次数后没获取到锁，锁升级为重量级锁。
1. 重量级锁是独占锁。通过monitor对象实现。monitor enter 进入数+1，当前线程获得锁，如果其他线程已经获得锁，那么该线程阻塞，直到锁计数为0。monitor exit表示锁计数-1，直到进入数为0时，当前线程释放锁。

### 与 Lock 相比优缺点
1. lock是一个接口，而synchronized是java的关键字，synchronized是内置的语言实现；synchronized优化以后性能和lock差不多。
1. synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而lock在发生异常时，如果没有主动通过unlock（）去释放锁，则很可能造成死锁现象，因此使用lock（）时需要在finally块中释放锁；
1. lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能响应中断
1. 通过lock可以知道有没有成功获取锁， lock的功能非常丰富，而synchronized却无法办到。
