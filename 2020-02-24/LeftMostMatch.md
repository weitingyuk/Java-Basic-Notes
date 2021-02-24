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