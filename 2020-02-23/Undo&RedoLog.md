## 简述 undo log、 redo log 和 binlog 的作用

### MySQL中有六种日志文件，分别是：

1. 重做日志（redo log）
2. 回滚日志（undo log）
3. 二进制日志（binlog）
4. 错误日志（errorlog）
5. 慢查询日志（slow query log）
6. 一般查询日志（general log）
7. 中继日志（relay log）。

### undo log
undo log是把所有没有COMMIT的事务回滚到事务开始前的状态，系统崩溃时，可能有些事务还没有COMMIT，在系统恢复时，这些没有COMMIT的事务就需要借助undo log来进行回滚。
- **作用**：undo log是回滚日志，提供回滚操作

### redo log
redo log是指在回放日志的时候把已经COMMIT的事务重做一遍，对于没有commit的事务按照abort处理，不进行任何操作。
redolog 用来保证数据库宕机后可以通过该文件进行恢复。
 InnoDB 在更新数据的时候会采用 WAL 技术，也就是 Write Ahead Logging
 - **作用**：redo log是重做日志，提供前滚操作

###  undo log 和 redo log  的区别是:
- undo log不是redo log的逆向过程，其实它们都算是用来恢复的日志：
- redo log通常是物理日志，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样，它用来恢复提交后的物理数据页，且只能恢复到最后一次提交的位置。
- undo log用来回滚行记录到某个版本，undo log一般是逻辑日志，根据每行记录进行记录。


### binlog
通过 binlog 来进行高可用，也就是通过 binlog 来将数据同步到集群内其他的 MySQL 实例。


### binlog 和 redolog 的区别是:
1. binlog 是在存储引擎上层 Server层写入的，而binlog也需要在事务提交前写入文件。
2. binlog记录的是逻辑操作，也就是对应的 sql,redolog 记录的底层某个数据页的物理操作，
3. redolog 是循环写的，而binlog是追加写的，不会覆盖以前写的数据。
4. binlog 的写入页需要通过 fsync来保证落盘.
为了提高tps,MySQL可以通过参数sync_binlog来控制是否需要同步刷盘，该策略会影响当主库宕机后备库数据可能并没有完全同步到主库数据。
5. 由于事务的原子性，需要保证事务提交的时候 redolog 和 binlog 都写入成功，所以 MySQL 执行层采用了两阶段提交来保证 redolog 和 binlog 都写入成功后才commit，如果一方失败则会进行回滚。

### 一条update语句的执行过程：
update person set age = 30 where id = 1;Text only
1. 分配事务 ID ，开启事务，获取锁，没有获取到锁则等待。
2. 执行器先通过存储引擎找到 id = 1 的数据页，如果缓冲池有则直接取出，没有则去主键索引上取出对应的数据页放入缓冲池。
3. 在数据页内找到 id = 1 这行记录，取出，将 age 改为 30 然后写入内存
4. 生成 redolog undolog 到内存，redolog 状态为 prepare
5. 将 redolog undolog 写入文件并调用 fsync
6. server 层生成 binlog 并写入文件调用 fsync
7. 事务提交，将 redolog 的状态改为 commited 释放锁


