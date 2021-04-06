## 简述 undo log， redo log和binlog的作用和区别

注： undo log 和redo log是为了支持事务才有的log，InnoDB引擎中有这两个log，MyISAM引擎没有。

###  一. undo log 和 redo log  的区别
- 1. undo log不是redo log的逆向过程，其实它们都算是用来恢复的日志：
- 2. 物理&逻辑日志
    - redo log通常是物理日志，记录的是数据页的物理修改。
    - undo log用来回滚行记录到某个版本，undo log一般是逻辑日志，根据每行记录进行记录。
- 3. 顺序&随机读写
    - redo log 是顺序写的，在数据库运行时不需要对redo log 文件进行读取操作。
    - undo log是随机读写的。

### 二.为什么需要redo log
####  mysql是如何保证一致性的呢？
- 最简单的做法是在每次事务提交的时候，将该事务涉及修改的数据页全部刷新到磁盘中。但是这么做会有严重的性能问题，主要体现在两个方面：

1. **太浪费资源**: 因为 Innodb 是以 页 为单位进行磁盘交互的，而一个事务很可能只修改一个数据页里面的几个字节，这个时候将完整的数据页刷到磁盘的话，太浪费资源了。
2. **数据页在物理上并不连续**: 一个事务可能涉及修改多个数据页，并且这些数据页在物理上并不连续，使用随机IO写入性能太差。
- 因此 mysql 设计了 redo log ， 具体来说就是只记录事务对数据页做了哪些修改

### 三.redo log作用和组成
####  1. **redo log的作用**
redo log是用来实现事务的**持久性**，即ACID中的D。
- 当事务提交（COMMIT）时，必须先将该事务的所有日志写入到redo log file进行持久化，待事务的COMMIT操作完成才算完成。
####  2. **redo log组成**
    - 1. 内存中的重做日志缓冲（redo log buffer), 是易失的
    - 2. 重做日志文件（redo log file）,是持久的
####  3. **WAL(Write-Ahead Logging)**
    -  先写日志，再写磁盘
    -  mysql 每执行一条 DML 语句，先将记录写入 redo log buffer，后续某个时间点再一次性将多个操作记录写到 redo log file 。


### 四.undo log
####  1. **undo log的作用**
undo log是用来实现事务的**原子性**，即ACID中的A。用来帮助**事务回滚**及**MVCC**的功能。   
- **1. 事务回滚**：如果执行的事务或者语句失败了， 或者ROLLBACK语句请求回滚的时候，可以利用undo log将数据回滚到修改之前
    -     undo log是逻辑日志, 事务回滚ROLLBACK时，实际上做的是先前相反的工作，即对于INSERT、DELETE、UPDATE，存储引擎分别执行的是DELETE、INSERT、相反的UPDATE
- **2. MVCC**：MVCC是通过undo log来完成的
    -     当用户读取一行记录时，若该记录已经被其他的事务占用，当前事务通过undo log读取之前的杭版本信息，实现非锁定读取。
- **3. undo log会产生redo log**：
    -     undo log会产生redo log，因为undo log也需要持久性保护。
####  2. **undo log的格式**
undo log 分为
- insert undo log:**insert**操作中产生的undo log
- update undo log:**delete**或**update** 操作中产生的undo log

### 五.redo log 和binlog
####  1. **binlog的作用**
    MySql数据库的binlog是用来POINT-IN-TIME（PIT）的恢复和主从复制的（Replication）环境的建立。
####  2. **binlog的格式**
- 三种格式，分别为 STATMENT, ROW 和 MIXED
- MySQL 5.7.7 之前，默认的格式是 STATEMENT ， MySQL 5.7.7 之后，默认值是 ROW
    -     STATMENT ： 基于 SQL 语句的复制，每一条会修改数据的sql语句会记录到 binlog 中
             优点：不需要记录每一行的变化，减少了日志量
             缺点： 在某些情况下会导致主从数据不一致
    -     ROW ： 基于行的复制，不记录每条sql语句的上下文信息，仅需记录哪条数据被修改
             优点： 不会出现某些特定情况下无法被正确复制的问题（例如，存储过程、或function、或trigger的调用和触发）
             缺点： 会产生大量的日志，尤其是 alter table 的时候会让日志暴涨
    -     MIXED ： 基于STATMENT和ROW两种模式的混合复制

#### 2. redo log 和binlog的相似点
    都记录了对数据库的操作日志

#### 3. redo log 和binlog的不同点
-  作用范围
    - redo log是InnoDB存储引擎层产生的
    - binlog是MySql数据库的上层产生的，Mysql任何引擎对于DB的修改都会产生binlog
-  内容形式
    - binlog是一种逻辑日志，binlog记录的是SQL语句的复制或者基于行的复制
    - redolog是物理日志，记录的是对每个页的修改
-  写入磁盘的时间点不同
    - binlog是事务提交完成后一次性写入
    - redolog是事务进行中不断地写入，并不是随着事务提交的顺序进行写入的。


### Reference
《MySql技术内幕：InnoDB存储引擎》：第七章 - 事务