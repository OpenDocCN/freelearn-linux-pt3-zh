# *第二章*：理解 systemd 目录和文件

在本章中，我们将探讨各种`systemd`单元文件和配置文件，并解释几种类型的目的。我们还将简要了解与`systemd`相关的一些可执行文件。在此过程中，我们还会查看这些文件所在的目录。

这是本章将涵盖的主题：

+   理解`systemd`配置文件

+   理解`systemd`单元文件

+   理解`systemd`可执行文件

本章中的主题包含`systemd`的基本基础知识。我们将在接下来的章节中以此为基础进行拓展。

如果你准备好了，我们开始吧。

# 技术要求

如果你想跟着我做，你需要准备几个**虚拟机**（**VMs**）。在这里，我使用 Ubuntu Server 20.04 来处理 Ubuntu 方面的内容，使用 AlmaLinux 8 来处理 Red Hat 方面的内容。（你还会看到我用 Fedora 来指出一些内容，但你自己并不需要有 Fedora 虚拟机。）

查看以下链接，查看代码实操视频：[`bit.ly/3xL4os5`](https://bit.ly/3xL4os5)

# 理解 systemd 配置文件

在本节中，我们将查看控制`systemd`各个组件操作的配置文件。如果你想跟随我一起操作，你使用的发行版不太重要，因为所有启用`systemd`的发行版大致是相同的。好了—现在你可能在对我大喊：

*大致相同？为什么，Donnie，你之前告诉我们，systemd 在所有发行版中实现得都是一致的！这是怎么回事？*

这是一致的，因为管理和控制命令在所有发行版中都是相同的，但`systemd`生态系统不仅仅包括`init`系统，还包括其他几个不同的组件。这些组件是可选的，某些 Linux 发行版在默认配置中并不使用所有组件。正如你在这里看到的，几个组件在`/etc/systemd/`目录下有配置文件：

```
[donnie@localhost systemd]$ pwd
/etc/systemd
[donnie@localhost systemd]$ ls -l *.conf
-rw-r--r--. 1 root root  720 May 31  2016 bootchart.conf
-rw-r--r--. 1 root root  615 Mar 26  2020 coredump.conf
-rw-r--r--. 1 root root 1041 Mar 26  2020 journald.conf
-rw-r--r--. 1 root root 1042 Mar 26  2020 logind.conf
-rw-r--r--. 1 root root  584 Mar 26  2020 networkd.conf
-rw-r--r--. 1 root root  529 Mar 26  2020 pstore.conf
-rw-r--r--. 1 root root  764 Mar 26  2020 resolved.conf
-rw-r--r--. 1 root root  790 Mar 26  2020 sleep.conf
-rw-r--r--. 1 root root 1762 Mar 26  2020 system.conf
-rw-r--r--. 1 root root  677 Mar 26  2020 timesyncd.conf
-rw-r--r--. 1 root root 1185 Mar 26  2020 user.conf
[donnie@localhost systemd]$
```

`timesyncd.conf`文件，你在前面的代码片段中看到它排在倒数第二的位置，是你在某些地方看不到的组件之一。它用于将机器的时间同步到一个可信的外部源。你在这里看到它，但在`chronyd`上是看不见的，而且仅仅看到某个`systemd`组件的配置文件并不意味着该组件正在使用。在我提取前面代码片段的 Fedora 机器上，`networkd`、`resolved`和`timesyncd`组件都被禁用了。（与 RHEL 发行版类似，Fedora 使用`chronyd`来保持时间同步，但它仍然安装了`timesyncd`组件。）另一方面，如果你查看最新版本的 Ubuntu Server，你会发现这些可选组件默认启用。（我们稍后会看到如何判断一个服务是启用还是禁用。）

好的——让我们来看看这些配置文件的内容。我们将从查看 `system.conf` 文件开始，它设置了 `systemd` `init` 进程的配置。（由于空间原因，我只能在这里显示文件的一部分。你可以通过在虚拟机上运行 `less /etc/systemd/system.conf` 来查看完整文件。）以下是文件片段：

```
[Manager]
#LogLevel=info
#LogTarget=journal-or-kmsg
#LogColor=yes
#LogLocation=no
. . .
. . .
#DefaultLimitNICE=
#DefaultLimitRTPRIO=
#DefaultLimitRTTIME=
```

现在，我不会一行行地解释这个文件，因为我不想让你因为无聊而讨厌我。但说真的，在正常情况下，你可能永远不需要更改这些配置文件。如果你认为你可能需要对它们做些什么，最好的办法是阅读它们的相关手册页，手册页会详细说明每个参数的作用。诀窍是，对于大多数这些文件，你需要在文件名的前面加上 `systemd-` 字符串，才能找到它们的手册页。例如，要查看 `system.conf` 文件的手册页，输入以下命令：

```
man systemd-system.conf
```

此外，你可能已经注意到，在所有这些配置文件中，每一行都是注释掉的。这并不意味着这些行没有效果。相反，这意味着这些是默认的编译参数。要更改某些内容，你需要取消注释你想更改的参数行，并修改它的值。

专业提示

你可以使用 `apropos` 命令查找所有包含特定文本字符串的手册页，无论是手册页名称还是手册页描述中包含该字符串。例如，要查找所有匹配 `systemd` 字符串的页面，只需输入以下命令：`apropos systemd`。

你还可以输入 `man -k systemd`，这是 `apropos systemd` 的同义词。（我一开始就养成了总是输入 `apropos` 的习惯，并且一直没有改掉这个习惯。）如果尝试后没有任何结果，你可能需要重新构建手册页数据库，你可以通过输入 `sudo mandb` 来完成这一步。

好了，我想我们已经讨论够了配置文件。接下来，我们将讨论 `systemd` 单元文件。

# 理解 systemd 单元文件

不同于使用一组复杂的 Bash 脚本，`systemd` `init` 系统通过各种类型的 *单元* 文件来控制系统和服务操作。每个单元文件的文件名都会附带一个扩展名，说明它是哪种类型的单元文件。在我们查看这些文件之前，让我们先看看它们存放在哪里。

`/lib/systemd/system/` 目录是操作系统默认的单元文件存放位置，或者是你可能安装的任何软件包附带的单元文件。你可能会遇到需要修改这些单元文件或甚至创建自己单元文件的情况，但你不会在这个目录下进行操作。相反，你会在 `/etc/systemd/system/` 目录下进行。这个目录下与 `/lib/systemd/system/` 目录下同名的单元文件具有优先权。

小贴士

你可以通过输入以下命令阅读关于单元文件的内容：`man systemd.unit`。

在这个手册页的底部，你会看到它会引导你查看每种特定类型单元文件的其他手册页。你很快会发现，最棘手的部分是每当你需要查找有关某个特定单元配置参数的内容时，都必须在不同的手册页中查找。为了简化这个过程，你可以在`systemd.directives`手册页中查找特定的指令，这将引导你到包含该指令信息的手册页。

现在你知道了单元文件的位置，让我们来看一看它们*是什么*。

## 单元文件的类型

在`/lib/systemd/system`目录中，你会看到各种类型的单元文件，每种文件执行不同的功能。以下是一些常见类型的列表：

+   `service`：这些是服务的配置文件。它们取代了旧**System V**（**SysV**）系统中的初始化脚本。

+   `socket`：套接字可以启用不同系统服务之间的通信，或者当接收到连接请求时，它们可以自动唤醒一个处于休眠状态的服务。

+   `slice`：切片单元用于配置`cgroups`。（我们将在*第二部分*，*理解 cgroups*中详细介绍这些。）

+   `mount`和`automount`：这些包含由`systemd`控制的文件系统的挂载点信息。通常，它们会自动创建，因此你不需要做太多操作。

+   `target`：目标单元在系统启动时使用，用于分组单元并提供众所周知的同步点。（我们将在*第六章*，*理解 systemd 目标*中详细讲解这些。）

+   `timer`：定时器单元用于按计划调度任务。它们取代了旧的 cron 系统。（我们将在*第七章*，*理解 systemd 定时器*中与它们一起工作。）

+   `path`：路径单元用于通过基于路径的激活启动服务。（我们将在*第三章*，*理解服务、路径和套接字单元*中介绍服务、路径和套接字单元。）

+   `swap`：交换单元包含关于交换分区的信息。

    关于我们单元文件的基本描述差不多就是这些。我们将在后续章节中深入探讨它们的细节。

# 理解 systemd 可执行文件

通常，我们会在`bin/`或`sbin/`目录中查找程序的可执行文件，的确你会在那里找到一些`systemd`工具的可执行文件，但大多数`systemd`的可执行文件实际上位于`/lib/systemd/`目录。为了节省空间，下面只列出部分：

```
donnie@donnie-TB250-BTC:/lib/systemd$ ls -l
total 7448
-rw-r--r--  1 root root 2367728 Feb  6  2020 libsystemd-shared-237.so
drwxr-xr-x  2 root root    4096 Apr  3  2020 network
-rw-r--r--  1 root root     699 Feb  6  2020 resolv.conf
-rwxr-xr-x  1 root root    1246 Feb  6  2020 set-cpufreq
drwxr-xr-x 24 root root   36864 Apr  3  2020 system
-rwxr-xr-x  1 root root 1612152 Feb  6  2020 systemd
-rwxr-xr-x  1 root root    6128 Feb  6  2020 systemd-ac-power
-rwxr-xr-x  1 root root   18416 Feb  6  2020 systemd-backlight
-rwxr-xr-x  1 root root   10304 Feb  6  2020 systemd-binfmt
-rwxr-xr-x  1 root root   10224 Feb  6  2020 systemd-cgroups-agent
-rwxr-xr-x  1 root root   26632 Feb  6  2020 systemd-cryptsetup
. . .
. . .
```

你会看到`systemd`本身的可执行文件在这里，还有`systemd`作为其系统的一部分运行的服务可执行文件。在一些 Linux 发行版中，你会在`/bin`或`/usr/bin`目录下看到指向这些可执行文件的符号链接。大部分情况下，你不会直接与这些文件交互，所以让我们继续讲解你会互动的内容。

`systemctl`工具用于控制`systemd`，你会经常使用它。它是一个多功能工具，可以为你做很多事情。它允许你查看不同的单元及其状态，并可以启用或禁用它们。现在，我们将查看一些`systemctl`命令，它们允许你查看不同类型的信息。稍后我们将讨论如何使用`systemctl`控制和编辑特定单元。如果你想跟着操作，启动一个虚拟机，开始动手吧。

需要注意的一点是，一些`systemctl`命令需要根权限，而另一些则不需要。如果你只是查看系统或单元信息，可以使用普通用户权限。如果需要更改配置，则需要使用根用户权限。好吧，我们开始吧。

我们首先列出`systemd`当前在内存中的活动单元。我们可以使用`systemctl list-units`命令来做到这一点。输出内容非常长，所以我这里只展示前几行：

```
[donnie@localhost ~]$ systemctl list-units
UNIT                                                                                      LOAD   ACTIVE SUB        DESCRIPTION
proc-sys-fs-binfmt_misc.automount                                                         loaded active waiting   Arbitrary Executable File Formats File System Automount Point
sys-devices-pci0000:00-0000:00:17.0-ata3-host2-target2:0:0-2:0:0:0-block-sda-sda1.device  loaded active plugged   WDC_WDS250G2B0A-00SM50 1
sys-devices-pci0000:00-0000:00:17.0-ata3-host2-target2:0:0-2:0:0:0-block-sda-sda2.device  loaded active plugged   WDC_WDS250G2B0A-00SM50 2
sys-devices-pci0000:00-0000:00:17.0-ata3-host2-target2:0:0-2:0:0:0-block-sda.device       loaded active plugged   WDC_WDS250G2B0A-00SM50
sys-devices-pci0000:00-0000:00:1b.2-0000:02:00.1-sound-card1.device                       loaded active plugged   GP104 High Definition Audio Controller
sys-devices-pci0000:00-0000:00:1b.3-0000:03:00.1-sound-card2.device                       loaded active plugged   GP104 High Definition Audio Controller
. . .
. . .
```

这是`automount`部分，展示了已挂载的各种设备。正如你所看到的，这不仅仅包括存储设备。

接下来，我们有挂载、路径和作用域单元，具体如下：

```
. . .
. . .
-.mount                                                                                   loaded active mounted   /
boot.mount                                                                                loaded active mounted   /boot
dev-hugepages.mount                                                                       loaded active mounted   Huge Pages File System
dev-mqueue.mount                                                                          loaded active mounted   POSIX Message Queue File System
home.mount                                                                                loaded active mounted   /home
run-user-1000.mount                                                                       loaded active mounted   /run/user/1000
sys-fs-fuse-connections.mount                                                             loaded active mounted   FUSE Control File System
sys-kernel-config.mount                                                                   loaded active mounted   Kernel Configuration File System
sys-kernel-debug.mount                                                                    loaded active mounted   Kernel Debug File System
tmp.mount                                                                                 loaded active mounted   Temporary Directory (/tmp)
var-lib-nfs-rpc_pipefs.mount                                                              loaded active mounted   RPC Pipe File System
cups.path                                                                                 loaded active running   CUPS Scheduler
systemd-ask-password-plymouth.path                                                        loaded active waiting   Forward Password Requests to Plymouth Directory Watch
systemd-ask-password-wall.path                                                            loaded active waiting   Forward Password Requests to Wall Directory Watch
init.scope                                                                                loaded active running   System and Service Manager
session-1.scope                                                                           loaded active abandoned Session 1 of user donnie
session-3.scope                                                                           loaded active abandoned Session 3 of user donnie
session-4.scope                                                                           loaded active running   Session 4 of user donnie
. . .
. . .
```

请注意，这里每个分区都有一个挂载单元。

继续向下滚动，你将看到服务、切片、套接字、交换、目标和定时器单元的类似显示。在底部，你将看到状态代码的简要说明以及简短的总结，内容如下：

```
LOAD   = Reflects whether the unit definition was properly loaded.
ACTIVE = The high-level unit activation state, i.e. generalization of SUB.
SUB    = The low-level unit activation state, values depend on unit type.
182 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
lines 136-190/190 (END)
```

使用`--all`选项还可以查看非活动的单元，像这样：

```
[donnie@localhost ~]$ systemctl list-units --all
  UNIT                                                                                                           LOAD      ACTIVE   SUB       DESCRIPTION
● boot.automount                                                                                                 not-found inactive dead      boot.automount
  proc-sys-fs-binfmt_misc.automount                                                                              loaded    active   waiting   Arbitrary Executable File Formats File System A
  dev-block-8:2.device                                                                                           loaded    active   plugged   WDC_WDS250G2B0A-00SM50 2
  dev-disk-by\x2did-ata\x2dWDC_WDS250G2B0A\x2d00SM50_181202802064.device                                         loaded    active   plugged   WDC_WDS250G2B0A-00SM50
  dev-disk-by\x2did-ata\x2dWDC_WDS250G2B0A\x2d00SM50_181202802064\x2dpart1.device                                 loaded    active   plugged   WDC_WDS250G2B0A-00SM50 1
  dev-disk-by\x2did-ata\x2dWDC_WDS250G2B0A\x2d00SM50_181202802064\x2dpart2.device                                loaded    active   plugged   WDC_WDS250G2B0A-00SM50 2
. . .
. . .
```

那是运气。我们在最上面找到了一单位未激活的单元。

你还可以使用`-t`选项查看特定类型的单元。例如，要查看仅服务单元，可以运行以下命令：

```
[donnie@localhost ~]$ systemctl list-units -t service
UNIT                                                                                      LOAD   ACTIVE SUB     DESCRIPTION
abrt-journal-core.service                                                                 loaded active running Creates ABRT problems from coredumpctl messages
abrt-oops.service                                                                         loaded active running ABRT kernel log watcher
abrt-xorg.service                                                                         loaded active running ABRT Xorg log watcher
abrtd.service                                                                             loaded active running ABRT Automated Bug Reporting Tool
alsa-state.service                                                                        loaded active running Manage Sound Card State (restore and store)
atd.service                                                                               loaded active running Deferred execution scheduler
auditd.service                                                                            loaded active running Security Auditing Service
avahi-daemon.service                                                                      loaded active running Avahi mDNS/DNS-SD Stack
. . .
. . .
```

你也可以以相同的方式查看其他单元。

现在，假设我们只想查看已死掉的服务。我们可以使用`--state`选项，像这样：

```
[donnie@localhost ~]$ systemctl list-units -t service --state=dead
  UNIT                                   LOAD      ACTIVE    SUB  DESCRIPTION
  abrt-vmcore.service                    loaded    inactive dead Harvest vmcores for ABRT
  alsa-restore.service                   loaded    inactive dead Save/Restore Sound Card State
  auth-rpcgss-module.service             loaded    inactive dead Kernel Module supporting RPCSEC_GSS
● autofs.service                         not-found inactive dead autofs.service
  blk-availability.service               loaded    inactive dead Availability of block devices
  dbxtool.service                        loaded    inactive dead Secure Boot DBX (blacklist) updater
  dm-event.service                       loaded    inactive dead Device-mapper event daemon
  dmraid-activation.service              loaded    inactive dead Activation of DM RAID sets
  dnf-makecache.service                  loaded    inactive dead dnf makecache
  dracut-cmdline.service                 loaded    inactive dead dracut cmdline hook
. . .
. . .
```

运行`systemctl --state=help`，你将看到一个列表，列出你可以查看的所有不同单元类型的状态。

除了查看当前在内存中的单元外，你还可以通过运行以下命令来查看系统中安装的单元文件：

```
[donnie@localhost ~]$ systemctl list-unit-files
UNIT FILE                                            STATE
proc-sys-fs-binfmt_misc.automount                    static
-.mount                                              generated
boot.mount                                           generated
dev-hugepages.mount                                  static
dev-mqueue.mount                                     static
home.mount                                           generated
proc-fs-nfsd.mount                                   static
. . .
. . .
session-1.scope                                      transient
session-3.scope                                      transient
session-4.scope                                      transient
abrt-journal-core.service                            enabled
abrt-oops.service                                    enabled
abrt-pstoreoops.service                              disabled
abrt-vmcore.service                                  enabled
abrt-xorg.service                                    enabled
abrtd.service                                        enabled
. . .
. . .
```

在这里，你会看到一些可能看起来很奇怪的东西。在顶部，你会看到一些处于 `generated` 状态的挂载文件。这些文件位于 `/run/systemd/units/` 目录，并由 `systemd` 自动生成。为了创建这些挂载文件，`systemd` 每次启动机器或手动重新加载 `fstab` 文件时，都会读取 `/etc/fstab` 文件。

`static` 状态的单元文件是你无法启用或禁用的。相反，其他单元会将这些静态单元作为依赖项调用。

处于 `transient` 状态的单元文件处理的是暂时性的事物。在这里，我们看到三个作用域单元，它们管理三个用户会话。当用户注销会话时，这些单元中的一个会消失。

当然，处于 `enabled` 状态的单元会在机器启动时自动启动，而处于 `disabled` 状态的单元则不会。

要查看单个单元是否启用或活动，你可以使用 `systemctl` 的 `is-enabled` 和 `is-active` 选项。之前，我告诉你 `networkd`、`resolved` 和 `timesyncd` 服务在我的 Fedora 机器上都是禁用的。下面是证明这一点的方法：

```
[donnie@localhost ~]$ systemctl is-enabled systemd-timesyncd
disabled
[donnie@localhost ~]$ systemctl is-enabled systemd-networkd
disabled
[donnie@localhost ~]$ systemctl is-enabled systemd-resolved
disabled
[donnie@localhost ~]$
```

这就是证明它们不活跃的方法：

```
[donnie@localhost ~]$ systemctl is-active systemd-timesyncd
inactive
[donnie@localhost ~]$ systemctl is-active systemd-resolved
inactive
[donnie@localhost ~]$ systemctl is-active systemd-timesyncd
inactive
[donnie@localhost ~]$
```

另一方面，`NetworkManager` 服务在我的 Fedora 机器上是启用且活动的，正如你在这里看到的：

```
[donnie@localhost ~]$ systemctl is-enabled NetworkManager
enabled
[donnie@localhost ~]$ systemctl is-active NetworkManager
active
[donnie@localhost ~]$
```

现在，我将留给你去验证所有这些内容，尤其是在 Ubuntu 机器上。

你还可以查看仅某一类型单元文件的信息。在这里，我们将仅查看交换单元文件的信息：

```
[donnie@localhost units]$ systemctl list-unit-files -t swap
UNIT FILE                                            STATE
dev-mapper-fedora_localhost\x2d\x2dlive\x2dswap.swap generated
1 unit files listed.
[donnie@localhost units]$
```

就像它处理挂载单元文件一样，`systemd` 通过读取 `/etc/fstab` 文件生成了这个文件。

之前，我向你展示了 `/etc/systemd/system.conf` 文件，该文件设置了 `systemd` 的全局配置。使用 `show` 选项，你可以通过运行 `systemctl show` 来查看实际运行的配置。以下是部分输出：

```
[donnie@localhost ~]$ systemctl show
Version=v243.8-1.fc31
Features=+PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 default-hierarchy=unified
Architecture=x86-64
Tainted=local-hwclock
FirmwareTimestampMonotonic=0
LoaderTimestampMonotonic=0
KernelTimestamp=Thu 2021-03-11 11:58:01 EST
KernelTimestampMonotonic=0
. . .
. . .
DefaultLimitRTPRIOSoft=0
DefaultLimitRTTIME=infinity
DefaultLimitRTTIMESoft=infinity
DefaultTasksMax=4608
TimerSlackNSec=50000
DefaultOOMPolicy=stop
```

使用 `--property=` 选项可以仅查看某一项，例如：

```
[donnie@localhost ~]$ systemctl show --property=DefaultLimitSIGPENDING
DefaultLimitSIGPENDING=15362
[donnie@localhost ~]$
```

`systemctl` 有一个手册页，你可以随时查阅。但如果你只是需要快速参考，可以运行 `systemctl -h`。

好的，我觉得现在差不多了。那我们就把这一章总结一下，收尾吧。

# 小结

好的，我们已经迅速进入正题并覆盖了相当多的概念。我们介绍了各种类型的配置文件和单元文件，并了解了它们存放的位置。最后，我们使用 `systemctl` 命令查看了我们运行系统的信息。

在下一章，我们将通过展示服务、路径和套接字单元文件的内部工作原理来进一步探讨这一点。我们下次见。

# 问题

1.  以下哪个命令可以告诉你当前运行的 `systemd` 配置？

    a. `systemctl list`

    b. `systemctl show`

    c. `systemd show`

    d. `systemd list`

1.  以下哪一项陈述是正确的？

    a. 要配置你的磁盘分区，你需要手动配置挂载单元。

    b. 当 `systemd` 读取 `fstab` 文件时，你的磁盘分区挂载单元会自动生成。

    c. 您的驱动分区的挂载单元是静态单元。

    d. 您的驱动分区不需要挂载单元。

1.  以下哪个命令可以告诉你 `NetworkManager` 服务是否正在运行？

    a. `systemctl active NetworkManager`

    b. `systemd active NetworkManager`

    c. `systemd enabled NetworkManager`

    d. `systemctl is-enabled NetworkManager`

    e. `systemctl is-active NetworkManager`

# 答案

1.  a

1.  b

1.  c

# 进一步阅读

`systemd` 单元和单元文件：

[`www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files`](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)
