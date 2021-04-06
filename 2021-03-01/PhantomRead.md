## 简述脏读和幻读的发生场景，InnoDB 是如何解决幻读的？

### 事务的四种隔离级别发生的问题和对应场景
数据库环境为 MySQL 5.7
#### 一.  READ UNCOMMITTED（读未提交）
该隔离级别的事务会读到其它未提交事务的数据，此现象也称之为 **脏读** 。
1. 准备两个终端，在此命名为 mysql 终端 1 和 mysql 终端 2，再准备一张测试表 test ，写入一条测试数据并调整隔离级别为 READ UNCOMMITTED ，任意一个终端执行即可。

```sql
SET @@session.transaction_isolation = 'READ-UNCOMMITTED';
create database test;
use test;
create table test(id int primary key);
insert into test(id) values(1);
```
2. 登录 mysql 终端 1，开启一个事务，将 ID 为 1 的记录更新为 2 。

```sql
begin;
update test set id = 2 where id = 1;
select * from test; -- 此时看到一条ID为2的记录
```

3. 登录 mysql终端2，开启一个事务后查看表中的数据。

```sql
use test;
begin;
select * from test; -- 此时看到一条 ID 为 2 的记录
```
最后一步读取到了 mysql 终端 1 中未提交的事务（没有 commit 提交动作），即产生了 脏读 ，大部分业务场景都不允许脏读出现，但是此隔离级别下数据库的并发是最好的。

#### 二. READ COMMITTED（读提交）
一个事务可以读取另一个已提交的事务，多次读取会造成不一样的结果，此现象称为 **不可重复读问题**
1. 准备两个终端，在此命名为 mysql 终端 1 和 mysql 终端 2，再准备一张测试表 test ，写入一条测试数据并调整隔离级别为 READ COMMITTED ，任意一个终端执行即可。

```
SET @@session.transaction_isolation = 'READ-COMMITTED';
create database test;
use test;
create table test(id int primary key);
insert into test(id) values(1);
```
2. 登录 mysql 终端 1，开启一个事务，将 ID 为 1 的记录更新为 2 ，并确认记录数变更过来。

```
begin;
update test set id = 2 where id = 1;
select * from test; -- 此时看到一条记录为 2
```
3. 登录 mysql终端2，开启一个事务后，查看表中的数据。

```
use test;
begin;
select * from test; -- 此时看一条 ID 为 1 的记录
```

4. 登录 mysql 终端 1，提交事务。

```
commit;
```

5. 切换到 mysql 终端 2。

```
select * from test; -- 此时看到一条 ID 为 2 的记录
```
mysql 终端 2 在开启了一个事务之后，在第一次读取 test 表（此时 mysql 终端 1 的事务还未提交）时 ID 为 1 ，在第二次读取 test 表（此时 mysql 终端 1 的事务已经提交）时 ID 已经变为 2 ，说明在此隔离级别下已经读取到已提交的事务。

#### 三. REPEATABLE READ（可重复读）
- 该隔离级别是 MySQL 默认的隔离级别
- 在同一个事务里， select 的结果是事务开始时时间点的状态，因此，同样的 select 操作读到的结果会是一致的，但是，会有 **幻读** 现象。
1. 准备两个终端，在此命名为 mysql 终端 1 和 mysql 终端 2，准备一张测试表 test 并调整隔离级别为 REPEATABLE READ ，任意一个终端执行即可。


```
SET @@session.transaction_isolation = 'REPEATABLE-READ';
create database test;
use test;
create table test(id int primary key,name varchar(20));
```
2. 登录 mysql 终端 1，开启一个事务。

```
begin;
select * from test; -- 无记录
```

3. 登录 mysql 终端 2，开启一个事务。

```
begin;
select * from test; -- 无记录
```

4. 切换到 mysql 终端 1，增加一条记录并提交。

```
insert into test(id,name) values(1,'a');
commit;
```

5. 切换到 msyql 终端 2。

```
select * from test; --此时查询还是无记录
```

6. 此时接着在 mysql 终端 2 插入一条数据。

```
insert into test(id,name) values(1,'b'); -- 此时报主键冲突的错误
```
明明在第 5 步没有数据，为什么在这里会报错呢？其实这就是该隔离级别下可能产生的问题，MySQL 称之为 **幻读** 。

#### 四. SERIALIZABLE（序列化）
在该隔离级别下事务都是串行顺序执行的，MySQL 数据库的 InnoDB 引擎会给读操作隐式加一把读共享锁，从而避免了脏读、不可重读复读和幻读问题。
1. 准备两个终端，在此命名为 mysql 终端 1 和 mysql 终端 2，分别登入 mysql，准备一张测试表 test 并调整隔离级别为 SERIALIZABLE ，任意一个终端执行即可。


```
SET @@session.transaction_isolation = 'SERIALIZABLE';
create database test;
use test;
create table test(id int primary key);
```

2. 登录 mysql终端1，开启一个事务，并写入一条数据。

```
begin;
insert into test(id) values(1);
```

3. 登录 mysql 终端 2，开启一个事务。
 
```
begin;
select * from test; -- 此时会一直卡住
```

4. 立马切换到 mysql 终端 1,提交事务。

```
commit;
```

一旦事务提交，msyql 终端 2 会立马返回 ID 为 1 的记录，否则会一直卡住，直到超时，其中超时参数是由 innodb_lock_wait_timeout 控制。由于每条 select 语句都会加锁，所以该隔离级别的数据库并发能力最弱，但是有些资料表明该结论也不一定对，如果感兴趣，您可以自行做个压力测试。


### MySQL 中的锁
InnoDB 实现了两种类型的行级锁：

- 共享锁 （也称为S锁）：允许事务读取一行数据。

可以使用 SQL 语句 select * from tableName where... lock in share mode; 手动加 S 锁。

- 独占锁 （也称为X锁）：允许事务删除或更新一行数据。

可以使用下面 SQL 语句手动加 X 锁。
 
```
select * from tableName where... for update;
```


**注**： S 锁和 S 锁是 兼容 的，X 锁和其它锁都 不兼容。

为了实现多粒度的锁机制，InnoDB 还有两种内部使用的 意向锁 ，由 InnoDB 自动添加，且都是表级别的锁。

- 意向共享锁 （IS）：事务即将给表中的各个行设置共享锁，事务给数据行加 S 锁前必须获得该表的 IS 锁。
- 意向排他锁 （IX）：事务即将给表中的各个行设置排他锁，事务给数据行加 X 锁前必须获得该表 IX 锁。

意向锁的主要目的是为了使得 行锁 和 表锁 共存。

#### 行锁的算法

InnoDB 存储引擎使用三种行锁的算法用来满足相关事务隔离级别的要求。

- Record Locks

该锁为索引记录上的锁，如果表中没有定义索引，InnoDB 会默认为该表创建一个隐藏的聚簇索引，并使用该索引锁定记录。

- Gap Locks

该锁会锁定一个范围，但是不括记录本身。可以通过修改隔离级别为 READ COMMITTED 或者配置 innodb_locks_unsafe_for_binlog 参数为 ON 。

- Next-key Locks

该锁就是 Record Locks 和 Gap Locks 的组合，即锁定一个范围并且锁定该记录本身。

### InnoDB 是如何解决幻读的？
InnoDB 使用 **Next-key Locks**解决幻读问题。
- 注： 如果索引有唯一属性，则 InnnoDB 会自动将 Next-key Locks 降级为 Record Locks。举个例子，如果一个索引有 1, 3, 5 三个值，则该索引锁定的区间为 (-∞,1], (1,3], (3,5], (5,+ ∞) 。