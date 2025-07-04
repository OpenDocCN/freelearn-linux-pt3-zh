# *第十二章*：调整 SELinux 策略

到目前为止，我们一直在通过调整系统来处理现有的 SELinux 策略，确保系统适应适当的 SELinux 上下文，并为文件、目录甚至网络端口分配正确的标签。我们已经了解到，SELinux 执行的行为是由策略定义的。为了微调策略执行规则，我们已经简要介绍了 SELinux 布尔值。

现在是时候更详细地研究 SELinux 布尔值了，学习如何查看布尔值的影响。在本章中，我们将考虑 SELinux 策略模块本身，以及管理员在处理这些模块时可以选择的选项。最后，我们将探讨如何更新甚至替换现有的策略。

在本章中，我们将讨论以下主要主题：

+   使用 SELinux 布尔值

+   处理策略模块

+   替换和更新现有策略

# 技术要求

查看以下视频，了解代码的实际应用：[`bit.ly/2T7MkVK`](https://bit.ly/2T7MkVK)

# 使用 SELinux 布尔值

操作 SELinux 策略的其中一种方法是切换 SELinux 布尔值。自*第二章*《理解 SELinux 决策和日志记录》以来，我们使用了`secure_mode_policyload`布尔值，这些可调设置在本书中随时出现。通过其简单的开/关状态，它们启用或禁用 SELinux 策略的部分内容。策略开发人员和管理员使用 SELinux 布尔值来切换策略的某些部分，因为并非所有部署都需要始终启用这些部分，但某些部署仍然需要。

这些布尔值是根据来自社区反馈并在社区帮助下，添加到策略中的。通过明确哪些策略规则是必要的，哪些是可选的，SELinux 开发人员可以提供适用于大多数系统的 SELinux 策略，即使这些系统的使用方式不同。

## 列出 SELinux 布尔值

可以通过使用`semanage`命令与`boolean`选项，获得 SELinux 布尔值的概览。在普通系统上，我们可以轻松找到上百个 SELinux 布尔值，因此需要筛选出我们需要的布尔值的描述：

```
# semanage boolean -l | grep policyload
secure_mode_policyload	(off, off)
  Boolean to determine whether the system permits loading
  policy, setting enforcing mode, and changing boolean values.
  Set this to true and you have to reboot to set it back.
```

输出不仅为我们简要描述了布尔值，还显示了当前值（实际上，它首先给出当前值，然后是等待策略变更的值，但这两个几乎总是相同的）。

获取布尔值当前值的另一种方法是通过`getsebool`命令，如下所示：

```
# getsebool secure_mode_policyload
secure_mode_policyload --> off
```

如果布尔值的名称不完全明确，我们可以请求所有布尔值（及其值）的概览，并筛选出我们需要的：

```
# getsebool -a | grep policy
secure_mode_policyload --> off
```

另一个可以用来查看 SELinux 布尔值描述的工具是`sepolicy booleans`命令：

```
# sepolicy booleans -b secure_mode_policyload
secure_mode_policyload=_("Boolean to ...")
```

然而，使用该命令无法显示布尔值的当前值。

最后，布尔值也通过`/sys/fs/selinux`文件系统表示：

```
# cat /sys/fs/selinux/booleans/secure_mode_policyload
0 0
```

在这里，布尔值可以像常规文件一样读取，并返回两个值：

+   第一个值是布尔值的当前状态，`0`表示关闭，`1`表示开启。

+   第二个值是布尔值的待提交状态。

待提交状态允许管理员同时更改多个布尔值，但只有在通过`/sys/fs/selinux`文件系统操作布尔值时才会发生，如我们接下来将要看到的那样。

## 更改布尔值

我们可以使用`setsebool`命令更改布尔值。例如，要切换`httpd_can_sendmail`的 SELinux 布尔值，可以使用以下命令：

```
# setsebool httpd_can_sendmail on
```

一些 Linux 发行版可能还提供`togglesebool`命令。此命令将切换布尔值，因此开启变为关闭，关闭变为开启：

```
# togglesebool httpd_can_sendmail
```

SELinux 布尔值的默认状态由策略管理员定义（因此由系统上活动的默认 SELinux 策略定义）。使用`setsebool`更改值时，会更新当前活动的访问控制，但该更改不会跨重启持久化（如果切换布尔值，则重启后会使用旧的值）。

为了使更改永久生效，请在`setsebool`命令中添加`-P`选项，如下所示：

```
# setsebool -P httpd_can_sendmail off
```

在后台，更新的 SELinux 布尔值会被包含在策略存储中。然后，当前的策略文件会被重建并加载。因此，位于`/etc/selinux/targeted/policy`中的策略文件（名为`policy.##`，其中`##`代表一个整数值）将会重新生成。这个过程需要一些时间，这也是为什么使布尔值持久生效（使用`-P`）比不持久更改布尔值（使用`setsebool`而不加`-P`或使用`togglesebool`）更耗时的原因。

改变并使布尔设置持久化的另一种方法是使用`semanage boolean`命令，如下所示：

```
# semanage boolean -m --on httpd_can_sendmail
```

在这种情况下，我们修改（`-m`）布尔值并将其设置为开启（`--on`）。

布尔值也可以通过其`/sys/fs/selinux/booleans`表示进行更改。当这种情况发生时，布尔值不会立即生效——该值的更改处于待提交状态。这允许管理员先通过`/sys/fs/selinux/booleans`修改多个布尔值：

```
# echo 0 > /sys/fs/selinux/booleans/httpd_can_sendmail
# getsebool httpd_can_sendmail
httpd_can_sendmail --> on pending: off
```

要提交更改，请将值`1`写入`/sys/fs/selinux/commit_pending_bools`：

```
# echo 1 > /sys/fs/selinux/commit_pending_bools
```

只要通过`semanage`或`setsebool`命令修改布尔值，变更将立即提交。只有通过`/sys/fs/selinux`结构进行的操作才允许布尔值更改处于待提交状态。

## 检查布尔值的影响

要发现一个布尔值操作了哪些策略规则，通常其描述就足够了。但有时我们可能想知道，当我们改变布尔值时，哪些 SELinux 规则会发生变化。通过`sesearch`应用程序，我们可以查询 SELinux 策略，显示受特定布尔值影响的规则。要详细显示这些信息，我们使用`-b`选项（表示布尔值）和`-A`选项（显示所有`allow`规则）：

```
# sesearch -b httpd_can_sendmail -A
allow httpd_suexec_t bin_t:dir { getattr open search }; [ httpd_can_sendmail ]:True
...
allow system_mail_t httpd_t:process sigchld; [ httpd_can_sendmail ]:True
```

当我们直接查询 SELinux 策略时，条件规则也可以作为输出的一部分显示：

```
# sesearch -s system_mail_t -t httpd_t -A
allow domain domain:key { link search };
allow system_mail_t httpd_t:fd use; [ httpd_can_sendmail ]:True
...
```

当`allow`规则后面跟着一个 SELinux 布尔值并用方括号括起来，后面加上`:True`时，这些规则仅在布尔值处于活动状态时应用。如果布尔值后面跟着`:False`，则在布尔值不活动时应用规则。

然而，并不是所有情况都能被策略编写者完美定义。有时我们需要创建自己的 SELinux 策略模块并加载它们。让我们看看如何专门处理 SELinux 策略模块。

# 处理策略模块

当系统将 SELinux 策略加载到内存时，它使用`policy.##`文件，其中`##`代表策略版本，如*第一章*《基本 SELinux 概念》结尾所述。此文件位于`/etc/selinux/targeted/policy`目录下，每次策略修改时都会生成。修改可能发生在布尔值改变（并持久化）时，或者当 SELinux 策略模块被添加或删除时。

## 列出策略模块

SELinux 策略模块是一组可以加载和卸载的 SELinux 规则。这些带有`.pp`或`.cil`后缀的模块可以根据需要由管理员加载和卸载。一旦加载，策略模块就成为 SELinux 策略存储的一部分，并将在系统重启后继续加载。与 SELinux 布尔值更改不同，SELinux 策略模块的加载始终是持久的。

要列出当前加载的 SELinux 策略模块，建议使用`semodule`命令。默认情况下，`semodule`会显示所有已加载的 SELinux 策略模块，但不包括任何详细信息：

```
# semodule -l
abrt
accountsd
...
zosremote
```

然而，SELinux 策略模块可以在指定的优先级下加载。这使得管理员能够加载一个覆盖已加载策略的策略：使用较高的`--list-modules=full`参数的 SELinux 策略模块：

```
# semodule --list-modules=full
100 abrt       pp
100 accountsd  pp
...
400 test       cil
...
100 zosremote  pp
```

除了优先级外，列表还显示策略模块是基于二进制模块格式（`pp`）还是较新的`cil`格式。

SELinux 工具将把活动策略模块复制到策略特定的位置。这使得管理员也可以通过常规的文件系统查询列出当前活动的模块：

```
# ls /var/lib/selinux/targeted/active/modules/*
/var/lib/selinux/targeted/active/modules/100:
abrt
accountsd
...
/var/lib/selinux/targeted/active/modules/400:
test
```

然而，使用文件系统位置查询活动策略并不推荐，因为我们无法保证加载的策略与文件系统匹配：非 SELinux 工具可以在不调整 SELinux 策略状态的情况下，添加或删除这些位置的文件。

## 加载和移除策略模块

在*替换和更新现有策略*一节中，我们将学习如何生成新的策略模块。创建后，需要加载和/或移除它们。无论策略格式是（`.pp`还是`.cil`），我们都使用`semodule`加载策略模块：

```
# semodule -i screen.pp
```

默认情况下，SELinux 策略模块在管理员调用时会以 `400` 优先级加载，而作为默认系统策略一部分加载的 SELinux 策略模块会以 `100` 优先级加载。在加载策略时，可以使用 `-X` 选项调整优先级。例如，要以 `500` 优先级加载 `test.cil` 策略，可以使用如下的 `-X` 选项：

```
# semodule -X 500 -i test.cil
libsemanage.semanage_direct_install_info: Overriding test module at lower priority 400 with module at priority 500.
```

要使用 `semodule` 移除一个策略模块，可以使用 `--remove` 或 `-r` 选项。在这里，我们并不是指 SELinux 策略模块的 *文件*，而是指通过 `semodule` 显示的 *模块名称*。因此，我们不需要传递后缀：

```
# semodule -r test
```

要从指定的优先级移除 SELinux 策略模块，可以使用 `-X` 选项：

```
# semodule -X 500 -r test
libsemanage.semanage_direct_remove_key: test module at priority 400 is now active.
```

参数的顺序非常重要：`-X` 选项会设置后续操作的优先级，而不是前面的操作。如果没有设置，则会使用优先级值 `400`。

最后，可以保留 SELinux 策略模块但将其禁用。这会将模块保留在策略存储中，但禁用其中所有的 SELinux 策略规则。我们使用 `--disable`（或 `-d`）选项来实现这一点：

```
# semodule -d screen
```

要重新启用该策略，可以使用 `--enable`（或 `-e`）选项：

```
# semodule -e screen
```

SELinux 策略模块的禁用和启用状态会在重启后保持不变。此外，如果你禁用了一个 SELinux 模块，该模块的所有实例（包括优先级较低的实例）都会被禁用。

强烈建议在策略模块是发行版 SELinux 策略的一部分时禁用该策略，因为这些模块本身并不总是存在于系统中，可能需要重新安装策略包才能恢复它们。

通过加载和卸载策略的说明，让我们来看看如何生成当前 SELinux 策略的更新。

# 替换和更新现有策略

当我们替换或更新现有策略时，需要使用 `semodule` 命令加载它们，如 *处理策略模块* 部分所示。但我们究竟如何创建或更新策略呢？让我们考虑一些触发 SELinux 策略调整的使用场景。

## 使用 audit2allow 创建策略

当 SELinux 阻止某些操作时，我们知道它会在审计日志中记录相应的拒绝信息（前提是没有定义 `dontaudit` 语句）。这个拒绝信息可以作为生成自定义 SELinux 策略的来源，以允许该活动。

考虑以下拒绝情况：当一个受限用户使用 `su` 切换到 root 用户时发生的拒绝：

```
type=AVC msg=audit(...): avc: denied { write } for pid=58002 comm="su" name="btmp" dev="vda1" ino=4213650 scontext=staff_u:staff_r:staff_t:s0 tcontext=system_u:object_r:faillog_t:s0 tclass=file permissive=0
```

如果我们确定需要授权这些操作，可以使用 `audit2allow` 命令为我们生成一个允许这些活动的策略模块。`allow` 规则。这些规则可以保存到一个文件中，准备构建成一个 SELinux 策略模块，然后加载。

要生成 SELinux 策略 `allow` 规则，将拒绝信息通过 `audit2allow` 工具传递：

```
# grep btmp /var/log/audit/audit.log | audit2allow
#============ staff_t ============
allow staff_t faillog_t:file write;
```

根据拒绝记录，`audit2allow`准备了一个`allow`规则。我们还可以要求`audit2allow`立即创建一个 SELinux 策略模块：

```
# grep btmp /var/log/audit/audit.log | audit2allow -M localpolicy
************ IMPORTANT ************
To make this policy package active, execute:
semodule -i localpolicy.pp
```

一个名为`localpolicy.pp`的文件将出现在当前目录中，我们可以使用`semodule`命令加载它。源文件也将存在，名为`localpolicy.te`。

如果发生的拒绝记录被认为是外观性的（即系统按预期运行，并且拒绝不应该对策略产生任何更新），可以使用`audit2allow`生成`dontaudit`规则，而不是`allow`规则。在这种情况下，拒绝记录将不再出现在审计日志中，但仍然会阻止相关操作发生：

```
# grep btmp /var/log/audit/audit.log | audit2allow -D -M localpolicy
```

在包含必要规则之后，可能会导致更多之前未触发的拒绝记录。只要之前的 AVC 拒绝记录仍然存在于审计日志中，重新生成策略并继续操作就足够了。毕竟，`audit2allow`会考虑到它遇到的所有 AVC 拒绝记录，而不管当前的 SELinux 策略状态如何。

另一种常见的方法是将系统（或应用程序领域）设置为宽松模式，以生成并填充所有与操作相关的 AVC 拒绝记录到审计日志中。虽然这会生成更多的 AVC 拒绝记录供我们处理，但也可能导致`audit2allow`命令做出错误的决策。因此，在生成新的策略构建之前，始终验证拒绝记录，并审查生成的策略，以确保它能够执行正确的访问控制集，并且不会授予超过所需的权限。

当先前的 AVC 拒绝记录不再出现在审计日志中时，就需要生成新的策略模块，否则之前已修复的访问将再次被拒绝：新生成的策略将不再包含以前的`allow`规则，而当我们加载新策略时，旧的策略不再生效。

## 使用合理的模块名称

在前面的章节中，我们使用了`audit2allow`命令生成了一个名为`localpolicy`的策略模块。然而，这个名称并没有揭示模块的具体用途。

一旦我们创建了一个（二进制）策略（如`localpolicy.pp`文件）并加载它，管理员和用户一开始通常无法清楚地知道这个模块的目的。虽然可以通过解包`.pp`文件（使用`semodule_unpackage`）然后将得到的`.mod`文件反汇编成`.te`文件，但这需要在大多数发行版上都没有的软件（例如，`dismod`应用程序，它是`checkpolicy`软件的一部分，通常不包括在内）。考虑到我们只是想了解模块中包含的规则，这是一个非常复杂且费时的方法。

模块的内容也可以从其 CIL 代码中推断出来。例如，活动的`screen`模块的代码可以在`/var/lib/selinux/targeted/active/modules/100/screen`中找到，文件名为`cil`。在某些发行版中，此文件可能是压缩文件，因此在查看之前可能需要解压缩：

```
# file screen/cil
cil: bzip2 compressed data, block size = 500k
# bzcat screen/cil
(typealias secadm_screen_home_t
...
```

然而，必须深入了解规则以了解`localpolicy`的内容，不仅非常繁琐，而且还需要足够的权限才能读取这些文件。

相反，最佳做法是为生成的模块命名以反映其预期用途。例如，修复`su`从`staff_t`域内执行时出现的几个 AVC 拒绝的 SELinux 策略，最好命名为`custom_staff_su_faillog`。

还建议对自定义策略添加前缀（或后缀），以便更容易找到：

```
# semodule -l | grep ^custom_
custom_staff_su_faillog
```

这表明该策略模块是由管理员（或组织）添加的，而不是来自默认 Linux 发行版的策略。

## 使用 audit2allow 生成参考策略模块

参考策略项目为发行版和策略编写者提供了一组功能，这些功能简化了 SELinux 策略的开发。例如，我们来看一下参考策略函数（称为`su`情境）：

```
# grep btmp /var/log/audit/audit.log | audit2allow -R
require {
  type staff_t;
}
#============ staff_t ============
auth_rw_faillog(staff_t)
```

示例中的规则是`auth_rw_faillog(staff_t)`。这是一个参考策略宏，它以更易读的方式解释了 SELinux 规则（或一组规则）。在这种情况下，它允许`staff_t`域对`faillog_t`标记的资源进行读/写操作。`faillog_t`类型是系统认证 SELinux 策略的一部分（由`auth_`前缀标识，该前缀表示源 SELinux 策略模块）。

重要说明

由于`audit2allow -R`使用自动化方法查找潜在的功能，我们需要仔细审查结果。有时它会选择一种方法，为一个域创建比所需更多的权限。

所有主要发行版都将其 SELinux 策略建立在参考策略提供的宏和内容之上。我们可以在本地文件系统的`/usr/share/doc/selinux-policy/html`中找到构建 SELinux 策略时可以调用的方法列表。

这些命名方法将一组与 SELinux 策略管理员希望启用的功能相关的规则捆绑在一起。例如，`storage_read_tape()`方法允许我们增强 SELinux 策略，授予指定域对磁带存储设备的读取访问权限。

## 构建参考策略风格模块

如果我们使用参考策略宏生成 SELinux 策略，但不再有访问二进制策略模块的权限，那么我们需要在加载之前构建该策略。基于 CIL 的策略可以直接加载，这就是为什么本书尽可能使用 CIL 的原因。然而，鉴于参考策略的广泛使用，了解如何构建这些模块同样很重要。

假设基于*参考策略*的 SELinux 策略代码位于名为`custom_staff_su_faillog.te`的文件中，那么我们可以按照以下方式将其构建为`.pp`文件：

```
# make -f /usr/share/selinux/devel/Makefile custom_staff_su_faillog.pp
Compiling targeted custom_staff_su_faillog.pp policy package
rm tmp/custom_staff_su_faillog.mod tmp/custom_staff_su_faillog.mod.fc
```

构建完成后，我们可以使用`semodule`加载它。每次我们修改策略代码（在`.te`文件中）或其他策略信息（如`.fc`文件中的文件上下文定义）时，我们都需要重新构建`.pp`文件，然后才能加载它。

## 构建传统风格的模块

如果我们要求`audit2allow`生成不使用参考策略样式宏（即我们称之为*传统风格*的 SELinux 策略）的策略规则，那么从中构建`.pp`文件需要不同的方法。

假设我们有`audit2allow -M`生成的`.te`文件，但没有`.pp`文件，那么我们可以按照以下步骤生成它：

1.  首先，使用`checkmodule`创建`.mod`文件：

    ```
    # checkmodule -M -m -o custom_nonrefpol.mod \
      custom_nonrefpol.te
    ```

1.  接下来，使用`semodule_package`生成`.pp`文件：

    ```
    # semodule_package -o custom_nonrefpol.pp \
      -m custom_nonrefpol.mod
    # semodule_package -o custom_nonrefpol.pp \
      -m custom_nonrefpol.mod -f custom_nonrefpol.fc
    ```

`audit2allow`命令将自动执行这些命令，因此仅在`.pp`文件不再存在，或者当这些较为传统风格的 SELinux 策略与您共享时，并且您需要手动构建和加载它们时才需要执行。

## 替换默认的分发策略

在添加自定义 SELinux 策略时，用户所能做的只是添加更多的`allow`规则。SELinux 没有可以用来移除当前允许访问规则的 deny 规则。

如果当前策略对管理员来说过于宽松，那么管理员需要更新策略，而不仅仅是增强它。这意味着管理员可以访问当前使用的 SELinux 策略规则。

要替换一个活动的 SELinux 策略，大多数 Linux 发行版允许你获取该策略的源代码。例如，对于基于 RPM 的 Linux 发行版，可以下载 SELinux 策略包的源 RPM 并解压，从而访问策略，方法如下：

1.  首先，找出当前 SELinux 策略的版本：

    ```
    $ rpm -qi selinux-policy
    Name		: selinux-policy
    Version	: 3.14.3
    ...
    Source RPM	: selinux-policy-3.14.3-20.el8.src.rpm
    ...
    ```

1.  接下来，尝试获取输出中显示的源 RPM。源 RPM 也可以从第三方仓库下载。如果难以找到该包，可以尝试通过[`rpmfind.net`](https://rpmfind.net)来查找。

1.  接下来，使用`rpmbuild`工具提取源 RPM：

    ```
    ~/rpmbuild directory.
    ```

1.  完成后，SELinux 策略源代码可以在`~/rpmbuild/SOURCES`目录下找到，可能命名为`selinux-policy-9c02e99.tar.gz`或类似名称，您可以进一步解压：

    ```
    screen.te file can be found in the ./selinux-policy-contrib-c8ebb*/policy/modules/contrib subdirectory.
    ```

现在，策略文件可以安全地复制、随意操作并构建，以替换现有的策略。如果我们以与已加载策略相同或更高的优先级加载更新后的 SELinux 策略模块，它将在策略中占据优先地位。

大多数发行版还将通过在线源代码控制仓库提供其活动的 SELinux 策略。例如，CentOS 当前的 SELinux 策略可以在[`github.com/fedora-selinux/selinux-policy`](https://github.com/fedora-selinux/selinux-policy)找到。

# 概述

管理员可以通过 SELinux 布尔值（由 SELinux 策略本身提供）或加载新的 SELinux 策略模块来调整 SELinux 策略。这些模块可以自动生成，或者由策略开发人员手动构建。

在本章中，我们学习了如何使用 SELinux 布尔值以及如何查询活动策略，以了解布尔值对系统的影响。接着，我们学习了如何使用 `semodule` 加载和卸载策略，或在系统上启用/禁用模块。我们在本章结束时介绍了如何生成和替换策略。

在下一章中，我们将扩展对 SELinux 策略的查询，不仅仅局限于布尔值，并学习如何使用专业工具详细分析策略行为。

# 问题

1.  如何将布尔值更改标记为待处理，但尚未提交？

1.  哪个命令可以用来查询布尔值的影响？

1.  为什么可以以不同的优先级加载 SELinux 策略模块？

1.  如何将拒绝转化为新的 SELinux 策略模块？
