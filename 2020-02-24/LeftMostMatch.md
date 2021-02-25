## 简述什么是最左匹配原则
- MySQL可以创建联合索引(即, 多列的索引). 一个索引可以包含最多16列. 对于 某些数据类型, 你可以索引列的前缀作为索引。
- 如果表拥有一个联合索引, 任何一个索引的最左前缀都会被优化器用于查找列. 比如,
如果你创建了一个三列的联合索引包含(col1, col2, col3), 你的索引会生效于(col1),
(col1, col2), 以及(col1, col2, col3)
- 如果查询的列不是索引的最左前缀, 那MySQL不会将索引用于执行查询.


例如：
你有下列查询语句:

```
SELECT * FROM tbl_name WHERE col1=val1;
SELECT * FROM tbl_name WHERE col1=val1 AND col2=val2;

SELECT * FROM tbl_name WHERE col2=val2;
SELECT * FROM tbl_name WHERE col2=val2 AND col3=val3;
```

如果索引存在于(col1, col2, col3), 那只有头两个查询语句用到了索引. 第三个和 第四个查询包含索引的列, 但是不会用索引去执行查询. 因为(col2)和(col2, col3) 不是(col1, col2, col3)的最左前缀


### 实战案例

1. 如果sql为如下，如何建立索引?

```
SELECT * FROM table WHERE a = 1 and b = 2 and c = 3;
```

(a,b,c)或者(c,b,a)或者(b,a,c)都可以，重点要的是将区分度高的字段放在前面，区分度低的字段放后面。像性别、状态这种字段区分度就很低，我们一般放后面。

例如假设区分度由大到小为b,a,c。那么我们就对(b,a,c)建立索引。在执行sql的时候，优化器会 帮我们调整where后a,b,c的顺序，让我们用上索引。

2. 如果sql为如下，如何建立索引?

```
SELECT * FROM table WHERE a > 1 and b = 2;
```
对(b,a)建立索引。如果你建立的是(a,b)索引，那么只有a字段能用得上索引，毕竟最左匹配原则遇到范围查询就停止匹配。
如果对(b,a)建立索引那么两个字段都能用上，优化器会帮我们调整where后a,b的顺序，让我们用上索引。

3. 如果sql为如下，如何建立索引?

```
SELECT * FROM `table` WHERE a > 1 and b = 2 and c > 3;

```
此题回答也是不一定，(b,a)或者(b,c)都可以，要结合具体情况具体分析。
同理下面的情况也一样：

```
SELECT * FROM `table` WHERE a = 1 and b = 2 and c > 3;
```

4. 如果sql为如下，如何建立索引?

```
SELECT * FROM `table` WHERE a = 1 ORDER BY b;
```
对(a,b)建索引，当a = 1的时候，b相对有序，可以避免再次排序！
那么如果下面情况如何建立索引？
```
SELECT * FROM `table` WHERE a > 1 ORDER BY b;
```
对(a)建立索引，因为a的值是一个范围，这个范围内b值是无序的，没有必要对(a,b)建立索引。
如下的情况呢？
```
SELECT * FROM `table` WHERE a = 1 AND b = 2 AND c > 3 ORDER BY c;
```
对(a,b,c)建索引

注： 实战写sql的时候，order by字段必须出现在where中，除了第一次范围，order by字段必须在where中全等的情况下才能索引排序

5. 如果sql为如下，如何建立索引?
```
SELECT * FROM `table` WHERE a IN (1,2,3) and b > 1;
```
还是对(a，b)建立索引，因为IN在这里可以视为等值引用，不会中止索引匹配，所以还是(a,b)!


