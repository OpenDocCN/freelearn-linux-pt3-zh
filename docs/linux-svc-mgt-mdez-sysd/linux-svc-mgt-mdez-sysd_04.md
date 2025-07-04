# *第三章*：理解服务单元、路径单元和套接字单元

在本章中，我们将检查服务单元、路径单元和套接字单元文件的内部工作原理。我们将检查每个文件中的部分内容，并查看你可以设置的一些参数。在这个过程中，我还会给你一些提示，教你如何查找有关各个参数的使用信息。

本章我们将讨论以下主题：

+   理解服务单元

+   理解套接字单元

+   理解路径单元

在你成为 Linux 管理员的过程中，你可能会被要求修改现有单元或创建新的单元。本章的知识可以帮助你实现这一点。所以，如果你准备好了，就开始吧。

# 技术要求

和往常一样，我将在 Ubuntu Server 20.04 虚拟机和 Alma Linux 8 虚拟机上进行演示。你可以随时启动自己的虚拟机来跟着做。

查看以下链接以观看《代码实战》视频：[`bit.ly/2ZQBHh6`](https://bit.ly/2ZQBHh6)

# 理解服务单元

服务单元相当于旧版 SysV 系统中的 init 脚本。我们将使用它们来配置我们的各种服务，这些服务在过去我们称之为*守护进程*。服务几乎可以是任何你希望自动启动并在后台运行的东西。服务的例子包括安全外壳（SSH）、你选择的 Web 服务器、邮件服务器以及系统正常运行所需的各种服务。虽然有些服务文件可能简短明了，其他则可能相当长，包含更多启用的选项。要了解所有这些选项，只需输入以下命令：

```
man systemd.directives
```

你可以设置的所有参数的描述分布在多个不同的手册页中。这个`systemd.directives`手册页是一个索引，它会引导你找到每个参数的正确手册页。

与其尝试解释服务文件可以使用的每个参数，不如让我们通过一些示例文件来解释它们的作用。

## 理解 Apache 服务文件

我们将从 Apache Web 服务器的服务文件开始。在我的 Ubuntu Server 20.04 虚拟机中，它是`/lib/systemd/system/apache2.service`文件。首先需要注意的是，服务单元文件被分为三个部分。顶部部分是`[Unit]`部分，它包含可以放入任何类型单元文件中的参数。它看起来像这样：

```
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=https://httpd.apache.org/docs/2.4/ 
```

在这里，我们看到这三个参数：

+   `Description=`：好吧，这个应该是相当自解释的。它的作用是告诉用户这个服务是什么。`systemctl status`命令从这一行获取其描述信息。

+   `After=`：我们不希望 Apache 在某些其他事情发生之前启动。我们还没有讨论`target`文件，但没关系。现在，只需知道我们希望阻止 Apache 启动，直到网络、任何可能的附加远程文件系统和名称切换服务可用之后。

+   `Documentation=`：这是另一个不言自明的参数。它只是显示了 Apache 文档的位置。

要了解可以放置在任何单元文件`[Unit]`部分的选项，请输入以下内容：

```
man systemd.unit
```

接下来，我们有`[Service]`部分，这里有些更有趣的参数可以放在服务单元文件中，看起来像这样：

```
[Service]
Type=forking
Environment=APACHE_STARTED_BY_SYSTEMD=true
ExecStart=/usr/sbin/apachectl start
ExecStop=/usr/sbin/apachectl stop
ExecReload=/usr/sbin/apachectl graceful
PrivateTmp=true
Restart=on-abort
```

在这个特定的文件中，我们看到了这些参数：

+   `Type=`：在`systemd.service`手册页面上描述了几种不同的服务类型。在这种情况下，我们有`forking`类型，这意味着第一个启动的 Apache 进程将生成一个子进程。当 Apache 启动完成并且建立了适当的通信通道后，原始进程——*父*进程将退出，子进程将继续作为主服务进程运行。当父进程退出时，`systemd`服务管理器最终将认为服务已完全启动。根据手册页面的说法，这是 Unix 服务的传统行为，`systemd`只是延续了这一传统。

+   `Environment=`：这设置了一个影响服务行为的环境变量。在这种情况下，它告诉 Apache 它是由`systemd`启动的。

+   `ExecStart=`, `ExecStop=`和`ExecReload=`：这三行指向 Apache 可执行文件的位置，并指定了启动、停止和重新加载服务的命令参数。

+   `PrivateTmp=`：许多服务因各种原因而写入临时文件，你可能习惯于在每个人都可以访问的`/tmp/`目录中看到它们。在这里，我们看到了一个很酷的`systemd`安全特性。当设置为`true`时，这个参数会强制 Apache 服务将其临时文件写入一个私有的`/tmp/`目录，其他人无法访问。因此，如果你担心 Apache 可能会将敏感信息写入其临时文件，你会想使用这个特性。（你可以在`systemd.exec`手册页面上阅读更多关于这个特性以及其他安全特性的信息。）另外，请注意，如果你完全不设置这个参数，它将默认为`false`，这意味着你将不会得到这种保护。

+   `Restart=`：有时候，如果服务停止了，你可能希望它自动重新启动。在这种情况下，我们使用了`on-abort`参数，这意味着如果 Apache 服务因不正常的信号而崩溃，`systemd`会自动重新启动它。

好的，`[Service]`部分就到此为止。让我们继续`[Install]`部分，看起来像这样：

```
[Install]
WantedBy=multi-user.target
```

这个命名法看起来有点奇怪，因为我们似乎并没有在这里安装什么东西。实际上，它的作用是控制在启用或禁用某个单元时会发生什么。在这种情况下，我们表示希望 Apache 服务在`multi-user.target`单元中启用，这将导致服务在机器启动到多用户目标时自动启动。（我们稍后会介绍目标和启动过程。现在，只需要理解多用户目标是指机器完全启动并准备好使用的时候。对于老一辈的 SysV 用户来说，这里的目标相当于 SysV 的运行级别。）

## 理解安全外壳服务文件

有一些不同的内容，让我们看看安全外壳服务的服务文件，在这台 Ubuntu 机器上，它是`/lib/systemd/system/ssh.service`文件。这里是`[Unit]`部分：

```
[Unit]
Description=OpenBSD Secure Shell server
Documentation=man:sshd(8) man:sshd_config(5)
After=network.target auditd.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run
```

在`[Unit]`部分，我们看到`ConditionPathExists=`参数，这是之前没有看到的。它检查文件是否存在或不存在。在这种情况下，我们在文件路径前面看到一个感叹号（`!`），这意味着我们在检查指定文件的不存在。如果`systemd`发现该文件，它将不会启动安全外壳服务。如果我们去掉感叹号，`systemd`只有在该文件*存在*时才会启动服务。所以，如果我们想防止安全外壳服务启动，所需要做的就是在`/etc/ssh/`目录中创建一个虚拟文件，如下所示：

```
sudo touch /etc/ssh/sshd_not_to_be_run
```

我不确定这个功能到底有多有用，因为如果你不希望服务运行，直接禁用它也一样简单。但是，如果你觉得以后可能会用到这个功能，它就在那里为你提供。

接下来是`[Service]`部分：

```
[Service]
EnvironmentFile=-/etc/default/ssh
ExecStartPre=/usr/sbin/sshd -t
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
ExecReload=/usr/sbin/sshd -t
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=255
Type=notify
RuntimeDirectory=sshd
RuntimeDirectoryMode=0755
```

在`[Service]`部分，我们看到了一些新的参数：

+   `EnvironmentFile=`：这个参数会让`systemd`从指定的文件中读取环境变量列表。路径前的减号（`-`）告诉`systemd`如果文件不存在，不用担心，还是照常启动服务。

+   `ExecStartPre=`：这告诉`systemd`在启动服务之前执行一个指定的命令，通过`ExecStart=`参数来启动服务。在这个例子中，我们希望运行`sshd -t`命令，它会测试安全外壳配置，以确保其有效。

+   `KillMode=`：我之前已经告诉过你，`systemd`的一个优点是它能在你需要向服务发送杀死信号时，停止该服务的所有进程。如果你没有在服务文件中包含这个参数，它就是默认行为。不过，有时你可能并不想这样。通过将此参数设置为`process`，杀死信号将只会终止服务的主进程，所有其他关联进程将继续运行。（你可以在`systemd.kill`手册页中阅读更多关于此参数的内容。）

+   `Restart=`：这次，它将不再因为`on-abort`自动重启停止的服务，而是因为`on-failure`重新启动。因此，除了因为异常信号重新启动服务外，`systemd`还会因不正常的退出代码、超时或看门狗事件重新启动该服务。（如果你在想，看门狗是一种内核特性，它可以在发生某些无法恢复的错误时重新启动服务。）

+   `RestartPreventExitStatus=`：当接收到某个退出代码时，这会防止服务自动重启。在这个例子中，我们不希望服务在退出代码为`255`时重启。（有关退出代码的更多信息，请参见`systemd.exec`手册页中的`$EXIT_CODE, $EXIT_STATUS_`部分。）

+   `Type=`：对于此服务，类型为`notify`，而不是我们在前一个示例中看到的`forking`。这意味着当服务启动完成后，服务将发送一个通知消息。发送通知消息后，`systemd`将继续加载后续单元。

+   `RuntimeDirectory=`和`RuntimeDirectoryMode=`：这两个指令会在`/run/`目录下创建一个运行时目录，然后设置该目录的权限值。在这个例子中，我们将目录的权限设置为`0755`，这意味着目录的所有者将拥有读、写和执行权限，而其他人将仅有读和执行权限。

最后，这里是`[Install]`部分：

```
[Install]
WantedBy=multi-user.target
Alias=sshd.service
```

在`[Install]`部分，我们看到`Alias=`参数，这非常实用。因为某些服务在不同的 Linux 发行版上可能有不同的名称。例如，安全外壳服务在 Red Hat 类型的系统上是`sshd`，而在 Debian/Ubuntu 系统上是`ssh`。通过包含`Alias=sshd.service`这一行，我们可以通过指定任意一个名称来控制服务。

## 理解 timesyncd 服务文件

对于最后一个示例，我想向你展示`timesyncd`服务的服务文件。它是`/lib/systemd/system/systemd-timesyncd.service`文件。首先是`[Unit]`部分：

```
[Unit]
Description=Network Time Synchronization
Documentation=man:systemd-timesyncd.service(8)
ConditionCapability=CAP_SYS_TIME
ConditionVirtualization=!container
DefaultDependencies=no
After=systemd-sysusers.service
Before=time-set.target sysinit.target shutdown.target
Conflicts=shutdown.target
Wants=time-set.target time-sync.target
```

对于这个文件，我主要想关注与安全相关的参数。在`[Unit]`部分，有`ConditionCapability=`参数，稍后我将解释。`Wants=`行与安全无关，它定义了该服务的依赖单元。如果这些依赖单元在该服务启动时未运行，`systemd`将尝试启动它们。如果它们未能启动，该服务仍将继续启动。

接下来，我们将查看`[Service]`部分，在这里我们将看到更多与安全相关的参数。（由于篇幅限制，这里只展示文件的一部分，欢迎在您自己的虚拟机上查看完整文件。）

```
[Service]
AmbientCapabilities=CAP_SYS_TIME
CapabilityBoundingSet=CAP_SYS_TIME
ExecStart=!!/lib/systemd/systemd-timesyncd
LockPersonality=yes
MemoryDenyWriteExecute=yes
. . .
. . .
ProtectSystem=strict
Restart=always
RestartSec=0
RestrictAddressFamilies=AF_UNIX AF_INET AF_INET6
RestrictNamespaces=yes
RestrictRealtime=yes
RestrictSUIDSGID=yes
RuntimeDirectory=systemd/timesync
StateDirectory=systemd/timesync
SystemCallArchitectures=native
SystemCallErrorNumber=EPERM
SystemCallFilter=@system-service @clock
Type=notify
User=systemd-timesync
WatchdogSec=3min 
```

`AmbientCapabilities=` 和 `CapabilityBoundingSet=` 参数都设置为 `CAP_SYS_TIME`，`[Unit]` 部分的 `ConditionCapability=` 参数也是如此。在 `[Service]` 部分的末尾，我们看到 `User=systemd-timesync` 行，这告诉 `systemd` 以非特权帐户运行此服务。但是，设置系统时间需要 root 权限，而 `systemd-timesync` 用户没有该权限。我们可以通过为此用户分配一个根级别的内核能力来解决这个问题。在这种情况下，我们允许此用户设置系统时间，但不允许做其他操作。不过，一些系统可能无法实现 `AmbientCapabilities=` 指令。因此，`ExecStart=` 行中的双重感叹号 (`!!`) 告诉 `systemd` 以最小权限运行指定的服务。请注意，只有在系统无法处理 `AmbientCapabilities=` 指令时，这个双重感叹号选项才会生效。

注意

你可以通过输入 `man capabilities` 了解更多关于内核能力的信息。了解内核能力的一个重要点是，它们在不同的 CPU 架构之间可能有所不同。因此，适用于 ARM CPU 的能力集合与适用于 x86_64 CPU 的能力集合不同。

阅读 `[Service]` 部分的其余内容，你会看到很多显然是为了增强安全性的参数。我不会一一讲解，因为大多数参数你仅从名称就能大致知道它们的作用。对于那些不太明显的参数，我建议你查阅手册页。这些安全设置是一个强大的功能，你可以看到它们几乎在做与强制访问控制系统相同的工作。

最后，我们有 `[Install]` 部分：

```
[Install]
WantedBy=sysinit.target
Alias=dbus-org.freedesktop.timesync1.service
```

这里要注意的主要一点是，这个服务是 `sysinit.target` 所需要的，这意味着它会在系统初始化过程中启动。

我们只是略微触及了服务文件的应用范围。但由于有如此多不同的参数，我们只能合理地期望做一些简单的探索。你最好的做法是浏览手册页，熟悉相关内容，并在有问题时查阅手册页。

接下来，我们将讨论套接字单元。（幸运的是，这一部分不需要太长。）

# 理解套接字单元

套接字单元文件也位于 `/lib/systemd/system/` 目录下，文件名以 `.socket` 结尾。以下是我在一台 Ubuntu 服务器机器上的部分文件列表：

```
donnie@ubuntu20-10:/lib/systemd/system$ ls -l *.socket
-rw-r--r-- 1 root root  246 Jun  1  2020 apport-forward.socket
-rw-r--r-- 1 root root  102 Sep 10  2020 dbus.socket
-rw-r--r-- 1 root root  248 May 30  2020 dm-event.socket
-rw-r--r-- 1 root root  197 Sep 16 16:52 docker.socket
-rw-r--r-- 1 root root  175 Feb 26  2020 iscsid.socket
-rw-r--r-- 1 root root  239 May 30  2020 lvm2-lvmpolld.socket
-rw-r--r-- 1 root root  186 Sep 11  2020 multipathd.socket
-rw-r--r-- 1 root root  281 Feb  2 08:21 snapd.socket
-rw-r--r-- 1 root root  216 Jun  7  2020 ssh.socket
. . .
. . .
-rw-r--r-- 1 root root  610 Sep 20 10:16 systemd-udevd-kernel.socket
-rw-r--r-- 1 root root  126 Aug 30  2020 uuidd.socket
donnie@ubuntu20-10:/lib/systemd/system$
```

Socket 单元可以为我们做几件事情。首先，它们可以代替旧的 SysV 系统上的 `inetd` 和 `xinetd` *超级服务器* 守护进程。这意味着，我们可以避免让一个服务器守护进程一直运行，即使在不需要的时候也要一直运行。相反，我们可以大部分时间将它关闭，只在系统检测到一个进来的网络请求时才启动它。我们来看一个简单的例子，看看 Ubuntu 机器上的`ssh.socket`文件：

```
[Unit]
Description=OpenBSD Secure Shell server socket
Before=ssh.service
Conflicts=ssh.service
ConditionPathExists=!/etc/ssh/sshd_not_to_be_run
[Socket]
ListenStream=22
Accept=yes
[Install]
WantedBy=sockets.target
```

即使这个 socket 文件默认情况下会被安装，但它并不会默认启用。在 Ubuntu 的默认配置下，Secure Shell 服务是一直在运行的。在`[Unit]`部分，我们可以看到这两个有趣的指令：

+   `Before=ssh.service`：这告诉`systemd`在启动 Secure Shell 服务之前先启动这个 socket。

+   `Conflicts=ssh.service`：这告诉`systemd`，如果启用了这个 socket，就不允许 Secure Shell 服务正常运行。如果你启用这个 socket，正常的 SSH 服务会被关闭。

在`[Socket]`部分，我们看到这个 socket 监听在端口`22/tcp`，这是 Secure Shell 的默认端口。`Accept=yes`这一行有点具有误导性，因为它并不完全意味着你想象的那样。它实际上是指每当有新的连接请求时，服务会为每个请求生成一个新的实例。根据`systemd.socket`的手册页面，这个设置应该仅用于设计时考虑到旧的`inetd`和`xinetd`方案的服务。为了更好的性能，新的服务应该设计成不以这种方式运行。

为了演示这个是如何工作的，我首先想向你展示一下我在 Ubuntu 虚拟机上运行的`ssh`服务是否正常：

```
donnie@ubuntu20-10:~$ sudo systemctl is-active ssh
active
donnie@ubuntu20-10:~$
```

所以，它是`active`的，这意味着它作为一个正常的守护进程在运行。现在，让我们启用`ssh.socket`，然后看一下差异：

```
donnie@ubuntu20-10:~$ sudo systemctl enable --now ssh.socket
Created symlink /etc/systemd/system/sockets.target.wants/ssh.socket → /lib/systemd/system/ssh.socket.
donnie@ubuntu20-10:~$ sudo systemctl is-active ssh
inactive
donnie@ubuntu20-10:~$
```

所以，一旦我启用这个 socket，`Conflicts=`这一行就会自动关闭`ssh`服务。但是我仍然可以连接到这台机器，因为这个 socket 会自动启动 SSH 服务，恰好足够长的时间来处理连接请求。当服务不再需要时，它会自动进入休眠状态。

其次，注意到这个 socket 并没有提到启动哪个服务，或者它的可执行文件在哪。这是因为，当该 socket 被激活时，它会从`ssh.service`文件中自动获取这些信息。你不需要告诉它怎么做，因为任何 socket 文件的默认行为是从一个文件名相同前缀的服务文件中获取它的信息。

最后，socket 单元可以启用操作系统进程之间的通信。例如，一个 socket 可以从各种系统进程接收消息并将它们传递给日志系统，正如我们在这个`systemd-journald.socket`文件中看到的那样：

```
[Unit]
Description=Journal Socket
Documentation=man:systemd-journald.service(8) man:journald.conf(5)
DefaultDependencies=no
Before=sockets.target
. . .
. . .
IgnoreOnIsolate=yes
[Socket]
ListenStream=/run/systemd/journal/stdout
ListenDatagram=/run/systemd/journal/socket
SocketMode=0666
PassCredentials=yes
PassSecurity=yes
ReceiveBuffer=8M
Service=systemd-journald.service 
```

我们在这里看到的是，这个套接字并没有监听网络端口，而是监听来自 `/run/systemd/journal/stdout` 的 TCP 输出，以及来自 `/run/systemd/journal/socket` 的 UDP 输出。（`ListenStream=` 指令用于 TCP 源，`ListenDatagram=` 指令用于 UDP 源。`systemd.socket` 手册页没有明确说明这一点，所以你需要通过 DuckDuckGo 搜索来了解这一点。）

这里没有 `Accept=yes` 指令，因为与我们之前看到的安全 shell 服务不同，`journald` 服务不需要为每个传入连接生成一个新的实例。省略此设置时，它的默认值为 `no`。

`PassCredentials=yes` 行和 `PassSecurity=yes` 行使发送过程将安全凭证和安全上下文信息传递给接收套接字。如果省略这两个参数，它们的默认值为 `no`。为了提高性能，`ReceiveBuffer=` 行为缓冲区分配了 8 MB 的内存。

最后，`Service=` 行指定了服务。根据 `systemd.socket` 的手册页，只有在设置了 `Accept=no` 时才能使用此选项。手册页还指出，通常不需要此项，因为默认情况下，套接字仍会引用与套接字同名的服务文件。但如果使用此项，它可能会引入一些额外的依赖项，原本不会引入这些依赖。

# 理解路径单元

你可以使用路径单元让 `systemd` 监控特定文件或目录，以查看它们何时发生变化。当 `systemd` 检测到文件或目录发生变化时，它将激活指定的服务。我们将以**通用 Unix 打印系统**（**CUPS**）为例。

在 `/lib/systemd/system/cups.path` 文件中，我们看到：

```
[Unit]
Description=CUPS Scheduler
PartOf=cups.service
[Path]
PathExists=/var/cache/cups/org.cups.cupsd
[Install]
WantedBy=multi-user.target
```

`PathExists=` 行告诉 `systemd` 监控特定文件的变化，在这个例子中是 `/var/cache/cups/org.cups.cupsd` 文件。如果 `systemd` 检测到此文件的变化，它将激活打印服务。

# 总结

好了，我们又完成了一个章节，这是件好事。在本章中，我们研究了服务、套接字和路径单元文件的结构。我们看到了每种类型单元的三个部分，并查看了一些可以为这些部分定义的参数。当然，解释每个可用参数几乎是不可能的，所以我只展示了几个例子。接下来的几个章节中，我会给你更多的例子。

任何 IT 管理员都应该具备的一个重要技能是知道如何查找自己不知道的内容。对于 `systemd` 来说，这可能有点挑战性，因为相关信息分布在多个手册页中。我已经为你提供了一些查阅手册页的技巧，希望能帮到你。

接下来你需要掌握的技能是控制服务单元，这是下一章的主题。我们在那里见。

# 问题

1.  哪种单元用于监控文件和目录的变化？

    a. system

    b. 文件

    c. 路径

    d. 定时器

    e. 服务

1.  套接字单元可以：

    a. 如果有网络请求进入，自动通知用户

    b. 自动设置 Linux 和 Windows 机器之间的通信

    c. 监听网络连接，并充当防火墙

    d. 在检测到对该服务的连接请求时自动启动网络服务

1.  `[Install]` 部分的目的是什么？

    a. 它定义了在安装服务时需要安装的其他软件包。

    b. 它定义了启用或禁用单元时发生的情况。

    c. 它定义了特定于安装单元的参数。

    d. 它定义了特定于服务单元的参数。

# 答案

1.  c

1.  d

1.  b

# 深入阅读

`Systemd` 套接字单元：

[`www.linux.com/training-tutorials/end-road-systemds-socket-units/`](https://www.linux.com/training-tutorials/end-road-systemds-socket-units/)

`ListenStream` 和 `ListenDatagram=` 的区别：

[`unix.stackexchange.com/questions/517240/systemd-socket-listendatagram-vs-listenstream`](https://unix.stackexchange.com/questions/517240/systemd-socket-listendatagram-vs-listenstream)

监控路径和目录：

[`www.linux.com/topic/desktop/systemd-services-monitoring-files-and-directories/`](https://www.linux.com/topic/desktop/systemd-services-monitoring-files-and-directories/)
