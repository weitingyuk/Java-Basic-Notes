## 简述 ArrayList 与 LinkedList 的底层实现以及常见操作的时间复杂度

ArrayList和LinkedList都是Java中列表接口的实现。两个类都是非同步的。但也有一些不同之处。
### ArrayList和LinkedList两者的区别主要有4点：
1. 内部实现：
    - ArrayList 是通过一个动态数组去保存数据的。
    - LinkedList基于双向链表的数据结构实现。
2. 实现：
    - ArrayList只能作为列表。
    - LinkedList既可以作为列表也可以充当队列。
1. 操作数据：对于新增add()和删除操作remove()，
    - ArrayList速度慢，因为数组操作速度慢。
    - LinkedList基于节点的速度更快，因为不需要太多的位移动。
3. 访问数据，比如get()，set()：
    - ArrayList在存储和访问数据方面更快，因为数组的特点：数据是连续的；随机访问速度快，是根据下标来的。
    - LinkedList处理数据的速度更快。