

# 启动 – `init`程序

我们在*第四章*中已经了解了内核如何启动并启动第一个程序`init`。在*第五章*和*第六章*中，我们创建了不同复杂度的根文件系统，这些文件系统都包含了`init`程序。现在，是时候更详细地了解`init`程序，并探索它对系统其他部分的重要性。

`init`有很多实现版本。在本章中，我将描述三种主要的实现：BusyBox `init`、System V `init` 和 `systemd`。我将解释它们是如何工作的，以及哪些类型的系统最适合使用每种实现。部分内容涉及在大小、复杂性和灵活性之间做出权衡。我们将学习如何使用 BusyBox `init`和 System V `init`启动一个守护进程。同时，我们还将学习如何向`systemd`添加一个服务。

在本章中，我们将讨论以下主题：

+   在内核启动后

+   介绍`init`程序

+   BusyBox `init`

+   System V `init`

+   `systemd`

# 技术要求

要跟随示例操作，确保你具备以下条件：

+   一台至少有 90GB 空闲磁盘空间的 Ubuntu 24.04 或更高版本的 LTS 主机系统

+   Buildroot 2024.02.6 LTS 版

+   Yocto 5.0（scarthgap）LTS 版

你应该已经为*第六章*安装了 Buildroot 2024.02.6 LTS 版。如果还没有，请参考[*Buildroot 用户手册*](https://buildroot.org/downloads/manual/manual.html)中的*系统要求*部分，按照*第六章*的说明，在 Linux 主机上安装 Buildroot。

你应该已经在*第六章*构建了 5.0（scarthgap）LTS 版的 Yocto。如果还没有，请参考*兼容的 Linux 发行版*和*构建主机包*部分，按照[*Yocto 项目快速构建*](https://docs.yoctoproject.org/brief-yoctoprojectqs/)指南的说明，在 Linux 主机上构建 Yocto。

本章中使用的代码可以在本书 GitHub 仓库的章节文件夹中找到：[`github.com/PacktPublishing/Mastering-Embedded-Linux-Development`](https://github.com/PacktPublishing/Mastering-Embedded-Linux-Development)。

# 在内核启动后

在**第四章**中，我们看到内核启动代码会寻找根文件系统，可能是 `initramfs` 或内核命令行中由 `root=` 指定的文件系统。内核启动代码接着执行一个程序，默认情况下，对于 `initramfs` 是 `/init`，对于常规文件系统则是 `/sbin/init`。`init` 程序具有 `root` 权限，并且由于它是第一个运行的进程，因此它的 **进程 ID** (**PID**) 是 1。如果因为某些原因无法启动 `init`，内核将会 panic，系统无法启动。

`init` 程序是所有其他进程的祖先，如下面 `pstree` 命令在一个简单的嵌入式 Linux 系统上显示的那样：

```
# pstree -gn
init(1)-+-syslogd(63)
        |-klogd(66)
        |-dropbear(99)
        `-sh(100)---pstree(109) 
```

`init` 程序的工作是接管用户空间中的引导过程并使其运行。它可能像是一个运行 shell 脚本的简单 shell 命令——在**第五章**的开始有一个例子——但在大多数情况下，你将使用一个专门的 `init` 守护进程来执行以下任务：

+   启动其他守护进程并配置系统参数以及将系统配置到工作状态所需的其他任务。

+   可选地，在允许登录 shell 的终端上启动登录守护进程，如 `getty`。

+   采用因其直接父进程终止且线程组中没有其他进程而成为孤儿的进程。

+   响应其任何直接子进程终止，捕获 `SIGCHLD` 信号并收集返回值，以防止它们成为僵尸进程。我将在**第十七章**中详细讨论僵尸进程。

+   可选地，重启其他已终止的守护进程。

+   处理系统关机。

换句话说，`init` 管理系统从启动到关机的生命周期。有一种观点认为，`init` 很适合处理其他运行时事件，如新硬件的加入和模块的加载与卸载。这正是 `systemd` 所做的工作。

# 引入 init 程序

你最有可能在嵌入式设备中遇到的三种 init 程序是 BusyBox `init`、System V `init` 和 `systemd`。Buildroot 提供这三种程序，其中 BusyBox `init` 是默认选项。Yocto 项目允许你在 System V `init` 和 `systemd` 之间选择，默认是 System V `init`。虽然 Yocto 的小型发行版包含 BusyBox `init`，但大多数其他发行版层并不包含它。

以下表格列出了一些指标，用于比较三者：

| **指标** | **BusyBox init** | **System V init** | **systemd** |
| --- | --- | --- | --- |
| 复杂度 | 低 | 中 | 高 |
| 启动速度 | 快 | 慢 | 中等 |
| 必需的 shell | ash | dash 或 bash | 无 |
| 可执行文件数量 | 1(*) | 4 | 50 |
| libc | 任何 | 任何 | glibc |
| 大小（MB） | < 0.1(*) | 0.1 | 34(**) |

表 13.1 – BusyBox init、System V init 和 systemd 的比较

(*) BusyBox `init` 是 BusyBox 单一可执行文件的一部分，经过优化以减少磁盘空间占用。

(**) 基于 `systemd` 的 Buildroot 配置。

从广义上讲，从 BusyBox `init` 到 `systemd`，灵活性和复杂性都有所增加。

# BusyBox init

BusyBox 有一个最小化的 `init` 程序，它使用 `/etc/inittab` 配置文件在启动时启动程序，在关机时停止程序。实际的工作由 shell 脚本完成，按照约定，这些脚本放在 `/etc/init.d` 目录中。

`init` 从读取 `/etc/inittab` 开始。该文件包含一系列程序，每行一个，格式如下：

```
<id>::<action>:<program> 
```

这些参数的作用是：

+   `id`：命令的控制终端

+   `action`：何时以及如何运行程序

+   `program`：要运行的程序及其所有命令行参数

这些操作是：

+   `sysinit`：在 `init` 启动时，先于其他类型的操作运行该程序。

+   `respawn`：运行该程序，并在其终止时重新启动。用于将程序作为守护进程运行。

+   `askfirst`：与 `respawn` 相同，但它会在控制台上显示**请按 Enter 键激活此控制台**的消息，按下 *Enter* 后运行程序。用于在终端启动交互式外壳，而无需提示输入用户名或密码。

+   `once`：运行该程序一次，但如果它终止，不会尝试重新启动它。

+   `wait`：运行该程序并等待其完成。

+   `restart`：当 `init` 接收到 `SIGHUP` 信号时运行该程序，表示它应该重新加载 `inittab` 文件。

+   `ctrlaltdel`：当 `init` 接收到 `SIGINT` 信号时运行该程序，通常是因为在控制台上按下 *Ctrl + Alt + Del*。

+   `shutdown`：当 `init` 关闭时运行该程序。

这里有一个小例子，它挂载了 `proc` 和 `sysfs`，然后在串口接口上运行一个外壳：

```
null::sysinit:/bin/mount -t proc proc /proc
null::sysinit:/bin/mount -t sysfs sysfs /sys
console::askfirst:-/bin/sh 
```

对于一些简单的项目，如果你只需要启动少量守护进程并在串口终端上启动登录外壳，手动编写脚本是非常容易的。如果你正在创建一个**自主定制**（**RYO**）的嵌入式 Linux，这种方法是适当的。然而，你会发现，随着需要配置的内容增多，手写的 `init` 脚本会迅速变得不可维护。它们不是很模块化，每次添加或删除新组件时都需要更新。

## Buildroot init 脚本

Buildroot 多年来有效地使用 BusyBox `init`。Buildroot 在 `/etc/init.d/` 中有两个脚本，分别是 `rcS` 和 `rcK`（`rc` 代表“运行命令”）。`rcS` 脚本在启动时运行。它遍历 `/etc/init.d/` 中所有名称以大写字母 `S` 开头并跟随两位数字的脚本，并按数字顺序运行它们。这些是启动脚本。`rcK` 脚本在关机时运行。它遍历所有以大写字母 `K` 开头并跟随两位数字的脚本，并按数字顺序运行它们。这些是停止脚本。

有了这个结构，Buildroot 软件包可以提供自己的启动和停止脚本，使得系统变得可扩展。两位数字控制 `init` 脚本的执行顺序。如果你正在使用 Buildroot，这个结构是透明的。如果没有使用，你可以将其作为编写自己 BusyBox `init` 脚本的模型。

像 BusyBox `init` 一样，System V `init` 依赖于 `/etc/init.d` 中的 shell 脚本和 `/etc/inittab` 配置文件。虽然这两种 `init` 系统在许多方面相似，但 System V `init` 拥有更多功能和更长的历史。

# System V init

这个 `init` 程序的灵感来源于 Unix System V，追溯到上世纪 80 年代中期。大多数 Linux 发行版中常见的版本最初由 Miquel van Smoorenburg 编写。直到最近，它一直是几乎所有桌面和服务器发行版以及许多嵌入式系统的 `init` 守护进程。然而，近年来它已被 `systemd` 替代，我们将在下一节中介绍它。

BusyBox 的 `init` 守护进程只是一个简化版的 System V `init`。与 BusyBox `init` 相比，System V `init` 有两个优势：

+   首先，启动脚本采用众所周知的模块化格式编写，便于在构建时或运行时添加新软件包。

+   其次，它具有 **运行级别** 的概念，允许在从一个运行级别切换到另一个运行级别时，一次性启动或停止一组程序。

有八个运行级别，从 `0` 到 `6`，再加上 `S`：

+   `S`：执行启动任务

+   `0`：停止系统

+   `1` 到 `5`：可供一般使用

+   `6`：重新启动系统

级别 `1` 到 `5` 可以根据需要使用。在大多数桌面 Linux 发行版中，它们被分配为：

+   `1`：单用户模式

+   `2`：没有网络配置的多用户模式

+   `3`：具有网络配置的多用户模式

+   `4`：未使用

+   `5`：具有图形登录的多用户模式

`init` 程序启动 `/etc/inittab` 文件中 `initdefault` 行指定的默认运行级别：

```
id:3:initdefault: 
```

你可以使用 `telinit <runlevel>` 命令在运行时更改运行级别，这会向 `init` 发送一条消息。你可以使用 `runlevel` 命令查找当前运行级别和先前的运行级别。以下是一个示例：

```
# runlevel
N 5
# telinit 3
INIT: Switching to runlevel: 3
# runlevel
5 3 
```

`runlevel` 命令的初始输出是 `N 5`。`N` 表示没有先前的运行级别，因为自启动以来运行级别没有变化。当前的运行级别是 `5`。在更改运行级别后，输出为 `5` `3`，表示从 `5` 过渡到 `3`。

`halt` 和 `reboot` 命令分别切换到运行级别 `0` 和 `6`。你可以通过在内核命令行中指定不同的运行级别（从 `0` 到 `6` 的单个数字）来覆盖默认的运行级别。例如，要强制默认运行级别为单用户模式，你可以在内核命令行中添加 `1`，如下所示：

```
console=ttyAMA0 root=/dev/mmcblk1p2 1 
```

每个运行级别都有一组杀死脚本，用于停止进程，还有一组启动脚本，用于启动进程。当进入新运行级别时，`init`首先运行杀死脚本，然后是新级别的启动脚本。当前正在运行的守护进程，如果在新运行级别中既没有启动脚本也没有杀死脚本，将会收到`SIGTERM`信号。换句话说，切换运行级别时的默认操作是终止守护进程，除非另有指示。

事实上，嵌入式 Linux 中并不常用运行级别（runlevel）。大多数设备直接启动到默认运行级别并保持在那里。我觉得这部分原因是大多数人并不了解它们。

**提示**

运行级别是一种简单便捷的方式，用于在不同模式之间切换，例如从生产模式切换到维护模式。

系统 V `init`是 Buildroot 和 Yocto 项目中的一种选项。在这两种情况下，`init`脚本都已去除任何`bash` shell 的特定内容，因此它们能够在 BusyBox 的`ash` shell 中工作。然而，Buildroot 通过将 BusyBox 的`init`程序替换为系统 V 的`init`并添加一个模拟 BusyBox 行为的`inittab`，稍微作弊了一下。Buildroot 除了`0`和`6`运行级别（它们用于停止或重启系统）外，不实现其他运行级别。

接下来，我们来看一些细节。以下示例取自 Yocto 项目 5.0 版本。其他 Linux 发行版可能会略有不同地实现`init`脚本。

## inittab

`init`程序首先通过读取`/etc/inittab`配置文件中的条目来定义每个运行级别发生的操作。其格式是前面章节中描述的 BusyBox `inittab`的扩展版。因为 BusyBox 本来就从系统 V 借用了这一格式，所以这并不令人惊讶。

每个`inittab`条目的格式如下：

```
<id>:<runlevels>:<action>:<process> 
```

这些字段包括：

+   `id`：最多四个字符的唯一标识符

+   `runlevels`：此条目所属的运行级别

+   `action`：命令的运行时机和方式

+   `process`：要运行的命令

这些操作与 BusyBox 的`init`相同：`sysinit`、`respawn`、`once`、`wait`、`restart`、`ctrlaltdel`和`shutdown`。然而，系统 V 的`init`没有`askfirst`，这是 BusyBox 特有的。

以下是 Yocto 项目在为`qemuarm`机器构建`core-image-minimal`时提供的完整`inittab`：

```
# /etc/inittab: init(8) configuration.
# $Id: inittab,v 1.91 2002/01/25 13:35:21 miquels Exp $
# The default runlevel.
id:5:initdefault:
# Boot-time system configuration/initialization script.
# This is run first except when booting in emergency (-b) mode.
si::sysinit:/etc/init.d/rcS
# What to do in single-user mode.
~~:S:wait:/sbin/sulogin
# /etc/init.d executes the S and K scripts upon change
# of runlevel.
#
# Runlevel 0 is halt.
# Runlevel 1 is single-user.
# Runlevels 2-5 are multi-user.
# Runlevel 6 is reboot.
l0:0:wait:/etc/init.d/rc 0
l1:1:wait:/etc/init.d/rc 1
l2:2:wait:/etc/init.d/rc 2
l3:3:wait:/etc/init.d/rc 3
l4:4:wait:/etc/init.d/rc 4
l5:5:wait:/etc/init.d/rc 5
l6:6:wait:/etc/init.d/rc 6
# Normally not reached, but fallthrough in case of emergency.
z6:6:respawn:/sbin/sulogin
AMA0:12345:respawn:/sbin/getty 115200 ttyAMA0
# /sbin/getty invocations for the runlevels
#
# The "id" field MUST be the same as the last
# characters of the device (after "tty").
#
# Format:
# <id>:<runlevels>:<action>:<process>
#
1:2345:respawn:/sbin/getty 38400 tty1 
```

第一个`id:5:initdefault`条目将默认运行级别设置为`5`。接下来的`si::sysinit`条目在启动时运行`/etc/init.d/rcS`脚本。`rcS`脚本所做的就是进入`S`运行级别：

```
#!/bin/sh
<…>
exec /etc/init.d/rc S 
```

因此，第一次进入的运行级别是`S`，接着是默认运行级别`5`。请注意，运行级别`S`不会被记录，也不会在`runlevel`命令中作为之前的运行级别显示出来。

从`l0`到`l6`的七个条目，在运行级别发生变化时会执行`/etc/init.d/rc`脚本。`rc`脚本负责处理启动和杀死脚本。

再往下查看，找到一个运行`getty`守护进程的条目：

```
AMA0:12345:respawn:/sbin/getty 115200 ttyAMA0 
```

该条目会在进入运行级别`1`到`5`时，在`/dev/ttyAMA0`上生成一个登录提示，允许你登录并获得一个交互式终端。`ttyAMA0`设备是 QEMU 仿真中的 Arm Versatile 开发板的串行控制台。其他开发板的串行控制台可能会有不同的设备名称。

最后一项会在`/dev/tty1`上运行另一个`getty`守护进程：

```
1:2345:respawn:/sbin/getty 38400 tty1 
```

该条目会在进入运行级别`2`到`5`时触发。`tty1`设备是一个虚拟控制台，当你在构建内核时启用了`CONFIG_FRAMEBUFFER_CONSOLE`或`VGA_CONSOLE`选项，它会映射到一个图形屏幕。

桌面 Linux 发行版通常会在虚拟终端 1 到 6 上启动六个`getty`守护进程，`tty7`则保留用于图形屏幕。Ubuntu 和 Arch Linux 是值得注意的例外，因为它们使用`tty1`来显示图形界面。你可以通过组合键*Ctrl + Alt + F1*到*Ctrl + Alt + F6*在虚拟终端之间切换。嵌入式设备很少使用虚拟终端。

## init.d 脚本

每个需要响应运行级别变化的组件，在`/etc/init.d`中都有一个脚本来执行该变化。脚本应该接收两个参数：`start`和`stop`。我会在*添加一个新的守护进程*部分给出每个的示例。

`/etc/init.d/rc`运行级别处理脚本接收要切换的运行级别作为参数。每个运行级别都有一个名为`rc<runlevel>.d`的目录：

```
# ls -d /etc/rc*
/etc/rc0.d  /etc/rc2.d  /etc/rc4.d  /etc/rc6.d
/etc/rc1.d  /etc/rc3.d  /etc/rc5.d  /etc/rcS.d 
```

你会找到一组脚本，这些脚本以大写字母`S`开头，后面跟着两个数字。你也可能会找到以大写字母`K`开头的脚本。这些是启动和终止脚本。以下是`5`级别运行时的脚本示例：

```
# ls /etc/rc5.d
S01networking   S20hwclock.sh  S99rmnologin.sh  S99stop-bootlogd
S15mountnfs.sh  S20syslog 
```

这些实际上是指向`init.d`中对应脚本的符号链接。`rc`脚本首先运行所有以`K`开头的脚本，传入`stop`参数。然后，它运行所有以`S`开头的脚本，传入`start`参数。再次强调，两个数字的代码是用来表示脚本执行的顺序的。

## 添加一个新的守护进程

假设你有一个名为`simpleserver`的程序，它作为传统的 Unix 守护进程运行；换句话说，它会分叉并在后台运行。这个程序的代码位于`MELD/Chapter13/simpleserver`。对应的`init.d`脚本（见下文）位于`MELD/Chapter13/simpleserver-sysvinit/init.d`：

```
#! /bin/sh
case "$1" in
   start)
     echo "Starting simpelserver"
     start-stop-daemon -S -n simpleserver -a /usr/bin/simpleserver
     ;;
   stop)
     echo "Stopping simpleserver"
     start-stop-daemon -K -n simpleserver
     ;;
   *)
     echo "Usage: $0 {start|stop}"
     exit 1
esac
exit 0 
```

`start-stop-daemon`是一个简化后台进程操作的程序。它最初来自 Debian 安装包（`dpkg`），但大多数嵌入式系统使用的是来自 BusyBox 的版本。使用`-S`参数运行`start-stop-daemon`时，会启动守护进程，确保每次只有一个实例在运行。使用`-K`参数运行`start-stop-daemon`时，会通过发送`SIGTERM`信号（默认）来停止守护进程，通知守护进程是时候终止了。

要使`simpleserver`正常工作，请将`init.d`脚本复制到`/etc/init.d`并使其可执行。然后，从你希望该程序运行的每个运行级别添加链接——在此情况下，只有默认运行级别`5`：

```
# cd /etc/init.d/rc5.d
# ln -s ../init.d/simpleserver S99simpleserver 
```

数字`99`表示这是最后启动的程序之一。

**重要说明**

请记住，可能还会有其他以`S99`开头的链接，在这种情况下，`rc`脚本会按字母顺序运行它们。

在嵌入式设备中，通常不需要过多担心关机操作，但如果有需要处理的事项，可以在`0`级和`6`级添加 kill 链接：

```
# cd /etc/init.d/rc0.d
# ln -s ../init.d/simpleserver K01simpleserver
# cd /etc/init.d/rc6.d
# ln -s ../init.d/simpleserver K01simpleserver 
```

我们可以绕过运行级别和顺序，直接测试和调试`init.d`脚本。

## 启动和停止服务

你可以通过直接调用脚本与`/etc/init.d`中的脚本交互。以下是一个使用`syslog`脚本的示例，该脚本控制`syslogd`和`klogd`守护进程：

```
# /etc/init.d/syslog --help
Usage: syslog { start | stop | restart }
# /etc/init.d/syslog stop
Stopping syslogd/klogd: stopped syslogd (pid 198)
stopped klogd (pid 201)
done
# /etc/init.d/syslog start
Starting syslogd/klogd: done 
```

所有脚本都实现了`start`和`stop`，它们还应该实现`help`。有些脚本还实现了`status`，可以告诉你服务是否正在运行。仍然使用 System V `init`的主流发行版有一个名为`service`的命令，用于启动和停止服务，该命令隐藏了直接调用脚本的细节。

System V `init`是一个简单的`init`守护进程，已经为 Linux 管理员服务了数十年。虽然运行级别提供了比 BusyBox `init`更高的复杂性，但 System V `init`仍然无法监控服务并在需要时重新启动它们。随着 System V `init`逐渐显得过时，最受欢迎的 Linux 发行版已经转向了`systemd`。

# systemd

`systemd` ([`systemd.io/`](https://systemd.io/)) 自我定义为*系统和服务管理器*。该项目由 Lennart Poettering 和 Kay Sievers 于 2010 年发起，旨在创建一套集成的工具，用于管理基于`init`守护进程的 Linux 系统。它还包括设备管理（`udev`）和日志记录等多个功能。`systemd`是最新的技术，并且仍在快速发展中。它在桌面和服务器 Linux 发行版中很常见，且在嵌入式 Linux 系统中越来越受欢迎。那么，它比 System V `init`更好在哪里呢？

+   配置更简单且更有逻辑性（理解后即可）。与复杂的 Shell 脚本不同，`systemd`使用单位配置文件，这些文件采用一种明确定义的格式编写。

+   服务之间有明确的依赖关系。这比只能控制脚本执行顺序的两位数字系统有了巨大的改进。

+   在安全性方面，为每个服务设置权限和资源限制是很容易的。

+   它可以监控服务并在需要时重新启动它们。

+   服务是并行启动的，从而减少了启动时间。

这里无法提供关于`systemd`的完整描述。与 System V `init`一样，我将重点介绍基于`systemd`版本 255 的 The Yocto Project 5.0 发布版的嵌入式用例示例。

## 使用 The Yocto Project 和 Buildroot 构建 systemd

The Yocto Project 中的默认`init`守护进程是 System V。要选择`systemd`，请在`conf/local.conf`中添加以下行：

```
INIT_MANAGER = "systemd" 
```

默认情况下，Buildroot 使用 BusyBox 的`init`。你可以通过`menuconfig`选择`systemd`，方法是进入**系统配置 | 初始化系统**菜单。你还需要将工具链配置为使用`glibc`作为 C 库，因为`systemd`官方不支持`uClibc-ng`或`musl`。此外，内核的版本和配置也有限制。`systemd`源代码顶层的`README`文件中列出了库和内核的所有依赖关系。

## 介绍目标、服务和单元

在描述`systemd`如何工作之前，我需要介绍三个关键概念：

+   **单元**：一个描述目标、服务或其他几个事物的配置文件。单元是包含属性和值的文本文件。

+   **服务**：一个可以启动和停止的守护进程，类似于 System V 的`init`服务。

+   **目标**：一组服务，类似于 System V 的`init`运行级别。默认目标由所有在启动时启动的服务组成。

你可以使用`systemctl`命令来更改状态并了解当前发生了什么。

### 单元

配置的基本项是**单元**文件。单元文件位于四个不同的位置：

+   `/etc/systemd/system`：本地配置

+   `/run/systemd/system`：运行时配置

+   `/usr/lib/systemd/system`：分发级别的配置（默认位置）

+   `/lib/systemd/system`：分发级别的配置（传统默认位置）

在查找单元时，`systemd`会按前述顺序搜索这些目录，找到匹配项后停止搜索。你可以通过在`/etc/systemd/system`中放置同名的单元来覆盖分发级别单元的行为。你还可以通过创建一个空文件或链接到`/dev/null`来完全禁用一个单元。

所有单元文件都以`[Unit]`标记的部分开始，其中包含基本信息和依赖关系。例如，以下是 D-Bus 服务`/lib/systemd/system/dbus.service`的`Unit`部分：

```
[Unit]
Description=D-Bus System Message Bus
Documentation=man:dbus-daemon(1)
Requires=dbus.socket 
```

除了描述和文档引用外，还有一个通过`Requires`关键字表达的对`dbus.socket`单元的依赖关系。这告诉`systemd`在启动 D-Bus 服务时创建一个本地套接字。

依赖关系通过`Requires`、`Wants`和`Conflicts`关键字表达：

+   `Requires`：此单元依赖的单元列表；这些单元会在该单元启动时启动。

+   `Wants`：一种比`Requires`更弱的形式；即使这些依赖项中的任何一个未能启动，该单元仍然会继续运行。

+   `Conflicts`：一种负依赖关系；当此单元启动时，这些单元会被停止，反之，如果其中一个单元随后重新启动，则此单元会被停止。

这三个关键字定义了**外向依赖**。它们用于在 *目标* 之间创建依赖关系。还有一组依赖关系称为**内向依赖**，用于在 *服务* 和 *目标* 之间创建链接。换句话说，外向依赖用于创建在系统从一个状态切换到另一个状态时需要启动的目标列表，内向依赖用于确定在进入任何状态时应启动或停止的服务。内向依赖通过 `WantedBy` 关键字创建，我将在接下来的章节 *添加自定义服务* 中描述。

处理依赖关系会生成一个应启动或停止的单元列表。`Before` 和 `After` 关键字决定它们的启动顺序。停止顺序只是启动顺序的反向：

+   `Before`：在列出的单元之前启动该单元。

+   `After`：在列出的单元之后启动该单元。

例如，`After` 指令确保在网络子系统启动后启动以下 Web 服务器：

```
[Unit]
Description=Lighttpd Web Server
After=network.target 
```

在没有 `Before` 或 `After` 指令的情况下，单元会并行启动或停止，没有特定顺序。

### 服务

**服务** 是一个守护进程，可以像 System V `init` 服务一样启动和停止。一个服务有一个以 `.service` 结尾的单元文件。

服务单元有一个 `[Service]` 部分，描述了服务如何运行。以下是 `lighttpd.service` 中的 `[Service]` 部分：

```
[Service]
ExecStart=/usr/sbin/lighttpd -f /etc/lighttpd/lighttpd.conf -D
ExecReload=/bin/kill -HUP $MAINPID 
```

这些是在启动和重启服务时运行的命令。你可以在这里添加更多配置项，详情请参考 `systemd.service(5)` 手册页。

### 目标

**目标** 是一个将服务或其他类型的单元组合在一起的单元。目标在这方面是一个元服务，起到同步点的作用。目标只有依赖关系。目标的名称以 `.target` 结尾，例如 `multi-user.target`。目标是一个期望的状态，扮演着 System V `init` 运行级别的角色。以下是完整的 `multi-user.target`：

```
[Unit]
Description=Multi-User System
Documentation=man:systemd.special(7)
Requires=basic.target
Conflicts=rescue.service rescue.target
After=basic.target rescue.service rescue.target
AllowIsolate=yes 
```

这意味着基本目标必须在多用户目标之前启动。这还意味着，由于它与救援目标冲突，启动救援目标将首先停止多用户目标。救援目标和多用户目标不能同时运行，因为救援目标启动的是单用户模式。只有在系统恢复时，激活救援目标才有意义。

## 如何 systemd 启动系统

让我们看看 `systemd` 如何实现启动过程。内核启动 `systemd`，因为 `/sbin/init` 被符号链接到 `/lib/systemd/systemd`。`systemd` 运行 `default.target`，它始终是指向目标的链接：如果是文本登录，指向 `multi-user.target`，如果是图形环境，指向 `graphical.target`。如果默认目标是 `multi-user.target`，你将看到这个符号链接：

```
/etc/systemd/system/default.target -> /lib/systemd/system/multi-user.target 
```

通过在内核命令行中传递`system.unit=<new target>`来覆盖默认目标。

查找默认目标：

```
# systemctl get-default
multi-user.target 
```

启动类似`multi-user.target`的目标会创建一个依赖树，将系统带入工作状态。在典型的系统中，`multi-user.target`依赖于`basic.target`，后者依赖于`sysinit.target`，而`sysinit.target`又依赖于需要早期启动的服务。

打印系统依赖的文本图：

```
# systemctl list-dependencies 
```

列出所有服务及其当前状态：

```
# systemctl list-units --type service 
```

列出所有目标：

```
# systemctl list-units --type target 
```

现在我们已经看到了系统的依赖树，那么如何插入一个额外的服务呢？

## 添加你自己的服务

这是我们的`simpleserver`服务的单元：

```
[Unit]
Description=Simple server
[Service]
Type=forking
ExecStart=/usr/bin/simpleserver
[Install]
WantedBy=multi-user.target 
```

你可以在`MELD/Chapter13/simpleserver-systemd`中找到这个`simpleserver.service`文件。

`[Unit]`部分只包含一个描述，这个描述会出现在`systemctl`下。没有依赖项，因为这个服务非常简单。

`[Service]`部分指向可执行文件，并有一个标志表示它会分叉。如果`simpleserver`更简单，并在前台运行，`systemd`会为我们进行守护进程化，因此不需要`Type=forking`。

`[Install]`部分创建了一个到`multi-user.target`的传入依赖，这样当系统进入多用户模式时，我们的服务器会被启动。

一旦你将`simpleserver.service`文件放入`/etc/systemd/system`目录，你就可以使用`systemctl start simpleserver`和`sytemctl stop simpleserver`命令来启动和停止该服务。你还可以使用`systemctl`获取它的当前状态：

```
# systemctl status simpleserver
simpleserver.service - Simple server
  Loaded: loaded (/etc/systemd/system/simpleserver.service; disabled)
  Active: active (running) since Thu 1970-01-01 02:20:50 UTC; 8s ago
Main PID: 180 (simpleserver)
  CGroup: /system.slice/simpleserver.service
          └─180 /usr/bin/simpleserver -n
Jan 01 02:20:50 qemuarm systemd[1]: Started Simple server. 
```

此时，该服务仅在命令下启动和停止。为了使其持久化，你需要添加一个指向目标的永久依赖。`[Install]`部分表示，当启用该服务时，它会依赖`multi-user.target`，以便在启动时启动。

启用该服务：

```
# systemctl enable simpleserver
Created symlink from /etc/systemd/system/multiuser.target.wants/simpleserver.service to /etc/systemd/system/simpleserver.service. 
```

更新`systemd`依赖树而无需重启：

```
# systemctl daemon-reload 
```

你也可以为服务添加依赖，而无需编辑目标单元文件。一个目标可以有一个名为`<target_name>.target.wants`的目录，其中包含指向服务的链接。在此目录中创建一个链接就相当于将单元添加到目标的`[Wants]`列表中。`systemctl enable` `simpleserver`命令创建了以下链接：

```
/etc/systemd/system/multi-user.target.wants/simpleserver.service -> /etc/systemd/system/simpleserver.service 
```

如果一个重要服务崩溃，你可能希望重启它。为此，在`[Service]`部分添加以下标志：

```
Restart=on-abort 
```

其他`Restart`选项包括`on-success`、`on-failure`、`on-abnormal`、`on-watchdog`和`always`。

## 添加看门狗

许多嵌入式系统需要看门狗：如果一个关键服务停止工作，你需要采取行动。这通常意味着需要重启系统。大多数嵌入式 SoC 都有一个硬件看门狗，可以通过`/dev/watchdog`设备节点访问。**看门狗**在启动时会初始化一个超时时间。如果在超时时间内没有重置这个计时器，看门狗将会被触发，系统将重启。与看门狗驱动程序的接口在内核源代码的`Documentation/watchdog/`下进行了描述，驱动程序的代码位于`drivers/watchdog/`。

当有两个或更多关键服务需要通过看门狗进行保护时，就会出现一个问题。`systemd`提供了一个有用的功能，可以在多个服务之间分配看门狗。可以配置`systemd`期望服务定期发送保持活跃信号，并在未收到该信号时采取行动，从而创建一个软件看门狗。为了实现这一功能，你需要在守护进程中添加代码，发送保持活跃信号。守护进程读取`WATCHDOG_USEC`环境变量的值，并在此时间段内调用`sd_notify(false, "WATCHDOG=1")`。该时间段应设置为看门狗超时的约一半。`systemd`源代码中有相关示例。

要在服务单元中启用软件看门狗，可以在`[Service]`部分添加如下内容：

```
WatchdogSec=30s
Restart=on-watchdog
StartLimitInterval=5min
StartLimitBurst=4
StartLimitAction=reboot-force 
```

在这个例子中，服务每 30 秒需要接收到一次保持活跃的信号。如果保持活跃信号未能发送，服务会被重启，但如果在 5 分钟内重启超过四次，`systemd`会立即重启整个系统。关于这些设置的详细描述，可以参考`systemd.service(5)`手册页。

软件看门狗负责监控单个服务，但如果`systemd`本身失败、内核崩溃或硬件死锁该怎么办？在这些情况下，我们需要告诉`systemd`使用硬件看门狗。将`RuntimeWatchdogSec=<N>`添加到`/etc/systemd/system.conf`中。这将会在给定的`N`时间内重置看门狗，从而在`systemd`因某种原因失败时重启系统。这将是一个立即的硬重启或系统“重置”，没有任何优雅的关机过程。

## 嵌入式 Linux 的影响

`systemd`有许多对嵌入式 Linux 有用的功能。本章仅提到了其中的一些。其他功能包括资源控制（可以参考`systemd.slice(5)`和`systemd.resource-control(5)`手册页）、设备管理（`udev(7)`）、系统日志功能（`journald(5)`）、自动挂载文件系统的挂载单元以及`cron`作业的定时器单元。

你需要平衡这些功能与`systemd`的体积。即使是仅包含核心组件（`systemd`、`udevd`和`journald`）的最小构建，其存储空间也接近 10 MB，包括共享库。

你还需要记住，`systemd`的开发与内核和`glibc`紧密跟随，因此`systemd`的版本无法在比其发布版本早一到两年的内核和`glibc`上运行。

# 总结

每个 Linux 设备都需要某种形式的 `init` 程序。如果你设计的系统只需要在启动时启动少量守护进程，那么 BusyBox `init` 足够用了。如果你使用 Buildroot 作为构建系统，BusyBox `init` 通常也是一个不错的选择。

另一方面，如果你的系统在启动或运行时具有复杂的服务依赖关系，那么 `systemd` 是最佳选择。即便没有这样的复杂性，`systemd` 也有一些有用的功能，比如看门狗、远程日志等。如果你的存储空间足够，应该认真考虑使用 `systemd`。

与此同时，System V `init` 仍然存在。它已经被广泛理解，并且每个对我们来说重要的组件都有对应的 `init` 脚本。System V 仍然是 Yocto 项目参考发行版（Poky）的默认 `init`。在启动时间方面，`systemd` 对于类似的工作负载来说更快。然而，如果你追求的是最快的启动速度，简单的 BusyBox `init` 配合最小化的启动脚本则更胜一筹。

# 进一步学习

+   *systemd 系统与服务管理器* – [`systemd.io/`](https://systemd.io/)
