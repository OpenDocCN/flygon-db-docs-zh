# 内存映射 I/O

> 原文：[`sqlite.com/mmap.html`](https://sqlite.com/mmap.html)

SQLite 访问和更新数据库磁盘文件的默认机制是通过 sqlite3_io_methods VFS 对象的 xRead()和 xWrite()方法。这些方法通常实现为导致操作系统在内核缓冲区高速缓存和用户空间之间复制磁盘内容的“read()”和“write()”系统调用。

从版本 3.7.17（2013-05-20）开始，SQLite 具有通过内存映射 I/O 直接访问磁盘内容的选项，并在 sqlite3_io_methods 上使用新的 xFetch()和 xUnfetch()方法。

使用内存映射 I/O 的优缺点各有所见。优点包括：

1.  许多操作，特别是 I/O 密集型操作，可能更快，因为内容无需在内核空间和用户空间之间复制。

1.  由于 SQLite 库与操作系统页面缓存共享页面，并且不始终需要其自己的工作页面副本，因此 SQLite 库可能需要更少的 RAM。

但也存在一些缺点：

1.  内存映射文件上的 I/O 错误无法被 SQLite 捕获和处理。相反，I/O 错误会导致信号，如果应用程序没有捕获到该信号，则会导致程序崩溃。

1.  操作系统必须具有统一的缓冲区高速缓存，以确保内存映射 I/O 扩展正常工作，特别是在两个进程访问同一数据库文件的情况下，其中一个进程使用内存映射 I/O，而另一个进程不使用。并非所有操作系统都具有统一的缓冲区高速缓存。在某些声称具有统一缓冲区高速缓存的操作系统中，实现可能存在错误，并可能导致数据库损坏。

1.  性能并非总是随着内存映射 I/O 的使用而增加。事实上，可以构造测试用例，证明内存映射 I/O 的使用会降低性能。

1.  Windows 无法截断内存映射文件。因此，在 Windows 上，如果执行诸如 VACUUM 或 auto_vacuum 之类的操作试图减少内存映射数据库文件的大小，则大小减少尝试将悄无声息地失败，使得数据库文件末尾留下未使用的空间。由于此问题不会导致数据丢失，并且未使用的空间将在数据库再次增长时重新使用。但是，如果在 SQLite 3.7.0 之前的版本上运行 PRAGMA integrity_check 来检查这样的数据库，则会（错误地）报告由于数据库末尾的未使用空间而导致的数据库损坏。或者如果在 SQLite 3.7.0 之前的版本中在数据库仍有未使用空间时对其进行写入，则可能使该未使用空间变得不可访问并在下一次 VACUUM 之后再次可用。

由于潜在的缺点，内存映射 I/O 默认是禁用的。要激活内存映射 I/O，请使用 mmap_size pragma 并将 mmap_size 设置为一些较大的数字，通常是 256MB 或更大，具体取决于应用程序可以提供多少地址空间。其余工作是自动完成的。在不支持内存映射 I/O 的系统上，PRAGMA mmap_size 语句将是一个静默的空操作。

## 内存映射 I/O 的工作原理

要使用传统的 xRead() 方法读取数据库内容的页面，SQLite 首先分配一个页面大小的堆内存块，然后调用 xRead() 方法，导致数据库页面内容被复制到新分配的堆内存中。这至少涉及整个页面的复制。

但是，如果 SQLite 想要访问数据库文件的某一页并启用了内存映射 I/O，则首先调用 xFetch() 方法。xFetch() 方法请求操作系统返回所请求页面的指针，如果可能的话。如果请求的页面已经被映射到应用程序地址空间中，那么 xFetch 返回该页面的指针供 SQLite 使用，而无需复制任何内容。跳过复制步骤是内存映射 I/O 更快的原因。

SQLite 不假定 xFetch() 方法会起作用。如果调用 xFetch() 返回一个空指针（表示请求的页面当前未映射到应用程序地址空间），那么 SQLite 会静默地回退到使用 xRead()。只有在 xRead() 也失败时才会报错。

当更新数据库文件时，SQLite 总是在修改页面之前将页面内容复制到堆内存中。这是出于两个原因的必要性。首先，数据库的更改在事务提交之前不应对其他进程可见，因此更改必须发生在私有内存中。其次，SQLite 使用只读内存映射来防止应用程序中的杂散指针覆盖和损坏数据库文件。

在完成所有必要的更改之后，xWrite() 用于将内容移回数据库文件。因此，使用内存映射 I/O 不会显著改变数据库更改的性能。内存映射 I/O 主要对查询有益。

## 配置内存映射 I/O

"mmap_size" 是 SQLite 尝试在进程地址空间中一次映射的数据库文件的最大字节数。mmap_size 分别适用于每个数据库文件，因此可以潜在使用的进程地址空间总量为 mmap_size 乘以打开的数据库文件数。

要激活内存映射 I/O，应用程序可以将 mmap_size 设置为较大的值。例如：

> ```sql
> PRAGMA mmap_size=268435456;
> 
> ```

要禁用内存映射 I/O，只需将 mmap_size 设置为零：

> ```sql
> PRAGMA mmap_size=0;
> 
> ```

如果 mmap_size 设置为 N，则所有当前的实现都会将数据库文件的前 N 字节映射到内存，并对超出 N 字节的内容使用传统的 xRead() 调用。如果数据库文件小于 N 字节，则整个文件都会被映射。未来，理论上新的操作系统接口可以映射文件除了前 N 字节之外的区域，但目前尚无此类实现。

"PRAGMA mmap_size" 语句用于分别为每个数据库文件设置 mmap_size。通常，默认的 mmap_size 是零，意味着默认情况下禁用内存映射 I/O。但是，可以在编译时使用 SQLITE_DEFAULT_MMAP_SIZE 宏或在启动时使用 sqlite3_config(SQLITE_CONFIG_MMAP_SIZE,...) 接口来增加默认的 mmap_size。

SQLite 还保持对 mmap_size 的硬上限。尝试将 mmap_size 增加到这个硬上限之上（使用 PRAGMA mmap_size）会自动将 mmap_size 限制在硬上限之内。如果硬上限为零，则无法进行内存映射 I/O。硬上限可以在编译时使用 SQLITE_MAX_MMAP_SIZE 宏进行设置。如果 SQLITE_MAX_MMAP_SIZE 设置为零，则用于实现内存映射 I/O 的代码将被省略。在某些平台（例如 OpenBSD）上，由于缺乏统一的缓冲区缓存，硬上限会自动设置为零，从而导致内存映射 I/O 不起作用。

如果在编译时 mmap_size 的硬上限不为零，可以在启动时使用 sqlite3_config(SQLITE_CONFIG_MMAP_SIZE,X,Y) 接口来减少或将其归零。X 和 Y 参数必须都是 64 位有符号整数。X 参数是进程的默认 mmap_size，Y 是新的硬上限。使用 SQLITE_CONFIG_MMAP_SIZE 不能将硬上限增加到编译时的设置之上，但可以减少或归零。