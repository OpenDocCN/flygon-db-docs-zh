> 原文：[`dev.mysql.com/doc/refman/8.0/en/problems-with-float.html`](https://dev.mysql.com/doc/refman/8.0/en/problems-with-float.html)

#### B.3.4.8 浮点值的问题

浮点数有时会引起混淆，因为它们是近似值，而不是存储为精确值。在 SQL 语句中写入的浮点值可能与内部表示的值不同。试图在比较中将浮点值视为精确值可能会导致问题。它们还受平台或实现依赖性的影响。`FLOAT` - FLOAT, DOUBLE")和`DOUBLE` - FLOAT, DOUBLE")数据类型受到这些问题的影响。对于`DECIMAL` - DECIMAL, NUMERIC")列，MySQL 执行具有 65 位小数位数的精度的操作，这应该解决大多数常见的不准确问题。

以下示例使用`DOUBLE` - FLOAT, DOUBLE")演示使用浮点运算进行的计算受浮点误差影响。

```sql
mysql> CREATE TABLE t1 (i INT, d1 DOUBLE, d2 DOUBLE);
mysql> INSERT INTO t1 VALUES (1, 101.40, 21.40), (1, -80.00, 0.00),
 -> (2, 0.00, 0.00), (2, -13.20, 0.00), (2, 59.60, 46.40),
 -> (2, 30.40, 30.40), (3, 37.00, 7.40), (3, -29.60, 0.00),
 -> (4, 60.00, 15.40), (4, -10.60, 0.00), (4, -34.00, 0.00),
 -> (5, 33.00, 0.00), (5, -25.80, 0.00), (5, 0.00, 7.20),
 -> (6, 0.00, 0.00), (6, -51.40, 0.00);

mysql> SELECT i, SUM(d1) AS a, SUM(d2) AS b
 -> FROM t1 GROUP BY i HAVING a <> b;

+------+-------+------+
| i    | a     | b    |
+------+-------+------+
|    1 |  21.4 | 21.4 |
|    2 |  76.8 | 76.8 |
|    3 |   7.4 |  7.4 |
|    4 |  15.4 | 15.4 |
|    5 |   7.2 |  7.2 |
|    6 | -51.4 |    0 |
+------+-------+------+
```

结果是正确的。尽管前五条记录看起来不应满足比较条件（`a`和`b`的值似乎没有不同），但它们可能会满足，因为数字之间的差异大约在第十位小数左右，具体取决于诸如计算机架构、编译器版本或优化级别等因素。例如，不同的 CPU 可能以不同方式评估浮点数。

如果列`d1`和`d2`被定义为`DECIMAL` - DECIMAL, NUMERIC")而不是`DOUBLE` - FLOAT, DOUBLE")，那么`SELECT`查询的结果将只包含最后一个上面显示的行。

浮点数比较的正确方法是首先确定两个数字之间的可接受容差，然后根据容差值进行比较。例如，如果我们认为浮点数在万分之一（0.0001）的精度内相同，则比较应该写成查找大于容差值的差异：

```sql
mysql> SELECT i, SUM(d1) AS a, SUM(d2) AS b FROM t1
 -> GROUP BY i HAVING ABS(a - b) > 0.0001;
+------+-------+------+
| i    | a     | b    |
+------+-------+------+
|    6 | -51.4 |    0 |
+------+-------+------+
1 row in set (0.00 sec)
```

相反，要获取数字相同的行，测试应该在容差值内找到差异：

```sql
mysql> SELECT i, SUM(d1) AS a, SUM(d2) AS b FROM t1
 -> GROUP BY i HAVING ABS(a - b) <= 0.0001;
+------+------+------+
| i    | a    | b    |
+------+------+------+
|    1 | 21.4 | 21.4 |
|    2 | 76.8 | 76.8 |
|    3 |  7.4 |  7.4 |
|    4 | 15.4 | 15.4 |
|    5 |  7.2 |  7.2 |
+------+------+------+
5 rows in set (0.03 sec)
```

浮点值受平台或实现依赖性的影响。假设您执行以下语句：

```sql
CREATE TABLE t1(c1 FLOAT(53,0), c2 FLOAT(53,0));
INSERT INTO t1 VALUES('1e+52','-1e+52');
SELECT * FROM t1;
```

在某些平台上，`SELECT`语句返回`inf`和`-inf`。在其他平台上，它返回`0`和`-0`。

前述问题的一个含义是，如果您尝试通过使用**mysqldump**在源端转储表内容并将转储文件重新加载到副本中来创建副本，那么包含浮点列的表可能会在两个主机之间有所不同。