## MVCC原理
### 1. 什么是MVCC
多版本并发控制
### 2. MVCC作用场景
MVCC只在InnoDB的事务中READ COMMITTED 和 REPEATABLE READ 两个隔离级别下工作。

### 3. 版本链
#### 版本链的作用
每条记录都有一个版本连，每次改变都一个版本号。
SELECT可以去版本链中拿记录，这就实现了读-写，写-读的并发执行
#### 版本链的组成
InnoDB引擎表中，它的聚簇索引记录中有两个必要的隐藏列：
##### 1. trx_id (DB_TRX_ID)
这个id用来存储的每次对某条聚簇索引记录进行修改的时候的事务id。
##### 2. roll_pointer （DB_ROLL_PTR）
roll_pointer是存了一个指针，它指向这条聚簇索引记录的上一个版本的位置，通过它来获得上一个版本的记录信息。
- 每次对哪条聚簇索引记录有修改的时候，都会把老版本写入undo日志中。
##### 3. id (DB_ROW_ID）
如果建了主键就是主键ID，否则引擎会默认添加ROW_ID字段，这是隐藏

### 4. ReadView
#### ReadView的作用
ReadView中主要就是有个列表来存储我们系统中当前活跃着的读写事务，也就是begin了还未提交的事务。
#### RR和RC的不同
RR和RC区别就在于它们生成ReadView的策略不同。
RC隔离级别下的事务在查询的开始都会生成一个独立的ReadView,而RR隔离级别则在第一次读的时候生成一个ReadView，之后的读都复用之前的ReadView。

