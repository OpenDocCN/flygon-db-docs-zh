> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-compression-tuning-monitoring.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-tuning-monitoring.html)

#### 17.9.1.4 在运行时监控 InnoDB 表压缩

整体应用性能、CPU 和 I/O 利用率以及磁盘文件大小是衡量压缩对应用效果的良好指标。本节基于第 17.9.1.3 节“调整 InnoDB 表压缩”中的性能调优建议，展示如何发现在初始测试中可能不会出现的问题。

要深入了解压缩表的性能考虑，您可以使用信息模式表在运行时监控压缩性能，详情请参阅示例 17.1“使用压缩信息模式表”。这些表反映了内存的内部使用和整体使用的压缩率。

`INNODB_CMP`表报告每个压缩页面大小（`KEY_BLOCK_SIZE`）的压缩活动信息。这些表中的信息是系统范围的：它总结了数据库中所有压缩表的压缩统计信息。您可以使用这些数据来帮助决定是否压缩表，通过在没有访问其他压缩表时检查这些表来检查这些表。它对服务器的开销相对较低，因此您可以定期在生产服务器上查询它，以检查压缩功能的整体效率。

`INNODB_CMP_PER_INDEX` 表报告了单个表和索引的压缩活动信息。这些信息更具针对性，更有助于逐个评估压缩效率和诊断性能问题的表或索引。（因为每个 `InnoDB` 表都表示为一个聚簇索引，MySQL 在这种情况下并不会对表和索引进行明显区分。）`INNODB_CMP_PER_INDEX` 表确实涉及相当大的开销，因此更适用于开发服务器，在那里你可以独立比较不同 工作负载、数据和压缩设置的影响。为了防止意外增加此监控开销，你必须在查询 `INNODB_CMP_PER_INDEX` 表之前启用 `innodb_cmp_per_index_enabled` 配置选项。

要考虑的关键统计数据是压缩和解压操作的数量和所花费的时间。由于 MySQL 在修改后无法容纳压缩数据时会拆分 B 树 节点，因此比较“成功”压缩操作的数量与总体操作数量。根据 `INNODB_CMP` 和 `INNODB_CMP_PER_INDEX` 表中的信息以及整体应用程序性能和硬件资源利用情况，你可能需要调整硬件配置、调整缓冲池大小、选择不同的页面大小或选择不同的要压缩的表。

如果压缩和解压所需的 CPU 时间较长，更换更快或多核 CPU 可以帮助提高性能，同时保持相同的数据、应用工作负载和一组压缩表。增加缓冲池的大小也可能有助于性能，这样更多未压缩的页面可以保留在内存中，减少只存在于内存中的压缩页面的解压需求。

如果整体压缩操作的数量（与应用程序中的`INSERT`、`UPDATE`和`DELETE`操作以及数据库大小相比）较多，可能表明一些被压缩的表正在过度更新，以至于无法有效压缩。如果是这样，选择更大的页面大小，或者更加谨慎地选择要压缩的表。

如果“成功”压缩操作（`COMPRESS_OPS_OK`）的数量占总压缩操作（`COMPRESS_OPS`）的比例很高，那么系统可能表现良好。如果比例较低，那么 MySQL 可能比理想情况下更频繁地重新组织、重新压缩和拆分 B 树节点。在这种情况下，避免压缩一些表，或者增加一些被压缩表的`KEY_BLOCK_SIZE`。您可能会关闭一些表的压缩，如果这些表导致应用程序中“压缩失败”的数量超过总数的 1%或 2%。（在诸如数据加载之类的临时操作期间，这种失败比率可能是可以接受的）。
