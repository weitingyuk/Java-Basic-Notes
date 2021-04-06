## 手写生产者消费者模型

现在用五种方式来实现生产者消费者模型

### 1. wait()和notify()方法
- 使用synchronized来进行同步；
- 缓冲区满和为空时都调用wait()方法等待；
- 当生产者生产了一个产品或者消费者消费了一个产品之后会唤醒所有线程。
``` java
package BasicKnowleage;

/** Method 1: synchronized + wait & notifyAll **/
public class ProducerConsumer01 {
  private static Integer count = 0;
  private static String LOCK = "lock";
  private static final Integer FULL = 10;
  private static final Integer EMPTY = 0;

  class Producer implements Runnable {
    @Override
    public void run() {
      for(int i = 0; i < 10; i++) {
        try {
          Thread.sleep(3000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        synchronized (LOCK) {
          while (count == FULL) {
            try {
              LOCK.wait();
            } catch (InterruptedException e) {
              e.printStackTrace();
            }
          }
          count ++;
          System.out.println(Thread.currentThread().getName() + " producer number: " + count);
          LOCK.notifyAll();
        }
      }
    }
  }

  class Consumer implements Runnable {

    @Override
    public void run() {
      for (int i = 0; i < 10; i++) {
        try {
          Thread.sleep(3000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        synchronized (LOCK){
          while (count == EMPTY) {
            try {
              LOCK.wait();
            } catch (InterruptedException e) {
              e.printStackTrace();
            }
          }
          count --;
          System.out.println(Thread.currentThread().getName() + " consumer number: " + count);
          LOCK.notifyAll();
        }
      }
    }
  }

  public static void main(String[] args) {
    ProducerConsumer01 producerConsumer01 = new ProducerConsumer01();
    new Thread(producerConsumer01.new Producer()).start();
    new Thread(producerConsumer01.new Consumer()).start();
    new Thread(producerConsumer01.new Producer()).start();
    new Thread(producerConsumer01.new Consumer()).start();
    new Thread(producerConsumer01.new Producer()).start();
    new Thread(producerConsumer01.new Consumer()).start();
    new Thread(producerConsumer01.new Producer()).start();
    new Thread(producerConsumer01.new Consumer()).start();
  }

}
```

### 2. 可重入锁ReentrantLock
- 创建一个锁对象ReentrantLock，注意；需要释放锁
- 为这个锁创建两个条件Condition变量，一个为缓冲区notFull，一个为缓冲区notEmpty


``` java
package BasicKnowleage;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

/** Method 2: ReentrantLock + wait & notifyAll **/
public class ProducerConsumer02 {

  private static Integer count = 0;
  private static ReentrantLock lock = new ReentrantLock();
  private static final Integer FULL = 10;
  private static final Integer EMPTY = 0;
  private static Condition notFull = lock.newCondition();
  private static Condition notEmpty = lock.newCondition();

  class Producer implements Runnable {
    @Override
    public void run() {
      for (int i = 0; i < 10; i++) {
        try {
          Thread.sleep(3000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        lock.lock();
        try {
          while (count == FULL) {
            try {
              notFull.await();
            } catch (InterruptedException e) {
              e.printStackTrace();
            }
          }
          count ++;
          System.out.println(Thread.currentThread().getName() + " producer number: " + count);
          notEmpty.signal();
        } finally {
          lock.unlock();
        }
      }
    }
  }

  class Consumer implements Runnable {
    @Override
    public void run() {
      for (int i =0; i< 10; i++) {
        try {
          Thread.sleep(3000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        lock.lock();
        try {
          while (count == EMPTY) {
            try {
              notEmpty.await();
            } catch (InterruptedException e) {
              e.printStackTrace();
            }
          }
          count --;
          System.out.println(Thread.currentThread().getName() + " consumer number: " + count);
          notFull.signal();
        } finally {
          lock.unlock();
        }
      }
    }
  }

  public static void main(String[] args) {
    ProducerConsumer02 producerConsumer02 = new ProducerConsumer02();
    new Thread(producerConsumer02.new Producer()).start();
    new Thread(producerConsumer02.new Consumer()).start();
    new Thread(producerConsumer02.new Producer()).start();
    new Thread(producerConsumer02.new Consumer()).start();
    new Thread(producerConsumer02.new Producer()).start();
    new Thread(producerConsumer02.new Consumer()).start();
    new Thread(producerConsumer02.new Producer()).start();
    new Thread(producerConsumer02.new Consumer()).start();
  }
}
```

### 3. 阻塞队列BlockingQueue
- 定义一个阻塞队列，其中lockingQueue的put和take时阻塞的方法


``` java
package BasicKnowleage;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

/** Method 3:  BlockingQueue **/
public class ProducerConsumer03 {
  private static Integer count = 0;
  final BlockingQueue blockingDeque = new ArrayBlockingQueue<Integer>(10);

  class Producer implements Runnable {
    @Override
    public void run() {
      for (int i = 0; i < 10; i++) {
        try {
          Thread.sleep(3000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        try {
          blockingDeque.put(1);
          count ++;
          System.out.println(Thread.currentThread().getName() + " producer number: " + count);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      }
    }
  }

  class Consumer implements Runnable {
    @Override
    public void run() {
      for (int i =0; i< 10; i++) {
        try {
          Thread.sleep(3000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        try {
          blockingDeque.take();
          count --;
          System.out.println(Thread.currentThread().getName() + " consumer number: " + count);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      }
    }
  }

  public static void main(String[] args) {
    ProducerConsumer03 producerConsumer03 = new ProducerConsumer03();
    new Thread(producerConsumer03.new Producer()).start();
    new Thread(producerConsumer03.new Consumer()).start();
    new Thread(producerConsumer03.new Producer()).start();
    new Thread(producerConsumer03.new Consumer()).start();
    new Thread(producerConsumer03.new Producer()).start();
    new Thread(producerConsumer03.new Consumer()).start();
    new Thread(producerConsumer03.new Producer()).start();
    new Thread(producerConsumer03.new Consumer()).start();
  }

}
```

### 4. 信号量Semaphore
- 添加两个信号量notFull和notEmpty作为许可集；
- 可以使用acquire()方法获得一个许可，当许可不足时会被阻塞，release()添加一个许可；
- 加入了另外一个mutex信号量，维护生产者消费者之间的同步关系，保证生产者和消费者之间的交替进行


``` java
package BasicKnowleage;

import java.util.concurrent.Semaphore;

/** Method 4:  Semaphore **/
public class ProducerConsumer04 {
  private static Integer count = 0;
  final Semaphore notFull = new Semaphore(10);
  final Semaphore notEmpty = new Semaphore(0);
  final Semaphore mutex = new Semaphore(1);

  class Producer implements Runnable {
    @Override
    public void run() {
      for (int i = 0; i < 10; i++) {
        try {
          Thread.sleep(3000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        try {
          notFull.acquire();
          mutex.acquire();
          count ++;
          System.out.println(Thread.currentThread().getName() + " producer number: " + count);
        } catch (InterruptedException e) {
          e.printStackTrace();
        } finally {
          notFull.release();
          mutex.release();
        }
      }
    }
  }

  class Consumer implements Runnable {
    @Override
    public void run() {
      for (int i =0; i< 10; i++) {
        try {
          Thread.sleep(3000);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        try {
          notEmpty.acquire();
          mutex.acquire();
          count --;
          System.out.println(Thread.currentThread().getName() + " consumer number: " + count);
        } catch (InterruptedException e) {
          e.printStackTrace();
        } finally {
          notEmpty.release();
          mutex.release();
        }
      }
    }
  }

  public static void main(String[] args) {
    ProducerConsumer04 producerConsumer04 = new ProducerConsumer04();
    new Thread(producerConsumer04.new Producer()).start();
    new Thread(producerConsumer04.new Consumer()).start();
    new Thread(producerConsumer04.new Producer()).start();
    new Thread(producerConsumer04.new Consumer()).start();
    new Thread(producerConsumer04.new Producer()).start();
    new Thread(producerConsumer04.new Consumer()).start();
    new Thread(producerConsumer04.new Producer()).start();
    new Thread(producerConsumer04.new Consumer()).start();
  }
}

```
 
### 5. 管道输入输出流PipedInputStream和PipedOutputStream

- 先创建一个管道输入流和管道输出流，然后将输入流和输出流进行连接
- 生产者线程往管道输出流中写入数据，消费者在管道输入流中读取数据，这样就可以实现了不同线程间的相互通讯
- **注意**：这种方式在生产者和生产者、消费者和消费者之间不能保证同步，也就是说在一个生产者和一个消费者的情况下是可以生产者和消费者之间交替运行的，多个生成者和多个消费者者之间则不行


``` java
package BasicKnowleage;

import java.io.IOException;
import java.io.PipedInputStream;
import java.io.PipedOutputStream;


public class ProducerConsumer05 {
  final PipedInputStream pipedInputStream = new PipedInputStream();
  final PipedOutputStream pipedOutputStream = new PipedOutputStream();
  {
    try {
      pipedInputStream.connect(pipedOutputStream);
    } catch (IOException e) {
      e.printStackTrace();
    }
  }

  class Producer implements Runnable {
    @Override
    public void run() {
        try {
          while(true) {
            Thread.sleep(1000);
            int num = (int) (Math.random() * 255);
            System.out.println(Thread.currentThread().getName() + " producer number: " + num);
            pipedOutputStream.write(num);
            pipedOutputStream.flush();
          }
        } catch (Exception e) {
          e.printStackTrace();
        } finally {
          try {
            pipedOutputStream.close();
            pipedInputStream.close();
          } catch (IOException e) {
            e.printStackTrace();
          }
        }
      }
  }
  class Consumer implements Runnable {
    @Override
    public void run() {
        try {
          Thread.sleep(1000);
          int num = pipedInputStream.read();
          System.out.println(Thread.currentThread().getName() + " consumer number: " + num);
        } catch (Exception e) {
          e.printStackTrace();
        } finally {
          try {
            pipedOutputStream.close();
            pipedInputStream.close();
          } catch (IOException e) {
            e.printStackTrace();
          }
        }
      }

  }

  public static void main(String[] args) {
    ProducerConsumer05 producerConsumer05 = new ProducerConsumer05();
    new Thread(producerConsumer05.new Producer()).start();
    new Thread(producerConsumer05.new Consumer()).start();
  }
}

```
 
