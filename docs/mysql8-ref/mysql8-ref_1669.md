> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-mgm-definition.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-mgm-definition.html)

#### 25.4.3.5 定义 NDB 集群管理服务器

`[ndb_mgmd]` 部分用于配置管理服务器的行为。如果使用多个管理服务器，您可以在 `[ndb_mgmd default]` 部分中指定所有服务器共同的参数。`[mgm]` 和 `[mgm default]` 是这些的旧别名，为了向后兼容而支持。

以下列表中的所有参数都是可选的，如果省略，则假定其默认值。

注意

如果既没有 `ExecuteOnComputer` 参数也没有 `HostName` 参数，则默认值为 `localhost`。

+   `Id`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认值 | [...] |
    | 范围 | 1 - 255 |
    | 重启类型 | **初始系统重启：** 需要完全关闭集群，从备份中擦除和恢复集群文件系统，然后重新启动集群。（NDB 8.0.13） |

    集群中的每个节点都有一个唯一的标识。对于管理节点，这由范围在 1 到 255 的整数值表示。此 ID 被所有内部集群消息用于寻址节点，因此对于每个 NDB 集群节点，无论节点类型如何，此 ID 必须是唯一的。

    注意

    数据节点 ID 必须小于 145。如果计划部署大量数据节点，最好将管理节点（和 API 节点）的节点 ID 限制在大于 144 的值。

    用于标识管理节点的 `Id` 参数的使用已被弃用，推荐使用 `NodeId`。尽管 `Id` 仍然为向后兼容而支持，但现在会生成警告，并且可能在将来的 NDB Cluster 版本中被移除。

+   `NodeId`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认值 | [...] |
    | 范围 | 1 - 255 |
    | 重启类型 | **初始系统重启：** 需要完全关闭集群，从备份中擦除和恢复集群文件系统，然后重新启动集群。（NDB 8.0.13） |

    集群中的每个节点都有一个唯一的标识。对于管理节点，这由范围在 1 到 255 的整数值表示。此 ID 被所有内部集群消息用于寻址节点，因此对于每个 NDB 集群节点，无论节点类型如何，此 ID 必须是唯一的。

    注意

    数据节点 ID 必须小于 145。如果计划部署大量数据节点，最好将管理节点（和 API 节点）的节点 ID 限制在大于 144 的值。

    `NodeId` 是标识管理节点时首选的参数名称。尽管旧的 `Id` 仍然受支持以实现向后兼容性，但现已弃用，并在使用时生成警告；它也可能在未来的 NDB Cluster 版本中被移除。

+   `ExecuteOnComputer`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 名称 |
    | 默认值 | [...] |
    | 范围 | ... |
    | 已弃用 | 是（在 NDB 7.5 中） |
    | 重启类型 | **系统重启：**需要完全关闭和重启集群。（NDB 8.0.13） |

    这指的是在 `config.ini` 文件的 `[computer]` 部分中定义的计算机之一设置的 `Id`。

    重要

    此参数已弃用，并可能在未来的版本中被移除。请改用 `HostName` 参数。

+   `PortNumber`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认值 | 1186 |
    | 范围 | 0 - 64K |
    | 重启类型 | **系统重启：**需要完全关闭和重启集群。（NDB 8.0.13） |

    这是管理服务器监听配置请求和管理命令的端口号。

+   仅当显式请求时才能提供此节点的节点 ID。请求“任何”节点 ID 的管理服务器无法使用此节点。当在同一主机上运行多个管理服务器时，且 `HostName` 不足以区分进程时，可以使用此参数。用于测试目的。

+   `HostName`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 名称或 IP 地址 |
    | 默认值 | [...] |
    | 范围 | ... |
    | 重启类型 | **节点重启：**需要进行集群的滚动重启。（NDB 8.0.13） |

    指定此参数定义管理节点所在计算机的主机名。使用 `HostName` 来指定除 `localhost` 外的主机名。

+   `LocationDomainId`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 整数 |
    | 默认值 | 0 |
    | 范围 | 0 - 16 |
    | 重启类型 | **系统重启：**需要完全关闭和重启集群。（NDB 8.0.13） |

    将管理节点分配给云中特定的[可用域](https://docs.us-phoenix-1.oraclecloud.com/Content/General/Concepts/regions.htm)（也称为可用区）。通过告知 `NDB` 哪些节点位于哪些可用域中，可以在云环境中通过以下方式提高性能：

    +   如果请求的数据在同一节点上找不到，则读取可以指向同一可用域中的另一个节点。

    +   不同可用性域中节点之间的通信保证使用 `NDB` 传输器的广域网支持，无需进一步手动干预。

    +   传输器的组号可以基于使用的可用性域，以便 SQL 和其他 API 节点尽可能与同一可用性域中的本地数据节点通信。

    +   仲裁者可以从不存在数据节点的可用性域中选择，或者如果找不到这样的可用性域，则可以从第三个可用性域中选择。

    `LocationDomainId` 接受一个介于 0 和 16 之间的整数值，其中 0 是默认值；使用 0 与未设置参数相同。

+   `LogDestination`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | {CONSOLE&#124;SYSLOG&#124;FILE} |
    | 默认值 | FILE: 文件名=ndb_nodeid_cluster.log, 最大大小=1000000, 最大文件数=6 |
    | 范围 | ... |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    此参数指定集群日志信息发送位置。在这方面有三个选项——`CONSOLE`、`SYSLOG` 和 `FILE`——其中 `FILE` 是默认值：

    +   `CONSOLE` 将日志输出到 `stdout`：

        ```sql
        CONSOLE
        ```

    +   `SYSLOG` 将日志发送到 `syslog` 设施，可能的值为 `auth`、`authpriv`、`cron`、`daemon`、`ftp`、`kern`、`lpr`、`mail`、`news`、`syslog`、`user`、`uucp`、`local0`、`local1`、`local2`、`local3`、`local4`、`local5`、`local6` 或 `local7`。

        注意

        并非每个设施一定被每个操作系统支持。

        ```sql
        SYSLOG:facility=syslog
        ```

    +   `FILE` 将集群日志输出到同一台机器上的常规文件。可以指定以下值：

        +   `文件名`：日志文件的名称。

            在这种情况下使用的默认日志文件名是 `ndb_*`nodeid`*_cluster.log`。

        +   `maxsize`：文件在日志滚动到新文件之前可以增长到的最大大小（以字节为单位）。发生这种情况时，旧日志文件的名称将通过在文件名后附加 *`.N`* 来重命名，其中 *`N`* 是尚未使用的下一个数字。

        +   `maxfiles`：日志文件的最大数量。

        ```sql
        FILE:filename=cluster.log,maxsize=1000000,maxfiles=6
        ```

        `FILE` 参数的默认值是 `FILE:文件名=ndb_*`node_id`*_cluster.log,最大大小=1000000,最大文件数=6`，其中 *`node_id`* 是节点的 ID。

    可以按照以下示例指定多个日志目的地，用分号分隔：

    ```sql
    CONSOLE;SYSLOG:facility=local0;FILE:filename=/var/log/mgmd
    ```

+   `ArbitrationRank`

    | 版本（或更高版本） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 0-2 |
    | 默认值 | 1 |
    | 范围 | 0 - 2 |
    | 重启类型 | **节点重启：** 需要进行滚动重启。 (NDB 8.0.13) |

    此参数用于定义哪些节点可以充当仲裁者。只有管理节点和 SQL 节点可以充当仲裁者。`ArbitrationRank` 可以取以下值之一：

    +   `0`：节点永远不会被用作仲裁者。

    +   `1`: 节点具有高优先级；也就是说，它优先作为仲裁者而不是低优先级节点。

    +   `2`: 表示低优先级节点，仅在没有更高优先级节点可用时用作仲裁者。

    通常，管理服务器应通过将其`ArbitrationRank`设置为 1（管理节点的默认值）以及将所有 SQL 节点的设置为 0（SQL 节点的默认值）来配置为仲裁者。

    您可以通过在所有管理和 SQL 节点上将`ArbitrationRank`设置为 0，或者在`config.ini`全局配置文件的`[ndbd default]`部分设置`Arbitration`参数来完全禁用仲裁。设置`Arbitration`会忽略`ArbitrationRank`的任何设置。

+   `ArbitrationDelay`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 毫秒 |
    | 默认值 | 0 |
    | 范围 | 0 - 4294967039 (0xFFFFFEFF) |
    | 重启类型 | **节点重启:** 需要进行滚动重启。 (NDB 8.0.13) |

    一个整数值，导致管理服务器对仲裁请求的响应延迟该毫秒数。默认值为 0；通常不需要更改。

+   `DataDir`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 路径 |
    | 默认值 | . |
    | 范围 | ... |
    | 重启类型 | **节点重启:** 需要进行滚动重启。 (NDB 8.0.13) |

    指定管理服务器输出文件的目录。这些文件包括集群日志文件、进程输出文件和守护进程的进程 ID（PID）文件。（对于日志文件，此位置可以通过为`LogDestination`设置`FILE`参数来覆盖，如本节前面讨论的。）

    默认值为**ndb_mgmd**所在目录的参数。

+   `PortNumberStats`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 无符号 |
    | 默认值 | [...] |
    | 范围 | 0 - 64K |
    | 重启类型 | **节点重启:** 需要进行滚动重启。 (NDB 8.0.13) |

    此参数指定用于从 NDB 集群管理服务器获取统计信息的端口号。它没有默认值。

+   `Wan`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 布尔值 |
    | 默认值 | false |
    | 范围 | true, false |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    使用 WAN TCP 设置为默认。

+   `HeartbeatThreadPriority`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 字符串 |
    | 默认值 | [...] |
    | 范围 | ... |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    设置管理和 API 节点的心跳线程的调度策略和优先级。

    设置此参数的语法如下：

    ```sql
    HeartbeatThreadPriority = *policy*[, *priority*]

    *policy*:
      {FIFO | RR}
    ```

    在设置此参数时，必须指定一个策略。这是`FIFO`（先进先出）或`RR`（轮询）之一。策略值后面可以选择性地跟着优先级（一个整数）。

+   `ExtraSendBufferMemory`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 字节 |
    | 默认值 | 0 |
    | 范围 | 0 - 32G |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    此参数指定要分配的传输器发送缓冲区内存的总量，除了使用`TotalSendBufferMemory`、`SendBufferMemory`或两者设置的内存外。

+   `TotalSendBufferMemory`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 字节 |
    | 默认值 | 0 |
    | 范围 | 256K - 4294967039 (0xFFFFFEFF) |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    此参数用于确定在此节点上为所有配置的传输器之间共享的发送缓冲区内存分配的总量。

    如果设置了此参数，其最小允许值为 256KB；0 表示未设置该参数。有关更详细信息，请参见第 25.4.3.14 节，“配置 NDB 集群发送缓冲区参数”。

+   `HeartbeatIntervalMgmdMgmd`

    | 版本（或更高） | NDB 8.0.13 |
    | --- | --- |
    | 类型或单位 | 毫秒 |
    | 默认值 | 1500 |
    | 范围 | 100 - 4294967039 (0xFFFFFEFF) |
    | 重启类型 | **节点重启：** 需要对集群进行滚动重启。（NDB 8.0.13） |

    指定心跳消息之间的间隔，用于确定另一个管理节点是否与此节点联系。管理节点在这些间隔之后等待 3 次才宣布连接中断；因此，默认设置为 1500 毫秒导致管理节点在超时前等待大约 1600 毫秒。

注意

在管理节点配置更改后，需要对集群执行滚动重启，以使新配置生效。

在运行中向 NDB 集群添加新的管理服务器后，还需要在修改任何现有的`config.ini`文件后对所有集群节点执行滚动重启。有关使用多个管理节点时出现的问题的更多信息，请参见第 25.2.7.10 节，“与多个 NDB 集群节点相关的限制”。

**重新启动类型。** 此部分参数描述中使用的重新启动类型信息如下表所示：

**表 25.9 NDB 集群重新启动类型**

| 符号 | 重新启动类型 | 描述 |
| --- | --- | --- |
| N | 节点 | 可以使用滚动重启更新参数（请参见第 25.6.5 节，“执行 NDB 集群的滚动重启”） |
| S | 系统 | 所有集群节点必须完全关闭，然后重新启动，以实现对该参数的更改 |
| I | 初始 | 数据节点必须使用`--initial`选项重新启动 |