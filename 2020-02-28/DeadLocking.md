## 什么情况下会发生死锁，如何解决死锁？

### 什么情况下会发生死锁
- 死锁是指由于每个事务都持有对方需要的锁而无法进行其他事务的情况。因为这两个事务都在 await 资源变得可用，所以两个都不会释放它持有的锁。
- InnoDB使用自动行级锁定。即使在仅插入或删除单行的事务中，也可能会遇到死锁。
    - 因为这些操作并不是 true 的“原子”操作；它们会自动对插入或删除的行的(可能是多个)索引记录设置锁定。

### 死锁检测和回滚
- 启用deadlock detection时(默认设置)，InnoDB自动检测到事务deadlocks并回滚一个或多个事务以 break 僵局。
    - 如果启用了死锁检测(默认设置)并且确实发生了死锁，则InnoDB检测到该情况并回滚其中一个事务(受害方)。
    - InnoDB尝试选择要回滚的小事务，其中事务的大小由插入，更新或删除的行数确定。
- 禁用死锁检测：在高并发系统上，当多个线程 await 相同的锁时，死锁检测会导致速度变慢。

### 如何解决死锁？
Deadlocks是事务数据库中的经典问题，但是它们并不危险，除非它们如此频繁以至于您根本无法运行某些事务。
一般可以使用以下技术来处理死锁并减少发生死锁的可能性：
- 1. SHOW ENGINE：在INNODB中可以使用SHOW ENGINE语句命令来确定最新死锁的原因。
- 2. 启用innodb_print_all_deadlocks：如果频繁出现死锁警告引起关注，请通过启用innodb_print_all_deadlocks配置选项来收集更多的调试信息。有关每个死锁的信息，不仅是最新的死锁，还记录在 MySQL error log中。完成调试后，请禁用此选项。
- 3. 如果由于死锁而失败，请始终准备重新触发事务。死锁并不危险。请再试一次。
- 4. 保持 Transaction 小巧且持续时间短，以使 Transaction 不易发生冲突。
- 5. 进行一系列相关更改后立即提交事务，以减少冲突的发生。特别是，不要长时间未提交事务而保持交互式mysql会话打开。
- 6. 如果使用locking reads(SELECT ... FOR UPDATE或SELECT ... LOCK IN SHARE MODE)，请尝试使用较低的隔离级别，例如READ COMMITTED。
- 7. 修改事务中的多个 table 或同一 table 中的不同行时，每次都要以一致的 Sequences 执行这些操作。然后，事务形成定义明确的队列，并且不会死锁。例如，将数据库操作组织到应用程序内的函数中，或调用存储的例程，而不是在不同的地方对INSERT，UPDATE和DELETE语句的多个相似序列进行编码。
- 8. 将精选的索引添加到 table 中。这样，您的查询就需要扫描更少的索引记录，从而设置更少的锁。使用EXPLAIN SELECT来确定 MySQL 服务器认为哪些索引最适合您的查询。
- 9. 使用较少的锁定。如果您有能力允许SELECT从旧快照返回数据，请不要在其上添加FOR UPDATE或LOCK IN SHARE MODE子句。在这里使用READ COMMITTED隔离级别是件好事，因为同一事务中的每个一致性读取均从其自己的新快照读取。
- 10. 如果没有其他帮助，请使用 table 级锁序列化事务。对事务 table(例如InnoDBtable)使用LOCK TABLES的正确方法是，先以SET autocommit = 0(不是START TRANSACTION)后跟LOCK TABLES来开始事务，并且在明确提交事务之前不要调用UNLOCK TABLES。
- 11. 序列化事务的另一种方法是创建一个仅包含一行的辅助“signal 量”table。在访问其他 table 之前，让每个事务更新该行。这样，所有事务都以串行方式发生。请注意，InnoDB即时死锁检测算法在这种情况下也适用，因为序列化锁是行级锁。对于 MySQLtable 级锁，必须使用超时方法来解决死锁。


### Reference
https://www.docs4dev.com/docs/zh/mysql/5.7/reference/innodb-deadlocks.html
https://www.docs4dev.com/docs/zh/mysql/5.7/reference/show-engine.html