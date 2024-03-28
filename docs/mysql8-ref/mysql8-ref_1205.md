> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-flushing.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-buffer-pool-flushing.html)

#### 17.8.3.5 配置缓冲池刷新

`InnoDB`在后台执行某些任务，包括从缓冲池中刷新脏页。脏页是已经被修改但尚未写入磁盘上的数据文件的页面。

在 MySQL 8.0 中，缓冲池刷新由页面清理线程执行。页面清理线程的数量由`innodb_page_cleaners`变量控制，默认值为 4。但是，如果页面清理线程的数量超过缓冲池实例的数量，`innodb_page_cleaners`会自动设置为与`innodb_buffer_pool_instances`相同的值。

当脏页的百分比达到由`innodb_max_dirty_pages_pct_lwm`变量定义的低水位值时，会启动缓冲池刷新。默认的低水位标记是缓冲池页面的 10%。`innodb_max_dirty_pages_pct_lwm`值为 0 会禁用这种早期刷新行为。

`innodb_max_dirty_pages_pct_lwm`阈值的目的是控制缓冲池中的脏页百分比，并防止脏页数量达到由`innodb_max_dirty_pages_pct`变量定义的阈值，其默认值为 90。如果缓冲池中脏页的百分比达到`innodb_max_dirty_pages_pct`阈值，`InnoDB`会积极刷新缓冲池页面。

在配置`innodb_max_dirty_pages_pct_lwm`时，该值应始终低于`innodb_max_dirty_pages_pct`值。

其他变量允许对缓冲池刷新行为进行微调：

+   `innodb_flush_neighbors`变量定义了是否刷新缓冲池中的页面也会刷新同一范围内的其他脏页。

    +   默认设置为 0 会禁用`innodb_flush_neighbors`。同一范围内的脏页不会被刷新。这个设置适用于非旋转存储（SSD）设备，其中寻道时间不是一个重要因素。

    +   设置为 1 会刷新同一范围内的连续脏页。

    +   设置为 2 会刷新同一范围内的脏页。

    当表数据存储在传统的 HDD 存储设备上时，一次刷新相邻页面可以减少 I/O 开销（主要是磁盘寻道操作）相对于在不同时间刷新单个页面。对于存储在 SSD 上的表数据，寻道时间不是一个重要因素，您可以禁用此设置以分散写操作。

+   `innodb_lru_scan_depth`变量指定了每个缓冲池实例中，页面清理线程扫描缓冲池 LRU 列表以查找要刷新的脏页的深度。这是一个后台操作，由页面清理线程每秒执行一次。

    一般来说，比默认值小的设置对大多数工作负载都是合适的。如果值显著高于必要值，可能会影响性能。只有在典型工作负载下有多余的 I/O 容量时才考虑增加该值。相反，如果写入密集型工作负载使您的 I/O 容量饱和，减少该值，特别是在大缓冲池的情况下。

    在调整`innodb_lru_scan_depth`时，从一个较低值开始，并将设置向上配置，目的是很少看到零空闲页面。此外，在更改缓冲池实例数量时，考虑调整`innodb_lru_scan_depth`，因为`innodb_lru_scan_depth` * `innodb_buffer_pool_instances`定义了每秒页清理线程执行的工作量。

`innodb_flush_neighbors`和`innodb_lru_scan_depth`变量主要用于写入密集型工作负载。在大量 DML 活动中，如果刷新不够积极，刷新可能会滞后，或者如果刷新过于积极，磁盘写入可能会饱和 I/O 容量。理想的设置取决于您的工作负载、数据访问模式和存储配置（例如，数据存储在 HDD 还是 SSD 设备上）。

##### 自适应刷新

`InnoDB`使用自适应刷新算法动态调整刷新速率，根据重做日志生成的速度和当前刷新速率。其目的是通过确保刷新活动跟上当前工作负载的步伐来平滑整体性能。自动调整刷新速率有助于避免由于缓冲池刷新导致的 I/O 活动突然下降，从而影响了用于普通读写活动的 I/O 容量。

尖锐的检查点通常与产生大量重做条目的写入密集型工作负载相关联，例如可能导致吞吐量突然变化。尖锐的检查点发生在`InnoDB`想要重用日志文件的一部分时。在这样做之前，必须刷新该日志文件部分中具有重做条目的所有脏页。如果日志文件变满，将发生尖锐的检查点，导致暂时的吞吐量降低。即使未达到`innodb_max_dirty_pages_pct`阈值，也可能发生这种情况。

自适应刷新算法通过跟踪缓冲池中脏页的数量以及重做日志记录生成的速率来避免这种情况。根据这些信息，它决定每秒从缓冲池中刷新多少脏页，从而使其能够管理工作负载的突然变化。

`innodb_adaptive_flushing_lwm`变量定义了重做日志容量的低水位标记。当超过该阈值时，即使`innodb_adaptive_flushing`变量被禁用，自适应刷新也会被启用。

内部基准测试显示，该算法不仅可以随时间保持吞吐量，还可以显著提高总体吞吐量。然而，自适应刷新可能会显著影响工作负载的 I/O 模式，并且在所有情况下可能并不适用。当重做日志面临填满的危险时，它会带来最大的好处。如果自适应刷新不适合您的工作负载特征，您可以禁用它。自适应刷新由`innodb_adaptive_flushing`变量控制，默认情况下启用。

`innodb_flushing_avg_loops`定义了`InnoDB`保持先前计算的刷新状态快照的迭代次数，控制自适应刷新对前台工作负载变化的快速响应。较高的`innodb_flushing_avg_loops`值意味着`InnoDB`会保持先前计算的快照时间更长，因此自适应刷新的响应速度更慢。设置较高的值时，重要的是确保重做日志利用率不会达到 75%（异步刷新开始的硬编码限制），并且`innodb_max_dirty_pages_pct`阈值保持脏页数量适合工作负载的水平。

对于工作负载一致、日志文件大小较大（`innodb_log_file_size`）且不会达到 75% 日志空间利用率的系统，应该使用较高的 `innodb_flushing_avg_loops` 值，以尽可能平滑地进行刷新。对于负载波动极大或日志文件提供的空间不多的系统，较小的值可以使刷新紧密跟踪工作负载变化，并有助于避免达到 75% 日志空间利用率。

请注意，如果刷新落后，缓冲池刷新速率可能超过 `InnoDB` 可用的 I/O 容量，即由 `innodb_io_capacity` 设置定义。`innodb_io_capacity_max` 值在这种情况下定义了 I/O 容量的上限，以防止 I/O 活动的激增消耗服务器的整个 I/O 容量。

`innodb_io_capacity` 设置适用于所有缓冲池实例。当脏页被刷新时，I/O 容量在缓冲池实例之间均等分配。

##### 限制空闲期间的缓冲区刷新

截至 MySQL 8.0.18 版本，您可以使用 `innodb_idle_flush_pct` 变量来限制在空闲期间缓冲池刷新的速率，空闲期间是指数据库页面未被修改的时间段。`innodb_idle_flush_pct` 值是 `innodb_io_capacity` 设置的百分比，该设置定义了每秒可用于 `InnoDB` 的 I/O 操作数。默认的 `innodb_idle_flush_pct` 值为 100，即 `innodb_io_capacity` 设置的 100%。为了在空闲期间限制刷新，请定义一个小于 100 的 `innodb_idle_flush_pct` 值。

在空闲期间限制页面刷新可以延长固态存储设备的寿命。限制空闲期间页面刷新的副作用可能包括在长时间空闲期之后更长的关闭时间，以及在发生服务器故障时更长的恢复时间。