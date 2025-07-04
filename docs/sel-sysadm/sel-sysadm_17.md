# 第十四章：处理新应用程序

新应用程序通常还未通过特定于应用程序的 SELinux 策略进行支持，因为大多数应用程序项目并未自行开发 SELinux 策略，而是依赖于社区（或更具体地说是 Linux 发行版）来创建和维护这些策略。一些 Linux 发行版已经实施了回退机制，允许这些应用程序运行，即使它们可能未正确隔离。然而，管理员可能不喜欢没有任何 SELinux 强制执行的情况下，运行不受信任的新应用程序。

因此，本章将讨论管理员如何在多个隔离环境中运行新应用程序，从（通常是默认的）未保护域，到沙箱系统，最终通过重用现有的 SELinux 域，而无需完全开发新的域。

本章将涵盖以下主要内容：

+   无限制地运行应用程序

+   使用沙箱应用程序

+   为新应用程序分配通用策略

+   扩展生成的策略

# 技术要求

查看以下视频，看看代码如何运行：[`bit.ly/3dGG5Bu`](https://bit.ly/3dGG5Bu)

# 无限制地运行应用程序

在许多 Linux 发行版中，默认行为是通过未受限制的域来运行新应用程序。这些是精心设计的域，虽然仍然受到 SELinux 的控制，但授予了非常广泛的权限。你可以将这样的未受限制域与允许任何可能流量的防火墙进行比较：尽管防火墙正在运行，但它几乎没有进行任何强制执行。

然而，也可以采用另一种方法，即将应用程序作为宽松域运行。与未受限制的域不同，宽松域不会通过 SELinux 强制执行：该域的所有操作都是允许的，即使 SELinux 可能会记录每一次违规行为。在*第三章*中，我们简要讨论了宽松域，*理解 SELinux 决策和日志记录*。

让我们首先看看未受限制的域，以及管理员如何修改系统配置，将未受限制的域应用于其他应用程序，或将应用程序移除出未受限制的状态。

## 理解未受限制的域如何工作

**未受限制的域**是一个具有广泛权限的 SELinux 域，仅限制域可以执行的极少数操作。未受限制的域实际上并不是 SELinux 作为技术所支持的概念。相反，它是由 SELinux 策略开发者使用的，他们创建了一组他们认为是未受限制的权限。

许多 Linux 发行版的最终用户会注意到，他们自己的上下文是 `unconfined_t`。虽然这确实是指一个未受限制的域，但实际上存在比 `unconfined_t` 更多的未受限制域。

SELinux 策略开发者将大部分与不受限制域相关的权限聚集在域本身（如参考策略所示）或 SELinux 属性中，如`unconfined_domain_type`和`unconfined_user_type`（如 CentOS 和相关的 Linux 发行版所示）。对于属性，这些属性然后会分配给一个或多个域，从而有效地使它们本质上不受限制：

```
$ seinfo -a | grep unconfined
$ seinfo -a unconfined_domain_type -x
```

一旦一个进程在不受限制的域中运行，并不意味着该域的每个操作都保持不受限制。当不受限制的域执行一个有适当 SELinux 策略的进程时，这个执行仍然可能会引发域转换，从而将执行的命令置于一个（可能受限的）SELinux 域中。

由于是否允许域转换的决定属于 SELinux 策略的一部分，建议管理员查询哪些域转换是被允许的，哪些不是。我们在*第十三章*中看到了如何分析域转换，*分析策略行为*。考虑到我们主要关心的是单步分析，我们可以使用`sesearch`工具快速概览支持的域转换：

```
$ sesearch -A -s unconfined_service_t -c process -p transition
allow unconfined_service_t chronyc_t:process transition;
allow unconfined_service_t rpm_script_t:process transition;
allow unconfined_service_t unconfined_service_t:process { transition ...};
allow unconfined_service_t virt_domain:process { transition ...};
```

我们可以通过检查单一域的权限或直接检查表示不受限制域的属性，来查看与不受限制域相关的（多个）权限：

```
$ sesearch -A -s unconfined_domain_type -ds
```

使用不受限制的域比使域宽松更为优先，因此让我们来看一下如何标记一个新的应用程序，以使其在不受限制的域中运行。

## 使新的应用程序在不受限制的域中运行

当执行应用程序时，有多个检查需要通过，才能导致域转换：

+   源 SELinux 域必须能够执行应用程序（意味着在与应用程序的二进制文件或脚本相关联的 SELinux 类型上拥有`执行`权限）。

+   源 SELinux 域必须能够转换到目标域。

+   目标域必须为其应用程序的二进制文件或脚本打上一个 SELinux 类型标签，该标签被标记为该域的入口点。

+   目标域必须允许源域所运行的 SELinux 角色（或者必须允许角色转换，但这是一个边缘案例）。

所有这些检查都与 SELinux 策略和标签相关。因此，毫不奇怪，为了使应用程序能够在不受限制的域中运行，我们需要关联正确的标签。

让我们在接下来的章节中考虑两个示例，一个是用户触发的应用程序，另一个是守护进程服务。

### 在显式不受限制的域中运行应用程序

对于用户执行的应用程序，举个例子，我们在*第七章*中介绍了 Jailkit，*配置特定应用程序的 SELinux 控制*。默认情况下，该应用程序未与任何域关联，因此它与父进程在同一域中运行。如果我们通过`unconfined_u`用户登录系统（在`unconfined_t` SELinux 域中），那么我们无需做任何事情。但是，假设我们的 staff 用户是受限的，但我们希望命令在`unconfined_t`域中运行。

重要说明

这是一个示例，展示了如何让应用程序在目标域中运行——在我们的案例中，是在一个非约束域中。允许受限用户运行非受限应用程序总是有一定风险的，因为他们可能利用这个机会突破限制。确保只有对那些你有信心不会违反安全的应用程序或用户才执行此操作。

要允许应用程序在`unconfined_t`域中运行，我们将使用`sudo`及其 SELinux 支持。虽然我们也可以扩展 SELinux 策略以透明地允许它，但这并不推荐。更新 SELinux 策略以允许受限用户运行非受限命令意味着要推翻策略中列出的一些原则。例如，你可能需要透明地允许受限用户切换到`unconfined_r`角色（通常出于安全原因不允许这样做）。这将需要进行大量分析，以确保它无法被用来突破受限角色。

使用`sudo`可以限制通过这些更高权限命令执行的方法。例如，SELinux-wise，适当的控制措施会被应用于`staff_sudo_t`域，这个域仅在执行`sudo`命令时才会分配，而不是`staff_t`域，后者是大多数用户交互所在的域。

让我们允许`lisa`用户作为一个非受限进程运行`jk_init`命令：

1.  首先，检查我们要执行命令的 SELinux 用户是否被允许使用`unconfined_r` SELinux 角色（如果没有，则将该角色添加到 SELinux 用户配置中）：

    ```
    # semanage user -l
    ```

    允许某个角色并不意味着用户域会自动在需要时切换角色，而是意味着该角色是用户的允许角色。

1.  接下来，更新`/etc/sudoers`文件，在执行以下命令时添加转换：

    ```
    ROLE and TYPE transition, but we also allow the command to be executed as the root user, as that is a requirement for the jk_init command. Of course, this can be adjusted as needed.
    ```

1.  我们的用户现在可以运行命令，并通过`sudo`前缀使其在正确的域和使用正确的角色下执行：

    ```
    $ sudo /usr/sbin/jk_init -v -j /srv/chroot \
      extshellplusnet
    ```

对于最终用户应用程序，使用`sudo`很常见，当用户的权限也需要切换时（从用户权限切换到 root 权限）。但在保持在 Linux 用户上下文中时，使用它则不太常见。

### 在显式非受限域中运行守护进程

第二种使用场景，或许比最终用户应用程序更常见，是将守护进程服务在非受限域中运行。大多数使用非受限域的 Linux 发行版（如 CentOS）默认会让新安装的软件也以非受限域运行。例如，任何通过 systemd 启用和激活的服务（它以`init_t` SELinux 域运行）且没有显式标签设置（即可执行命令被标记为`bin_t`）将运行在`unconfined_service_t`域中。

但如果我们有一个受限应用程序，想要在非受限域中运行呢？以 PostgreSQL 为例。假设这是一个隔离的数据库，启用了某些与现有 PostgreSQL SELinux 域（`postgresql_t`）不兼容的扩展。管理员可能没有时间使用`audit2allow`等方法扩展现有的 SELinux 策略，正如在*第十二章*《调整 SELinux 策略》中所看到的那样。

幸运的是，我们可以轻松地将 PostgreSQL 移至非受限域并运行。有两种方法可以实现这一点：

+   我们可以移除其可执行文件上的现有标签（`postgresql_exec_t`），并将其设置为`bin_t`。这将触发启动 PostgreSQL 二进制文件时的默认转换，转换到`unconfined_service_t`域。

+   我们可以更新 SELinux 策略，使`postgresql_t`成为一个非受限域。

切换标签很容易，但这不是最推荐的方法。然而，这是一种快速且简单的方式，可以立即查看在`unconfined_service_t`域中运行服务是否足以解决问题：

```
# chcon -t bin_t /usr/bin/postgres
```

如果可以接受，确保标签更改在重新标签操作后仍然保持：

```
# semanage fcontext -a -t bin_t /usr/bin/postgres
```

推荐更新 PostgreSQL 守护进程的 SELinux 策略，因为它保留了策略中现有的支持（包括文件转换和`postgresql_t`域与其他域及资源的集成）。它还允许管理员在以后有更多时间时，根据需要更新策略。

为了使`postgresql_t`域变为非受限域，我们需要将`unconfined_domain_type`属性分配给`postgresql_t`域。这可以通过加载以下基于 CIL 的 SELinux 策略来完成：

```
(typeattributeset cil_gen_require postgresql_t)
(typeattributeset cil_gen_require unconfined_domain_type)
(typeattributeset unconfined_domain_type (postgresql_t))
```

将此内容保存到文件中，并使用`semodule -i`加载，从此以后，`postgresql_t`域将扩展为与`unconfined_domain_type`属性相关的权限。

## 扩展非受限域

由于非受限域仍然受到强制执行，可能仍然有一些操作被 SELinux 阻止。我们可以调整 SELinux 策略，以扩展非受限域并授予更多权限。尽管默认的`unconfined_service_t`域几乎设置了所有可能的权限，但更具体地说，某些已识别的域可能没有那么广泛。

向域添加更多权限的技巧是为其分配适当的属性。这个方法与在*运行守护进程于明确的非限制性域*中看到的方法相同，根据需要添加更多属性。我们可以添加的属性列表非常重要（如从`seinfo -a`中看到的那样），但最重要的几个属性，尤其是对于基于 CentOS 的 SELinux 策略，主要包括以下几个：

+   `files_unconfined_type` 允许该域管理任何可能的文件或基于文件系统的资源。

+   `devices_unconfined_type` 允许该域与任何设备资源进行交互和管理。

+   `filesystem_unconfined_type` 允许该域与所有文件系统进行交互和管理。

+   `selinux_unconfined_type` 允许该域与 SELinux 子系统及其配置进行交互和管理。

+   `storage_unconfined_type` 允许该域与存储系统和可移动设备进行交互。

+   `dbusd_unconfined` 允许该域与所有可能的 D-Bus 服务进行交互。

+   `xserver_unconfined_type` 允许该域与并管理所有 X 服务器资源。

此外，还有若干个`can_*`属性，它们精细调整了非常具体的、安全敏感的操作。这些属性的名称很好地解释了它们的用途。例如，`can_write_shadow_passwords` 允许该域写入`/etc/shadow`，而`can_change_object_identity` 表示该域可以更改对象的 SELinux 用户。

并非所有属性的权限都能通过常规的`allow`规则或过渡来反映，这些规则或过渡可以通过`sesearch`查询。例如，`can_change_object_identity` 是在 SELinux 约束中使用的：

```
# seinfo --constrain | grep can_change_object_identity
```

查询约束是一个常常被忽视的方法，可以查看某个特定权限是否被分配到域，或者为什么没有被分配。

假设现在一个应用程序在非限制性域内仍然无法正确运行，那么我们可以使用宽容域来允许该应用程序在没有保护的情况下运行，同时保持系统的其余部分处于强制模式。

## 将域标记为宽容

正如我们在*第二章*中看到的，*理解 SELinux 决策和日志记录*，我们可以使用`semanage permissive`将一个域标记为宽容：

```
# semanage permissive -a postgresql_t
```

同样的命令可以用来查询（`-l`）或移除（`-d`）宽容状态。然而，管理员在将域标记为宽容时应特别小心：

+   首先，如果你将一个域标记为宽容，那么该 SELinux 域下运行的所有进程都将没有任何活动的 SELinux 强制执行。作为管理员，你确实希望限制通过宽容域运行的进程数量，因此不要将广泛使用的 SELinux 域标记为宽容。

    在一个非限制性域内运行的守护进程，如果仍然存在问题，不应导致该非限制性域被标记为宽容。相反，应该让该守护进程以不同的域运行，并将该域标记为宽容。

+   其次，宽松的域仍然会触发 SELinux 子系统的 SELinux 行为。过渡规则，包括进程过渡和文件过渡，仍然会执行。这当然是设计上的考虑，因为宽松域旨在短期使用，允许管理员和开发人员捕获信息并根据需要调整策略，然后才能再次移除宽松标志。

    这也意味着，如果域没有设置适当的过渡规则，可能会导致系统上创建具有错误 SELinux 类型的文件。因此，对于对系统影响较大的应用程序或守护进程，不应考虑使用宽松的域，而应使用更为隔离的情况，在这些情况下，作为管理员的你确信在需要时可以轻松地微调策略。

假设我们部署 pgpool-II，这是一个 PostgreSQL 数据库的负载均衡器，并发现该应用程序在非受限域中无法正常运行，尽管它已经在 `unconfined_service_t` SELinux 域中运行。虽然我们可以将此域设置为宽松模式，但这也会影响在 `unconfined_service_t` 域中运行的其他各种服务。

我们可以做的是重新标记其资源（主要是可执行文件），使得该应用程序通过另一个 SELinux 域运行，然后将该域标记为宽松模式。我们可以重用一个现有的、未使用的域，或者生成一个新域，正如我们将在 *使用 sepolicy generate 生成策略* 章节中看到的那样。

然而，当我们希望以（严格）受限的方式运行应用程序时，我们需要采取完全不同的方法，并寻求将此类应用程序放入类似沙盒的域中。

# 使用沙盒化的应用程序

应该仅具有非常有限权限且天生不可信的新应用程序应完全受限。虽然我们可以为这些应用程序查看自定义的 SELinux 策略，但对于每个应用程序来说，这几乎是不可能的。

相反，我们可以考虑将应用程序沙盒化，将其与系统的访问隔离开来。借助其他 Linux 原语，如命名空间支持，已经创建了一个名为 SELinux 沙盒的工具，它将在一个严格受限的域中启动应用程序。这主要适用于终端用户应用程序。

重要提示

SELinux 沙盒、其 SELinux 策略以及与其相关的命令，特定于使用或遵循 Red Hat 包的 Linux 发行版，如 CentOS。这可能不适用于你的 Linux 发行版。

对于面向服务的域，使用容器运行时和保护措施更为合适。有关使用容器保护的更多信息，请参见 *第十一章*，*增强容器化工作负载的安全性*。

## 理解 SELinux 沙盒

**SELinux 沙箱**是多种技术和保护措施的结合体。虽然 SELinux 策略在其中起着重要作用，但也采取了其他隔离措施，真正为应用程序和用户创建了沙箱体验。

沙箱的目的是创建一个低权限环境，阻止任何可能危及系统或用户数据安全的操作。这也意味着网络交互默认是被阻止的（没有数据外泄），并且许多系统资源对沙箱进程是隐藏的。

许多访问控制本身是由 SELinux 策略处理的。沙箱 SELinux 域` sandbox_t`及其衍生域（如`sandbox_xserver_t`）没有太多对其他资源的权限。沙箱实用程序还将应用类似 sVirt 的类别，以区分不同的沙箱进程。

然而，隔离是通过不同的手段实现的。`seunshare`应用程序负责执行这些隔离任务。

让我们看看 SELinux 沙箱在实践中是如何工作的。

## 使用沙箱命令

SELinux 沙箱使用` sandbox`命令。现在，在我们使用它之前，需要确保我们的 SELinux 用户已经设置了多个类别，否则 SELinux 沙箱无法随机分配两个类别进行隔离：

```
# semanage login -l
# semanage login -m -r "s0-s0:c0.c100" lisa
```

一旦分配了类别，我们就可以准备在沙箱中运行不可信的应用程序。例如，我们可以从[`www.ioccc.org`](https://www.ioccc.org)下载国际混淆 C 代码竞赛的一个应用程序，编译它，然后仅在沙箱模式下运行它，以防代码表现出恶意行为：

1.  假设我们使用的是` adamovsky`的 2019 年条目，我们应该已经准备好了` prog`二进制文件和` advent.unl`文件。创建一个存储这些文件的位置，并将它们复制过来：

    ```
    $ mkdir sandbox
    $ cp 2019/adamovsky/* sandbox
    ```

1.  接下来，从沙箱中运行` prog`命令：

    ```
    $ sandbox -H sandbox/ prog advent.unl
    Welcome to Adventure!! Would you like instructions?
    **
    ```

1.  在应用程序运行时，我们可以通过` ps`查看其当前上下文：

    ```
    prog command itself, which will be running in the sandbox_t SELinux domain and with a certain category pair set, you will notice that a seunshare command will run alongside it. This command provides the isolation for the process, not only by triggering the SELinux context change, but also removing unnecessary mount and filesystem views from the process's viewpoint.
    ```

1.  如果退出应用程序，我们可以看到沙箱位置已被标记为一个类似 sVirt 的 MCS 对：

    ```
    $ ls -Z sandbox/
    staff_u:object_r:sandbox_file_t:s0:c29,c94 advent.unl
    staff_u:object_r:sandbox_file_t:s0:c29,c94 prog
    ```

我们在这里使用的方法是明确告诉沙箱基于` sandbox/`文件夹创建一个隔离的主目录，并从该位置运行` prog`二进制文件（并且将` advent.unl`作为` prog`命令的参数）。然而，这并不是唯一的方法。

如果没有明确提供主目录，则沙箱将创建一个临时目录（并在之后清理）。但是，在这种情况下，除非我们允许沙箱域执行标有` user_home_t`标签的资源，否则我们无法执行系统中未安装的命令：

```
(typeattributeset cil_gen_require sandbox_t)
(allow sandbox_t user_home_t (file (execute map)))
```

加载此策略后，我们可以使用最少选项来使用沙箱。例如，使用 Burton 竞赛提交（也来自 IOCCC 2019 年竞赛），我们有以下内容：

```
$ cat prog.c | sandbox ./prog
      1      1   127
```

然而，使用一个更常见的位置可以提供更多的灵活性，同时还允许沙箱在多个会话之间保持数据（因为指向的目录不会被清理）。

SELinux 沙箱还支持在沙箱中运行图形应用程序。要实现这一点，可以在`sandbox`命令中添加`-X`选项。由此启动的进程将运行在`sandbox_xserver_t`域中，而不是`sandbox_t`域，因为运行图形应用程序需要更多的权限。不过需要注意的是，沙箱域的权限非常有限；不允许连接到网络资源，因此，在没有额外修改和 SELinux 策略调整的情况下，无法使用沙箱（无法与不安全网站交互的沙箱浏览器）。

# 将通用策略分配给新应用程序

在 SELinux 沙箱的强隔离与未受限制域（或甚至宽松域）的广泛权限之间，存在着具有足够特权的应用程序域。对于大多数管理员来说，拥有一个适当的 SELinux 域来管理应用程序是最好的方法，因为它可以允许所有常见的行为，并限制不希望出现的行为。

然而，当我们开始查看应用程序域时，我们会注意到复杂性存在差异，作为管理员，我们需要理解这种复杂性到底是什么，然后才能做出正确的选择。

## 了解域复杂性

SELinux 能够提供完整的系统限制：每个应用程序都在自己无法突破的受限环境中运行。但这需要细粒度的策略，这些策略必须随着所有应用程序的新版本快速开发。

以这样的速度制定细粒度的策略是不可能的，因此必须在策略的可维护性和域的安全性之间找到平衡。这个平衡就是策略设计的复杂性或域复杂性，可以大致分为以下几类：

+   **细粒度策略**为应用程序或服务的每个子组件设置独立的、单独的域。这种策略的优点在于它们尽可能地限制应用程序。通过细粒度策略，针对用户和管理员开发的角色也会变得更精细，例如通过区分应用程序中的子角色。

    这种策略的缺点是它们很难维护，需要随着应用程序本身的演变频繁更新。策略还需要考虑应用程序所支持的各种配置选项的影响。

    这样的细粒度策略并不常见。一个例子是为 Postfix 邮件基础设施提供的策略集。Postfix 基础设施的每个子服务都有自己的 SELinux 域。

+   **应用程序级别的策略**为应用程序使用单一域，无论其子组件如何。这在应用程序的限制性需求与应用程序及其 SELinux 策略的可维护性之间达到了平衡。

    这种应用程序级别的策略是 SELinux 策略中最常见的策略。尽管随着应用程序功能的扩展，它们确实面临常规维护问题，但其复杂性是有限的，SELinux 策略开发人员应该不会在维护这些策略时遇到太多问题。

+   **类别广泛的策略**为一组实现相同功能的应用程序使用单一的域定义。这种策略适用于那些行为非常相似的服务，它们的用户角色定义可以在不考虑特定应用性质的情况下描述。

    类别广泛策略的一个好例子是针对 web 服务器的策略。虽然这个策略最初是为 Apache HTTP 守护进程编写的，但它已经变得可以重用，适用于多个 web 服务器，例如 Cherokee、Hiawatha、Nginx 和 Lighttpd 项目。

    虽然此类策略更易于维护，但类别广泛策略的缺点是，它们往往拥有比实际需要更广泛的权限。随着更多应用加入类别广泛策略，为了支持这些特定功能，会增加更多的规则和权限。

+   **粗粒度策略**用于那些难以定义行为的应用程序或服务。最终用户域是粗粒度策略的一个例子，未受限的域也是如此。

当我们处理一个新的应用程序，并希望快速分配一个足够合适的策略时，最常见的方法是查看是否存在可以为该应用程序重用的类别广泛策略。

## 在特定策略中运行应用程序

让我们考虑一下 pgpool-II 应用程序的情况。当我们在没有任何额外更改的情况下安装它时，按照 *标记域为宽松* 部分所述，它将以 `unconfined_service_t` 域运行。但也许我们可以找到一个合适的策略，使得 pgpool-II 应用程序能够在更受限制的环境中运行。

由于 pgpool-II 解决方案是一个类似负载均衡的 PostgreSQL 数据库应用程序，因此我们可能可以将其运行在 PostgreSQL 域中。如果系统上没有运行 PostgreSQL 数据库，那么将这个域分配给 pgpool-II 应用程序可能不会造成太大损害。让我们看看效果如何：

1.  PostgreSQL 策略使用 `postgresql_exec_t` SELinux 类型来处理其可执行文件，因此我们将其分配给 `pgpool` 二进制文件：

    ```
    # chcon -t postgresql_exec_t /usr/bin/pgpool
    ```

1.  如果我们尝试启动 `pgpool` 系统服务，可能会遇到一个或多个失败：

    ```
    # systemctl start pgpool
    # systemctl status pgpool
    ...
    WARNING: Failed to open status file at: "/var/log/pgpool/pgpool_status"
    FATAL: could not read pid file
    ```

1.  提到的一个故障是守护进程无法访问其日志（位于`/var/log/pgpool`），另一个则抱怨进程 ID 文件（位于`/var/run/pgpool`）无法访问。由于这些文件之前是由一个不受限的域创建的，确实有可能它们的上下文也不正确。让我们将 PostgreSQL 特定的类型应用到这些位置：

    ```
    # chcon -R -t postgresql_log_t /var/log/pgpool
    # chcon -R -t postgresql_var_run_t /var/run/pgpool
    ```

1.  在重新启动`pgpool`后，我们发现它出现了一个新的故障：

    ```
    pgpool wants to listen on port 9999, but SELinux is refusing this.
    ```

1.  让我们创建一个小的策略增强，允许`postgresql_t`绑定到这个端口：

    ```
    (typeattributeset cil_gen_require jboss_management_port_t)
    (typeattributeset cil_gen_require postgresql_t)
    (allow postgresql_t jboss_management_port_t (tcp_socket (name_bind)))
    ```

1.  加载这个策略并重启`pgpool`。这样一来，`pgpool`就可以正常启动了。

    当然，守护进程能够启动而不出错并不意味着它能正常工作，因此建议继续进行测试，按照预期使用该服务。

找出哪个策略可以重用给一个进程需要一点实践和搜索。例如，你可以查询哪个域能够绑定到守护进程需要的端口，或者你可以搜索一个行为非常类似于应用程序的域。在我们的例子中，我们只需要允许该域绑定到`9999`端口。我们也可以使用这一信息点来寻找一个不同的策略——一个允许绑定到该端口的策略（例如`httpd_t`域），然后看看它是否更合适。

虽然这种方法是试错法，但它可能会使服务在比不受限域更受限的域中运行。然而，更好的方法是生成一个新的自定义策略，并从那里开始工作。

# 扩展生成的策略

当我们为一个新的应用程序分配一个不同的策略时，我们实际上是在重用并可能扩展现有的策略。我们可以进一步生成新的策略，然后进一步扩展这些策略，实际上进入了自己开发新策略的领域。

在*第十五章*，《使用参考策略》，以及*第十六章*，《使用 SELinux CIL 开发策略》中，我们将进一步扩展策略开发的相关内容，以实现更细粒度的控制。通过使用策略生成工具，我们可以快速创建一个初稿策略，并根据需要进行调整。

一个重要的警告是，策略生成工具通常只限于单一策略格式，要么是参考策略风格，要么是 CIL 风格。管理员和组织应该尝试专注于一种风格并坚持使用，这样新开发人员和管理员的学习曲线就不会太陡峭。

## 理解生成的策略的局限性

策略生成器，例如我们在 *第十一章* 中看到的 `udica` 工具，*增强容器化工作负载的安全性*，通常有一个非常特定的目的。例如，`udica` 工具专注于生成新的容器 SELinux 域，仅对这些容器有用。生成器通常有一个明确的目标，来定义它们的策略应该是什么样的。

生成的策略通常是应用层的策略。通过生成器创建精细粒度的策略是困难的，而定义跨类别的策略需要多个步骤和多次操作，而生成器通常采用单步生成。

此外，大多数生成的策略仅一般支持基于角色的 SELinux 访问控制：用户要么被允许访问目标 SELinux 域并与其交互，要么不允许访问。生成的策略中通常不包含角色区分（例如应用管理员与应用用户）。

管理员应当意识到，生成器也必须假设应用程序的工作方式。虽然这使得生成器适用于大多数简单的服务和应用程序，但它们显然还不能替代一支有经验的 SELinux 策略开发团队。

## 引入 sepolicy generate

`sepolicy` 命令能够生成初始的 SELinux 策略模块，管理员和开发者可以进一步调整。这些生成器会利用系统上的一些资源（例如发行版的包数据库），更好地了解应包含哪些资源，并生成一系列 SELinux 策略文件。

由于应用程序种类繁多，`sepolicy generate` 命令还需要用户提供应用程序类型。目前支持以下类型：

+   用户应用程序通过 `--application` 选项进行识别。这类应用程序是供最终用户启动并与之交互的。

+   系统服务应用程序通过 `--init` 选项进行识别。以守护进程模式运行或拥有自己用户的应用程序通常是系统服务应用程序。

+   D-Bus 系统服务应用程序通过 `--dbus` 选项进行识别。这类服务由 D-Bus 调用。

+   `--cgi` 选项。使用 CGI 特定域可以让 CGI 应用程序在自己的域中运行，而不是扩展 Web 服务器域本身的权限。

+   `--inetd` 选项。

+   沙箱应用程序类似于用户应用程序，但更为封闭，并通过 `--sandbox` 选项提供支持。

除了应用层策略生成，`sepolicy generate` 还支持生成用户域和角色：

+   支持图形桌面的标准用户可以通过 `--desktop_user` 选项生成。这是一种常见的非管理型用户角色。

+   使用`--x_user`选项可以生成一个更轻量、最小的用户角色，仍然支持图形桌面。此域专注于最小权限，因此需要进一步扩展，才能更好地发挥作用。

+   如果不需要支持图形用户界面，则可以使用`--term_user`选项。这将生成一个没有桌面支持的受限用户域。

+   可以使用`--admin_user`选项生成面向管理的用户域。此选项适用于具有广泛管理权限的用户。

+   可以使用`--confined_admin`选项生成更多受限的管理域。这使您能够生成仅对少数应用程序域具有管理角色的用户域，而不是对整个系统进行管理。

生成器还支持进一步自定义现有域（使用`--customize`）或生成特定类型（使用`--newtype`）。

我们使用`sepolicy generate`为 pgpool-II 应用程序生成策略。

## 使用 sepolicy generate 生成策略

`sepolicy generate`命令将创建一个骨架 SELinux 策略，采用参考策略代码样式。然后可以逐步扩展此策略，以满足应用程序所需的权限。

让我们为`pgpool`创建并调整策略：

1.  首先，我们告诉`sepolicy`生成一个名为`pgpool`的新策略，适用于`/usr/bin/pgpool`二进制文件：

    ```
    # sepolicy generate -n pgpool --init /usr/bin/pgpool
    ```

1.  接下来，构建生成的 SELinux 策略：

    ```
    # make -f /usr/share/selinux/devel/Makefile pgpool.pp
    ```

1.  将策略加载到内存中：

    ```
    # semodule -i pgpool.pp
    ```

1.  重新标记文件系统，或者至少是`pgpool.fc`文件中提到的位置：

    ```
    # restorecon -RvF /usr/bin/pgpool /var/log/pgpool \
      /var/run/pgpool
    ```

1.  启动`pgpool`服务：

    ```
    pgpool is running flawlessly.
    ```

现在，您可能会觉得这太简单了。是的，的确如此。`sepolicy generate`提供的默认 SELinux 策略是宽容的，正如您可以在`pgpool.te`文件中看到的那样：

```
permissive pgpool_t;
```

如果我们删除此语句，重新构建并重新加载策略，那么我们会注意到失败再次出现，例如进程无法绑定到所选端口。我们现在可以使用`audit2allow`，例如，帮助我们根据需要扩展策略：

```
# cat /var/log/audit/audit.log | audit2allow -R
```

逐步扩展、重建并重新加载策略，直到应用程序无故障运行。

# 总结

Linux 管理员可以使用 SELinux 控制来防止或限制对应用程序的访问，但这并不总是当前的需求。能够以*正确*的权限集运行应用程序才是关键，而正确的权限集取决于用户的意图和环境。

在本章中，我们学习了如何对应用程序域应用适当的限制，从高度隔离的容器环境，到常规应用程序域、类别范围的权限集，再到无限制域甚至宽容域。我们了解到，这是通过首先找到适当的域，理解该域使用的标签，然后为文件分配正确的标签，从而确保应用程序在正确的域中执行。

我们还学习了如何自己生成新的策略（使用`sepolicy generate`），而不需要立即深入全面的 SELinux 策略开发方法，这将在最后两章中讨论。

# 问题

1.  非受限域和宽容域有什么区别？

1.  我们如何在一个非常受限的域中运行应用程序？

1.  我们如何轻松切换服务运行的域？

1.  为什么由`sepolicy`生成的策略似乎可以顺利运行？
