# 25.3.7 NDB Cluster 的升级和降级

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-upgrade-downgrade.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-upgrade-downgrade.html)

+   升级到 NDB 8.0 支持的版本

+   回滚 NDB Cluster 8.0 升级

+   升级或降级 NDB Cluster 时已知问题

本节提供有关 NDB Cluster 软件以及不同 NDB Cluster 8.0 版本之间的兼容性信息，涉及执行升级和降级操作。在尝试升级或降级之前，您应该已经熟悉安装和配置 NDB Cluster。请参阅第 25.4 节，“NDB Cluster 配置”。

重要提示

在 NDB 8.0 中支持 `NDB` 存储引擎的小版本之间的在线升级和降级。支持包含的 MySQL Server（SQL 节点**mysqld**）的原地升级；对于多个 SQL 节点，可以在重新启动单个**mysqld** 进程的同时保持 SQL 应用在线。不支持包含的 MySQL Server 的原地降级（参见第四章，“降级 MySQL”）。

在某些情况下，可能会将最近的 NDB 8.0 小版本升级回到较新版本，并恢复运行为 SQL 节点的任何 MySQL Server 实例的所需状态。如果出现这种需求或必要性，强烈建议在升级 NDB Cluster 之前对每个 SQL 节点进行完整备份。出于同样的原因，您还应该使用新版本的`--ndb-schema-dist-upgrade-allowed=0`启动**mysqld**二进制文件，并在确保不会回退到旧版本之前不允许将其设置为 1。有关更多信息，请参阅回滚 NDB Cluster 8.0 升级。

有关从 8.0 之前版本升级到 NDB 8.0 的信息，请参阅升级到 NDB 8.0 支持的版本。

有关升级或降级 NDB 8.0 时遇到的已知问题和问题的信息，请参见升级或降级 NDB Cluster 时的已知问题。

#### 支持升级至 NDB 8.0 的版本

支持将以下版本的 NDB Cluster 升级至 GA 版本的 NDB Cluster 8.0（8.0.19 及更高版本）：

+   NDB Cluster 7.6：NDB 7.6.4 及更高版本

+   NDB Cluster 7.5：NDB 7.5.4 及更高版本

+   NDB Cluster 7.4：NDB 7.4.6 及更高版本

要从 NDB 7.4 之前的版本系列升级，必须分阶段升级，首先升级到刚列出的版本之一，然后从该版本升级到最新的 NDB 8.0 版本。在这种情况下，建议首先升级至最新的 NDB 7.6 版本。有关从旧版本升级至 NDB 7.6 的信息，请参见升级和降级 NDB 7.6。

#### 撤销 NDB Cluster 8.0 升级

最近将 NDB Cluster 软件升级至 NDB 8.0 版本后，可以将 `NDB` 软件回滚到较早版本，前提是在升级之前、集群运行较新版本期间以及将 NDB Cluster 软件回滚到较早版本之后满足某些条件。具体取决于当地情况；本节提供了关于在上述升级和回滚过程中每个点应该做什么的一般信息。

在大多数情况下，可以无问题地升级和降级数据节点，如其他地方所述；请参见第 25.6.5 节，“执行 NDB Cluster 的滚动重启”。（在执行升级或降级之前，您应执行 `NDB` 备份；有关如何执行此操作，请参见第 25.6.8 节，“NDB Cluster 的在线备份”。）不支持在线降级 SQL 节点，原因如下：

+   从 8.0 版本开始的**mysqld**如果检测到来自较新版本的 MySQL 的文件系统，则无法启动。

+   在许多情况下，**mysqld** 无法打开由较新版本的 MySQL 创建或修改的表。

+   在大多数情况下，**mysqld**无法读取由较新版本的 MySQL 创建或修改的二进制日志文件。

接下来概述的过程提供了升级集群从版本*X*到版本*Y*所需的基本步骤，同时允许将来可能回滚到*X*版本。 （将升级后的集群恢复到*X*版本的过程稍后在本节中介绍。）为此，版本*X*是任何 NDB 8.0 GA 版本，或任何支持升级到 NDB 8.0 的先前 NDB 版本（请参阅支持升级到 NDB 8.0 的版本），而版本*Y*是晚于*X*的 NDB 8.0 版本。

+   *升级前*：备份 NDB *X* SQL 节点状态。可以通过以下一种或多种方式完成：

    +   使用诸如**cp**、**rsync**、**fwbackups**、Amanda 等一个或多个系统工具在静止状态下复制版本*X* SQL 节点文件系统的副本。

        未存储在`NDB`中的任何版本*X*表的转储。您可以使用**mysqldump**生成此转储。

        使用 MySQL Enterprise Backup 创建的备份；有关更多信息，请参见第 32.1 节，“MySQL Enterprise Backup 概述”。

    在任何升级之前建议备份 SQL 节点，无论您是否打算将集群恢复到先前的`NDB`版本。

+   *升级到 NDB *Y*：所有 NDB *Y* **mysqld**二进制文件必须使用`--ndb-schema-dist-upgrade-allowed=0`启动，以防止任何自动模式升级。（一旦降级的可能性过去，您可以安全地将相应的系统变量`ndb_schema_dist_upgrade_allowed`更改回默认值 1，在**mysql**客户端中。）当每个 NDB *Y* SQL 节点启动时，它会连接到集群并同步其`NDB`表模式。之后，您可以从备份中恢复 MySQL 表和状态数据。

    为确保 NDB 复制的连续性，有必要以一种方式升级集群的 SQL 节点，以便在升级过程中任何给定时间点至少有一个**mysqld**充当复制源。有两个 SQL 节点*A*和*B*，您可以这样做：

    1.  在使用 SQL 节点*B*作为复制通道时，将 SQL 节点*A*从 NDB 版本*X*升级到版本*Y*。这将导致*A*上的二进制日志在时期*E1*处出现间隙。

    1.  在所有复制应用程序消耗了 SQL 节点*B*过去时期*E1*的二进制日志后，将复制通道切换到使用 SQL 节点*A*。

    1.  将 SQL 节点*B*升级到 NDB 版本*Y*。这将导致*B*上的二进制日志在时期*E2*处出现间隙。

    1.  在所有复制应用程序消耗了 SQL 节点*`A`*过去时期*`E2`*的二进制日志后，您可以再次根据需要切换复制通道以使用任一 SQL 节点。

    不要对任何现有的`NDB`表使用`ALTER TABLE`；在降级之前不要创建任何无法安全删除的新`NDB`表。

以下过程展示了在执行如上所述的升级后，从版本*`X`*回滚（恢复）到版本*`Y`*的 NDB 集群所需的基本步骤。这里，版本*`X`*是任何 NDB 8.0 GA 版本，或任何支持升级到 NDB 8.0 的先前 NDB 版本（请参见支持升级到 NDB 8.0 的版本）；版本*`Y`*是晚于*`X`*的 NDB 8.0 版本。

+   *回滚之前*：从应保留的 NDB *`Y`* SQL 节点中收集任何**mysqld**状态信息。在大多数情况下，您可以使用**mysqldump**来执行此操作。

    在备份状态数据后，删除自升级以来已创建或更改的所有`NDB`表。

    在进行任何 NDB 集群软件版本更改之前，始终建议备份 SQL 节点。

    您必须为每个**mysqld**（SQL 节点）提供与 MySQL *`X`*兼容的文件系统。您可以使用以下两种方法之一：

    +   通过重新初始化版本*`X`* SQL 节点的磁盘状态，创建一个新的兼容文件系统状态。您可以通过删除 SQL 节点文件系统，然后运行**mysqld** `--initialize`来实现这一点。

    +   从升级前的备份中恢复一个兼容的文件系统（参见第 9.4 节，“使用 mysqldump 进行备份”）。

+   *在`NDB`降级后*：将数据节点降级为 NDB *`X`*后，启动版本*`X`* SQL 节点（**mysqld**的实例）。在每个 SQL 节点上恢复或修复任何其他所需的本地状态信息。可以通过以下操作的某种组合（0 个或多个）将 MySQLD 状态调整为必要的状态：

    +   初始化命令，如**mysqld** `--initialize`。

    +   恢复从版本*`X`* SQL 节点捕获的任何所需或必要的状态信息。

    +   恢复从版本*`Y`* SQL 节点捕获的任何所需或必要的状态信息。

    +   执行清理操作，例如删除过时的日志，如二进制日志或中继日志，并删除任何不再有效的时间相关状态。

    在降级时，为了保持 NDB 复制的连续性，需要以一种方式降级集群的 SQL 节点，以确保在降级过程中至少有一个**mysqld**在任何给定时间点充当复制源。这可以通过与升级 SQL 节点类似的方式来完成。有两个 SQL 节点 *`A`* 和 *`B`*，你可以在降级过程中保持二进制日志没有任何间隙，就像这样：

    1.  以 SQL 节点 *`B`* 充当复制通道，将 SQL 节点 *`A`* 从 NDB 版本 *`Y`* 降级到版本 *`X`。这会导致在时期 *`F1`* 上的 SQL 节点 *`A`* 上的二进制日志中出现间隙。

    1.  在所有复制应用程序消耗了 SQL 节点 *`B`* 过去时期 *`F1`* 的二进制日志后，切换复制通道以使用 SQL 节点 *`A`*。

    1.  将 SQL 节点 *`B`* 降级到 NDB 版本 *`X`*。这会导致在时期 *`F2`* 上的 SQL 节点 *`B`* 上的二进制日志中出现间隙。

    1.  在所有复制应用程序消耗了 SQL 节点 *`A`* 过去时期 *`F2`* 的二进制日志后，二进制日志的冗余性得以恢复，你可以根据需要再次使用任一 SQL 节点作为复制通道。

    另请参阅 Section 25.7.7, “Using Two Replication Channels for NDB Cluster Replication”。

#### 升级或降级 NDB 集群时已知的问题

在本节中，提供关于升级或降级到、从或在 NDB 8.0 版本之间发生的已知问题的信息。

我们建议您在任何 NDB 集群软件升级或降级期间不要尝试进行模式更改。以下是其中一些原因：

+   在某些数据节点启动阶段，不允许对 `NDB` 表执行 DDL 语句。

+   如果在执行过程中停止任何数据节点，则可能会拒绝对 `NDB` 表的 DDL 语句；停止每个数据节点的二进制文件（以便用目标版本的二进制文件替换）是升级或降级过程的一部分。

+   在同一集群中运行不同版本的 NDB Cluster 软件的数据节点时，不允许对 `NDB` 表执行 DDL 语句。

有关执行数据节点在线升级或降级的滚动重启过程的更多信息，请参阅 Section 25.6.5, “Performing a Rolling Restart of an NDB Cluster”。

在执行 NDB 8.0 的次要版本之间的在线升级时，应注意以下列表中的问题。这些问题也适用于从 NDB Cluster 的先前主要版本升级到任何已声明的 NDB 8.0 版本。

+   NDB 8.0.22 为 `config.ini` 文件中的管理节点和数据节点添加了对 IPv6 地址的支持。要在升级过程中开始使用 IPv6 地址，请执行以下步骤：

    1.  以通常方式将集群升级到版本 8.0.22 或更高版本的 NDB Cluster 软件。

    1.  将 `config.ini` 文件中使用的地址更改为 IPv6 地址。

    1.  执行集群的系统重启。

    在 Linux 平台上运行 NDB 8.0.22 及更高版本时，已知问题是即使没有使用任何 IPv6 地址，操作系统内核也需要提供 IPv6 支持。此问题在 NDB 8.0.34 及更高版本中已修复（Bug #33324817，Bug #33870642）。

    如果您使用的是受影响的版本，并希望在系统上禁用对 IPv6 的支持（因为您不打算为 NDB Cluster 节点使用任何 IPv6 地址），请在系统启动后执行以下操作：

    ```sql
    $> sysctl -w net.ipv6.conf.all.disable_ipv6=1
    $> sysctl -w net.ipv6.conf.default.disable_ipv6=1
    ```

    （或者，您可以将相应行添加到 `/etc/sysctl.conf` 中。）在 NDB Cluster 8.0.34 及更高版本中，前述操作是不必要的，如果您不需要或不想要 IPv6 支持，可以简单地在 Linux 内核中禁用 IPv6 支持。

+   由于内部 `mysql.ndb_schema` 表的更改，如果您升级到 8.0.24 之前的 NDB 8.0 版本，则建议您使用 `--ndb-schema-dist-upgrade-allowed = 0` 来避免意外的中断（Bug #30876990，Bug #31016905）。

    另外，如果有可能在升级到新版本后恢复到以前的 NDB Cluster 版本，您必须从新版本启动所有**mysqld**进程，并使用 `--ndb-schema-dist-upgrade-allowed = 0` 来防止对`ndb_schema`表进行与旧版本不兼容的更改。有关如何执行此操作，请参阅回滚 NDB Cluster 8.0 升级。

+   `EncryptedFileSystem` 配置参数在 NDB 8.0.29 中引入，有时可能会导致撤消日志文件被加密，即使明确设置为 `0`，这可能会在使用 Disk Data 表并尝试升级或降级到 NDB 8.0.29 时出现问题。在这种情况下，您可以通过在滚动重启过程中执行数据节点的初始重启来解决问题。

+   如果您正在使用多线程数据节点（**ndbmtd**）和`ThreadConfig`配置参数，则在从先前版本升级到 NDB 8.0.30 或更高版本时，可能需要更改`config.ini`文件中为其设置的值。在从 NDB 8.0.23 或更早版本升级时，必须显式设置在先前版本中隐含的`main`、`rep`、`recv`或`ldm`线程的任何使用。在从 NDB 8.0.23 或更高版本升级到 NDB 8.0.30 或更高版本时，必须在`ThreadConfig`字符串中显式设置任何`recv`线程的使用。此外，为了避免在 NDB 8.0.30 或更高版本中使用`main`、`rep`或`ldm`线程，必须显式将给定类型的线程计数设置为`0`。 

    以下是一个示例。

    *NDB 8.0.22 及更早版本*：

    +   `config.ini`文件包含`ThreadConfig=ldm`。

    +   在这些版本的`NDB`中，这被解释为`ThreadConfig=main,ldm,recv,rep`。

    +   在`config.ini`中需要设置以下内容以匹配 NDB 8.0.30 或更高版本的效果：`ThreadConfig=main,ldm,recv,rep`。

    *NDB 8.0.23—8.0.29*：

    +   `config.ini`文件包含`ThreadConfig=ldm`。

    +   在这些版本的`NDB`中，这被解释为`ThreadConfig=ldm,recv`。

    +   在`config.ini`中需要设置以下内容以匹配 NDB 8.0.30 或更高版本的效果：`ThreadConfig=main={count=0},ldm,recv,rep={count=0}`。

    更多信息，请参阅`ThreadConfig`配置参数的描述。

支持从 NDB Cluster 的先前主要版本（7.4、7.5、7.6）升级到 NDB 8.0；请参阅升级到 NDB 8.0 支持的版本，以获取具体版本信息。此类升级受到以下问题的影响：

+   在 NDB 8.0 中，`log_bin`的默认值为 1，与之前版本有所改变。此外，从 NDB 8.0.16 开始，`ndb_log_bin`的默认值从 1 改为 0，这意味着必须显式设置`ndb_log_bin`为 1 才能在此版本及以后的版本中启用二进制日志记录。

+   在先前的发布系列中实现的 MySQL 服务器之间共享的分布式权限（请参见使用共享授权表进行分布式权限）在 NDB Cluster 8.0 中不受支持。在启动时，NDB 8.0 及更高版本提供的**mysqld**会检查是否存在使用`NDB`存储引擎的任何授权表；如果找到任何授权表，它将使用`InnoDB`创建本地副本（“影子表”）。对于每个连接到 NDB Cluster 的 MySQL 服务器都是如此。在所有作为 NDB Cluster SQL 节点的 MySQL 服务器上执行此操作后，可以使用 NDB Cluster 发行版提供的**ndb_drop_table**实用程序安全地删除`NDB`授权表，如下所示：

    ```sql
    ndb_drop_table -d mysql user db columns_priv tables_priv proxies_priv procs_priv
    ```

    可以保留`NDB`授权表，但它们不用于访问控制，实际上被忽略。

    有关 NDB 8.0 中使用的 MySQL 权限系统的更多信息，请参见第 25.6.13 节，“权限同步和 NDB_STORED_USER”，以及第 8.2.3 节，“授权表”。

+   在将任何 NDB 7.6 之前的版本升级到任何 NDB 8.0 版本时，需要使用`--initial`重新启动所有数据节点。这是因为 NDB 8.0 增加了对节点数量的支持。

尝试从 NDB 8.0 降级到以前的主要版本时遇到的问题可以在以下列表中找到：

+   由于 NDB 8.0 中对`NDB`表实现的额外元数据属性的使用方式发生了变化，导致与 NDB 7.6 和更早版本不兼容，因此在降级之前需要采取额外步骤来保留集群 SQL 节点的任何所需状态信息，然后在降级后进行恢复。

    更具体地说，支持`NDBCLUSTER`存储引擎的在线降级，即数据节点的降级，但 SQL 节点无法在线降级。这是因为给定 MySQL 8.0 或更早版本的 MySQL 服务器（**mysqld**）无法使用来自（稍后的）8.0 版本的系统文件，并且无法打开在稍后版本中创建的表。可能可以回滚最近从以前的 NDB 版本升级的集群；有关何时以及如何执行此操作，请参见回滚 NDB Cluster 8.0 升级。

    有关这些问题的更多信息，请参见 NDB 表额外元数据的更改；另请参见 第十六章，*MySQL 数据字典*。

+   在 NDB 8.0 中，二进制配置文件格式已经改进，以支持比以前版本更多的节点。新格式对于运行旧版本`NDB`的节点不可访问，尽管新的管理服务器可以检测到旧节点并使用适当的格式与它们通信。

    升级到 NDB 8.0 在这方面不应该有问题，但是旧的管理服务器无法读取更新的二进制配置文件格式，因此在从 NDB 8.0 降级到之前的主要版本时需要一些手动干预。在执行这样的降级时，需要在使用旧版`NDB`软件版本启动管理之前删除任何缓存的二进制配置文件，并确保管理服务器可以读取明文配置文件。或者，您可以使用 `--initial` 选项启动旧的管理服务器（同样，需要有`config.ini`可用）。如果集群使用多个管理服务器，则必须为每个管理服务器二进制文件执行这两个操作。

    与支持增加节点数量有关，由于在数据节点 LCP `Sysfile` 中实施的不兼容更改，因此在从 NDB 8.0 在线降级到先前的主要版本时，需要重新启动所有数据节点并使用 `--initial` 选项。

+   不支持将运行超过 48 个数据节点的集群或使用大于 48 的节点 ID 的数据节点在线降级到 NDB 8.0 之前的 NDB Cluster 版本。在这种情况下，需要减少数据节点的数量，更改所有数据节点的配置，使它们使用小于或等于 48 的节点 ID，或者根据需要同时执行这两个操作，以确保不超过旧的最大限制。

+   如果要从 NDB 8.0 降级到 NDB 7.5 或 NDB 7.4，则必须在集群配置文件中为 `IndexMemory` 设置一个显式值，如果尚未设置。这是因为 NDB 8.0 不使用此参数（在 NDB 7.6 中已删除），并默认将其设置为 0，而在 NDB 7.5 和 NDB 7.4 中，集群要求`IndexMemory`设置为非零值，否则集群将拒绝启动并显示来自管理服务器的无效配置...。

    从 NDB 8.0 降级到 NDB 7.6 *不* 需要设置`IndexMemory`。
