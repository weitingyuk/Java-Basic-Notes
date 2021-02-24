## 简述 undo log 和 redo log 的作用

注： undo log 和redo log是为了支持事务才有的log，InnoDB引擎中有这两个log，MyISAM引擎没有。


### undo log
撤消日志是与单个读写事务关联的撤消日志记录的集合。
- **作用**：undo log是回滚日志，提供回滚操作

### redo log
redo log是基于磁盘的数据结构，在崩溃恢复期间用于纠正不完整事务写入的数据。
 - **作用**：redolog 用来保证数据库宕机后可以通过该文件进行恢复。

###  undo log 和 redo log  的区别是:
- undo log不是redo log的逆向过程，其实它们都算是用来恢复的日志：
- redo log通常是物理日志，记录的是数据页的物理修改，而不是某一行或某几行修改成怎样，它用来恢复提交后的物理数据页，且只能恢复到最后一次提交的位置。
- undo log用来回滚行记录到某个版本，undo log一般是逻辑日志，根据每行记录进行记录。
