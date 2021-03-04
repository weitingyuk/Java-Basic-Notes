## 数据库查询中左外连接和内连接的区别是什么？

### 1. 内连接
- 1.1 **关键词**：inner join on
- 1.2 **含义**：组合两个表中的记录，返回关联字段相符的记录，也就是返回两个表的交集（阴影）部分。
- 1.3 **语句**：  

```
select * from a_table a inner join b_table b on a.a_id = b.b_id;
```

### 2. 左连接（左外连接）
- 2.1 **关键词**：left join on / left outer join on
- 2.2 **含义**：返回左表全部，右表符合条件的数据。 右表记录不足的地方均为NULL。
- 2.3 **语句**：  

```
select * from a_table a left join b_table bon a.a_id = b.b_id;
```

### 3. 右连接（右外连接）
- 2.1 **关键词**：right join on / right outer join on
- 2.2 **含义**：参考左连接
- 2.3 **语句**：  

```
select * from a_table a right outer join b_table b on a.a_id = b.b_id;
```
