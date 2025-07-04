# *第八章*：SEPostgreSQL——通过 SELinux 扩展 PostgreSQL

在上一章中，我们讨论了几个示例 SELinux 感知应用程序：这些应用程序能够识别并与 SELinux 子系统交互，以进一步增强应用程序上下文中的安全性。部分应用程序使用现有的策略构件，例如 Apache 的`mod_selinux`，而其他应用程序则通过自定义类来增强策略，从而更精细地调整其行为（如 D-Bus 和`acquire_svc`权限）。

通过**安全增强型 PostgreSQL**（**SEPostgreSQL**），我们可以得到一个更复杂的 SELinux 感知应用程序示例，该程序使用多个额外的 SELinux 类，并对其内部数据库对象进行标签化，以进一步强化安全规则。在本章中，我们将学习如何在 PostgreSQL 中应用标签，调试其执行规则，将正确的标签与 PostgreSQL 资源关联，并展示这种基于标签的安全方法如何增强关系数据库中的特定安全实践。

在本章中，我们将涵盖以下主要主题：

+   介绍 PostgreSQL 和 `sepgsql`

+   理解 SELinux 的数据库特定对象类和权限

+   使用 MCS 和 MLS

+   将 SEPostgreSQL 集成到网络中

# 技术要求

请查看以下视频，了解代码的实际应用：[`bit.ly/3dDcg4Z`](https://bit.ly/3dDcg4Z)

# 介绍 PostgreSQL 和 sepgsql

PostgreSQL 是一个流行的、功能丰富且成熟的关系数据库管理系统。与 Apache 类似，它也通过可加载模块支持功能的模块化扩展。我们将研究的模块称为`sepgsql`，它为 PostgreSQL 提供了 SELinux 支持，以实现额外的访问控制，提供基于 SELinux 策略规则的细粒度数据流控制。

请注意，`sepgsql`并没有在 PostgreSQL 中实现完整的强制访问控制系统，因为并非所有的 PostgreSQL 语句都会导致策略检查。虽然它增强了 PostgreSQL 数据库的安全性，但该模块也有一些限制，详见其在线文档，网址为[`www.postgresql.org/docs/10/sepgsql.html`](https://www.postgresql.org/docs/10/sepgsql.html)（如有需要，请根据版本号调整 URL；此 URL 中的参考文档适用于 PostgreSQL 10，这是 CentOS 8 中使用的版本，也是本章所使用的版本）。

## 使用 sepgsql 重新配置 PostgreSQL

在我们安装`sepgsql`之前，必须确保有一个可用的 PostgreSQL 系统。大多数 Linux 发行版都有现成的教程，指导如何部署 PostgreSQL，这通常涉及创建与之关联的数据库。

在本章中，我们假设数据库本身位于`/var/lib/pgsql/data`，这是基于 CentOS 的 PostgreSQL 安装的默认位置。PostgreSQL 的配置文件也位于此位置。

要安装 `sepgsql`，应执行以下步骤：

1.  让我们首先通过以（默认）`postgres` 超级用户身份登录，并列出当前可用的数据库，来查看数据库是否正常工作：

    ```
    /var/lib/pgsql/data/log to get more information. This log file is the default log file for all PostgreSQL-related activities, as we will see when troubleshooting its SELinux support in the *Troubleshooting sepgsql* section.
    ```

1.  假设 PostgreSQL 正常工作，接下来我们将配置它使用 `sepgsql` 模块。该模块是 PostgreSQL 中的一个贡献模块，由 PostgreSQL 社区维护。在 CentOS 中，`sepgsql` 模块属于 `postgresql-contrib` 包，如果系统中尚未安装，可以通过 `yum install postgresql-contrib` 命令轻松安装。

1.  编辑 `/var/lib/pgsql/data` 目录下的 `postgresql.conf` 文件，并查找 `shared_preload_libraries` 语句。默认情况下，它会被注释掉，因此需要取消注释，并在其中添加 `sepgsql`：

    ```
    shared_preload_libraries = 'sepgsql' # (change requires restart)
    ```

1.  如前所述，修改此参数需要重新启动数据库。我们稍后会进行，但首先，我们将关闭数据库，因为接下来的操作需要在离线模式下进行：

    ```
    # systemctl stop postgresql
    ```

1.  接下来，我们需要重新配置所有数据库并启用与 `sepgsql` 相关的函数。我们将在 *使用 sepgsql 特定函数* 部分讲解这些函数。为了启用这些函数，我们需要再次成为 `postgres` 超级用户，并且对每个可用的数据库加载特定的 SQL 文件：

    ```
    \l command, which we used earlier to check whether the database is functioning properly.
    ```

1.  让我们通过启动 PostgreSQL 数据库、登录到 PostgreSQL 并请求当前上下文来验证 `sepgsql` 是否正常工作：

    ```
    sepgsql function sepgsql_getcon(), which retrieves the current context for the session.
    ```

让我们进一步配置数据库，创建一个测试账户，用于验证 `sepgsql` 控制：

## 创建测试账户

为了验证 `sepgsql` 控制是否生效，我们应该创建一个非 `postgres` 超级用户的测试账户，并且创建一个本地用户，以便将其映射到不同的 SELinux 上下文中。由于 SELinux 上下文会决定会话所关联的权限，我们希望能够展示不同上下文之间的影响。

首先，在 PostgreSQL 中（使用 `postgres` 超级用户），创建一个名为 `testuser` 的测试账户，并允许该账户使用指定密码进行身份验证：

```
postgres=# CREATE USER testuser PASSWORD 'somepassword';
```

我们还需要配置数据库以允许基于密码的身份验证（因为默认的 PostgreSQL 配置会使用系统信任或其他身份验证方式）。为此，在 `/var/lib/pgsql/data` 目录中编辑 `pg_hba.conf` 文件，并进行如下设置：

```
local		all	postgres					peer
local		all	testuser					md5
host		all	testuser		127.0.0.1/32	md5
host		all	testuser		192.168.100.0/24	md5
```

`pg_hba.conf` 文件管理 PostgreSQL 的基于主机的身份验证规则。我们更新该文件，允许 `testuser` 账户（使用 `md5` 作为标识符）进行基于密码的身份验证，同时允许 `postgres` 超级用户继续使用对等信任进行身份验证。

在这些更改生效后，PostgreSQL 允许 `testuser` 账户进行基于密码的身份验证，无论是当用户通过本地基于套接字的方式发起通信，还是通过网络通信方式进行连接。

我们还需要告诉 SELinux 策略，允许普通用户连接到 PostgreSQL 服务：

```
# setsebool -P selinuxuser_postgresql_connect_enabled on
```

虽然这足以访问 PostgreSQL 服务，但不足以允许常规用户域（`user_t`）与`sepgsql`交互。为了实现这一点，我们需要调整 SELinux 策略，使得`user_t`域也与`sepgsql_client_type`属性相关联，并且`user_r`角色可以激活与`sepgsql`相关的类型。

我们通过一个小的 CIL 策略来实现此操作，如下所示：

```
(typeattributeset cil_gen_require sepgsql_client_type)
(typeattributeset cil_gen_require user_t)
(typeattributeset cil_gen_require sepgsql_trusted_proc_t)
(typeattributeset cil_gen_require sepgsql_ranged_proc_t)
(typeattributeset sepgsql_client_type (user_t))
(roleattributeset cil_gen_require user_r)
(roletype user_r sepgsql_trusted_proc_t)
(roletype user_r sepgsql_ranged_proc_t)
```

也可以通过参考策略风格模块来完成此操作，如下所示：

```
policy_module(local_sepgsql, 1.0)
gen_require(`
	role user_r;
	type user_t;
')
postgresql_role(user_r, user_t)
```

假设我们继续使用基于 CIL 的策略，让我们将文件（即`local_sepgsql.cil`）加载为 SELinux 策略模块：

```
# semodule -i local_sepgsql.cil
```

在更改`pg_hba.conf`文件后，别忘了重启 PostgreSQL 服务。

## 在 PostgreSQL 内部调整 sepgsql

`sepgsql`模块引入了两个配置参数，可以用于调整 PostgreSQL 内部的`sepgsql`：

+   `sepgsql.permissive`参数告诉 PostgreSQL 在 PostgreSQL 内部不强制执行 SELinux 策略规则。这类似于 SELinux 在系统上的宽容状态，但仅涵盖 PostgreSQL 内部的`sepgsql`相关功能。

+   `sepgsql.debug_audit`参数告诉 PostgreSQL 始终记录与 SELinux 相关的决策，即使它们是允许处理某个语句。这类似于系统上 SELinux 的`auditallow`语句。

然而，非常重要的一点是要理解，正如在*第七章*中解释的那样，`sepgsql`是一个用户空间对象管理器，*配置应用程序特定的 SELinux 控制*：Linux 内核中的 SELinux 子系统并不用于强制执行访问控制，只有`sepgsql`才负责。SELinux 子系统的唯一作用是允许 PostgreSQL 查询当前的 SELinux 策略或获取当前的 SELinux 上下文信息。

因此，之前的配置参数大多独立于系统的配置工作。虽然 SELinux 必须在系统上激活，但它不需要处于强制模式才能使`sepgsql`在 PostgreSQL 内部强制执行规则，系统处于宽容模式也不会使`sepgsql`的强制执行变为宽容模式。

`sepgsql.debug_audit`参数与系统策略有一定关系。我们可以在 SELinux 策略中添加`auditallow`语句，以强制记录即使是被允许的事件。`sepgsql.debug_audit`参数的作用是强制记录所有事件，这对排查`sepgsql`问题非常有用，接下来我们将看到这一点。

## 排查 sepgsql 问题

让我们为单个会话启用调试语句，并再次调用`sepgsql_getcon`函数：

```
# su postgres -c "/usr/bin/psql postgres"
postgres=# SET sepgsql.debug_audit = true;
SET
postgres=# SELECT sepgsql_getcon();
...
```

如果你想为整个系统启用此配置，可以将配置放入`postgresql.conf`文件中：

```
sepgsql.debug_audit = true
```

在 PostgreSQL 日志中，我们将注意到以下信息：

```
STATEMENT:  SET sepgsql.debug_audit = true
STATEMENT:  SELECT sepgsql_getcon();
LOG:  SELinux: allowed { execute } \
  scontext=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 \
  tcontext=system_u:object_r:sepgsql_proc_exec_t:s0 \
  tclass=db_procedure name="pg_catalog.sepgsql_getcon()"
```

前两行记录了我们在会话中执行的语句，而第三行是与执行 `sepgsql_getcon` 相关的 SELinux 日志事件。

该事件告诉我们，`unconfined_t` 域（源上下文）尝试（并成功）执行数据库过程（如 `db_procedure` 类所示），该过程标记为 `sepgsql_proc_exec_t` 类型。数据库中的功能是 `pg_catalog` 模式中的 `sepgsql_getcon` 函数。

如果发生拒绝，这将导致日志中出现类似的事件，并且拒绝触发的最终用户将看到错误信息，PostgreSQL 会显示类似以下的错误消息：

```
ERROR:  SELinux: security policy violation
```

与例如 D-Bus 执行的审计日志（会在常规审计日志中产生 `USER_AVC` 事件）不同，`sepgsql` 将遵循 PostgreSQL 数据库本身的日志配置，因此在尝试故障排除 `sepgsql` 时，请密切关注该日志文件（或 PostgreSQL 中配置的其他日志目标）。

在这个简单的示例中，你可能已经注意到事件引用了一个数据库特定的类（`db_procedure`）。在接下来的部分，我们将深入探讨与 `sepgsql` 相关的各种类、权限和类型，并因此支持 SELinux 策略。

# 理解 SELinux 的数据库特定对象类和权限

`sepgsql` 模块使用多个数据库特定的 SELinux 类来精细调控策略和访问控制。可以通过 `/sys/fs/selinux/class` 或 `seinfo` 命令列出支持的类：

```
# seinfo --class | grep db_
db_blob
db_column
db_database
db_language
db_procedure
db_schema
db_sequence
db_table
db_tuple
db_view
```

这些类具有明显的关系数据库含义：`db_database` 用于与数据库相关的权限，`db_table` 用于表权限，`db_procedure` 用于数据库过程，依此类推。虽然并非所有类都仍然被 `sepgsql` 支持（`db_database` 类不再直接支持），但大多数类仍然在 PostgreSQL 数据库中具有其常规映射。

让我们看看 `sepgsql` 支持哪些权限，以及如何利用这些权限在数据库中精细调控访问控制。

## 理解 sepgsql 权限

`sepgsql` 强制执行的访问控制是在 PostgreSQL 已经支持的自主访问控制基础上实现的。与其使用当前在数据库中操作的角色或用户的权限，`sepgsql` 模块将使用与会话关联的上下文。

由于我们可以为使用相同数据库角色认证的会话使用不同的 SELinux 上下文，我们可以在数据库中创建不同的访问控制，而无需将其与用户帐户本身关联。例如，我们可以根据数据库会话的初始化方式进行区分：远程会话可能与本地启动的会话具有不同的上下文，或者即使在数据库中共享相同帐户，不同的 Linux 用户也可能具有独特的授权。

重要说明

由于远程连接需要对等方上下文是可访问的，`sepgsql`要求使用带标签的 IPSec，或者我们需要通过 NetLabel 和 CIPSO 引入回退标签，正如在*第五章*中看到的，*控制网络通信*。我们将在解释各种权限映射后，在*将 SEPostgreSQL 集成到网络中*部分中建立这样的映射。

一旦登录，对表的查询将触发针对 SELinux 策略的几项检查：

+   对表执行任何`SELECT`、`INSERT`、`UPDATE`或`DELETE`语句都会导致对`db_table`类中的`select`、`insert`、`update`或`delete`权限进行权限检查。

+   当`WHERE`子句列出一个或多个不同的表时，还会检查这些不同表的`select`权限。

+   此外，还会检查每个引用列的列级权限，并且这会与`db_column`类中的权限进行比较。再次强调，对`select`权限的检查验证了读取访问权限，而`update`或`insert`权限反映了在值发生变化时需要检查的控制。

有关支持的权限的更详细概述，可以参考 PostgreSQL 的`sepgsql`文档。

## 使用默认支持的类型

默认的 SELinux 策略有多种类型可以在`sepgsql`设置中直接使用。大多数 SEPostgreSQL 配置不会偏离这些默认类型，而是依赖于我们在*第三章*中提到的基于类别和敏感度的控制，*管理用户登录*。

要查看这些默认类型是什么，它们的用途以及如何在 PostgreSQL 中分配这些标签，我们先从创建一个名为`db_test`的新数据库开始：

```
# su postgres -c "/usr/bin/psql postgres"
postgres=# CREATE DATABASE db_test;
CREATE DATABASE
```

接下来，我们连接到这个新创建的数据库，并创建一个简单的表，命名为`tb_users`，它有以下列：

+   用户的 ID，命名为`uid`

+   用户的姓名，命名为`name`

+   用户的电子邮件地址，命名为`mail`

+   用户的邮寄地址，命名为`address`

+   用户的密码盐和哈希，分别命名为`salt`和`phash`

    重要提示

    所使用的示例仅仅是一个示范，旨在展示如何处理 SELinux 标签和`sepgsql`。适当的数据库设计以及处理密码哈希和其他敏感数据的最佳实践远远超出了本书的范围！

正如你所想，我们将进一步保护这些列：虽然密码哈希显然应该被视为非常敏感的信息，但我们还应该确保妥善保护邮件和地址字段，因为这些是**个人身份信息**（**PII**），在世界许多地方受特定隐私法律的管辖：

```
postgres=# \c db_test;
db_test=# CREATE TABLE tb_users(uid int primary key, name text, mail text, address text, salt text, phash text);
```

现在，这个表关联的标签是什么？为此，我们需要查询 PostgreSQL 的内部表/视图，特别是`pg_seclabels`：

```
db_test=# SELECT objname,provider,label FROM pg_seclabels WHERE objname='tb_users';
 objname  | provider | label
----------+----------+----------------------------------------
 tb_users | selinux  | unconfined_u:object_r:sepgsql_table_t:s0
```

如你所见，表已获得 `sepgsql_table_t` 类型和默认敏感度（`s0`）。

`sepgsql_table_t` 是表的默认类型。我们通常会看到该类型用于一般的表支持和列。除了 `sepgsql_table_t` 类型外，策略还有一些其他表和列相关的类型，管理员可以用来区分 `sepgsql` 强制执行的控制：

+   `sepgsql_fixed_table_t` 类型可用于只能追加（插入）但不能更新的表或列。这可以用于与日志相关的表或审计事件，我们希望通过 `sepgsql` 控制进一步加强这一点（超出可以用于此的数据库内控件）。

+   `sepgsql_ro_table_t` 类型可用于只能读取（只读）的表或列。

+   `sepgsql_secret_table_t` 类型可用于常规用户或会话无法访问的表或列，只能由管理员访问。这通常用于只能通过受保护和/或特权程序访问的表或列。

+   `unpriv_sepgsql_table_t` 类型类似于 `sepgsql_table_t` 类型，但特定于由管理员或未受限用户管理的表或列，这些表或列无法被受限用户访问。

+   另一方面，`user_sepgsql_table_t` 类型专门为受限用户管理的表或列而构建。这使得管理员能够区分用户特定的表和一般表。

让我们授予 `testuser` 账户对该表和数据库的（完全）访问权限，并向表中添加一些数据：

```
db_test=# GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO testuser;
db_test=# INSERT INTO tb_users VALUES (1, 'Sven Vermeulen', 'some@example.com', 'Some Place 10001, Somewhere', 'abc123', 'f5ba94...3');
db_test=# INSERT INTO tb_users VALUES (2, 'Lisa McCarthy', 'lisa@example.com', 'Lisa Place 15, Someplace', 'def456', 'ba53f2...0');
```

如果我们通过测试用户查询数据，我们可以看到所有已添加到表中的数据：

```
db_test=> SELECT * FROM tb_users;
```

让我们将 `phash` 列的类型更改为 `sepgsql_secret_table_t`：

```
db_test# SECURITY LABEL ON COLUMN tb_users.phash IS 'system_u:object_r:sepgsql_secret_table_t:s0';
```

然而，仅此一点并不能防止 `testuser` 用户访问数据。这取决于 `testuser` 如何登录到数据库——会话是从哪个上下文启动的。如果我们从未受限域启动会话，那么该会话仍然允许访问数据。我们不如从常规用户会话（`user_t`）登录，并再次尝试访问数据：

```
db_test=> SELECT * FROM tb_users;
ERROR:  SELinux: security policy violation
```

即使用户在数据库中拥有所有权限，我们仍然注意到策略阻止了访问。然而，我们可以查询那些未标记为 `sepgsql_secret_table_t` 的列：

```
db_test=> SELECT uid, name, mail, address, salt from tb_users;
```

由于 `phash` 列现在被标记为 `sepgsql_secret_table_t`，我们仍然希望常规数据库用户能够查询哈希值是否与数据库中的哈希值匹配，或者设置新的哈希值。这使得数据库用户可以在不轻易泄漏密码哈希的情况下管理账户。我们通过函数实现这一点，接下来我们将描述这些函数。

## 创建受信任的过程

PostgreSQL 支持函数和过程，以更结构化和可管理的方式促进数据库内或数据上的操作的隔离或组合。过程允许在数据库中进行事务更新，但自身不返回值。函数返回值，但不允许进行事务更新。在我们的示例中，我们将创建两个函数，一个用于将哈希与存储的哈希进行比较（但不将存储的哈希显示给数据库用户），另一个用于更新存储的哈希。

信息提示

虽然我们应该为第二个函数使用过程，但并非所有今天使用的 PostgreSQL 版本都支持它们。支持过程的功能仅从 PostgreSQL 版本 11 开始引入，而我们的示例使用 PostgreSQL 10.6，因为那是 CentOS 8 支持的当前版本。

首先，我们来创建这两个函数：

```
postgres=# CREATE FUNCTION compare_hash(fuid int, fphash text) RETURNS boolean AS 'SELECT phash = regexp_replace(fphash, ''[^a-f0-9]*'', '''', ''g'') FROM tb_users WHERE uid = fuid' LANGUAGE sql;
postgres=# CREATE FUNCTION set_hash(fuid int, fphash text) RETURNS int AS 'UPDATE tb_users SET phash = regexp_replace(fphash, ''[^a-f0-9]*'', '''', ''g'') WHERE uid = fuid RETURNING uid' LANGUAGE sql;
```

我们在函数中引入了正则表达式来清理输入，因为我们稍后会将这些函数标记为受信任的，我们不希望这些函数成为 SQL 注入等攻击的跳板。

一旦定义了这些函数，授权用户可以使用它们来访问更多受保护的数据。当然，我们需要正确标记这些函数。在默认的 SELinux 策略中，以下类型可用于处理过程和函数：

+   `sepgsql_proc_exec_t` 是分配给常规函数或过程的类型。执行后，过程将在用户的当前上下文中运行，因此不会发生转换。

+   `sepgsql_trusted_proc_exec_t` 是分配给受信任的过程或函数的类型。执行后，这些函数将在 `sepgsql_trusted_proc_t` 域内运行，该域具有访问更多特权类型的权限，例如 `sepgsql_secret_table_t`。

+   `sepgsql_ranged_proc_exec_t` 是分配给受信任的过程或函数的类型，但它有一个额外的权限：允许范围过程改变当前的敏感性。范围过程权限对于可以访问当前上下文无法访问的类别标签列的函数或过程很有用。执行后，这些函数和过程将在 `sepgsql_ranged_proc_t` 域内运行。

+   用户管理的过程可以标记为 `unpriv_sepgsql_proc_exec_t`（对于未受限用户）和 `user_sepgsql_proc_t`（对于受限用户）。这些过程和函数将继续在用户域内运行。

要获取当前分配给函数的标签，可以使用 `LIKE` 语句，因为函数在定义时（在 `objname` 列中）包含了变量，因此它们并不总是那么显而易见，无法立即选择：

```
db_test=# SELECT objname,provider,label FROM pb_seclabels WHERE objname LIKE 'compare_hash%';
```

让我们将这些函数标记为受信任的：

```
db_test=# SECURITY LABEL ON FUNCTION compare_hash(fuid integer, fphash text) IS 'system_u:object_r:sepgsql_trusted_proc_exec_t:s0';
db_test=# SECURITY LABEL ON FUNCTION set_hash(fuid integer, fphash text) IS 'system_u:object_r:sepgsql_trusted_proc_exec_t:s0';
```

有了这些标签，数据库用户即使无法访问 `phash` 列本身，也可以执行适当的检查和更改：

```
db_test=> SELECT compare_hash(1, 'abc123');
f
db_test=> SELECT set_hash(1, 'abc123');
1
db_test=> SELECT compare_hash(1, 'abc123');
t
```

当然，防止未授权用户访问敏感数据并不是 PostgreSQL 在没有`sepgsql`的情况下无法做到的事。PostgreSQL 可以将过程和函数标记为以函数或过程所有者的权限而非执行会话的权限运行。`sepgsql`提供的是另一种实现此功能的方式，或者通过其他安全模型提供数据保护。

例如，在我们的示例中，`testuser`账户的数据库内权限仍然适用，我们并没有为`testuser`账户授予其他权限或将其权限提升到更高的级别——相反，我们是使用 SELinux 标签和上下文信息来进一步过滤权限。

## 使用 sepgsql 特定的函数

`sepgsql` PostgreSQL 模块添加了一些可以用来与数据库内标签进行交互的函数：

+   使用`sepgsql_getcon()`，我们可以获取当前会话的上下文。

+   使用`sepgsql_setcon()`，我们可以更改当前会话的上下文，前提是当前上下文具有执行此操作的权限。

+   使用`sepgsql_restorecon()`，当前数据库中的所有对象会重新标记回默认设置。该函数支持一个参数，可以是`NULL`，也可以是定义新默认值的文件引用。

+   使用`sepgsql_mcstrans_in()`和`sepgsql_mcstrans_out()`，我们可以与`mcstrans`守护进程（如果它正在运行）进行交互，将人类可读的敏感度范围转换为原始（`_in()`）或反之（`_out()`）。

这些函数在维护标签或定义依赖于上下文信息的逻辑函数时非常有用。

# 使用 MCS 和 MLS

启用`sepgsql`模块的最常见用例是使用**多类别支持**（**MCS**）和**多级安全性**（**MLS**）支持，结合 SELinux 来细化资源访问控制。

## 基于类别限制对列的访问

假设我们使用从`c900`到`c909`的类别编号范围来处理特定的 PII 数据集，并通过直接授予访问权限或使用特定的 SELinux 上下文来授权用户访问这些类别。

在数据库中，我们可以使用该范围内的类别编号来标记敏感的 PII 数据：

```
db_test=# SECURITY LABEL ON COLUMN tb_users.mail IS 'system_u:object_r:sepgsql_table_t:s0:c903';
db_test=# SECURITY LABEL ON COLUMN tb_users.address IS 'system_u:object_r:sepgsql_table_t:s0:c903';
```

应用标签后，没有访问该类别权限的用户将无法访问数据：

```
db_test=> SELECT sepgsql_getcon();
user_u:user_r:user_t:s0-s0:c0.c100
db_test=> SELECT uid,name,mail,address FROM tb_users;
ERROR:  SELinux: security policy violation;
```

当为用户正确设置类别范围时，访问数据将被授权：

```
db_test=> SELECT sepgsql_getcon();
user_u:user_r:user_t:s0-s0:c0.c100,c900.c904
db_test=> SELECT uid,name,mail,address FROM tb_users;
```

但是需要理解的是，大多数域都允许切换其类别集，只要它仍然在允许的范围内：

```
# semanage login -l
Login Name     SELinux User    MLS/MCS Range    ...
...
taylor         user_u          s0-s0:c0.c100,c900.c903 ...
```

这意味着，即使该用户的会话启动时使用了更有限的类别集（例如，使用`runcon`命令），用户仍然可以再次调用`runcon`以扩展类别范围，或者使用`sepgsql_setcon()`函数：

```
db_test=> SELECT sepgsql_getcon();
user_u:user_r:user_t:s0-s0:c0.c100;
db_test=> SELECT sepgsql_setcon('user_u:user_r:user_t:s0-s0:c0.c100,c900.c903');
db_test=> SELECT sepgsql_getcon();
user_u:user_r:user_t:s0-s0:c0.c100,c900.c903
```

为了解决这个问题，我们需要使目标域受到 MCS 约束。

## 限制用户域以操作敏感度范围

SELinux 策略始终允许减少类别范围，因此，最初包含 `c900` 类别的范围可以始终切换到一个排除该类别的范围。在 SELinux 中，授予域减少类别范围特权的规则使用了支配规则，这些规则本质上是运行数学集合表达式的算法：如果目标集合完全包含在源集合中，SELinux 将允许类别范围的转换。

然而，该策略也允许扩展类别范围（如果该范围仍然在 SELinux 配置中为用户定义的允许范围内），除非该域本身被标记为 **MCS 限制**。默认的 MCS 限制域通常是那些用于沙箱使用或虚拟化的域，正如我们将在 *第九章* *安全虚拟化* 中看到的那样。

然而，我们可以轻松添加更多的域。例如，要将用户域标记为 MCS 限制，只需加载以下 CIL 策略：

```
(typeattributeset cil_gen_require mcs_constrained_type)
(typeattributeset cil_gen_require user_t)
(typeattributeset mcs_constrained_type (user_t))
```

这将防止 `user_t` 域再次扩展其类别范围。

# 将 SEPostgreSQL 集成到网络中

当我们在 PostgreSQL 中使用 `sepgsql` 模块时，所有数据库会话都需要与之关联一个安全上下文。对于本地通信（使用 Unix 域套接字）来说，这个上下文是容易获取的，但网络会话（最常见的类型）并不会自动设置上下文。

如果系统没有参与标记的网络设置，正如我们在 *第五章* *控制网络通信* 中看到的那样，与数据库的交互将失败：

```
$ psql -U testuser -h ppubssa3ed db_test
psql: FATAL:  SELinux: unable to get peer label: Protocol not available
```

为了解决这个问题，推荐的方法是开始使用标记的 IPSec。不过，我们也可以使用 NetLabel 在需要时引入回退标记。

## 为远程会话创建回退标签

通过 Linux 的 NetLabel 和 CIPSO 支持（如在 *第五章* *控制网络通信* 中看到的那样），我们可以同时引入回退标签（基于源地址关联标签），并且为本地通信使用完整的标签。

通过完整的本地标签支持，NetLabel 可以将源上下文传递给目标，如果所有的通信仅在回环设备上进行（因为此类通信不会离开系统，允许 NetLabel 跟踪并支持端到端的流动，并向接收服务提供上下文信息）。

让我们为本地标签创建 CIPSO 定义：

```
# netlabelctl cipsov4 add local doi:2
```

我们现在为来自网络的通信（通过 `eth0` 接口和 `192.168.100.1/24` 网络）创建一个默认上下文。正是这个上下文我们会在通过网络连接到 PostgreSQL 服务器时看到：

```
# netlabelctl unlbl add interface:eth0 address:192.168.100.0/24 label:user_u:user_r:user_t:s0
```

我们现在可以删除默认映射规则，并为不同的通信类型添加映射规则：

```
# netlabelctl map del default
# netlabelctl map add default address:0.0.0.0/0 protocol:unlbl
# netlabelctl map add default address:::/0 protocol:unlbl
# netlabelctl map add default address:127.0.0.1 protocol:cipsov4,2
```

我们创建的映射将允许一切通信不带标签（但请记住，我们已为来自`192.168.100.0/24`的通信定义了特定标签），并在本地主机上进行基于回环的完整标签设置。

## 调整 SELinux 策略

除了标签配置外，我们可能还需要进一步微调 PostgreSQL 的 SELinux 策略。这里有几个 SELinux 布尔值值得一提：

+   `postgresql_selinux_transmit_client_label` SELinux 布尔值（默认禁用）允许`postgresql_t`域设置其自己的会话上下文。当 PostgreSQL 服务器与其他远程数据库建立连接时，可能需要设置自己的会话上下文（例如，使用 PostgreSQL 的**外部数据包装器**（**FDW**）支持）。启用后，客户端上下文也会传递给远程数据库。

+   `postgresql_selinux_unconfined_dbadm` SELinux 布尔值（默认启用）授予任何未限制的用户域在`sepgsql`中的数据库管理员权限。

+   `postgresql_selinux_users_ddl` SELinux 布尔值（默认启用）允许非特权用户使用`user_sepgsql_table_t`。

+   `selinuxuser_postgresql_connect_enabled` SELinux 布尔值（默认禁用）允许用户域通过 Unix 域套接字连接到 PostgreSQL 守护进程。

别忘了使布尔值更改持久化（使用`setsebool -P`），否则系统重启后设置将恢复为默认值。

# 总结

PostgreSQL 数据库可以通过`sepgsql`模块扩展 SELinux 支持。该模块为数据库中的各种对象添加了标签支持，并检查会话上下文与目标标签之间的访问权限。为了获取会话上下文，`sepgsql`依赖于基于套接字的通信或带标签的网络通信。

在本章中，我们学习了如何启用`sepgsql`模块以及如何排除可能的策略问题。然后，我们在示例数据库中使用了各种默认类型，并用这些类型展示了`sepgsql`中的访问控制如何工作。接着，我们利用 SELinux 的 MCS 支持进一步处理基于类别的访问控制。最后，我们在使用回退标签支持的网络中集成了 PostgreSQL。

在下一章中，我们将探讨 Linux 中的安全虚拟化，并看看 SELinux 如何帮助隔离虚拟客体。

# 问题

1.  SEPostgreSQL 是默认 PostgreSQL 技术的一部分吗？

1.  在`sepgsql`能够正常使用之前，还需要启用哪些其他设置？

1.  如何设置或查询数据库对象上的标签？

1.  为什么`sepgsql`决策事件没有出现在系统审计日志中？
