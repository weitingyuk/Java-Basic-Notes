## ThreadLocal 实现原理是什么


### 一. 作用：
- ThreadLocal 用于在多线程环境中安全的保存线程本地变量，同一线程在某个地方保存数据，在随后的任意地方都可以获取到。
- 即使是同一个 threadLocal 变量，由于每个线程自己都有一个独立的 ThreadLocalMap，所以不同线程中保存的数据也是互不影响的。

### 二. 保存步骤
1. 每个线程（Thread 类对象）中都会有一个 ThreadLocalMap 类型的变量 threadLocals 来保存 ThreadLocal 中的数据。
2. 在保存数据的 threadLocal.set() 方法中，先通过 Thread.currentThread() 方法获取到当前线程
3. 通过 getMap(Thread t) 方法获取到当前线程的 ThreadLocalMap 对象
4. 以当前 threadLocal 对象为 key、需要保存的数据对象为 value，构造一个键值对 Entry 对象保存在 ThreadLocalMap 中
5. 调用 threadLocal.get() 方法取数据也是先获取到这个线程对象，然后以对应 threadLocal 对象为 key 从这个 ThreadLocalMap 中取出数据的。

注意，一个 ThreadLocal 只能存储一个 Object 对象，如果需要存储多个 Object 对象，那么就需要多个 ThreadLocal 来转化成多个 Entry 对象（键值对）进行存储。

### 三.原理 + 内存泄露问题
#### 数据结构
1. 当使用 ThreadLocal 保存一个 value 时，会构造一个键值对 Entry 对象插入到 ThreadLocalMap 的数组中
2. Entry 继承自弱引用 **WeakReference 类**。构建 Entry 对象时，**key 是一个指向 ThreadLocal 对象的弱引用**，value 则为强引用。
#### 设计原理
对于一个 ThreadLocal 对象，通常会有两个引用指向它：
- 一个是线程中声明的 threadLocal 变量，这是个强引用；
- 一个是线程底层 ThreadLocalMap 中键值对的 key，这是弱引用。
- 不再需要使用某 ThreadLocal 对象时，会采用将变量设置为 null（threadLocal = null）的方式释放掉线程中 threadLocal 变量与对象之间的引用关系，方便 GC 对 ThreadLocal 对象的回收。
- 但此时线程的 ThreadLocalMap 中还有 key 引用着这个 ThreadLocal 对象：如果这个引用是强引用，那么这个 **ThreadLocal 对象就可能永远不会被回收了，造成内存泄露**；但现在这里设计成弱引用，那么当垃圾收集器发现这个 ThreadLocal 对象只有**弱引用相关联时，就会回收它的内存**。
- 当前 Entry 对象的 key 设计成弱引用：Entry 对象中的 value 仍存在着内存泄露的隐患，**因为在垃圾回收 key 被清理掉的时候，强引用 value 中的对象并不会被清理掉**。
- 最佳实践：应当在我们使用完 ThreadLocak 对象之后，主动调用 remove() 方法进行清理。

#### key是弱引用，value怎么清理
- 虽然key是弱引用，当不存在外部强引用的时候，就会自动被回收
- 但是Entry中的value依然是强引用。这个value的引用链条如下：

```
Thread -> ThreadLocal ->Entry -> value
```

- 所以只有当Thread被回收时，这个value才有被回收的机会，否则，只要线程不退出，value总是会存在一个强引用。
- 但是，要求每个Thread都会退出，是一个极其苛刻的要求，对于线程池来说，大部分线程会一直存在在系统的整个生命周期内，那样的话，就会造成value对象出现泄漏的可能。
- 处理的方法是，在ThreadLocalMap进行set(),get(),remove()的时候，都会进行清理：


```
以getEntry()为例：

private Entry getEntry(ThreadLocal<?> key) {
    int i = key.threadLocalHashCode & (table.length - 1);
    Entry e = table[i];
    if (e != null && e.get() == key)
        //如果找到key，直接返回
        return e;
    else
        //如果找不到，就会尝试清理，如果你总是访问存在的key，那么这个清理永远不会进来
        return getEntryAfterMiss(key, i, e);
}
下面是getEntryAfterMiss()的实现：

private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
    Entry[] tab = table;
    int len = tab.length;

    while (e != null) {
        // 整个e是entry ，也就是一个弱引用
        ThreadLocal<?> k = e.get();
        //如果找到了，就返回
        if (k == key)
            return e;
        if (k == null)
            //如果key为null，说明弱引用已经被回收了
            //那么就要在这里回收里面的value了
            expungeStaleEntry(i);
        else
            //如果key不是要找的那个，那说明有hash冲突，这里是处理冲突，找下一个entry
            i = nextIndex(i, len);
        e = tab[i];
    }
    return null;
}
```

真正用来回收value的是expungeStaleEntry()方法，在remove()和set()方法中，都会直接或者间接调用到这个方法进行value的清理。

从这里可以看到，ThreadLocal为了避免内存泄露，也算是花了一番大心思。不仅使用了弱引用维护key，还会在每个操作上检查key是否被回收，进而再回收value。


###### 总结
- ThreadLocal并不能100%保证不发生内存泄漏。
- 如果一直没有机会调用get(), set()和remove()，那么这个内存泄漏依然会发生。
- 因此，一个良好的习惯依然是：当你不需要这个ThreadLocal变量时，主动调用remove()，这样对整个系统是有好处的。

### 四. 四种引用类型
#### 1. 强引用：
- Object obj = new Object()，对于这种引用，除非显示将obj=null，否则虚拟机不会将其回收
- 应用场景：对象的一般状态
#### 2. 软引用：
- 则内存空间充足时，垃圾回收器就不会回收它；如果内存空间不足了，就会回收这些对象的内存。。
  
```
SoftReference<String> softReference = new SoftReference<String>(str);
```
注意：就算扫描到软引用对象也不一定会回收它，只有内存不够的时候才会回收。
- 应用场景：对象缓存
#### 3. 弱引用：
- 引用强度比软引用更弱。在垃圾回收器线程扫描过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。
- 应用场景：对象缓存
#### 4. 虚引用：
- 虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收。
- 应用场景：
虚引用主要用来跟踪对象被垃圾回收器回收的活动。
