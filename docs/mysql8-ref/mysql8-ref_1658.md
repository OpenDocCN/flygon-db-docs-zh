# 25.4.2 NDB 集群配置参数、选项和变量概述

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-configuration-overview.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-configuration-overview.html)

25.4.2.1 NDB 集群数据节点配置参数

25.4.2.2 NDB 集群管理节点配置参数

25.4.2.3 NDB 集群 SQL 节点和 API 节点配置参数

25.4.2.4 其他 NDB 集群配置参数

25.4.2.5 NDB 集群 mysqld 选项和变量参考

接下来的几个部分提供了用于管理节点行为各个方面的`config.ini`文件中使用的 NDB 集群节点配置参数的摘要表，以及由**mysqld** 从`my.cnf`文件或从命令行读取的选项和变量的摘要表，当作为 NDB 集群进程运行时。每个节点参数表列出了给定类型（`ndbd`、`ndb_mgmd`、`mysqld`、`computer`、`tcp`或`shm`）的参数、选项或变量，以及适用的默认值、最小值和最大值。

**重新启动节点时的考虑。** 对于节点参数，这些表还指示所需的重新启动类型（节点重新启动或系统重新启动）以及是否必须使用`--initial`来更改给定配置参数的值。在执行节点重新启动或初始节点重新启动时，必须依次重新启动所有集群的数据节点（也称为滚动重启）。可以在线更新标记为`node`的集群配置参数，即在不关闭集群的情况下以这种方式进行。初始节点重新启动需要使用`--initial`选项重新启动每个**ndbd** 进程。

系统重新启动需要完全关闭并重新启动整个集群。初始系统重新启动需要备份集群，在关闭后擦除集群文件系统，然后在重新启动后从备份中恢复。

在任何集群重新启动中，所有集群的管理服务器都必须重新启动，以便它们读取更新后的配置参数值。

重要提示

数值集群参数的值通常可以增加而不会出现任何问题，尽管建议逐步进行，以相对较小的增量进行此类调整。许多这些参数可以在线增加，使用滚动重启。

然而，减少这些参数的值——无论是通过节点重启、节点初始重启，甚至是整个集群系统的重启——都不应轻率进行；建议您在经过仔细规划和测试后才这样做。特别是涉及内存使用和磁盘空间的那些参数，如`MaxNoOfTables`、`MaxNoOfOrderedIndexes`和`MaxNoOfUniqueHashIndexes`。此外，通常情况下，与内存和磁盘使用相关的配置参数可以通过简单的节点重启来提高，但需要通过初始节点重启来降低。

因为其中一些参数可用于配置多种类型的集群节点，所以它们可能会出现在多个表中。

注意

`4294967039`经常出现在这些表中作为最大值。这个值在`NDBCLUSTER`源代码中被定义为`MAX_INT_RNIL`，等于`0xFFFFFEFF`，或者`2³² − 2⁸ − 1`。
