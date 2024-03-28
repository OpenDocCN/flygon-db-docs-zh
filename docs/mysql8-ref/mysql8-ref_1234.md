# 17.11.3 InnoDB 检查点

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-checkpoints.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-checkpoints.html)

使你的日志文件变得非常大可能会在检查点期间减少磁盘 I/O。通常情况下，将日志文件的总大小设置为缓冲池的大小甚至更大是有意义的。

#### 检查点处理的工作原理

`InnoDB`实现了一种被称为模糊检查点的检查点机制。`InnoDB`会将修改过的数据库页面以小批量方式从缓冲池刷新出去。在检查点过程中，没有必要一次性刷新缓冲池，这样会干扰用户 SQL 语句的处理过程。

在崩溃恢复期间，`InnoDB`会寻找写入日志文件的检查点标签。它知道标签之前对数据库的所有修改都存在于数据库的磁盘镜像中。然后，`InnoDB`从检查点开始向前扫描日志文件，将记录的修改应用到数据库中。