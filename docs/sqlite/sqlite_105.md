# SQLite 版本 2 中的数据类型

> 原文：[`sqlite.com/datatypes.html`](https://sqlite.com/datatypes.html)

### 1.0   无类型

SQLite 是"无类型"的。这意味着你可以将任何类型的数据存储在任何表的任何列中，而不管该列声明的数据类型是什么（请参阅下面 2.0 节中的一个例外）。这种行为是一种特性，而不是错误。数据库应该能够存储和检索数据，而数据库不应关心数据的格式。大多数其他 SQL 引擎中的强类型系统以及 SQL 语言规范中的这种系统都是一个反面教材 - 它显示了实现细节。SQLite 通过允许将任何类型的数据存储到任何类型的列中，并允许在数据类型规范中灵活使用来克服这种反面教材。

对于 SQLite 来说，数据类型是任何零个或多个名字序列，后面可以跟随一个或两个带括号的有符号整数的列表。特别要注意的是，数据类型可能是*零*个或多个名字。这意味着空字符串在 SQLite 看来是一个有效的数据类型。因此，你可以声明表，其中每个列的数据类型都未指定，就像这样：

> ```sql
> CREATE TABLE ex1(a,b,c);
> 
> ```

即使 SQLite 允许省略数据类型，但在你的 CREATE TABLE 语句中包含数据类型仍然是一个好主意，因为数据类型通常可以为其他程序员提供关于你打算放置在列中的内容的良好提示。如果将代码移植到另一个数据库引擎时，该引擎可能会要求某种数据类型。SQLite 接受所有常见的数据类型。例如：

> ```sql
> CREATE TABLE ex2(
>   a VARCHAR(10),
>   b NVARCHAR(15),
>   c TEXT,
>   d INTEGER,
>   e FLOAT,
>   f BOOLEAN,
>   g CLOB,
>   h BLOB,
>   i TIMESTAMP,
>   j NUMERIC(10,5)
>   k VARYING CHARACTER (24),
>   l NATIONAL VARYING CHARACTER(16)
> );
> 
> ```

依此类推。基本上，任何一个名字序列，后面可以跟随一个或两个带括号的有符号整数，都是可以的。

### 2.0   INTEGER 主键

一个例外是 SQLite 的无类型特性，即列类型为 INTEGER PRIMARY KEY。（并且必须使用 "INTEGER" 而不是 "INT"。类型为 INT PRIMARY KEY 的列与其他列一样是无类型的。）INTEGER PRIMARY KEY 列必须包含一个 32 位有符号整数。任何尝试插入非整数数据都会导致错误。

INTEGER PRIMARY KEY 列可用于实现类似 AUTOINCREMENT 的功能。如果尝试向 INTEGER PRIMARY KEY 列插入 NULL，则该列实际上会填入一个比表中已有的最大键大一的整数。或者如果最大键为 2147483647，则该列将填入一个随机整数。无论哪种方式，INTEGER PRIMARY KEY 列都将被分配一个唯一的整数。您可以使用 **sqlite_last_insert_rowid()** API 函数或在后续的 SELECT 语句中使用 **last_insert_rowid()** SQL 函数来检索此整数。

### 3.0   比较和排序顺序

SQLite 在决定允许存储在列中的数据时是无类型的。但在排序和比较数据时会考虑某种类型的概念。为了这些目的，列或表达式可以是两种类型之一：**数值**和**文本**。根据排序或比较的数据类型不同，排序或比较可能会产生不同的结果。

如果数据是 **文本** 类型，则比较由标准 C 数据比较函数 **memcmp()** 或 **strcmp()** 决定。比较逐个比较两个输入的字节，并返回第一个非零差异。字符串以 '\000' 结尾，因此较短的字符串会排在较长的字符串之前，如您所预期的那样。

对于数值数据，情况更为复杂。如果两个输入看起来都像格式良好的数字，则它们将使用**atof()**转换为浮点值，并进行数字比较。如果一个输入不是格式良好的数字但另一个是，则该数字被视为小于非数字。如果两个输入都不是格式良好的数字，则使用**strcmp()**进行比较。

不要被列可能具有"numeric"数据类型所困扰。这并不意味着该列只能包含数字。它仅仅意味着如果该列包含一个数字，则该数字将按照数值顺序排序。

对于文本和数值，NULL 在任何其他值之前排序。使用诸如"<"或">="的运算符比较任何值与 NULL 始终为 false。

### 4.0   SQLite 如何确定数据类型

对于 SQLite 版本 2.6.3 及更早版本，所有值都使用数值数据类型。文本数据类型出现在版本 2.7.0 及更高版本中。在后续中假定您使用的是 SQLite 版本 2.7.0 或更高版本。

对于表达式，结果的数据类型通常由最外层运算符决定。例如，算术运算符（"+"、"*"、"%"）始终返回数值结果。字符串连接运算符（"||"）返回文本结果，依此类推。如果对表达式的数据类型存有疑问，可以使用特殊的**typeof()** SQL 函数来确定数据类型。例如：

> ```sql
> sqlite> SELECT typeof('abc'+123);
> numeric
> sqlite> SELECT typeof('abc'||123);
> text
> 
> ```

对于表列，数据类型由 CREATE TABLE 语句的类型声明确定。仅当类型声明包含以下一个或多个字符串时，数据类型为文本：

> BLOB
> 
> CHAR
> 
> CLOB
> 
> TEXT

在类型声明中查找这些字符串是不区分大小写的。如果类型声明中出现上述任何字符串，则列的数据类型为文本。请注意，类型"VARCHAR"包含"CHAR"作为子字符串，因此被视为文本。

如果上述字符串在类型声明中没有出现，则数据类型为数字。特别注意，对于类型声明为空的列，数据类型为数字。

### 5.0   示例

考虑以下两个命令序列：

> ```sql
> CREATE TABLE t1(a INTEGER UNIQUE);        CREATE TABLE t2(b TEXT UNIQUE);
> INSERT INTO t1 VALUES('0');               INSERT INTO t2 VALUES(0);
> INSERT INTO t1 VALUES('0.0');             INSERT INTO t2 VALUES(0.0);
> 
> ```

在左侧的序列中，第二次插入将会失败。在这种情况下，字符串 '0' 和 '0.0' 被视为数字，因为它们被插入到一个数字列中，但是 0==0.0 违反了唯一性约束。然而，右侧序列中的第二次插入有效。在这种情况下，常量 0 和 0.0 被视为字符串，这意味着它们是不同的。

SQLite 总是将数字转换为双精度（64 位）浮点数进行比较。这意味着，如果它们在数字列中，则仅在无关紧要的数字上有所不同的长数字序列将会相等，但如果它们在文本列中则会不相等。我们有：

> ```sql
> INSERT INTO t1                            INSERT INTO t2
>    VALUES('12345678901234567890');           VALUES(12345678901234567890);
> INSERT INTO t1                            INSERT INTO t2
>    VALUES('12345678901234567891');           VALUES(12345678901234567891);
> 
> ```

和之前一样，左侧的第二次插入将会失败，因为比较将首先将这两个字符串转换为浮点数，并且字符串唯一的差异是在第 20 位数字，这超出了 64 位浮点数的分辨率。相比之下，右侧的第二次插入将会成功，因为在这种情况下，被插入的是字符串，并使用 memcmp() 进行比较。

对于 DISTINCT 关键字，数字和文本类型也会有所不同：

> ```sql
> CREATE TABLE t3(a INTEGER);               CREATE TABLE t4(b TEXT);
> INSERT INTO t3 VALUES('0');               INSERT INTO t4 VALUES(0);
> INSERT INTO t3 VALUES('0.0');             INSERT INTO t4 VALUES(0.0);
> SELECT DISTINCT * FROM t3;                SELECT DISTINCT * FROM t4;
> 
> ```

左侧的 SELECT 语句将返回一行，因为 '0' 和 '0.0' 被视为数字，因此是不相同的。但右侧的 SELECT 语句将返回两行，因为 0 和 0.0 被视为不同的字符串。