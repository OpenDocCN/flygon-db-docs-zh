> 原文：[`dev.mysql.com/doc/refman/8.0/en/crashing.html`](https://dev.mysql.com/doc/refman/8.0/en/crashing.html)

#### B.3.3.3 MySQL 持续崩溃时该怎么办

每个 MySQL 版本在发布之前都会在许多平台上进行测试。这并不意味着 MySQL 没有错误，但如果有错误，它们应该很少，并且很难找到。如果您遇到问题，尝试找出究竟是什么导致系统崩溃，因为这样您就有更好的机会快速解决问题。

首先，您应该尝试找出问题是**mysqld**服务器崩溃了还是您的问题与客户端有关。您可以通过执行**mysqladmin version**来检查您的**mysqld**服务器已经运行了多长时间。如果**mysqld**已经崩溃并重新启动，您可以通过查看服务器的错误日志找到原因。请参考 Section 7.4.2, “The Error Log”。

在某些系统上，您可以在错误日志中找到**mysqld**死机的堆栈跟踪。请注意，错误日志中写入的变量值可能并不总是 100%正确。

如果在`InnoDB`恢复期间，**mysqld**在启动时失败，请参考 Section 17.21.2, “Troubleshooting Recovery Failures”。

许多意外的服务器退出是由于损坏的数据文件或索引文件引起的。每个 SQL 语句执行后，MySQL 都会使用`write()`系统调用更新磁盘上的文件，并在通知客户端结果之前执行此操作。（如果启用了`delay_key_write`系统变量，则不适用此规则，此时只会写入数据文件而不写入索引文件。）这意味着即使**mysqld**崩溃，数据文件内容也是安全的，因��操作系统会确保未刷新的数据被写入磁盘。您可以通过使用`--flush`选项启动**mysqld**来强制 MySQL 在每个 SQL 语句后将所有内容刷新到磁盘。

前面的意思是通常情况下，除非发生以下情况之一，否则您不应该遇到损坏的表：

+   MySQL 服务器或服务器主机在更新过程中被终止。

+   您发现了一个导致**mysqld**在更新过程中崩溃的错误。

+   一些外部程序在没有正确锁定表的情况下同时操作数据文件或索引文件，导致**mysqld**崩溃。

+   您正在运行许多使用相同数据目录的 **mysqld** 服务器，但系统不支持良好的文件系统锁（通常由 `lockd` 锁管理器处理），或者您正在运行禁用外部锁定的多个服务器。

+   您有一个包含非常损坏数据的崩溃数据文件或索引文件，这些数据混淆了 **mysqld**。

+   您在数据存储代码中发现了一个错误。这不太可能，但至少有可能。在这种情况下，您可以尝试通过在修复后的表的副本上使用 `ALTER TABLE` 来将存储引擎更改为另一个引擎。

因为很难知道为什么会崩溃，首先尝试检查对其他人有效的事物是否导致您意外退出。尝试以下事项：

+   使用 **mysqladmin shutdown** 停止 **mysqld** 服务器，从数据目录运行 **myisamchk --silent --force */*.MYI** 检查所有 `MyISAM` 表，并重新启动 **mysqld**。这确保您从干净状态运行。参见 第七章，“MySQL 服务器管理”。

+   启动 **mysqld** 并启用常规查询日志（参见 第 7.4.3 节，“常规查询日志”）。然后尝试从写入日志的信息中确定是否有某个特定查询导致服务器崩溃。大约 95% 的所有错误都与特定查询有关。通常，这是在服务器重新启动之前日志文件中的最后一个查询之一。参见 第 7.4.3 节，“常规查询日志”。如果您可以重复使用特定查询使 MySQL 崩溃，即使在发出查询之前已检查了所有表，那么您已经隔离出错误，并应为其提交错误报告。参见 第 1.5 节，“如何报告错误或问题”。

+   尝试创建一个我们可以用来重复出现问题的测试用例。参见 第 7.9 节，“调试 MySQL”。

+   尝试运行 `fork_big.pl` 脚本。（它位于源分发的 `tests` 目录中。）

+   配置 MySQL 以进行调试，可以在出现问题时更轻松地收集可能的错误信息。重新使用 `-DWITH_DEBUG=1` 选项对 **CMake** 进行 MySQL 重新配置，然后重新编译。参见 第 7.9 节，“调试 MySQL”。

+   确保已为您的操作系统应用了最新的补丁。

+   使用`--skip-external-locking`选项到**mysqld**。在某些系统上，`lockd`锁管理器无法正常工作；`--skip-external-locking`选项告诉**mysqld**不要使用外部锁定。（这意味着您不能在同一数据目录上运行两个**mysqld**服务器，并且如果使用**myisamchk**，必须小心。尽管如此，作为测试尝试该选项可能是有益的。）

+   如果**mysqld**看起来正在运行但没有响应，请尝试**mysqladmin -u root processlist**。有时候**mysqld**虽然看起来没有响应，但并没有挂起。问题可能是所有连接都在使用中，或者可能存在一些内部锁问题。在这些情况下，**mysqladmin -u root processlist**通常能够建立连接，并提供有关当前连接数及其状态的有用信息。

+   在单独的窗口中运行命令**mysqladmin -i 5 status**或**mysqladmin -i 5 -r status**以在运行其他查询时生成统计信息。

+   尝试以下操作：

    1.  从**gdb**（或其他调试器）启动**mysqld**。参见第 7.9 节，“调试 MySQL”。

    1.  运行你的测试脚本。

    1.  打印三个最低级别的回溯和本地变量。在**gdb**中，当**mysqld**在**gdb**内部崩溃时，可以使用以下命令执行此操作：

        ```sql
        backtrace
        info local
        up
        info local
        up
        info local
        ```

        使用**gdb**，你也可以通过`info threads`查看存在哪些线程，并通过`thread *`N`*`切换到特定线程，其中*`N`*是线程 ID。

+   尝试使用 Perl 脚本模拟你的应用程序，强制 MySQL 退出或表现异常。

+   发送一个普通的错误报告。参见第 1.5 节，“如何报告错误或问题”。比平常更详细。因为 MySQL 适用于许多人，崩溃可能是由于仅存在于您的计算机上的某些东西（例如，与您特定的系统库相关的错误）。

+   如果您在使用只包含`VARCHAR`列（而不是`BLOB`或`TEXT`列）的动态长度行的表时遇到问题，可以尝试将所有`VARCHAR`更改为`CHAR`，并使用`ALTER TABLE`。这将强制 MySQL 使用固定大小的行。固定大小的行会占用一些额外空间，但更容忍数据损坏。

    当前的动态行代码已经使用了几年，几乎没有问题，但动态长度行本质上更容易出错，因此尝试这种策略看看是否有帮助可能是个好主意。

+   在诊断问题时，请考虑硬件故障的可能性。有缺陷的硬件可能是数据损坏的原因。在故障排除硬件时，特别注意内存和磁盘子系统。
