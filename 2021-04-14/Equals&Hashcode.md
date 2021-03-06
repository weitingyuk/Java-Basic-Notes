## hashCode() 和 equals() 的关系
 这里以“类的用途”来将“hashCode() 和 equals()的关系”分2种情况来说明。

#### 1. 第一种 不会创建“类对应的散列表”

-  这里所说的“不会创建类对应的散列表”是说：我们不会在HashSet, Hashtable, HashMap等等这些本质是散列表的数据结构中，用到该类。例如，不会创建该类的HashSet集合。
- 在这种情况下，该类的“hashCode() 和 equals() ”没有半毛钱关系的！
- 这种情况下，equals() 用来比较该类的两个对象是否相等。而hashCode() 则根本没有任何作用，所以，不用理会hashCode()。

#### 2. 第二种 会创建“类对应的散列表”

- 这里所说的“会创建类对应的散列表”是说：我们会在HashSet, Hashtable, HashMap等等这些本质是散列表的数据结构中，用到该类。例如，会创建该类的HashSet集合。
- 在这种情况下，该类的“hashCode() 和 equals() ”是有关系的：
- 1)、如果两个对象相等，那么它们的hashCode()值一定相同。
-       这里的相等是指，通过equals()比较两个对象时返回true。
- 2)、如果两个对象hashCode()相等，它们并不一定相等。
-        因为在散列表中，hashCode()相等，即两个键值对的哈希值相等。然而哈希值相等，并不一定能得出键值对相等。补充说一句：“两个不同的键值对，哈希值相等”，这就是哈希冲突。
- 此外，在这种情况下。若要判断两个对象是否相等，除了要覆盖equals()之外，也要覆盖hashCode()函数。否则，equals()无效。
- 例如，创建Person类的HashSet集合，必须同时覆盖Person类的equals() 和 hashCode()方法。