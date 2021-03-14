##  HashMap & Hashtable & ConcurrentHashMap
### HashMap
- HashMap是非线程安全的哈希表，常用于单线程程序中。
### Hashtable
- Hashtable是线程安全的哈希表，它是通过synchronized来保证线程安全的；
- Hashtable在线程竞争激烈时，效率比较低(此时建议使用ConcurrentHashMap)！因为当一个线程访问Hashtable的同步方法时，其它线程就访问Hashtable的同步方法时，可能会进入阻塞状态。
### ConcurrentHashMap
- ConcurrentHashMap是线程安全的哈希表，它是通过“锁分段”来保证线程安全的。
- ConcurrentHashMap将哈希表分成许多片段(Segment)，每一个片段除了保存哈希表之外，**本质上也是一个“可重入的互斥锁”(ReentrantLock)**。多线程对同一个片段的访问，是互斥的；但是，对于不同片段的访问，却是可以同步进行的。
  
##  ConcurrentHashMap源码（1.8之前）
put()的作用是将key-value键值对插入到“当前Segment对应的HashEntry中”，在**插入前它会获取Segment对应的互斥锁**，插入后会释放锁。具体的插入过程如下：
- (01) 首先根据**“hash值”获取“当前Segment的HashEntry**数组对象”中的“HashEntry节点”，每个HashEntry节点都是一个单向链表。
- (02) 接着，遍历HashEntry链表。
    - **已存在，根据参数判断是否插入**：若在遍历HashEntry链表时，找到与“要key-value键值对”对应的节点，即“要插入的key-value键值对”的key已经存在于HashEntry链表中。则根据onlyIfAbsent进行判断，若onlyIfAbsent为true，即“当要插入的key不存在时才插入”，则不进行插入，直接返回；否则，用新的value值覆盖原始的value值，然后再返回。
    - **不存在，获取锁，添加到entry**：若在遍历HashEntry链表时，没有找到与“要key-value键值对”对应的节点。当node!=null时，即在scanAndLockForPut()获取锁时，已经新建了key-value对应的HashEntry节点，则”将HashEntry添加到Segment中“；否则，新建key-value对应的HashEntry节点，然后再“将HashEntry添加到Segment中”。
    - **长度超过阈值，需要rehash**”将HashEntry添加到Segment中“前，会判断是否需要rehash。如果在添加key-value键值之后，容量会超过阈值，并且HashEntry数组的长度没有超过限制，则进行rehash；否则，直接通过setEntryAt()将key-value键值对添加到Segment中。
-（03）rehash()
    - rehash()的作用是将”**Segment的容量**“变为”原始的Segment容量的2倍“。
    - 在将原始的数据拷贝到“新的Segment”中后，会将新增加的key-value键值对添加到“新的Segment”中。
    - 注意：rehash的是**Segment**