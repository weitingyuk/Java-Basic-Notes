## Redis数据类型与数据结构


#### 1. 字符串（String）对象
String 有 int、raw、embst 三种编码格式：

- int：整数值，可以用 long 类型表示，使用整数值保存对象
- raw：字符串值且长度 > 32字节，使用 SDS 保存对象
- embstr：字符串值且长度 < 32字节，使用 embstr 编码的 SDS 保存对象

##### embstr 和 raw 有啥区别？
1. raw 分配内存和释放内存的次数是两次，embstr 是一次
1. embstr 编码的数据保存在一块连续的内存里面

##### 编码的转换
- int 类型的字符串，当保存的不再是整数值，将转换成 raw 类型
- embstr 类型的字符串是只读的，修改时会转换成 raw 类型。原因：Redis 没有为 embstr 提供修改程序，所以它是只读的；要修改只能先转成 raw。

#### 2. 列表（list）对象
列表的编码可以是 ziplist 或 linkedlist：

1. ziplist：所有元素长度都小于 64 字节且元素数量少于 512 个
- 以上两个条件的上限值可以通过配置文件的 list-max-ziplist-value和list-max-ziplist-entries修改
2. linkedlist：不满足上述条件时，将从 ziplist 转换成 linkedlist，就是双端链表作为底层实现。
执行 RPUSH 命令将创建一个列表对象，比如：


```
redis> RPUSH numbers 1 "three" 5
(integer) 3
```

#### 3. 哈希（hash）对象
哈希的编码可以是 ziplist 或 hashtable：

1. ziplist：哈希对象保存的所有键值对的键和值的字符串长度都小于 64 字节且键值对数量小于 512
- 以上两个条件的上限值可以通过配置文件的 hash-max-ziplist-value和hash-max-ziplist-entries修改
2. hashtable：不能满足上述条件，将从 ziplist 转成 hashtable

hashtable 保存的 hash 对象：
- 字典中每个键都是一个字符串对像，对象中保存键值对的键
- 字典中每个值都是一个字符串对像，对象中保存键值对的值

执行 HSET 命令，可以创建一个 hash 对象并保存数据：

```
redis> HSET profile name "Tom"
(integer) 1
redis> HSET profile age 25
(integer) 1
redis> HSET profile career "Programmer"
(integer) 1
```


#### 4. 集合（set）对象
set的编码可以是 intset 或 hashtable：

1. intset：集合对象保存的所有元素都是整数值且元素数量小于 512 个
- 以上两个条件的上限值可以通过配置文件的 set-max-intset-entries修改
2. hashtable：不能满足上述条件，将从 intset 转成 hashtable

使用 SADD 命令可构建一个 hashtable 编码的 set 对象并保存数据：
```
redis> SADD fruits "apple" "banana" "cherry"
(integer) 3
```
#### 5. 有序集合（Sorted Set）对象
有序集合的编码可以是 ziplist 或 skiplist：

1. ziplist：保存的元素数量小于 128 个且所有元素长度都小于 64 字节
- 以上两个条件的上限值可以通过配置文件的 zset-max-ziplist-entries和zset-max-ziplist-value修改
2. skiplist：不能同时满足上述条件，将从 ziplist 转成 skiplist

使用 ZADD 命令可以构建一个 Sorted Set 对象并保存数据：


```
redis> ZADD price 8.5 apple 5.0 banana 6.0 cherry
(integer) 3
```

1. 跳跃表 zsl 按分值从小到大保存所有集合元素；每个节点保存一个集合元素；object 属性保存元素成员、score 属性保存元素分值。目的：实现快速的范围查询操作。
1. 字典 dict 创建一个从成员到分值的 key-value；字典中每个键值对都保存一个集合元素；键保存元素成员、值保存元素分值。目的：用 O(1) 复杂度 get 元素分值。
