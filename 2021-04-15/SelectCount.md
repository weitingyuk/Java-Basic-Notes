## SELECT COUNT(*) 

EXPLAIN 来查询了一下执行计划

- EXPLAIN SELECT COUNT(*) FROM SomeTable
- 结果是Extra = Using Index
- 此条语句在此例中用到的并不是主键索引，而是辅助索引
- 而且COUNT(1)， COUNT( * )，MySQL 都会用成本最小的辅助索引查询方式来计数，也就是使用 COUNT(*) 由于 MySQL 的优化已经保证了它的查询性能是最好的！