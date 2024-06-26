# 10.5.2 优化 InnoDB 事务管理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-transaction-management.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-transaction-management.html)

为了优化`InnoDB`事务处理，找到事务特性的性能开销和服务器工作负载之间的理想平衡。例如，如果一个应用程序每秒提交数千次，可能会遇到性能问题，如果每 2-3 小时才提交一次，则可能会遇到不同的性能问题。

+   默认的 MySQL 设置`AUTOCOMMIT=1`可能会对繁忙的数据库服务器施加性能限制。在实际情况下，通过发出`SET AUTOCOMMIT=0`或`START TRANSACTION`语句，然后在进行所有更改后发出`COMMIT`语句，将几个相关的数据更改操作包装成一个单独的事务。

    如果事务对数据库进行了修改，则每次事务提交时`InnoDB`必须将日志刷新到磁盘。当每个更改后跟着一个提交时（与默认的自动提交设置一样），存储设备的 I/O 吞吐量限制了每秒潜在操作的数量。

+   或者，对于仅包含单个`SELECT`语句的事务，打开`AUTOCOMMIT`有助于`InnoDB`识别只读事务并对其进行优化。有关要求，请参阅第 10.5.3 节，“优化 InnoDB 只读事务”。

+   避免在插入、更新或删除大量行后执行回滚。如果一个大型事务正在减慢服务器性能，回滚可能会使问题变得更糟，可能需要执行的时间是原始数据更改操作的几倍。杀死数据库进程并不会有帮助，因为回滚会在服务器启动时重新开始。

    为了最大程度减少出现此问题的机会：

    +   增加缓冲池的大小，以便所有数据更改都可以被缓存而不是立即写入磁盘。

    +   设置`innodb_change_buffering=all`，以便在插入操作之外还缓冲更新和删除操作。

    +   在大数据更改操作期间定期发出`COMMIT`语句，可能将单个删除或更新拆分为操作较少行数的多个语句。

    一旦发生失控的回滚，增加缓冲池大小，使回滚变为 CPU 限制并快速运行，或者杀死服务器并使用`innodb_force_recovery=3`重新启动，如第 17.18.2 节，“InnoDB 恢复”中所述。

    默认设置`innodb_change_buffering=all`预计不会经常出现这个问题，它允许更新和删除操作在内存中被缓存，使它们在第一次执行时更快，并且在需要时回滚也更快。确保在处理有许多插入、更新或删除的长时间运行事务的服务器上使用这个参数设置。

+   如果可以容忍在意外退出时丢失一些最新提交的事务，可以将`innodb_flush_log_at_trx_commit`参数设置为 0。`InnoDB`尝试每秒刷新一次日志，尽管刷新不是保证的。

+   当行被修改或删除时，行和相关的撤销日志不会立即被物理删除，甚至在事务提交后也不会立即删除。旧数据会被保留，直到比它更早或同时开始的事务完成，这样这些事务就可以访问被修改或删除行的先前状态。因此，长时间运行的事务可能会阻止`InnoDB`清除被不同事务更改的数据。

+   当在长时间运行的事务中修改或删除行时，使用`READ COMMITTED`和`REPEATABLE READ`隔离级别的其他事务需要更多工作来重建旧数据，如果它们读取相同的行。

+   当一个长时间运行的事务修改表时，来自其他事务对该表的查询不会利用覆盖索引技术。通常可以从二级索引检索所有结果列的查询，而是从表数据中查找适当的值。

    如果发现二级索引页的`PAGE_MAX_TRX_ID`太新，或者如果二级索引中的记录被标记为删除，`InnoDB`可能需要使用聚簇索引查找记录。
