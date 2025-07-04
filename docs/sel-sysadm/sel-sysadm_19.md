# 第十六章：使用 SELinux CIL 开发策略

虽然参考策略是 SELinux 策略中最常用的语言和开发风格，但 **通用中间语言** (**CIL**) 是一个功能强大但更低级的语言结构，可以用于开发 SELinux 策略。尽管它比较低级，但它仍然非常易读，并且得到了良好的支持，因为 SELinux 工具在使用其他语言时会在后台使用 CIL。

由于 CIL 是主要使用的语言，我们知道它可以用于构建整个策略。遗憾的是，与参考策略不同，开发人员没有可以使用的支持构造。然而，我们仍然可以学习如何定制当前的策略，创建参考策略无法实现的特定定义，甚至如果我们选择的话，构建一个完整的应用程序策略。

在本章中，我们将讨论以下主要内容：

+   介绍 CIL

+   创建细粒度定义

+   构建完整的应用程序策略

# 技术要求

查看以下视频，查看代码实际运行：[`bit.ly/3dLYP2Q`](https://bit.ly/3dLYP2Q)

# 介绍 CIL

CIL 的设计目的是成为构建策略的主要语言，并且是最低级的可读格式。在 CIL 之后，SELinux 代码会被转换为二进制格式，以便发送到 Linux 内核（和 SELinux 子系统）以加载到内存中。

管理员可能倾向于认为，在使用参考策略方法构建 SELinux 策略模块时生成的二进制文件就是最终的二进制文件。然而，正如我们在 *第一章*《基础 SELinux 概念》中看到的那样，`semodule` 命令会在构建最终格式之前将其转换并翻译成 CIL。

让我们看看这些翻译是如何工作的，以及我们可以从中学到什么。

## 将 .pp 文件转换为 CIL

当加载一个非 CIL 的 SELinux 策略模块时，`semodule` 命令会首先将该模块视为一种未知格式，并提取 `.pp` 文件，支持作为高级语言（HLL）。

由于使用参考策略（或其他经典 SELinux 开发）构建的 SELinux 策略模块的高级语言（HLL）格式与生成的模块相同，因此此阶段通常只是涉及创建一个副本。我们可以通过将一个 HLL 文件与原始文件进行比较来看到这一点：

```
$ cp /var/lib/selinux/targeted/active/modules/400/pgpool/hll pgpool.hll.bz2
$ bunzip2 pgpool.hll.bz2
$ sha512sum pgpool.pp pgpool.hll
b81ba4ac...c0db pgpool.pp
b81ba4ac...c0db pgpool.hll
```

一旦 `semodule` 转换或提取了数据，它将把数据转换为 CIL 代码。对于每种支持的高级语言（HLL），都可以在 `/usr/libexec/selinux/hll` 中找到相应的转换器。目前，只有 `pp` 命令可用，用于将这种旧的风格转换为 CIL。

让我们看看它是如何工作的：

```
$ /usr/libexec/selinux/hll/pp pgpool.pp
(type pgpool_t)
(roletype object_r pgpool_t)
(type pgpool_exec_t)
...
(filecon "/var/run/pgpool(/.*)?" any (system_u object_r pgpool_var_run_t ((s0) (s0))))
```

所以，从本质上讲，当我们开发参考策略风格的模块时，它们无论如何都会被转换为 CIL。然而，生成的 CIL 代码内部并没有任何辅助的构造。例如，所有权限都会展开，并且与 SELinux 策略模块外部其他资源或类型的所有交互也会列出。那里不再有任何支持宏或接口。在 *构建完整的应用程序策略* 部分，我们将看到 CIL 确实支持抽象，因此当前的观察仅仅是由于 `pp` 命令执行的转换所致。

## 理解 CIL 语法

在我们开发 CIL 时，最明显的观察是它喜欢括号。CIL 使用 S 表达式语法，这种语法由 Lisp 推广，使得数据呈树状结构。表达式中的第一个标识符告诉 CIL 该构造是关于什么的。

让我们来看一下我们在将 `pgpool.pp` 二进制文件转换为 CIL 时收到的最后一条语句，现在为方便起见已格式化：

```
(filecon
  "/var/run/pgpool(/.*)?"
  any
  (
    system_u
    object_r
    pgpool_var_run_t
    (
      (s0)
      (s0)
    )
  )
)
```

如果我们仔细查看此语句，我们可以推断出以下内容：

+   我们有一个 `filecon` 语句，它接受三个参数：路径表达式、适用的资源类型，以及要与之关联的 SELinux 上下文。

+   SELinux 上下文有四个相关字段：SELinux 用户、SELinux 角色、SELinux 类型和 SELinux 敏感度范围。

+   SELinux 敏感度范围有两个值：低端值和高端值。

这条语句等同于参考策略在策略模块的文件上下文部分（带 `.fc` 后缀的文件）中定义的内容，如下所示：

```
/var/run/pgpool(/.*)?  gen_context(system_u:object_r:pgpool_var_run_t,s0)
```

幸运的是，我们不需要为了理解和查看发生了什么而去查找和解释代码。SELinux 项目提供了广泛的 CIL 文档，解释了该语言的工作原理及其所支持的内容。相关信息可以在 [`github.com/SELinuxProject/selinux/tree/master/secilc/docs`](https://github.com/SELinuxProject/selinux/tree/master/secilc/docs) 查阅。不过需要注意的是，CIL 策略开发仍处于初期阶段，因此 HLL 转换机制未使用的 CIL 构造的覆盖率非常低。

现在我们来看看如何使用 CIL。

# 创建细粒度的定义

在本书中，大多数小型 SELinux 策略调整都是使用 CIL 完成的。这些是小型、细粒度的定义，开发工作量很小，且具有直接加载的优点。

## 根据角色或类型

CIL 语言要求在策略中如何链接类型或角色有一定的顺序。有时候，在开发 CIL 策略时，类型的顺序可能没有得到正确处理。

为了绕过这个问题，使用了一个默认属性 `cil_gen_require`。当类型或角色被分配给 `cil_gen_require` 属性时，它们会在策略中自动正确链接。然而，这并不是 CIL 的要求，而是 SELinux 工具使用的一种约定。

这个属性实际上存在两次，一次作为类型属性，一次作为角色属性。它们可能有相同的名称，但是是两个不同的属性：

```
(roleattributeset cil_gen_require system_r)
(typeattributeset cil_gen_require direct_run_init)
```

使用的函数 `roleattributeset` 和 `typeattributeset`，将第二个参数（即属性名称）分配给第三个参数（即角色或类型）。角色或类型本身可以是属性，就像 `direct_run_init` 属性所示。

对于开发人员来说，建议始终将要使用的类型或角色作为 `cil_gen_require` 属性的一部分，并尽可能使用属性来简化开发活动。与其为每个可能与您的策略资源交互的域授予所有可能的 `allow` 规则，不如将它们授予一个属性，然后将此属性分配给域。这样可以创建一个更小的策略，并且更容易维护。

## 定义一个新的端口类型

当开发更复杂的 SELinux 策略时，有一些设置我们不能在 SELinux 模块中添加，至少在使用传统或参考策略样式编码时不能添加。当我们想要使用这些设置时，我们需要重建整个策略并在所谓的基本策略中进行调整；这是加载模块之前加载的主要策略。然而，这需要访问完整的 SELinux 策略源代码，并使用一个过程来使用它们（因为您将覆盖 Linux 发行版的 SELinux 策略，应确保任何系统更新不会再次覆盖您的策略）。

除了简单测试之外，确定一个语句是否在 SELinux 模块中受支持的另一种方法是查看在线文档。例如，让我们看看端口声明语句 `portcon`，它是网络导向语句的一部分。在 [`selinuxproject.org/page/NetworkStatements`](https://selinuxproject.org/page/NetworkStatements) 上有文档说明，我们可以看到 `portcon` 在模块策略中不是有效的，也不能通过 SELinux 布尔值进行切换。

幸运的是，在使用 CIL 时情况并非如此。例如，让我们创建一个自定义端口类型，`pgpool_port_t`，并将其映射到一个空闲端口，比如 TCP 端口 `50123`：

```
; Port type
(type pgpool_port_t)
; Some attributes to match the current SELinux policy requirements
(typeattributeset defined_port_type pgpool_port_t)
(typeattributeset reserved_port_type pgpool_port_t)
(typeattributeset port_type pgpool_port_t)
; Our dependency mappings
(roleattributeset cil_gen_require object_r)
(typeattributeset cil_gen_require pgpool_port_t)
; Make sure object_r is allowed for pgpool_port_t
(roletype object_r pgpool_port_t)
; The port mapping itself
(portcon tcp 50123 (system_u object_r pgpool_port_t ((s0) (s0))))
```

在加载策略之前，我们可以清楚地看到这个端口没有被分配：

```
# seinfo --port 50123 | grep tcp
 portcon tcp 32768-60999 system_u:object_r:ephemeral_port_t:s0
```

正如我们在整本书中看到的，我们可以立即加载这个策略文件，而不需要构建或编译它：

```
# semodule -i pgpool_port.cil
```

加载后，将类型分配给端口，我们可以使用它来优化我们的 SELinux 策略：

```
# seinfo --port 50123 | grep tcp
 portcon tcp 50123 system_u:object_r:pgpool_port_t:s0
 portcon tcp 32768-60999 system_u:object_r:ephemeral_port_t:s0
# semanage port -l | grep 50123
pgpool_port_t     tcp    50123
```

当使用传统的 SELinux 风格开发时，许多策略开发者遇到的约束条件在使用 CIL 时不再适用。由于 SELinux 工具将所有高级结构转换为 CIL，SELinux 开发人员可能会完全移除这些约束，尽管这必须经过仔细评估，以确保不会出现意外的副作用。

## 添加策略的约束条件：

另一个对于普通策略开发者来说不可访问的领域是向策略中添加约束。约束基于整个 SELinux 上下文（而不仅仅是类型）来限制操作，它们是我们能够找到的最接近否定现有规则的机制。

重要提示

我们不建议仅仅为了绕过不喜欢的规则而向现有策略中添加约束。约束在常规的策略查询中不可见，比如在 `sesearch` 中。管理员可能会非常困惑，当 `sesearch` 表明某个操作是允许的，但系统却拒绝执行该操作。

使用 CIL，我们可以向实时策略中添加约束，做到这一点。但请记住，约束本身并不实际允许任何操作 —— 它们仅仅是限制 SELinux 子系统认为是有效操作的范围。一个支持读取所有可能类型的约束语句并不意味着它实际允许此操作，因为仍然需要有类型强制规则来实际允许这些操作。

例如，如果我们想要移除 `staff_t` 域读取 `/etc/passwd` 文件（该文件具有 SELinux 类型 `passwd_file_t`）的能力，我们可以添加一个支持读取所有可能类型的约束，除非源域是 `staff_t`，在这种情况下，我们支持读取所有可能类型，除了 `passwd_file_t`：

```
(constrain (file (read))
  (or
    (and
      (eq t1 staff_t)
      (not (eq t2 passwd_file_t))
    )
    (not (eq t1 staff_t))
  )
)
```

加载后，我们可以确认约束已经生效：

```
# seinfo --constrain | grep passwd_file_t
constrain file read (t1 == staff_t and not ( ( t2 == passwd_file_t ) ) or not ( t1 == staff_t ) ));
```

而实际上，尝试读取 `passwd` 文件是被禁止的：

```
$ cat /etc/passwd
cat: /etc/passwd: Permission denied
```

为了展示约束对管理员的困扰，我们来看一下 `sesearch` 对此有什么说法：

```
# sesearch -A -s staff_t -t passwd_file_t -c file -p read;
allow nsswitch_domain passwd_file_t:file { ... read };
allow staff_t passwd_file_t:file { ... read };
```

因此，尽管策略中有两个规则允许此操作（一个针对 `nsswitch_domain` 属性，另一个明确针对 `staff_t` 域），约束已限制了该操作，但从 `sesearch` 输出中并不明显。

# 构建完整的应用策略

我们也可以使用 CIL 构建完整的应用策略。然而，请记住，目前没有现成的接口或支持宏可以快速开发策略。此外，也没有可用的模板或类似工具来启动这些工作。

但这不应该阻止我们，这将让我们展示更多 CIL 语言的细节。我们还会看到，CIL 语言确实支持接口构造（它们甚至被推荐使用），但社区尚未通过类似参考策略的项目完全接受它。

## 使用命名空间

CIL 语言支持命名空间，这使得在开发策略时具有更高的灵活性。生成的 CIL 策略始终使用主全局命名空间，因此在生成的策略中我们不会找到命名空间的示例。

然而，我们可以轻松地展示它是如何工作的。让我们创建一个骨架文件，里面包含我们用 CIL 开发的 `pgpool` 策略：

```
; Dependencies
; Pgpool support
(block pgpool
  ; Declarations
  (type domain)
  ; Local policy
  ; Behavior
  ; File contexts
)
```

前面的代码中创建的命名空间是`pgpool`命名空间，通过`block`语句标识。在 CIL 中，命名空间是层次结构的。如果需要，我们可以通过在`block`语句内嵌套另一个`block`语句，在`pgpool`中创建一个新的命名空间。

当我们在一般结构中遇到命名空间时，需要使用点分隔符。在示例中，我们在`pgpool`命名空间中定义了一个名为`domain`的类型。如果以后想通过`sesearch`查询它，完整的名称将是`pgpool.domain`（这与在更传统开发的策略中使用的`pgpool_t`名称有相同的作用）。

说明性注释

在后续的策略代码中，我们将只显示添加的最相关语句，而不是包括所有先前添加的语句。如果没有支持的接口（CIL 称之为宏），策略文件将迅速变得相当庞大，这对可读性没有帮助。通过集中展示相关和新增的语句，CIL 策略的开发模式更容易解释。

由于主策略中的所有对象都位于全局命名空间中，我们需要显式地引用这个全局命名空间。这是通过在名称前加一个点来实现的。例如，如果我们想将`pgpool.domain`类型分配给`system_r`角色，我们的策略需要调整如下：

```
(roleattributeset cil_gen_require system_r)
(block pgpool
  (type domain)
  (roletype .system_r domain)
)
```

在这里，`roletype`语句用于将`pgpool`命名空间中定义的`domain`类型分配给`system_r`角色（该角色在`global`命名空间中定义）。

## 扩展策略并分配属性

在开发策略时，建议尽可能使用属性。许多属性会自动授予必要的权限，以启动策略开发，减少需要参与策略的`allow`语句数量。

系统中仍在使用的主策略定义了很多属性，正如前面各章节所见。所以，让我们将`daemon`属性分配给我们的域：

```
(typeattributeset cil_gen_require daemon)
(block pgpool
  (type domain)
  (typeattributeset .daemon domain)
)
```

通过分配`daemon`属性，所有现有的守护进程策略规则会自动应用到`pgpool.domain` SELinux 域。

要找出哪些属性是合理添加的，我们可以看看现有的守护进程域：

```
# seinfo -t postgresql_t -x
Types: 1
  type postgresql_t, nsswitch_domain, can_change_object_identity, corenet_unlabeled_type, domain, kernel_system_state_reader, netlabel_peer_type, daemon, syslog_client_type, pcmcia_typeattr_1;
```

现在，我们不需要盲目地采用所有属性，而是从我们有信心的属性开始。

## 添加入口点信息

下一步是将入口点信息添加到策略中。在我们开始测试之前，这是一个必要的步骤，因为我们希望该域变为活动状态。为了实现这一点，它必须能够被初始化系统（或 systemd）执行，并且过渡到我们刚刚声明的域。

让我们从定义`entrypoint`类型（`pgpool.exec`）并将其与正确的属性关联开始：

```
(roleattributeset cil_gen_require object_r)
(typeattributeset cil_gen_require file_type)
(typeattributeset cil_gen_require direct_init_entry)
(block pgpool
  (type exec)
  (roletype .object_r exec)
  (typeattributeset .file_type exec)
  (typeattributeset .direct_init_entry exec)
  (allow domain exec (file (entrypoint ioctl read getattr lock map execute open)))
  (typetransition .initrc_domain exec process domain)
)
```

在这个代码块中，我们执行了多个步骤以确保会发生过渡：

+   我们将`pgpool.exec`类型与`file_type`属性（这是文件的通用属性）和`direct_init_entry`属性（这是用于启动系统服务的文件类型）关联起来。

+   我们将`pgpool.exec`类型标记为`pgpool.domain`类型的`entrypoint`，并授予该域必要的权限来读取、打开和执行`pgpool.exec`标记的资源（这是启动进程所需的）。

+   我们声明了一个类型转换，使得任何标记为`initrc_domain`的进程执行标记为`pgpool.exec`的资源时，会导致域向`pgpool.domain`转换。

我们现在可以通过添加文件上下文定义来完成这一步：

```
(block pgpool
  (filecon "/usr/bin/pgpool" file 
    (.system_u .object_r exec ((s0) (s0)))
  )
)
```

做出这些更改后，我们可以加载策略并重新标记文件：

```
# restorecon -v /usr/bin/pgpool
Relabeled /usr/bin/pgpool from system_u:object_r:bin_t:s0 to system_u:object_r:pgpool.exec:s0
```

我们现在可以尝试启动`pgpool`服务，并希望它失败（因为这将表明转换成功，考虑到`pgpool.domain` SELinux 域几乎没有足够的权限来成功启动整个服务）。

## 逐步扩展策略

一旦域的转换成功，我们可以通过反复试验逐步扩展策略，就像我们在使用参考策略风格开发 SELinux 策略时一样。然而，与其使用`audit2allow`来引导我们，我们需要自己解释拒绝信息，并看看如何更好地处理它。

考虑启动服务后出现的失败情况：

```
# ausearch -i -m avc -ts recent
(Output reformatted for readability)
avc: denied { map } for scontext=pgpool.domain 
                        tcontext=ld_so_t
                        tclass=file
avc: denied { read write open } for scontext=pgpool.domain 
                                    tcontext=null_device_t
                                    tclass=chr_file
```

现在，在立即为这些类型添加`allow`规则之前，让我们看看其他系统守护进程是如何完成这个任务的：

```
# sesearch -A -s postgresql_t -t ld_so_t -c file -p map
allow domain file_type:file map; [ domain_can_mmap_files ]:True
allow domain ld_so_t:file { execute getattr ioctl map open read };
```

因此，这个权限是基于`domain`属性的，事实上我们确实忘记将其添加到策略中。让我们纠正这个问题并重新尝试：

```
(typeattributeset cil_gen_require domain)
(block pgpool
  (type domain)
  (typeattributeset .domain domain)
)
```

在这个例子中，我们还可以清楚地看到 CIL 中的命名空间的影响。我们将（全局命名空间托管的）`domain`属性分配给（`pgpool`命名空间托管的）`domain`类型。它们都被命名为`domain`，但有不同的命名空间。这也展示了属性的重要性。

当然，并不是所有权限都可以通过属性授予。通过将目标类型作为依赖项添加，我们可以像对`entrypoint`声明一样，直接在策略中包含`allow`语句。

例如，如果我们希望显式地允许我们的域向`postgresql_t`域发送信号，可以执行以下命令：

```
(typeattributeset cil_gen_require postgresql_t)
(block pgpool
  (allow domain .postgresql_t (process (signal)))
)
```

随着我们向策略中添加越来越多的权限，我们可能希望优化一些定义。CIL 支持两种优化，这两种优化与参考策略的简化方式一致，因为 CIL 是由同一社区开发的。

## 引入权限集

我们可以做的第一个简化是简化我们使用的权限集。记得我们添加的`allow`规则，以允许我们的域执行其入口文件：

```
(allow domain exec 
  (file 
    (entrypoint ioctl read getattr lock map execute open)
  )
)
```

如果我们使用引用策略风格的方法，我们将通过`exec_file_perms`宏组合这些权限。好消息是，CIL 通过一个名为`classpermissionset`的语句支持类似的功能。

如果我们想要完全模拟引用策略风格的方法，我们会在全局命名空间中定义`classpermissionset`，然后按如下方式使用它：

```
(classpermission exec_file_perms)
(classpermissionset exec_file_perms (file (ioctl read getattr lock map execute open)))
(block pgpool
  (allow domain exec (file (entrypoint)))
  (allow domain exec exec_file_perms)
)
```

在这个例子中，我们在全局命名空间中定义了`classpermissionset`，然后引用它。然而，与引用策略不同的是，我们不能仅仅将`exec_file_perms`与`entrypoint`一起添加到权限中。`classpermissionset`语句明确引用了与之关联的类。因此，CIL 中的`allow`语句是一个独立的语句，它本身不包含类引用。

此外，在示例中，你会注意到我们并没有在`exec_file_perms`名前加上点符号，以引用全局命名空间。虽然我们可以加上点符号，以与其余策略保持一致，但如果没有潜在的冲突，使用点符号前缀并不是强制性的。如果当前命名空间中不存在某个名称的本地定义，策略将检查父命名空间（因此也包括全局命名空间）中是否定义了该名称。

所以，尽管前面的策略完全可以正常工作，但我们确实建议在全局命名空间相关的名称前加上点符号，以确保后续没有本地覆盖可能会干扰策略。

## 添加宏

我们可以引入的最后一个简化是添加**宏**。CIL 明确支持宏，允许它们成为加载策略的一部分，而不仅仅是在文件系统中引用。通过 CIL 宏，代码成为策略的一部分。构建策略时无需再引用 CIL 代码。

虽然这是与面向对象编程一致的最佳实践（因为我们可以将宏添加到命名空间中，使它们保持在同一对象内），但缺点是当前的 SELinux 工具无法快速显示策略中可用的宏（以及它们需要的接口）。

现在，让我们通过一个域转换宏来增强我们的`pgpool`策略，类似于通过引用策略风格开发创建的`pgpool_domtrans()`接口：

```
(block pgpool
  (macro domtrans ((type SOURCEDOMAIN))
    (allow SOURCEDOMAIN exec exec_file_perms)
    (allow SOURCEDOMAIN domain (process (transition)))
    (typetransition SOURCEDOMAIN exec process domain)
  )
)
```

宏定义本身以名称开始（在我们的例子中是`domtrans`），后跟接口。该接口定义了传递给宏的参数数量及其类型。在我们的示例中，只传递了一个参数，它是一个 SELinux 类型。

宏后面跟着应用的代码。代码中引用了参数本身（`SOURCEDOMAIN`），并将在宏被显式调用时用后面给出的参数进行替换。虽然我们的示例使用了大写的变量名，但这并不是强制性的，仅仅是为了视觉效果，表示将要被替换的内容。

在另一个 CIL 策略中，我们可以通过`call`语句引用这个宏。例如，要允许`postgresql_t`域转到`pgpool.domain` SELinux 域，我们可以将以下`call`语句添加到我们的策略中：

```
; Equivalent to "pgpool_domtrans(postgresql_t)" in refpolicy
(typeattributeset cil_gen_require postgresql_t)
(call pgpool.domtrans (postgresql_t))
```

CIL 宏提供了所需的一切，以便在开发 SELinux 策略时实现与参考策略相同的简洁性，甚至更多，因为有许多约束不适用于 CIL 策略。

虽然未来有可能会发生这种情况，但目前由于种种原因并没有计划这样做：

+   当前的参考策略包含大量代码，这些代码都需要重新编写。此外，Linux 发行版已经在此策略基础上做了许多扩展，因此将 SELinux 策略代码重写为 CIL 的工作量非常大。这并非不可能，但不是几周内能够完成的任务。

+   几乎所有帮助开发者编写 SELinux 策略的在线信息和文档都是基于当前的参考策略。这个重要的信息源一旦发生切换就会变得过时，而针对基于 CIL 的策略开发的在线文档目前仍然非常稀少。

+   CIL 策略虽然非常强大，但由于其 S 表达式，它也有些复杂。CIL 的设计初衷并不是用 CIL 替代 SELinux 策略开发，而是允许开发更高级的语言，这些语言可以轻松地转换为 CIL。因此，如果需要重新设计，用户友好和开发者友好的语言很可能会被设计出来，并且能够轻松转换为 CIL。

随着 SELinux 开发的进展，无论是在策略层面，还是在用户空间和内核支持方面，我们可以期待 CIL 及其支持工具会有更多的新增功能。

# 总结

SELinux 的 CIL 是一种强大且较低级的语法和语言，用于表达所有可能的 SELinux 策略代码。SELinux 用户空间工具会自动将现有策略转换为 CIL 代码，但通过这种转换，许多 CIL 构造并未被使用：转换只使用了一个较小的 CIL 功能集来建立有效的翻译。

更高级的 CIL 功能，例如命名空间支持、宏以及通过`classpermissionset`语句进行的权限集，在开发我们自己的基于 CIL 的 SELinux 策略时非常有用。在本章中，我们已经学习了如何使用 CIL 来构建完整的应用程序策略。由于没有类似参考策略的框架来简化开发，我们必须自己编写所有必要的代码构造。

虽然这意味着开发基于 CIL 的策略需要更多的资源，但我们也看到 CIL 具有一些参考策略开发无法解决的优势，例如声明端口或向活动策略添加 SELinux 约束的能力。

我们以简要概述结束本章，解释了为什么基于 CIL 的开发没有被广泛使用，但在可预见的未来，我们将在 SELinux 中看到这一方面的持续改进。

这就是我们本书的全部内容以及我们能提供的信息。然而，这仅仅是一个旅程的开始，而非结束。SELinux 是一种广泛使用的技术，我们希望本书能为您提供正确的材料和知识，以便理解、成长并为生态系统做出贡献。感谢您的关注与投入。

# 问题

1.  我们如何知道 CIL 会持续存在？

1.  `cil_gen_require` 属性在 CIL 开发中是必需的吗？

1.  开发者可以使用 CIL 进行的声明，其他 SELinux 语言风格不能做的例子有哪些？

1.  我们如何在 CIL 中创建类似接口的支持结构？
