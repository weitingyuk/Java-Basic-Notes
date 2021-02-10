## HashMap 与 ConcurrentHashMap 的相同点
HashMap 与 ConcurrentHashMap 的实现都是哈希表。

## HashMap 与 ConcurrentHashMap 的不同点
1. **主要不同点**:在于hashMap不是线程安全的，ConcurrentHashMap是线程安全的，ConcurrentHashMap是在hashMap的基础上加锁，解决多线程竞争问题。
2. **HashMap基本元素**:哈希表的基本元素是一个Entry数组，每一个Entry包含一个key-value键值对。
3. **HashMap组成结构**：HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的，如果定位到的数组位置不含链表（当前entry的next指向null）,那么对于查找，添加等操作很快，仅需一次寻址即可；如果定位到的数组包含链表，对于添加操作，其时间复杂度依然为O(1)，因为最新的Entry会插入链表头部，急需要简单改变引用链即可，而对于查找操作来讲，此时就需要遍历链表，然后通过key对象的equals方法逐一比对查找。

#### ConcurrentHashMap 是如何保证线程安全的？
1. **jdk1.8之前**：使用“分段锁”，将整个hashmap分成几个小的segment，每个segment都是一个lock；与hashtable相比，这么设计的目的是对于put, remove等操作，可以减少并发冲突，对不属于同一个片段的节点可以并发操作，大大提高了性能
1. **jdk1.8+之后**：取消segments字段，直接采用transient volatile HashEntry<K,V>[] table保存数据，直接对每一行数据进行加锁，进一步减少并发冲突的概率。
