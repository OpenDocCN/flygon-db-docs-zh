# 15.7.1 账户管理语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/account-management-statements.html`](https://dev.mysql.com/doc/refman/8.0/en/account-management-statements.html)

15.7.1.1 修改用户语句

15.7.1.2 创建角色语句

15.7.1.3 创建用户语句

15.7.1.4 删除角色语句

15.7.1.5 删除用户语句

15.7.1.6 授权语句

15.7.1.7 重命名用户语句

15.7.1.8 撤销语句

15.7.1.9 设置默认角色语句

15.7.1.10 设置密码语句

15.7.1.11 设置角色语句

MySQL 账户信息存储在`mysql`系统模式的表中。这个数据库和访问控制系统在第七章，*MySQL 服务器管理*中有详细讨论，您应该查阅以获取更多细节。

重要

一些 MySQL 版本对授权表进行更改以添加新的权限或功能。为了确保您能够利用任何新功能，每次升级 MySQL 时都要将授权表更新到当前结构。请参阅第三章，*升级 MySQL*。

当`read_only`系统变量启用时，账户管理语句需要`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限），除了其他所需权限。这是因为它们修改了`mysql`系统模式中的表。

账户管理语句是原子性的并且具有崩溃安全性。更多信息请参见第 15.1.1 节，“原子数据定义语句支持”。
