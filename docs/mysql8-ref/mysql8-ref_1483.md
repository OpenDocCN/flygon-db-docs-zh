> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-group-membership.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-group-membership.html)

#### 20.1.4.1 群组成员

在 MySQL Group Replication 中，一组服务器形成一个复制组。群组有一个以 UUID 形式的名称。群组是动态的，服务器可以随时离开（自愿或非自愿）和加入。每当服务器加入或离开时，群组会自动调整自身。

如果一个服务器加入群组，它会通过从现有服务器获取缺失状态来自动更新自身。如果一个服务器离开群组，例如因为维护而关闭，剩余的服务器会注意到它已经离开，并自动重新配置群组。

Group Replication 具有一个群组成员服务，定义了哪些服务器在线并参与群组。在线服务器列表被称为*视图*。群组中的每个服务器在给定时间点都对哪些服务器是活跃参与群组的成员有一致的视图。

群组成员不仅必须就事务提交达成一致意见，还必须就当前视图达成一致。如果现有成员同意新服务器应该成为群组的一部分，群组将重新配置以将该服务器整合进去，从而触发视图更改。如果一个服务器离开群组，无论是自愿还是非自愿，群组会动态重新排列其配置，并触发视图更改。

当一个成员自愿离开群组时，首先启动动态群重新配置，在此期间，所有成员必须就没有离开的服务器达成新的共识。然而，如果一个成员非自愿离开群组，例如因为意外停止或网络连接中断，它无法启动重新配置。在这种情况下，Group Replication 的故障检测机制在短时间内识别出成员已经离开，并提出了一个没有失败成员的群组重新配置。与自愿离开的成员一样，重新配置需要群组中大多数服务器的同意。然而，如果群组无法达成一致意见，例如因为分区导致没有大多数服务器在线，系统无法动态更改配置，并阻止分裂脑的情况发生。这种情况需要管理员干预。

一个成员可能会短暂离线，然后在故障检测机制检测到其故障之前，尝试重新加入组，而在组被重新配置以移除该成员之前。在这种情况下，重新加入的成员会忘记其先前的状态，但如果其他成员向其发送旨在其崩溃前状态的消息，这可能会导致问题，包括可能的数据不一致性。如果处于这种情况的成员参与 XCom 的共识协议，它有可能导致 XCom 在同一共识轮中传递不同的值，因为在故障前后做出不同的决定。

为了应对这种可能性，在 MySQL 5.7.22 版本和 MySQL 8.0 版本中，Group Replication 会检查这样一种情况：当同一台服务器的新实例尝试加入组时，而其旧实例（具有相同的地址和端口号）仍然被列为成员。新实例将被阻止加入组，直到旧实例通过重新配置被移除。请注意，如果`group_replication_member_expel_timeout`系统变量添加了等待时间，以允许成员在被驱逐之前重新连接到组，那么受到怀疑的成员在怀疑超时之前重新连接到组，可以作为其当前实例再次活跃在组中。当成员超过驱逐超时并被驱逐出组，或者当服务器上的 Group Replication 被`STOP GROUP_REPLICATION`语句或服务器故障停止时，它必须作为新实例重新加入。
