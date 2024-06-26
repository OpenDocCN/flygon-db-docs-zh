# 14.17.4 修改 JSON 值的函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/json-modification-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/json-modification-functions.html)

本节中的函数修改 JSON 值并返回结果。

+   [`JSON_ARRAY_APPEND(*`json_doc`*, *`path`*, *`val`*[, *`path`*, *`val`*] ...)`](json-modification-functions.html#function_json-array-append)

    将值附加到 JSON 文档中指定数组的末尾，并返回结果。如果任何参数为`NULL`，则返回`NULL`。如果*`json_doc`*参数不是有效的 JSON 文档，或任何*`path`*参数不是有效的路径表达式，或包含`*`或`**`通配符，则会发生错误。

    路径-值对从左到右进行评估。通过评估一个对产生的文档成为下一个对进行评估的新值。

    如果路径选择标量或对象值，则该值会自动包装在数组中，并将新值添加到该数组中。在 JSON 文档中，路径未识别任何值的对将被忽略。

    ```sql
    mysql> SET @j = '["a", ["b", "c"], "d"]';
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$[1]', 1);
    +----------------------------------+
    | JSON_ARRAY_APPEND(@j, '$[1]', 1) |
    +----------------------------------+
    | ["a", ["b", "c", 1], "d"]        |
    +----------------------------------+
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$[0]', 2);
    +----------------------------------+
    | JSON_ARRAY_APPEND(@j, '$[0]', 2) |
    +----------------------------------+
    | [["a", 2], ["b", "c"], "d"]      |
    +----------------------------------+
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$[1][0]', 3);
    +-------------------------------------+
    | JSON_ARRAY_APPEND(@j, '$[1][0]', 3) |
    +-------------------------------------+
    | ["a", [["b", 3], "c"], "d"]         |
    +-------------------------------------+

    mysql> SET @j = '{"a": 1, "b": [2, 3], "c": 4}';
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$.b', 'x');
    +------------------------------------+
    | JSON_ARRAY_APPEND(@j, '$.b', 'x')  |
    +------------------------------------+
    | {"a": 1, "b": [2, 3, "x"], "c": 4} |
    +------------------------------------+
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$.c', 'y');
    +--------------------------------------+
    | JSON_ARRAY_APPEND(@j, '$.c', 'y')    |
    +--------------------------------------+
    | {"a": 1, "b": [2, 3], "c": [4, "y"]} |
    +--------------------------------------+

    mysql> SET @j = '{"a": 1}';
    mysql> SELECT JSON_ARRAY_APPEND(@j, '$', 'z');
    +---------------------------------+
    | JSON_ARRAY_APPEND(@j, '$', 'z') |
    +---------------------------------+
    | [{"a": 1}, "z"]                 |
    +---------------------------------+
    ```

    在 MySQL 5.7 中，此函数的名称为`JSON_APPEND()`。在 MySQL 8.0 中不再支持该名称。

+   [`JSON_ARRAY_INSERT(*`json_doc`*, *`path`*, *`val`*[, *`path`*, *`val`*] ...)`](json-modification-functions.html#function_json-array-insert)

    更新 JSON 文档，将其插入到文档中的数组中，并返回修改后的文档。如果任何参数为`NULL`，则返回`NULL`。如果*`json_doc`*参数不是有效的 JSON 文档，或任何*`path`*参数不是有效的路径表达式，或包含`*`或`**`通配符，或不以数组元素标识符结尾，则会发生错误。

    路径-值对从左到右进行评估。通过评估一个对产生的文档成为下一个对进行评估的新值。

    对于路径未识别 JSON 文档中任何数组的对将被忽略。如果路径标识数组元素，则相应值将插入到该元素位置，将任何后续值向右移动。如果路径标识超出数组末尾的数组位置，则该值将插入到数组末尾。

    ```sql
    mysql> SET @j = '["a", {"b": [1, 2]}, [3, 4]]';
    mysql> SELECT JSON_ARRAY_INSERT(@j, '$[1]', 'x');
    +------------------------------------+
    | JSON_ARRAY_INSERT(@j, '$[1]', 'x') |
    +------------------------------------+
    | ["a", "x", {"b": [1, 2]}, [3, 4]]  |
    +------------------------------------+
    mysql> SELECT JSON_ARRAY_INSERT(@j, '$[100]', 'x');
    +--------------------------------------+
    | JSON_ARRAY_INSERT(@j, '$[100]', 'x') |
    +--------------------------------------+
    | ["a", {"b": [1, 2]}, [3, 4], "x"]    |
    +--------------------------------------+
    mysql> SELECT JSON_ARRAY_INSERT(@j, '$[1].b[0]', 'x');
    +-----------------------------------------+
    | JSON_ARRAY_INSERT(@j, '$[1].b[0]', 'x') |
    +-----------------------------------------+
    | ["a", {"b": ["x", 1, 2]}, [3, 4]]       |
    +-----------------------------------------+
    mysql> SELECT JSON_ARRAY_INSERT(@j, '$[2][1]', 'y');
    +---------------------------------------+
    | JSON_ARRAY_INSERT(@j, '$[2][1]', 'y') |
    +---------------------------------------+
    | ["a", {"b": [1, 2]}, [3, "y", 4]]     |
    +---------------------------------------+
    mysql> SELECT JSON_ARRAY_INSERT(@j, '$[0]', 'x', '$[2][1]', 'y');
    +----------------------------------------------------+
    | JSON_ARRAY_INSERT(@j, '$[0]', 'x', '$[2][1]', 'y') |
    +----------------------------------------------------+
    | ["x", "a", {"b": [1, 2]}, [3, 4]]                  |
    +----------------------------------------------------+
    ```

    早期的修改会影响数组中后续元素的位置，因此同一`JSON_ARRAY_INSERT()`调用中的后续路径应考虑这一点。在最后一个示例中，第二个路径不插入任何内容，因为在第一次插入后路径不再匹配任何内容。

+   [`JSON_INSERT(*`json_doc`*, *`path`*, *`val`*[, *`path`*, *`val`*] ...)`](json-modification-functions.html#function_json-insert)

    将数据插入 JSON 文档并返回结果。如果任何参数为`NULL`，则返回`NULL`。如果*`json_doc`*参数不是有效的 JSON 文档，或任何*`path`*参数不是有效的路径表达式，或包含`*`或`**`通配符，则会发生错误。

    路径-值对从左到右进行评估。通过评估一个对生成的文档成为下一个对要评估的新值。

    对文档中现有路径的路径-值对被忽略，不会覆盖现有文档值。对文档中不存在路径的路径-值对，如果路径标识以下类型的值之一，则将该值添加到文档中：

    +   不存在于现有对象中的成员。将该成员添加到对象中并与新值关联。

    +   超出现有数组末尾的位置。数组将使用新值扩展。如果现有值不是数组，则会自动包装为数组，然后使用新值扩展。

    否则，文档中不存在路径的路径-值对将被忽略且不起作用。

    有关`JSON_INSERT()`、`JSON_REPLACE()`和`JSON_SET()`的比较，请参见`JSON_SET()`的讨论。

    ```sql
    mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
    mysql> SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]');
    +----------------------------------------------------+
    | JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]') |
    +----------------------------------------------------+
    | {"a": 1, "b": [2, 3], "c": "[true, false]"}        |
    +----------------------------------------------------+
    ```

    结果中列出的第三个和最后一个值是带引号的字符串，而不像第二个值那样是数组（在输出中没有引号）；不执行将值转换为 JSON 类型的操作。要将数组插入为数组，必须显式执行此类转换，如下所示：

    ```sql
    mysql> SELECT JSON_INSERT(@j, '$.a', 10, '$.c', CAST('[true, false]' AS JSON));
    +------------------------------------------------------------------+
    | JSON_INSERT(@j, '$.a', 10, '$.c', CAST('[true, false]' AS JSON)) |
    +------------------------------------------------------------------+
    | {"a": 1, "b": [2, 3], "c": [true, false]}                        |
    +------------------------------------------------------------------+
    1 row in set (0.00 sec)
    ```

+   [`JSON_MERGE(*`json_doc`*, *`json_doc`*[, *`json_doc`*] ...)`](json-modification-functions.html#function_json-merge)

    合并两个或多个 JSON 文档。`JSON_MERGE_PRESERVE()`的同义词；在 MySQL 8.0.3 中已弃用，并可能在将来的版本中删除。

    ```sql
    mysql> SELECT JSON_MERGE('[1, 2]', '[true, false]');
    +---------------------------------------+
    | JSON_MERGE('[1, 2]', '[true, false]') |
    +---------------------------------------+
    | [1, 2, true, false]                   |
    +---------------------------------------+
    1 row in set, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Warning
       Code: 1287
    Message: 'JSON_MERGE' is deprecated and will be removed in a future release. \
     Please use JSON_MERGE_PRESERVE/JSON_MERGE_PATCH instead 1 row in set (0.00 sec)
    ```

    更多示例，请参见`JSON_MERGE_PRESERVE()`的条目。

+   [`JSON_MERGE_PATCH(*`json_doc`*, *`json_doc`*[, *`json_doc`*] ...)`](json-modification-functions.html#function_json-merge-patch)

    执行符合[RFC 7396](https://tools.ietf.org/html/rfc7396)的两个或多个 JSON 文档的合并，并返回合并结果，不保留具有重复键的成员。如果传递给此函数的至少一个文档无效，则会引发错误。

    注意

    有关此函数与`JSON_MERGE_PRESERVE()`之间差异的解释和示例，请参见 JSON_MERGE_PATCH()与 JSON_MERGE_PRESERVE()的比较与 JSON_MERGE_PRESERVE()的比较")。

    `JSON_MERGE_PATCH()`执行如下合并：

    1.  如果第一个参数不是对象，则合并的结果与将空对象与第二个参数合并的结果相同。

    1.  如果第二个参数不是对象，则合并的结果是第二个参数。

    1.  如果两个参数都是对象，则合并的结果是具有以下成员的对象：

        +   第一个对象中所有没有与第二个对象中具有相同键的成员。

        +   第二个对象的所有成员，这些成员在第一个对象中没有相应的键，并且其值不是 JSON `null`文字。

        +   所有具有在第一个和第二个对象中都存在的键的成员，并且在第二个对象中的值不是 JSON `null`文字。这些成员的值是通过递归地将第一个对象中的值与第二个对象中的值合并而得到的结果。

    有关更多信息，请参阅 JSON 值的规范化、合并和自动包装。

    ```sql
    mysql> SELECT JSON_MERGE_PATCH('[1, 2]', '[true, false]');
    +---------------------------------------------+
    | JSON_MERGE_PATCH('[1, 2]', '[true, false]') |
    +---------------------------------------------+
    | [true, false]                               |
    +---------------------------------------------+

    mysql> SELECT JSON_MERGE_PATCH('{"name": "x"}', '{"id": 47}');
    +-------------------------------------------------+
    | JSON_MERGE_PATCH('{"name": "x"}', '{"id": 47}') |
    +-------------------------------------------------+
    | {"id": 47, "name": "x"}                         |
    +-------------------------------------------------+

    mysql> SELECT JSON_MERGE_PATCH('1', 'true');
    +-------------------------------+
    | JSON_MERGE_PATCH('1', 'true') |
    +-------------------------------+
    | true                          |
    +-------------------------------+

    mysql> SELECT JSON_MERGE_PATCH('[1, 2]', '{"id": 47}');
    +------------------------------------------+
    | JSON_MERGE_PATCH('[1, 2]', '{"id": 47}') |
    +------------------------------------------+
    | {"id": 47}                               |
    +------------------------------------------+

    mysql> SELECT JSON_MERGE_PATCH('{ "a": 1, "b":2 }',
         >     '{ "a": 3, "c":4 }');
    +-----------------------------------------------------------+
    | JSON_MERGE_PATCH('{ "a": 1, "b":2 }','{ "a": 3, "c":4 }') |
    +-----------------------------------------------------------+
    | {"a": 3, "b": 2, "c": 4}                                  |
    +-----------------------------------------------------------+

    mysql> SELECT JSON_MERGE_PATCH('{ "a": 1, "b":2 }','{ "a": 3, "c":4 }',
         >     '{ "a": 5, "d":6 }');
    +-------------------------------------------------------------------------------+
    | JSON_MERGE_PATCH('{ "a": 1, "b":2 }','{ "a": 3, "c":4 }','{ "a": 5, "d":6 }') |
    +-------------------------------------------------------------------------------+
    | {"a": 5, "b": 2, "c": 4, "d": 6}                                              |
    +-------------------------------------------------------------------------------+
    ```

    您可以使用此函数通过在第二个参数中将相同成员的值指定为`null`来删除成员，如下所示：

    ```sql
    mysql> SELECT JSON_MERGE_PATCH('{"a":1, "b":2}', '{"b":null}');
    +--------------------------------------------------+
    | JSON_MERGE_PATCH('{"a":1, "b":2}', '{"b":null}') |
    +--------------------------------------------------+
    | {"a": 1}                                         |
    +--------------------------------------------------+
    ```

    此示例显示该函数以递归方式运行；即，成员的值不仅限于标量，而是可以是 JSON 文档本身：

    ```sql
    mysql> SELECT JSON_MERGE_PATCH('{"a":{"x":1}}', '{"a":{"y":2}}');
    +----------------------------------------------------+
    | JSON_MERGE_PATCH('{"a":{"x":1}}', '{"a":{"y":2}}') |
    +----------------------------------------------------+
    | {"a": {"x": 1, "y": 2}}                            |
    +----------------------------------------------------+
    ```

    `JSON_MERGE_PATCH()`在 MySQL 8.0.3 及更高版本中受支持。

    **JSON_MERGE_PATCH()与 JSON_MERGE_PRESERVE()的比较。** `JSON_MERGE_PATCH()`的行为与`JSON_MERGE_PRESERVE()`相同，但有以下两个例外：

    +   `JSON_MERGE_PATCH()`会删除第一个对象中具有与第二个对象中匹配键的成员，前提是与键相关联的值在第二个对象中不是 JSON `null`。

    +   如果第二个对象具有与第一个对象中的成员匹配的键的成员，`JSON_MERGE_PATCH()`会用第二个对象中的值*替换*第一个对象中的值，而`JSON_MERGE_PRESERVE()`会*附加*第二个值到第一个值。

    此示例比较了合并具有匹配键`"a"`的相同 3 个 JSON 对象的结果，每个函数都有：

    ```sql
    mysql> SET @x = '{ "a": 1, "b": 2 }',
         >     @y = '{ "a": 3, "c": 4 }',
         >     @z = '{ "a": 5, "d": 6 }';

    mysql> SELECT  JSON_MERGE_PATCH(@x, @y, @z)    AS Patch,
     ->         JSON_MERGE_PRESERVE(@x, @y, @z) AS Preserve\G
    *************************** 1\. row ***************************
       Patch: {"a": 5, "b": 2, "c": 4, "d": 6}
    Preserve: {"a": [1, 3, 5], "b": 2, "c": 4, "d": 6}
    ```

+   [`JSON_MERGE_PRESERVE(*`json_doc`*, *`json_doc`*[, *`json_doc`*] ...)`](json-modification-functions.html#function_json-merge-preserve)

    合并两个或多个 JSON 文档并返回合并后的结果。如果任何参数为`NULL`，则返回`NULL`。如果任何参数不是有效的 JSON 文档，则会发生错误。

    合并按照以下规则进行。有关更多信息，请参阅 JSON 值的规范化、合并和自动包装。

    +   相邻的数组会合并为一个数组。

    +   相邻的对象会合并为一个对象。

    +   标量值会自动包装为数组并作为数组合并。

    +   通过将对象自动包装为数组并合并两个数组，可以合并相邻的数组和对象。

    ```sql
    mysql> SELECT JSON_MERGE_PRESERVE('[1, 2]', '[true, false]');
    +------------------------------------------------+
    | JSON_MERGE_PRESERVE('[1, 2]', '[true, false]') |
    +------------------------------------------------+
    | [1, 2, true, false]                            |
    +------------------------------------------------+

    mysql> SELECT JSON_MERGE_PRESERVE('{"name": "x"}', '{"id": 47}');
    +----------------------------------------------------+
    | JSON_MERGE_PRESERVE('{"name": "x"}', '{"id": 47}') |
    +----------------------------------------------------+
    | {"id": 47, "name": "x"}                            |
    +----------------------------------------------------+

    mysql> SELECT JSON_MERGE_PRESERVE('1', 'true');
    +----------------------------------+
    | JSON_MERGE_PRESERVE('1', 'true') |
    +----------------------------------+
    | [1, true]                        |
    +----------------------------------+

    mysql> SELECT JSON_MERGE_PRESERVE('[1, 2]', '{"id": 47}');
    +---------------------------------------------+
    | JSON_MERGE_PRESERVE('[1, 2]', '{"id": 47}') |
    +---------------------------------------------+
    | [1, 2, {"id": 47}]                          |
    +---------------------------------------------+

    mysql> SELECT JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }',
         >    '{ "a": 3, "c": 4 }');
    +--------------------------------------------------------------+
    | JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }','{ "a": 3, "c":4 }') |
    +--------------------------------------------------------------+
    | {"a": [1, 3], "b": 2, "c": 4}                                |
    +--------------------------------------------------------------+

    mysql> SELECT JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }','{ "a": 3, "c": 4 }',
         >    '{ "a": 5, "d": 6 }');
    +----------------------------------------------------------------------------------+
    | JSON_MERGE_PRESERVE('{ "a": 1, "b": 2 }','{ "a": 3, "c": 4 }','{ "a": 5, "d": 6 }') |
    +----------------------------------------------------------------------------------+
    | {"a": [1, 3, 5], "b": 2, "c": 4, "d": 6}                                         |
    +----------------------------------------------------------------------------------+
    ```

    此函数在 MySQL 8.0.3 中作为`JSON_MERGE()`的同义词添加。`JSON_MERGE()`函数现已弃用，并可能在将来的 MySQL 版本中被移除。

    此函数与 `JSON_MERGE_PATCH()` 在重要方面类似但不同；有关更多信息，请参阅 JSON_MERGE_PATCH() 与 JSON_MERGE_PRESERVE() 的比较 与 JSON_MERGE_PRESERVE() 的比较")。

+   [`JSON_REMOVE(*`json_doc`*, *`path`*[, *`path`*] ...)`](json-modification-functions.html#function_json-remove)

    从 JSON 文档中移除数据并返回结果。如果任何参数为 `NULL`，则返回 `NULL`。如果 *`json_doc`* 参数不是有效的 JSON 文档，或任何 *`path`* 参数不是有效的路径表达式，或包含 `$`，或包含 `*` 或 `**` 通配符，则会发生错误。

    *`path`* 参数从左到右进行评估。通过评估一个路径，生成的文档成为下一个路径要评估的新值。

    如果要移除的元素在文档中不存在，不会出错；在这种情况下，路径不会影响文档。

    ```sql
    mysql> SET @j = '["a", ["b", "c"], "d"]';
    mysql> SELECT JSON_REMOVE(@j, '$[1]');
    +-------------------------+
    | JSON_REMOVE(@j, '$[1]') |
    +-------------------------+
    | ["a", "d"]              |
    +-------------------------+
    ```

+   [`JSON_REPLACE(*`json_doc`*, *`path`*, *`val`*[, *`path`*, *`val`*] ...)`](json-modification-functions.html#function_json-replace)

    替换 JSON 文档中的现有值并返回结果。如果任何参数为 `NULL`，则返回 `NULL`。如果 *`json_doc`* 参数不是有效的 JSON 文档，或任何 *`path`* 参数不是有效的路径表达式，或包含 `*` 或 `**` 通配符，则会发生错误。

    路径-值对从左到右进行评估。通过评估一个对，生成的文档成为下一个对要评估的新值。

    对于文档中现有路径的路径-值对，将现有文档值覆盖为新值。对于文档中不存在的路径的路径-值对，将被忽略且不起作用。

    在 MySQL 8.0.4 中，优化器可以对 `JSON` 列执行部分、原地更新，而不是删除旧文档并将新文档完全写入列中。这种优化可以用于使用 `JSON_REPLACE()` 函数的更新语句，并满足 JSON 值的部分更新 中概述的条件。

    比较 `JSON_INSERT()`、`JSON_REPLACE()` 和 `JSON_SET()`，请参阅 `JSON_SET()` 的讨论。

    ```sql
    mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
    mysql> SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]');
    +-----------------------------------------------------+
    | JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]') |
    +-----------------------------------------------------+
    | {"a": 10, "b": [2, 3]}                              |
    +-----------------------------------------------------+
    ```

+   [`JSON_SET(*`json_doc`*, *`path`*, *`val`*[, *`path`*, *`val`*] ...)`](json-modification-functions.html#function_json-set)

    在 JSON 文档中插入或更新数据并返回结果。如果 *`json_doc`* 或 *`path`* 为 `NULL`，或者给定 *`path`* 时未定位到对象，则返回 `NULL`。否则，如果 *`json_doc`* 参数不是有效的 JSON 文档，或任何 *`path`* 参数不是有效的路径表达式或包含 `*` 或 `**` 通配符，则会发生错误。

    路径-值对从左到右进行评估。通过评估一个对产生的文档成为下一个对进行评估的新值。

    文档中现有路径的路径-值对将使用新值覆盖现有文档值。文档中不存在的路径的路径-值对将在路径标识以下类型值之一时将值添加到文档中：

    +   一个不在现有对象中的成员。该成员将添加到对象中并与新值关联。

    +   超出现有数组末尾的位置。数组将使用新值扩展。如果现有值不是数组，则会自动包装为数组，然后使用新值扩展。

    否则，文档中不存在的路径的路径-值对将被忽略且不起作用。

    在 MySQL 8.0.4 中，优化器可以对 `JSON` 列执行部分、原地更新，而不是将旧文档删除并完整地写入新文档到列中。这种优化可以用于使用 `JSON_SET()` 函数的更新语句，并满足 JSON 值的部分更新 中概述的条件。

    `JSON_SET()`、`JSON_INSERT()` 和 `JSON_REPLACE()` 函数相关联：

    +   `JSON_SET()` 替换现有值并添加不存在的值。

    +   `JSON_INSERT()` 插入值而不替换现有值。

    +   `JSON_REPLACE()` *仅* 替换现有值。

    以下示例说明了这些差异，使用一个存在于文档中的路径 (`$.a`) 和另一个不存在的路径 (`$.c`)：

    ```sql
    mysql> SET @j = '{ "a": 1, "b": [2, 3]}';
    mysql> SELECT JSON_SET(@j, '$.a', 10, '$.c', '[true, false]');
    +-------------------------------------------------+
    | JSON_SET(@j, '$.a', 10, '$.c', '[true, false]') |
    +-------------------------------------------------+
    | {"a": 10, "b": [2, 3], "c": "[true, false]"}    |
    +-------------------------------------------------+
    mysql> SELECT JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]');
    +----------------------------------------------------+
    | JSON_INSERT(@j, '$.a', 10, '$.c', '[true, false]') |
    +----------------------------------------------------+
    | {"a": 1, "b": [2, 3], "c": "[true, false]"}        |
    +----------------------------------------------------+
    mysql> SELECT JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]');
    +-----------------------------------------------------+
    | JSON_REPLACE(@j, '$.a', 10, '$.c', '[true, false]') |
    +-----------------------------------------------------+
    | {"a": 10, "b": [2, 3]}                              |
    +-----------------------------------------------------+
    ```

+   `JSON_UNQUOTE(*`json_val`*)`

    取消 JSON 值的引号并将结果作为 `utf8mb4` 字符串返回。如果参数为 `NULL`，则返回 `NULL`。如果值以双引号开头和结尾但不是有效的 JSON 字符串文字，则会发生错误。

    在字符串中，除非启用了`NO_BACKSLASH_ESCAPES` SQL 模式，否则某些序列具有特殊含义。每个序列都以反斜杠（`\`）开头，称为*转义字符*。MySQL 识别表格 14.23, “JSON_UNQUOTE() 特殊字符转义序列” 特殊字符转义序列")中显示的转义序列。对于所有其他转义序列，反斜杠将被忽略。也就是说，转义字符将被解释为未转义的字符。例如，`\x` 就是 `x`。这些序列是区分大小写的。例如，`\b` 被解释为退格，但 `\B` 被解释为 `B`。

    **表格 14.23 JSON_UNQUOTE() 特殊字符转义序列**

    | 转义序列 | 序列表示的字符 |
    | --- | --- |
    | `\"` | 双引号（`"`）字符 |
    | `\b` | 退格字符 |
    | `\f` | 换页字符 |
    | `\n` | 换行（换行）字符 |
    | `\r` | 回车字符 |
    | `\t` | 制表符 |
    | `\\` | 反斜杠（`\`）字符 |
    | `\u*`XXXX`*` | Unicode 值 *`XXXX`* 的 UTF-8 字节 |

    这个函数的两个简单示例如下所示：

    ```sql
    mysql> SET @j = '"abc"';
    mysql> SELECT @j, JSON_UNQUOTE(@j);
    +-------+------------------+
    | @j    | JSON_UNQUOTE(@j) |
    +-------+------------------+
    | "abc" | abc              |
    +-------+------------------+
    mysql> SET @j = '[1, 2, 3]';
    mysql> SELECT @j, JSON_UNQUOTE(@j);
    +-----------+------------------+
    | @j        | JSON_UNQUOTE(@j) |
    +-----------+------------------+
    | [1, 2, 3] | [1, 2, 3]        |
    +-----------+------------------+
    ```

    以下一组示例展示了`JSON_UNQUOTE`如何处理启用和禁用`NO_BACKSLASH_ESCAPES`时的转义：

    ```sql
    mysql> SELECT @@sql_mode;
    +------------+
    | @@sql_mode |
    +------------+
    |            |
    +------------+

    mysql> SELECT JSON_UNQUOTE('"\\t\\u0032"');
    +------------------------------+
    | JSON_UNQUOTE('"\\t\\u0032"') |
    +------------------------------+
    |       2                           |
    +------------------------------+

    mysql> SET @@sql_mode = 'NO_BACKSLASH_ESCAPES';
    mysql> SELECT JSON_UNQUOTE('"\\t\\u0032"');
    +------------------------------+
    | JSON_UNQUOTE('"\\t\\u0032"') |
    +------------------------------+
    | \t\u0032                     |
    +------------------------------+

    mysql> SELECT JSON_UNQUOTE('"\t\u0032"');
    +----------------------------+
    | JSON_UNQUOTE('"\t\u0032"') |
    +----------------------------+
    |       2                         |
    +----------------------------+
    ```
