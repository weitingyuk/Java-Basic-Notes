## SQL优化的方案有哪些，如何定位问题并解决问题？

### 1. 定位问题
- 1.1 使用EXPLAIN
-  通过EXPLAIN查看SQL执行计划。
    - type列，连接类型。一个好的sql语句至少要达到range级别。杜绝出现all级别
    - key列，使用到的索引名。如果没有选择索引，值是NULL。可以采取强制索引方式
    - key_len列，索引长度
    - rows列，扫描行数。该值是个预估值
    - extra列，详细说明。注意常见的不太友好的值有：Using filesort, Using temporary

    
### 2. SQL优化的方案有哪些
- 2.1 使用索引（Index All Columns Used in 'where', 'order by', and 'group by' Clauses），同时注意如下
    -   对于连接查询，只需要在关联顺序中的第二张表的相应列上创建索引即可。
    -   覆盖索引（包含所有需要查询的字段的值）对于字段少的查询非常有用
    -   应避免冗余和重复索引
    -   不支持函数索引，但支持前缀索引（也就是对前N个字符创建索引）
    -   列的基数（比如数值范围）越大，索引效果越好
    -   尽量使用短索引
    -   利用最左前缀
- 2.2 用Union代替like（Optimize Like Statements With Union Clause） 
    - UNION ALL 要比 UNION 快很多（如果知道不重复，就应该使用Union All）
- 2.3 避免在开头使用通配符(Avoid Like Expressions With Leading Wildcards)
    -  解决方法：
        - 1）对于'%xx'，使用reverse函数：reverse(name) like reverse(‘%245′) 
        - 2）对于'%xx%'，使用instr函数： instr(t.column,’xx’)> 0 
- 2.4 使用Mysql的全文搜索（FTS：full-text search）
    - 关于FTS，有几个需要注意的地方：
    -  1）只在MyISAM或者InnoDB适用
    -  2）只用于字段char，varchar，text
    -  3）对于大量数据迁移，可以先迁移数据再创建FTS，这样比先建FTS再迁移快得多
- 2.5 优化表结构
    -   1）数据类型：shorter is always better. 
    -   2）查询语句：shorter is alwasys better：查询中尽量避免使用SELECT *以及加上LIMIT限制
    -   3）避免使用Null：如果使用，则在sum这列值的时候就需要筛选，否则就会出现错误
    -   4）避免定义的列数太多，可以用多个小表代替一个大表，但也要注意不要过度设计
    -   5）join时尽可能少关联表
- 2.6 使用Mysql的缓存机制
- 查看是否使用缓存：show variables like 'have_query_cache';
- 关于缓存的参数可以在my.cnf中设置
- 对于缓存的命中，需要注意：
    - 1）两个查询在任何字符上的不同（例如：空格、注释），都会导致缓存不会命中。
    - 2）如果查询中包含任何用户自定义函数、存储函数、用户变量、临时表、mysql库中的系统表，其查询结果都不会被缓存。
    - 3）缓存失效时机：查询对应的表数据发生变化
    - 4）只有当缓存带来的资源节约大于其本身消耗的资源时，才会给系统带来性能提升
    - 5）可以使用SQL_CACHE和SQL_NO_CACHE控制查询语句是否使用缓存，比如：select SQL_NO_CACHE count(*) from users where email = 'hello';
    - 6）不要轻易打开查询缓存，特别是写密集型应用。


### Ref:
https://dzone.com/articles/how-to-optimize-mysql-queries-for-speed-and-perfor