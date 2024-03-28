# 29.4.1 性能模式事件计时

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-timing.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-timing.html)

事件是通过添加到服务器源代码的仪器来收集的。仪器计时事件，这就是性能模式提供事件持续时间的方式。还可以配置仪器不收集计时信息。本节讨论可用计时器及其特性，以及事件中计时值的表示方式。

#### 性能模式计时器

性能模式计时器在精度和开销量上有所不同。要查看可用的计时器及其特性，请查看`performance_timers`表：

```sql
mysql> SELECT * FROM performance_schema.performance_timers;
+-------------+-----------------+------------------+----------------+
| TIMER_NAME  | TIMER_FREQUENCY | TIMER_RESOLUTION | TIMER_OVERHEAD |
+-------------+-----------------+------------------+----------------+
| CYCLE       |      2389029850 |                1 |             72 |
| NANOSECOND  |      1000000000 |                1 |            112 |
| MICROSECOND |         1000000 |                1 |            136 |
| MILLISECOND |            1036 |                1 |            168 |
| THREAD_CPU  |       339101694 |                1 |            798 |
+-------------+-----------------+------------------+----------------+
```

如果与给定计时器名称相关联的值为`NULL`，则该计时器在您的平台上不受支持。

列的含义如下：

+   `TIMER_NAME`列显示可用计时器的名称。`CYCLE`指的是基于 CPU（处理器）周期计数器的计时器。

+   `TIMER_FREQUENCY`表示每秒的计时器单位数。对于循环计时器，频率通常与 CPU 速度有关。显示的值是在一个 2.4GHz 处理器系统上获得的。其他计时器基于固定的秒分数。

+   `TIMER_RESOLUTION`表示计时器值每次增加的计时器单位数。如果计时器的分辨率为 10，那么每次增加 10。

+   `TIMER_OVERHEAD`是获取给定计时器的一个计时所需的最小周期数。每个事件的开销是显示值的两倍，因为计时器在事件开始和结束时被调用。

性能模式将计时器分配如下：

+   等待计时器使用`CYCLE`。

+   空闲、阶段、语句和事务计时器在支持`NANOSECOND`计时器的平台上使用`NANOSECOND`，否则使用`MICROSECOND`。

在服务器启动时，性能模式会验证构建时关于计时器分配的假设是否正确，并在计时器不可用时显示警告。

对于计时等待事件，最重要的标准是减少开销，可能会牺牲计时器的准确性，因此使用`CYCLE`计时器是最好的选择。

语句（或阶段）执行所需的时间通常比执行单个等待所需的时间大几个数量级。为了计时语句，最重要的标准是要有一个准确的测量，不受处理器频率变化的影响，因此使用不基于循环的计时器是最好的。语句的默认计时器是`NANOSECOND`。与`CYCLE`计时器相比的额外“开销”并不显著，因为由于调用计时器两次（一次在语句开始时，一次在语句结束时）引起的开销与执行语句本身所用的 CPU 时间相比，数量级要小得多。在这里使用`CYCLE`计时器没有好处，只有缺点。

循环计数器提供的精度取决于处理器速度。如果处理器运行速度为 1 GHz（十亿个周期/秒）或更高，则循环计数器提供亚纳秒级精度。使用循环计数器比获取实际的当天时间要便宜得多。例如，标准的`gettimeofday()`函数可能需要数百个周期，这对于可能每秒发生数千次或数百万次的数据收集来说是不可接受的开销。

循环计数器也有缺点：

+   最终用户希望看到墙钟单位的时间，例如秒的分数。从循环到秒的转换可能很昂贵。因此，转换是一个快速且相当粗糙的乘法运算。

+   处理器的循环速率可能会发生变化，例如当笔记本电脑进入节能模式或当 CPU 减速以减少热量产生时。如果处理器的循环速率波动，从循环到实时单位的转换可能会出现错误。

+   根据处理器或操作系统的不同，循环计数器可能不可靠或不可用。例如，在奔腾处理器上，指令是`RDTSC`（汇编语言而不是 C 指令），理论上操作系统可以阻止用户模式程序使用它。

+   与乱序执行或多处理器同步相关的一些处理器细节可能导致计数器看起来快或慢多达 1000 个周期。

MySQL 在 x386（Windows，macOS，Linux，Solaris 和其他 Unix 变种），PowerPC 和 IA-64 上使用循环计数器。

#### Performance Schema 事件中的计时器表示

Performance Schema 表中存储当前事件和历史事件的行有三列用于表示时间信息：`TIMER_START`和`TIMER_END`表示事件开始和结束的时间，`TIMER_WAIT`表示事件持续时间。

`setup_instruments` 表具有一个 `ENABLED` 列，用于指示要收集事件的仪器。该表还有一个 `TIMED` 列，用于指示哪些仪器是定时的。如果一个仪器未启用，则不会产生事件。如果启用的仪器未定时，则由仪器产生的事件的 `TIMER_START`、`TIMER_END` 和 `TIMER_WAIT` 计时器值为 `NULL`。这反过来导致在汇总表（总和、最小值、最大值和平均值）中计算聚合时间值时忽略这些值。

在事件内部，事件中的时间以事件定时开始时的计时器给定的单位存储。当从性能模式表中检索事件时，时间以皮秒（万亿分之一秒）显示，以将它们归一化为标准单位，而不管选择了哪个计时器。

计时器基线（“零时间点”）发生在服务器启动期间的性能模式初始化。事件中的 `TIMER_START` 和 `TIMER_END` 值表示自基线以来的皮秒数。`TIMER_WAIT` 值是以皮秒为单位的持续时间。

事件中的皮秒值是近似值。它们的准确性受到与从一个单位转换到另一个单位相关的常见误差形式的影响。如果使用 `CYCLE` 计时器并且处理器速率变化，可能会出现漂移。因此，查看事件的 `TIMER_START` 值作为自服务器启动以来经过的时间的准确度测量是不合理的。另一方面，使用 `TIMER_START` 或 `TIMER_WAIT` 值在 `ORDER BY` 子句中对事件按开始时间或持续时间排序是合理的。

事件中选择皮秒而不是微秒等值具有性能基础。一个实现目标是以统一的时间单位显示结果，而不管计时器是什么。在理想世界中，这个时间单位看起来像一个挂钟单位，并且相当精确；换句话说，微秒。但是要将周期或纳秒转换为微秒，就需要对每个仪器执行除法。在许多平台上，除法是昂贵的。乘法不昂贵，所以这就是所使用的。因此，时间单位是最高可能的 `TIMER_FREQUENCY` 值的整数倍，使用足够大的乘数来确保没有主要的精度损失。结果是时间单位是“皮秒”。这种精度是虚假的，但这个决定使得开销最小化。

当等待、阶段、语句或事务事件正在执行时，相应的当前事件表显示当前事件的定时信息：

```sql
events_waits_current
events_stages_current
events_statements_current
events_transactions_current
```

为了能够确定尚未完成的事件运行了多长时间，计时器列设置如下：

+   `TIMER_START` 已填充。

+   `TIMER_END` 已填充，显示当前计时器值。

+   `TIMER_WAIT` 已填充，显示到目前为止经过的时间（`TIMER_END` − `TIMER_START`）。

尚未完成的事件具有`END_EVENT_ID`值为`NULL`。要评估到目前为止事件经过的时间，使用`TIMER_WAIT`列。因此，要识别尚未完成且到目前为止已经花费超过*`N`*皮秒的事件，监控应用程序可以在查询中使用这个表达式：

```sql
WHERE END_EVENT_ID IS NULL AND TIMER_WAIT > *N*
```

事件识别如上所述，假设相应的仪器`ENABLED`和`TIMED`设置为`YES`，并且相关的消费者已启用。