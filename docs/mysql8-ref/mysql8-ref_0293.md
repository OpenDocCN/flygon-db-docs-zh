> 原文：[`dev.mysql.com/doc/refman/8.0/en/thread-pool-operation.html`](https://dev.mysql.com/doc/refman/8.0/en/thread-pool-operation.html)

#### 7.6.3.3 线程池操作

线程池由多个线程组组成，每个组管理一组客户端连接。随着连接的建立，线程池以轮询方式将它们分配给线程组。

线程池公开了可用于配置其操作的系统变量：

+   `thread_pool_algorithm`: 用于调度的并发算法。

+   `thread_pool_dedicated_listeners`: 在每个线程组中专门指定一个监听线程，以监听分配给该组的连接的传入语句。

+   `thread_pool_high_priority_connection`: 如何为会话安排语句执行。

+   `thread_pool_max_active_query_threads`: 每组允许的活动线程数。

+   `thread_pool_max_transactions_limit`: 线程池插件允许的最大事务数。

+   `thread_pool_max_unused_threads`: 允许的休眠线程数。

+   `thread_pool_prio_kickup_timer`: 在低优先级队列中等待执行的语句移动到高优先级队列之前的时间。

+   `thread_pool_query_threads_per_group`: 在线程组中允许的查询线程数（默认为单个查询线程）。如果由于长时间运行的事务导致响应时间变慢，考虑增加该值。

+   `thread_pool_size`: 线程池中线程组的数量。这是控制线程池性能最重要的参数。

+   `thread_pool_stall_limit`: 执行语句被视为停滞之前的时间。

+   `thread_pool_transaction_delay`: 开始新事务之前的延迟时间。

要配置线程组的数量，请使用`thread_pool_size`系统变量。默认组数为 16。有关设置此变量的指导，请参见第 7.6.3.4 节“线程池调优”。

每个组的最大线程数为 4096（在某些系统上为 4095，其中一个线程在内部使用）。

线程池将连接和线程分开，因此连接和执行来自这些连接的语句的线程之间没有固定关系。这与默认的线程处理模型不同，后者将一个线程与一个连接关联起来，以便给定线程执行来自其连接的所有语句。

默认情况下，线程池尝试确保每个组中最多有一个线程在任何时候执行，但有时允许更多线程暂时执行以获得最佳性能：

+   每个线程组都有一个监听线程，用于监听分配给该组的连接的传入语句。当语句到达时，线程组要么立即开始执行它，要么将其排队以供稍后执行：

    +   如果接收到的语句是唯一的，并且没有排队或当前正在执行的语句，则会立即执行。

        从 MySQL 8.0.31 开始，可以通过配置`thread_pool_transaction_delay`来延迟立即执行，这对事务有限制作用。有关更多信息，请参考后续讨论中对该变量的描述。

    +   如果语句由于同时排队或执行的语句而无法立即开始执行，则会发生排队。

+   `thread_pool_transaction_delay` 变量指定了以毫秒为单位的事务延迟。工作线程在执行新事务之前会休眠指定的时间段。

    在并行事务影响其他操作的性能的情况下，可以使用事务延迟。例如，如果并行事务影响索引创建或在线缓冲池调整操作，可以配置事务延迟以减少资源争用。延迟对事务有限制作用。

    `thread_pool_transaction_delay` 设置不影响从特权连接（分配给`Admin`线程组的连接）发出的查询。这些查询不受配置的事务延迟影响。

+   如果立即执行发生，则监听线程执行它。（这意味着暂时没有线程在组中监听。）如果语句很快完成，执行线程将返回到监听语句。否则，线程池会认为语句被阻塞，并启动另一个线程作为监听线程（必要时创建）。为了确保没有线程组被阻塞在被阻塞的语句上，线程池有一个定期监视线程组状态的后台线程。

    通过使用监听线程执行可以立即开始的语句，如果语句很快完成，则无需创建额外的线程。这确保在并发线程数量较低的情况下实现最有效的执行。

    当线程池插件启动时，它会为每个组创建一个线程（监听线程），以及后台线程。根据需要创建额外的线程来执行语句。

+   `thread_pool_stall_limit`系统变量的值确定了前一项中“快速完成”的含义。在被视为停顿之前的默认时间为 60ms，但可以设置为最大 6 秒。此参数可配置，以便您能够在服务器工作负载中找到适当的平衡。较短的等待值允许线程更快地启动。较短的值也更有利于避免死锁情况。较长的等待值对包含长期运行语句的工作负载很有用，以避免在当前语句执行时启动过多的新语句。

+   如果`thread_pool_max_active_query_threads`为 0，则默认算法适用于确定每个组的活动线程的最大数量，就像前面描述的那样。默认算法考虑了停顿的线程，并可能暂时允许更多的活动线程。如果`thread_pool_max_active_query_threads`大于 0，则它会限制每个组的活动线程数量。

+   线程池专注于限制并发短期运行的语句数量。在执行语句达到停顿时间之前，它会阻止其他语句开始执行。如果语句执行超过停顿时间，它将被允许继续执行，但不再阻止其他语句开始。通过这种方式，线程池试图确保在每个线程组中永远不会有多个短期运行的语句，尽管可能存在多个长期运行的语句。让长期运行的语句阻止其他语句执行是不可取的，因为可能需要等待的时间没有限制。例如，在复制源服务器上，负责向副本发送二进制日志事件的线程实际上会永远运行。

+   如果语句遇到磁盘 I/O 操作或用户级锁（行锁或表锁），则该语句将被阻塞。阻塞将导致线程组变为未使用状态，因此会有回调到线程池，以确保线程池可以立即在该组中启动一个新线程来执行另一个语句。当阻塞的线程返回时，线程池允许其立即重新启动。

+   有两个队列，一个高优先级队列和一个低优先级队列。事务中的第一条语句进入低优先级队列。如果事务正在进行中（其语句已开始执行），则事务的任何后续语句进入高优先级队列，否则进入低优先级队列。通过启用`thread_pool_high_priority_connection`系统变量，可以影响队列分配，导致会话的所有排队语句进入高优先级队列。

    针对非事务性存储引擎的语句，或者如果启用了`autocommit`的事务性引擎，被视为低优先级语句，因为在这种情况下每条语句都是一个事务。因此，对于`InnoDB`和`MyISAM`表的语句混合，线程池优先处理`InnoDB`表的语句而不是`MyISAM`表的语句，除非启用了`autocommit`。启用`autocommit`后，所有语句都具有低优先级。

+   当线程组选择一个排队的语句进行执行时，首先查看高优先级队列，然后查看低优先级队列。如果找到一条语句，则将其从队列中移除并开始执行。

+   如果一条语句在低优先级队列中停留时间过长，线程池将移动到高优先级队列。`thread_pool_prio_kickup_timer`系统变量的值控制移动前的时间。对于每个线程组，每 10ms（每秒 100 次）最多将一条语句从低优先级队列移动到高优先级队列。

+   线程池重复使用最活跃的线程以更好地利用 CPU 缓存。这是一个对性能有很大影响的小调整。

+   当一个线程执行来自用户连接的语句时，性能模式仪表化将线程活动归因于用户连接。否则，性能模式将活动归因于线程池。

以下是线程组可能启动多个线程执行语句的条件示例：

+   一个线程开始执行一条语句，但运行时间足够长以被视为停滞。线程组允许另一个线程开始执行另一条语句，即使第一个线程仍在执行。

+   一个线程开始执行一条语句，然后变为阻塞状态并将此情况报告给线程池。线程组允许另一个线程开始执行另一条语句。

+   一个线程开始执行一个语句，变得阻塞，但没有报告它被阻塞，因为阻塞不发生在已经使用线程池回调进行仪器化的代码中。在这种情况下，该线程对线程组来说仍在运行。如果阻塞持续时间足够长，使语句被视为停滞不前，组允许另一个线程开始执行另一个语句。

线程池设计为可扩展到越来越多的连接。它还设计为避免由于限制活动执行语句的数量而可能产生的死锁。重要的是，不向线程池报告的线程不应该阻止其他语句的执行，从而导致线程池陷入死锁。以下是此类语句的示例：

+   长时间运行的语句。这些将导致仅有少数语句使用所有资源，它们可能阻止其他所有人访问服务器。

+   读取二进制日志并将其发送到副本的二进制日志转储线程。这是一种长时间运行的“语句”，运行时间很长，不应该阻止其他语句的执行。

+   在行锁、表锁、休眠或任何其他未被 MySQL 服务器或存储引擎报告给线程池的阻塞活动上被阻塞的语句。

在每种情况下，为了防止死锁，当语句不能快速完成时，将该语句移至停滞类别，以便线程组可以允许另一个语句开始执行。通过这种设计，当线程执行或长时间阻塞时，线程池将该线程移至停滞类别，并在语句的其余执行过程中，不会阻止其他语句的执行。

可发生的线程最大数量是`max_connections`和`thread_pool_size`的总和。这可能发生在所有连接都处于执行模式且每个组创建了一个额外线程来监听更多语句的情况下。这不一定经常发生，但在理论上是可能的。

##### 特权连接

当达到`thread_pool_max_transactions_limit`定义的限制时，新连接似乎会挂起，直到一个或多个现有事务完成。当尝试在现有连接上启动新事务时也会发生相同情况。如果现有连接被阻塞或长时间运行，访问服务器的唯一方法是使用特权连接。

要建立特权连接，发起连接的用户必须具有`TP_CONNECTION_ADMIN`权限。特权连接会忽略由`thread_pool_max_transactions_limit`定义的限制，并允许连接到服务器以增加限制、移除限制或终止正在运行的事务。`TP_CONNECTION_ADMIN`权限必须显式授予，不会默认授予任何用户。

特权连接可以执行语句和启动事务，并分配给指定为`Admin`线程组的线程组。

当查询`performance_schema.tp_thread_group_stats`表时，该表报告每个线程组的统计信息，`Admin`线程组的统计信息报告在结果集的最后一行。例如，如果`SELECT * FROM performance_schema.tp_thread_group_stats\G`返回 17 行（每个线程组一行），`Admin`线程组的统计信息将报告在第 17 行。
