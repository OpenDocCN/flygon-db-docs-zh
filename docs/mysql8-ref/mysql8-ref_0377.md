# 8.2.17 可插拔认证

> 原文：[`dev.mysql.com/doc/refman/8.0/en/pluggable-authentication.html`](https://dev.mysql.com/doc/refman/8.0/en/pluggable-authentication.html)

当客户端连接到 MySQL 服务器时，服务器使用客户端提供的用户名和客户端主机来从`mysql.user`系统表中选择适当的账户行。然后服务器对客户端进行认证，从账户行确定适用于客户端的认证插件：

+   如果服务器找不到插件，则会发生错误并拒绝连接尝试。

+   否则，服务器会调用该插件来对用户进行认证，插件会向服务器返回一个状态，指示用户是否提供了正确的密码并被允许连接。

可插拔认证使得以下重要功能成为可能：

+   **认证方法的选择。** 可插拔的认证使得数据库管理员可以轻松选择和更改用于各个 MySQL 账户的认证方法。

+   **外部认证。** 可插拔认证使得客户端可以使用适合于在`mysql.user`系统表之外存储凭据的认证方法进行连接到 MySQL 服务器。例如，可以创建插件来使用外部认证方法，如 PAM、Windows 登录 ID、LDAP 或 Kerberos。

+   **代理用户：** 如果用户被允许连接，认证插件可以向服务器返回一个与连接用户名称不同的用户名，以指示连接用户是另一个用户（被代理用户）的代理。在连接持续期间，代理用户在访问控制方面被视为具有被代理用户的权限。实际上，一个用户冒充另一个用户。更多信息，请参见第 8.2.19 节，“代理用户”。

注意

如果使用`--skip-grant-tables`选项启动服务器，则即使加载了认证插件，也不会使用认证插件，因为服务器不执行客户端认证并允许任何客户端连接。由于这是不安全的，如果使用`--skip-grant-tables`选项启动服务器，则还会通过启用`skip_networking`来禁用远程连接。

+   可用认证插件

+   默认认证插件

+   认证插件使用

+   认证插件客户端/服务器兼容性

+   认证插件连接器编写注意事项

+   可插拔认证的限制

#### 可用的认证插件

MySQL 8.0 提供了这些认证插件：

+   一个执行本地认证的插件；即，在 MySQL 引入可插拔认证之前使用的基于密码哈希方法的认证。`mysql_native_password`插件实现了基于此本地密码哈希方法的认证。参见 Section 8.4.1.1，“本地可插拔认证”。

    注意

    截至 MySQL 8.0.34，`mysql_native_password`认证插件已被弃用，并可能在将来的 MySQL 版本中被移除。

+   使用 SHA-256 密码哈希进行认证的插件。这比本地认证提供的加密更强大。参见 Section 8.4.1.2，“缓存 SHA-2 可插拔认证”，以及 Section 8.4.1.3，“SHA-256 可插拔认证”。

+   一个客户端插件，将密码以明文形式发送到服务器，不经过哈希或加密。此插件与需要以客户端用户提供的密码完全一致的服务器端插件一起使用。参见 Section 8.4.1.4，“客户端明文插件认证”。

+   一个使用 PAM（可插拔认证模块）执行外部认证的插件，使 MySQL 服务器能够使用 PAM 对 MySQL 用户进行认证。此插件还支持代理用户。参见 Section 8.4.1.5，“PAM 可插拔认证”。

+   一个在 Windows 上执行外部认证的插件，使 MySQL 服务器能够使用本地 Windows 服务对客户端连接进行认证。已登录到 Windows 的用户可以根据其环境中的信息从 MySQL 客户端程序连接到服务器，而无需指定额外的密码。此插件还支持代理用户。参见 Section 8.4.1.6，“Windows 可插拔认证”。

+   使用 LDAP（轻量级目录访问协议）进行认证的插件，通过访问诸如 X.500 之类的目录服务来认证 MySQL 用户。这些插件还支持代理用户。参见第 8.4.1.7 节，“LDAP 可插拔认证”。

+   一个使用 Kerberos 进行认证的插件，用于认证与 Kerberos 主体对应的 MySQL 用户。参见第 8.4.1.8 节，“Kerberos 可插拔认证”。

+   一个阻止所有使用它的帐户的客户端连接的插件。此插件的用例包括永远不允许直接登录但仅通过代理帐户访问的代理帐户和必须能够以提升的权限执行存储程序和视图而不向普通用户公开这些权限的帐户。参见第 8.4.1.9 节，“无登录可插拔认证”。

+   一个插件，用于验证通过 Unix 套接字文件从本地主机连接的客户端。参见第 8.4.1.10 节，“套接字对等凭证可插拔认证”。

+   一个通过 FIDO 认证将用户认证到 MySQL 服务器的插件。参见第 8.4.1.11 节，“FIDO 可插拔认证”。

    注意

    截至 MySQL 8.0.35 版本，`authentication_fido`和`authentication_fido_client`认证插件已被弃用，并可能在将来的 MySQL 版本中被移除。

+   一个测试插件，检查帐户凭据并将成功或失败记录到服务器错误日志中。此插件旨在用于测试和开发目的，并作为编写认证插件的示例。参见第 8.4.1.12 节，“测试可插拔认证”。

注意

有关当前对可插拔认证使用的限制的信息，包括哪些连接器支持哪些插件，请参阅可插拔认证限制。

第三方连接器开发人员应阅读该部分，以确定连接器可以利用可插拔认证功能的程度，以及成为更加符合规范的步骤。

如果您有兴趣编写自己的认证插件，请参阅编写认证插件。

#### 默认认证插件

`CREATE USER`和`ALTER USER`语句有用于指定账户如何进行认证的语法。这种语法的某些形式没有明确指定认证插件（没有`IDENTIFIED WITH`子句）。例如：

```sql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY '*password*';
```

在这种情况下，服务器会为账户分配默认的认证插件。在 MySQL 8.0.27 之前，这个默认值是`default_authentication_plugin`系统变量的值。

截至 MySQL 8.0.27，引入了多因素认证，可以有最多三个子句指定账户如何进行认证。确定没有指定插件的认证方法的默认认证插件的规则是特定于因素的：

+   因素 1：如果`authentication_policy`元素 1 指定了一个认证插件，那个插件就是默认的。如果`authentication_policy`元素 1 是`*`，那么`default_authentication_plugin`的值就是默认的。

    根据上述规则，以下语句创建了一个双因素认证账户，第一因素认证方法由`authentication_policy`或`default_authentication_plugin`设置确定：

    ```sql
    CREATE USER 'wei'@'localhost' IDENTIFIED BY '*password*'
      AND IDENTIFIED WITH authentication_ldap_simple;
    ```

    同样地，这个示例创建了一个三因素认证账户：

    ```sql
    CREATE USER 'mateo'@'localhost' IDENTIFIED BY '*password*'
      AND IDENTIFIED WITH authentication_ldap_simple
      AND IDENTIFIED WITH authentication_fido;
    ```

    你可以使用`SHOW CREATE USER`来查看应用的认证方法。

+   因素 2 或 3：如果相应的`authentication_policy`元素指定了一个认证插件，那个插件就是默认的。如果`authentication_policy`元素是`*`或空的，那么就没有默认值；尝试为没有指定插件的因素定义账户认证方法是一个错误，就像以下示例一样：

    ```sql
    mysql> CREATE USER 'sofia'@'localhost' IDENTIFIED WITH authentication_ldap_simple 
           AND IDENTIFIED BY 'abc';
    ERROR 1524 (HY000): Plugin '' is not loaded 
    mysql> CREATE USER 'sofia'@'localhost' IDENTIFIED WITH authentication_ldap_simple 
           AND IDENTIFIED BY 'abc';
    ERROR 1524 (HY000): Plugin '*' is not loaded
    ```

#### 认证插件使用

本节提供了安装和使用认证插件的一般说明。有关特定插件的说明，请参阅描述该插件的部分，位于 Section 8.4.1, “Authentication Plugins”下。

一般来说，可插拔认证在服务器和客户端两侧使用一对相应的插件，因此你可以像这样使用给定的认证方法：

+   如果需要，安装包含适当插件的插件库或插件库。在服务器主机上，安装包含服务器端插件的库，以便服务器可以用它来验证客户端连接。同样，在每个客户端主机上，安装包含客户端插件的库，供客户端程序使用。内置的认证插件无需安装。

+   对于每个创建的 MySQL 帐户，指定要用于验证的适当服务器端插件。如果帐户要使用默认的认证插件，则帐户创建语句无需明确指定插件。服务器分配默认的认证插件，如默认认证插件中所述确定的那样。

+   当客户端连接时，服务器端插件告诉客户端程序要使用哪个客户端插件进行验证。

如果一个帐户使用的认证方法对于服务器和客户端程序都是默认的，那么服务器无需通知客户端要使用哪个客户端插件，可以避免客户端/服务器协商中的往返。

对于标准的 MySQL 客户端，如**mysql**和**mysqladmin**，可以在命令行上指定`--default-auth=*`plugin_name`*`选项，作为程序可以期望使用的客户端插件的提示，尽管如果与用户帐户关联的服务器端插件需要不同的客户端插件，则服务器会覆盖此设置。

如果客户端程序找不到客户端插件库文件，请指定`--plugin-dir=*`dir_name`*`选项以指示插件库目录位置。

#### 认证插件客户端/服务器兼容性

可插拔认证为 MySQL 帐户的认证方法选择提供了灵活性，但在某些情况下，由于客户端和服务器之间的认证插件不兼容，可能无法建立客户端连接。

对于成功连接到给定服务器上给定帐户的客户端的一般兼容性原则是，客户端和服务器都必须支持帐户所需的认证*方法*。因为认证方法由认证插件实现，所以客户端和服务器都必须支持帐户所需的认证*插件*。

认证插件不兼容可能以各种方式出现。例如：

+   使用 5.7.22 或更低版本的 MySQL 5.7 客户端连接到一个使用`caching_sha2_password`进行认证的 MySQL 8.0 服务器账户。这会失败，因为 5.7 客户端不识别这个插件，这个插件是在 MySQL 8.0 中引入的。（这个问题在 MySQL 5.7 中已经得到解决，从 5.7.23 开始，MySQL 客户端库和客户端程序添加了对`caching_sha2_password`的客户端支持。）

+   使用 MySQL 5.7 客户端连接到一个使用`mysql_old_password`进行认证的低于 5.7 版本的服务器账户。这会因为多种原因而失败。首先，这样的连接需要`--secure-auth=0`，这不再是一个支持的选项。即使它被支持，5.7 客户端也不会识别这个插件，因为它在 MySQL 5.7 中被移除了。

+   使用来自 Community 发行版的 MySQL 5.7 客户端连接到一个使用企业专用 LDAP 认证插件之一进行认证的 MySQL 5.7 企业服务器账户。这会失败，因为 Community 客户端无法访问企业插件。

一般来说，当客户端和服务器来自同一 MySQL 发行版时，这些兼容性问题不会出现。当客户端和服务器来自不同的 MySQL 系列时，问题可能会出现。当 MySQL 引入新的认证插件或移除旧插件时，这些问题是开发过程中固有的。为了最大程度地减少不兼容性的可能性，定期及时升级服务器、客户端和连接器。

#### 认证插件连接器编写注意事项

存在多种 MySQL 客户端/服务器协议的实现。`libmysqlclient` C API 客户端库是其中一种实现。一些 MySQL 连接器（通常不是用 C 语言编写的）提供了它们自己的实现。然而，并非所有协议实现都以相同的方式处理插件认证。本节描述了协议实现者应该考虑的一个认证问题。

在客户端/服务器协议中，服务器会告诉连接的客户端它认为是默认的认证插件。如果客户端使用的协议实现尝试加载默认插件，而该插件在客户端上不存在，加载操作将失败。如果默认插件不是客户端尝试连接的账户实际需要的插件，这是一个不必要的失败。

如果客户端/服务器协议实现没有自己的默认认证插件概念，并始终尝试加载服务器指定的默认插件，如果该插件不可用，则会出现错误。

为了避免这个问题，客户端使用的协议实现应该有自己的默认插件，并将其作为首选（或者在无法加载服务器指定的默认插件时，回退到这个默认插件）。例如：

+   在 MySQL 5.7 中，`libmysqlclient`默认选择要么是`mysql_native_password`，要么是通过`MYSQL_DEFAULT_AUTH`选项为`mysql_options()`指定的插件。

+   当 5.7 客户端尝试连接到 8.0 服务器时，服务器将`caching_sha2_password`指定为其默认认证插件，但客户端仍然根据`mysql_native_password`或通过`MYSQL_DEFAULT_AUTH`指定的凭据详细信息发送。

+   客户端仅在更改插件请求时加载服务器指定的插件，但在这种情况下，它可以是任何取决于用户帐户的插件。在这种情况下，客户端必须尝试加载插件，如果该插件不可用，则错误是不可选的。

#### 可插拔认证限制

本节的第一部分描述了在第 8.2.17 节，“可插拔认证”中描述的可插拔认证框架的适用性的一般限制。第二部分描述了第三方连接器开发人员如何确定连接器可以利用可插拔认证功能的程度以及采取哪些步骤以变得更加符合规范。

此处使用的“本机认证”术语指的是针对存储在`mysql.user`系统表中的密码进行的认证。这是在实施可插拔认证之前由较旧的 MySQL 服务器提供的相同认证方法。“Windows 本机认证”是指使用已经登录到 Windows 的用户的凭据进行认证，由 Windows 本机认证插件（简称“Windows 插件”）实施。

+   通用可插拔认证限制

+   可插拔认证和第三方连接器

##### 通用可插拔认证限制

+   **Connector/C++:** 使用此连接器的客户端只能通过使用本机认证的帐户连接到服务器。

    例外：如果连接器是动态链接到`libmysqlclient`（而不是静态链接）构建的，并且如果安装了该版本，则加载当前版本的`libmysqlclient`，或者如果连接器从源代码重新编译以链接到当前的`libmysqlclient`。

    有关编写连接器以处理有关默认服务器端认证插件信息的信息，请参阅认证插件连接器编写注意事项。

+   **Connector/NET：** 使用 Connector/NET 的客户端可以通过使用本地认证或 Windows 本地认证的帐户连接到服务器。

+   **Connector/PHP：** 使用此连接器的客户端只能通过使用 MySQL 本机驱动程序（`mysqlnd`）编译时使用本地认证的帐户连接到服务器。

+   **Windows 本地认证：** 通过使用 Windows 插件的帐户连接需要 Windows 域设置。如果没有设置，将使用 NTLM 认证，然后只有本地连接是可能的；也就是说，客户端和服务器必须在同一台计算机上运行。

+   **代理用户：** 代理用户支持程度取决于客户端是否可以通过使用实现代理用户功能的插件进行身份验证的帐户连接（即，可以返回与连接用户不同的用户名的插件）。例如，PAM 和 Windows 插件支持代理用户。`mysql_native_password`和`sha256_password`认证插件默认不支持代理用户，但可以进行配置；请参阅服务器支持代理用户映射。

+   **复制：** 复制副本不仅可以使用本地认证的复制用户帐户，还可以通过使用非本地认证的复制用户帐户连接，如果所需的客户端端插件可用。如果插件内置于`libmysqlclient`中，则默认可用。否则，插件必须安装在副本端的由副本的`plugin_dir`系统变量命名的目录中。

+   **`FEDERATED`表：** `FEDERATED`表只能通过在远程服务器上使用本地认证的帐户访问远程表。

##### 可插拔认证和第三方连接器

第三方连接器开发人员可以使用以下准则来确定连接器是否准备好利用可插拔认证功能，并采取哪些步骤以提高兼容性：

+   对于未进行任何更改的现有连接器，使用本地认证，使用该连接器的客户端只能通过使用本地认证的帐户连接到服务器。*但是，您应该针对服务器的最新版本测试连接器，以验证这样的连接是否仍然可以正常工作。*

    异常情况：如果连接器动态链接到`libmysqlclient`（而不是静态链接），并且加载当前安装的`libmysqlclient`版本，则连接器可能可以在不进行任何更改的情况下与可插拔认证一起使用。

+   要利用可插拔认证功能，基于 `libmysqlclient` 的连接器应该重新链接到当前版本的 `libmysqlclient`。这使连接器能够支持通过现在内置到 `libmysqlclient` 中的客户端插件的账户进行连接（例如用于 PAM 认证所需的明文插件和用于 Windows 本地认证所需的 Windows 插件）。与当前的 `libmysqlclient` 链接还使连接器能够访问安装在默认 MySQL 插件目录中的客户端插件（通常是由本地服务器的 `plugin_dir` 系统变量的默认值命名的目录）。

    如果连接器动态链接到 `libmysqlclient`，必须确保在客户端主机上安装了更新版本的 `libmysqlclient`，并且连接器在运行时加载它。

+   连接器支持给定认证方法的另一种方式是直接在客户端/服务器协议中实现它。Connector/NET 使用这种方法来提供对 Windows 本地认证的支持。

+   如果连接器应该能够从与默认插件目录不同的目录加载客户端插件，那么它必须实现一些方式让客户用户指定目录。这方面的可能性包括通过命令行选项或环境变量，从中连接器可以获取目录名称。标准的 MySQL 客户端程序，如**mysql**和**mysqladmin**实现了一个 `--plugin-dir` 选项。另请参阅 C API 客户端插件接口。

+   连接器对代理用户的支持取决于，正如本节前面所述的，它支持的认证方法是否允许代理用户。