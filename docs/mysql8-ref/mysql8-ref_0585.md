# 10.5.6 优化 InnoDB 查询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-queries.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-innodb-queries.html)

要为`InnoDB`表调整查询，为每个表创建适当的索引集。有关详细信息，请参见第 10.3.1 节，“MySQL 如何使用索引”。遵循这些`InnoDB`索引指南：

+   因为每个`InnoDB`表都有一个主键（无论您是否请求），请为每个表指定一组主键列，这些列在最重要和时间关键的查询中使用。

+   不要在主键中指定太多或太长的列，因为这些列值会在每个二级索引中重复。当索引包含不必要的数据时，读取这些数据的 I/O 和缓存它的内存会降低服务器的性能和可伸缩性。

+   不要为每一列创建单独的二级索引，因为每个查询只能使用一个索引。很少被测试的列或者只有少量不同值的列上的索引可能对任何查询都没有帮助。如果对同一表有许多查询，测试不同列的组合，请尝试创建少量的连接索引而不是大量的单列索引。如果一个索引包含了结果集所需的所有列（称为覆盖索引），查询可能可以完全避免读取表数据。

+   如果一个索引列不能包含任何`NULL`值，请在创建表时将其声明为`NOT NULL`。当优化器知道每列是否包含`NULL`值时，可以更好地确定哪个索引对查询最有效。

+   您可以使用第 10.5.3 节，“优化 InnoDB 只读事务”中的技术，为`InnoDB`表优化单个查询事务。