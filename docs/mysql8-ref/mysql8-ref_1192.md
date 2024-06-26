# 17.7.5 InnoDB 中的死锁

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-deadlocks.html)

17.7.5.1 InnoDB 死锁示例

17.7.5.2 死锁检测

17.7.5.3 如何最小化和处理死锁

死锁是指不同事务由于各自持有对方需要的锁而无法继续进行的情况。因为两个事务都在等待资源可用，所以它们都不会释放自己持有的锁。

当事务以相反的顺序锁定多个表中的行（通过诸如`UPDATE`或`SELECT ... FOR UPDATE`等语句）时，可能会发生死锁。当这些语句锁定索引记录和间隙的范围时，也可能发生死锁，因为每个事务由于时间问题而获取了一些锁，但没有获取其他锁。有关死锁示例，请参见第 17.7.5.1 节，“InnoDB 死锁示例”。

为减少死锁的可能性，请使用事务而不是`LOCK TABLES`语句；确保插入或更新数据的事务足够小，不会长时间保持打开状态；当不同事务更新多个表或大范围的行时，在每个事务中使用相同的操作顺序（如`SELECT ... FOR UPDATE`）；在`SELECT ... FOR UPDATE`和`UPDATE ... WHERE`语句中使用的列上创建索引。死锁的可能性不受隔离级别的影响，因为隔离级别改变了读操作的行为，而死锁是由写操作引起的。有关避免和恢复死锁条件的更多信息，请参见第 17.7.5.3 节，“如何最小化和处理死锁”。

当死锁检测被启用（默认情况下），并且确实发生了死锁时，`InnoDB`会检测到这种情况，并回滚其中一个事务（受害者）。如果使用`innodb_deadlock_detect`变量禁用了死锁检测，`InnoDB`会依赖于`innodb_lock_wait_timeout`设置来在发生死锁时回滚事务。因此，即使您的应用逻辑是正确的，您仍然必须处理事务必须重试的情况。要查看`InnoDB`用户事务中的最后一个死锁，请使用`SHOW ENGINE INNODB STATUS`。如果频繁的死锁突显出事务结构或应用程序错误处理的问题，请启用`innodb_print_all_deadlocks`以将所有死锁的信息打印到**mysqld**错误日志中。有关死锁如何自动检测和处理的更多信息，请参见 Section 17.7.5.2, “Deadlock Detection”。
