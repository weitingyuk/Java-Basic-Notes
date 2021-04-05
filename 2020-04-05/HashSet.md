## 简述 HashSet 实现原理

#### HashSet简述
##### HashSet和HashMap的关系
- HashSet中不允许有重复元素，这是因为HashSet是基于HashMap实现的
- HashSet中的元素都存放在HashMap的key上面，而value中的值都是统一的一个private static final Object PRESENT = new Object()。
- HashSet跟HashMap一样，都是一个存放链表的数组。

#### HashSet的组成结构
- HashSet是基于HashMap实现的，HashSet底层使用HashMap来保存所有元素。
- HashSet中的元素，只是存放在了底层HashMap的key上， 而value使用一个static final的Object对象标识。
- 因此HashSet 的实现比较简单，相关HashSet的操作，基本上都是直接调用底层HashMap的相关方法来完成。

#### 总结
- HashSet底层由HashMap实现
- HashSet的值存放于HashMap的key上
- HashMap的value统一为PRESENT

