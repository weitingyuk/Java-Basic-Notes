## 单例模式

- 单例模式某个类的实例对象只有一个。
- getInstance()的返回值是一个对象的引用，并不是一个新的实例，所以不要错误的理解成多个对象。

### 1. 懒汉写法（线程不安全）
- 缺点：当并发的时候，A线程判断singleton=null,就创造实例，但B线程在A未创造之前也判断singleton=null,就创造两个实例
```java
public class Singleton {
    private static Singleton singleton;
    private Singleton() {}
    public static Singleton getInstance() {
         if (singleton == null) {
             singleton = new Singleton();
         }
         return singleton;
    }
}
```


### 2. 懒汉式写法（线程安全）
- 缺点：方法上加synchronized，当一个线程访问方法的时候，其他的线程都要挂起等待，造成无谓等待
```java
public class Singleton {  
   private static Singleton instance;  
   private Singleton (){}  
   public static synchronized Singleton getInstance() {  
   if (instance == null) {  
       instance = new Singleton();  
   }  
   return instance;  
   }  
}
```


### 3. 双重校验锁
- 优点：通过两次判断解决了多线程等待的问题,
- 特点：需要加上volatile避免指令重排序
```java
public class Singleton {  
   private static volatile Singleton singleton;  
   private Singleton (){}  
   public static Singleton getSingleton() {  
   if (singleton == null) {  
       synchronized (Singleton.class) {  
       if (singleton == null) {  
           singleton = new Singleton();  
       }  
       }  
   }  
   return singleton;  
   }  
}
```

### 4. 静态内部类（标准单例）
- 原因：因为一个类的静态属性只会在第一次类加载的时候初始化，所以能保证单例
```java
public class Singleton {  
   private static class SingletonHolder {  
   private static final Singleton INSTANCE = new Singleton();  
   }  
   private Singleton (){}  
   public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;  
   }  
}
```

### 5. 枚举 （简单）
- 这是EffectiveJava中提倡的方式，天然的单例，能避免多线程同步问题，而且还能防止反序列化重新创建新的对象。
```java
public enum Singleton {  
   INSTANCE;  
   public void whateverMethod() {  
   }  
}
```


### 6. 饿汉式写法 （简单）
- 缺点：一旦访问了这个类的其他任何静态域，就会造成实例初始化。如果没用这个instance，会造成内存浪费
```java
public class Singleton {  
   private static volatile Singleton instance = new Singleton();  
   private Singleton (){}  
   public static Singleton getInstance() {  
       return instance;  
   }  
}
```

### 方法3 - 双重校验锁为什么需要加volatile
- 线程A获取到锁在创建实例，线程B执行到第一次检测，读取到instance不到null时，instance的引用对象可能没有完成初始化
- JAVA创建对象正常的顺序

```
1.分配内存    
2.初始化对象
3.将对象指向分配的内存
```

指令重排序的时候可能会将步骤2和步骤3倒过来运行，即：
```
1.分配内存    
2.将对象指向分配的内存
3.初始化对象
```
后面的线程B去getInstance的时候，认为instance已经实例化，有且返回一个没有实例化的引用