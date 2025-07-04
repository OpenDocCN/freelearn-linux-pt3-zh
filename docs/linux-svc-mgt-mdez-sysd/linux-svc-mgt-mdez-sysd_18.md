# 第十五章：使用 systemd-networkd 和 systemd-resolved

systemd 生态系统有其自己的网络组件。使用这些组件完全是可选的，甚至你可能会发现自己根本不需要使用它们。然而，有时候使用 systemd 的网络组件可以帮助你做一些传统 Linux 的 NetworkManager 做不到的事情。

本章将覆盖以下主题：

+   理解`networkd`和`resolved`

+   理解 Ubuntu 上的 Netplan

+   理解 RHEL 类型机器上的 networkd 和 resolved

+   使用`networkctl`和`resolvectl`

+   查看 networkd 和 resolved 单元文件

好的，让我们开始吧！

# 技术要求

我们将从 Ubuntu Server 虚拟机开始。（请注意，你需要使用 Ubuntu *Server*，因为 Ubuntu 的桌面版本仍然默认使用旧版的 NetworkManager。）在本章的后面，我们将与 AlmaLinux 一起操作。

那么，让我们从简单介绍 networkd 和 resolved 开始吧。

查看以下链接，观看《代码实战》视频：[`bit.ly/31mmXXZ`](https://bit.ly/31mmXXZ)

# 理解 networkd 和 resolved

传统的 NetworkManager 已经存在了相当长的时间，它仍然是大多数 Linux 桌面和笔记本的最佳解决方案。Red Hat 开发它的主要原因是为了使得 Linux 笔记本能够在有线和无线网络之间，或者从一个无线域切换到另一个无线域时立即切换。NetworkManager 对于普通 Linux 服务器来说也仍然运行良好。所有 RHEL 类型的发行版和所有 Ubuntu 桌面版本仍然默认使用 NetworkManager。

注意

我并不总是会输入`systemd-networkd`或`systemd-resolved`。除非我输入的是实际命令，否则我会将它们简写为`networkd`和`resolved`，这也是大多数人通常做的事。

你已经知道我有一个很怪异的习惯，就是读懂你的心思。所以，我知道你在想，*但是 Donnie，如果 NetworkManager 这么好，为什么我们还需要 networkd 和 resolved 呢？* 啊，我很高兴你问了这个问题。其实，使用`networkd`和`resolved`你可以做一些 NetworkManager 做不到的事。例如，你可以使用 networkd 为运行容器设置一个桥接网络，这样你可以直接给容器分配 IP 地址，让它们能够从外部直接访问。而使用 resolved，你可以设置分割 DNS 解析，从 DHCP 服务器或 IPv6 路由广告中获取 DNS 服务器地址，并使用 DNSSEC、MulticastDNS 和 DNS-over-TLS。另一方面，由于 NetworkManager 能在需要时即时切换网络，它依然是最适合普通桌面和笔记本使用的解决方案。

在运行纯 networkd 环境的系统上，网络配置会存储在 `/etc/systemd/network/` 目录中的一个或多个 `.network` 文件中。然而，正如我之前所说，RHEL 系统默认并不使用 networkd，所以目前在 AlmaLinux 机器上没有 `.network` 文件可供展示。Ubuntu Server 默认使用 networkd，但 Ubuntu 工程师做了一些使事情变得更加有趣的工作。他们并没有按照常规方式配置 networkd 或 NetworkManager，而是创建了 Netplan，我们接下来会详细看一下。

# 理解 Ubuntu 上的 Netplan

Netplan 是 Ubuntu 新的网络配置工具。在桌面机器上，它的作用不大，仅仅是告诉系统使用 NetworkManager。在服务器上，你会在 `/etc/netplan/` 目录下创建一个 `.yaml` 文件来配置 networkd。Netplan 会将这个 `.yaml` 文件的内容转换成 networkd 格式。

## 查看安装程序生成的 Netplan 配置

首先，我想给你展示一下 Ubuntu 桌面机器上的默认配置。（是的，我知道，我没有告诉你需要一个 Ubuntu 桌面虚拟机，但没关系。这是唯一一次我们需要它，所以你只需要看我在这里展示的内容。）在 `/etc/netplan/` 目录下，我们有一个在我创建虚拟机时生成的默认配置文件：

```
donnie@donald-virtualbox:~$ cd /etc/netplan/
donnie@donald-virtualbox:/etc/netplan$ ls -l
total 4
-rw-r--r-- 1 root root 104 Feb  3  2021 01-network-manager-all.yaml
donnie@donald-virtualbox:/etc/netplan$
```

在 `01-network-manager-all.yaml` 文件中，我们有如下内容：

```
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
```

这里是详细说明：

+   `network:` 这一行与屏幕左边对齐，这意味着这是一个新节点。接下来的两行缩进了一个空格，这意味着它们是该节点定义的一部分。

+   `version: 2` 这一行的含义不明确。`netplan` 手册页表示它是一个 *网络映射*，但并没有进一步解释，只是指出它 *可能* 与 Netplan 使用的 YAML 版本有关。或者，它可能是 `netplan` 配置的语法版本。由于似乎没有文档能解决这个谜团，所以很难确定。无论如何，手册页确实表明，这个 `version: 2` 这一行在定义 `network:` 节点时始终必须存在。

+   `renderer:` 这一行告诉系统使用 NetworkManager，所有其他的配置则是在普通的 NetworkManager 文件中完成的。由于这是台桌面机器，大多数人会使用 GUI 管理工具来重新配置网络。（大多数 GUI 类型的工具都是显而易见的，所以我们不会再多说了。）如果没有 `renderer:` 这一行，系统则会使用 networkd 而非 NetworkManager。

    注意

    创建或编辑 `.yaml` 文件时，记住正确的缩进非常重要。如果缩进不正确，配置将无法正常工作。而且 `.yaml` 文件中不允许使用制表符，因此你需要使用空格键来进行缩进。

相比之下，Ubuntu Server 被配置为使用 `networkd`。因此，由 Ubuntu Server 安装程序创建的 Netplan 配置看起来是这样的：

```
donnie@ubuntu2004:/etc/netplan$ ls
00-installer-config.yaml
donnie@ubuntu2004:/etc/netplan$ cat 00-installer-config.yaml 
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2
donnie@ubuntu2004:/etc/netplan$
```

`network:` 节点总是顶级节点。在它下面，我们可以看到 `ethernets:` 节点，它定义了网络接口。（在这个例子中，接口的名称是 `enp0s3`。）`dhcp4: true` 这一行告诉系统从 *DHCP* 服务器获取 IPv4 地址。（在这种情况下，DHCP 服务器内置在我的互联网网关路由器中。）

当网络启动时，Netplan 会将这个 `.yaml` 文件转换成 `networkd` 格式。然而，它并不会存储 `.network` 文件的永久副本。相反，它会在 `/run/systemd/network/` 目录下创建一个临时的 `.network` 文件，就像我们在这里看到的：

```
donnie@ubuntu2004:/run/systemd/network$ ls -l
total 4
-rw-r--r-- 1 root root 102 Aug 13 15:33 10-netplan-enp0s3.network
donnie@ubuntu2004:/run/systemd/network$
```

在这个文件中，我们可以看到如下内容：

```
[Match]
Name=enp0s3
[Network]
DHCP=ipv4
LinkLocalAddressing=ipv6
[DHCP]
RouteMetric=100
UseMTU=true
```

最终，我们终于可以看到一个实际的 `networkd` 配置文件是什么样子的，并且我们看到它被划分为 `[Match]`、`[Network]` 和 `[DHCP]` 部分。如我之前所提到的，这个文件并不是永久性的。它会在你关机或重启时消失，并在机器重新启动时重新出现。（在不使用 Netplan 的非 Ubuntu 系统中，你会在 `/etc/systemd/network/` 目录下找到这个文件的永久副本。）这个文件大部分内容是自解释的，但也有一些有趣的地方。在 `[Network]` 部分，我们看到的是 IPv6 `.yaml` 文件。在 `[DHCP]` 部分，我们看到此网络链路的 *最大传输单元* 的值将从 DHCP 服务器获取。（大多数情况下，该值会被设置为 `1500`。）`RouteMetric=100` 定义了将给予此网络链路的优先级。（当然，这里只有一个网络链路，所以这对我们来说并没有什么作用。）

为了展示静态 IP 地址配置的样子，我创建了一台新的 Ubuntu Server 机器，并告诉安装程序创建一个静态配置。那台机器上的 Netplan `.yaml` 文件是这样的：

```
donnie@ubuntu2004-staticip:/etc/netplan$ cat 00-installer-config.yaml 
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      addresses:
      - 192.168.0.49/24
      gateway4: 192.168.0.1
      nameservers:
        addresses:
        - 192.168.0.1
        - 8.8.8.8
  version: 2
donnie@ubuntu2004-staticip:/etc/netplan$
```

生成的临时 `.network` 文件看起来是这样的：

```
donnie@ubuntu2004-staticip:/run/systemd/network$ cat 10-netplan-enp0s3.network 
[Match]
Name=enp0s3
[Network]
LinkLocalAddressing=ipv6
Address=192.168.0.49/24
Gateway=192.168.0.1
DNS=192.168.0.1
DNS=8.8.8.8
donnie@ubuntu2004-staticip:/run/systemd/network$
```

这次，我们只有 `[Match]` 和 `[Network]` 两个部分。由于我们在这台机器上没有使用 DHCP，所以没有 `[DHCP]` 部分。

现在，请记住，你不必使用在安装操作系统时创建的默认 Netplan 配置。你可以根据需要编辑或替换默认的 `.yaml` 配置文件。接下来我们来看看这个。

## 创建 Netplan 配置

现在，假设我们想将第一台 Ubuntu Server 机器从 DHCP 地址转换为静态地址配置。我做的第一件事是重命名当前的 `.yaml` 文件，保留它作为备份，以防将来想恢复：

```
donnie@ubuntu2004:/etc/netplan$ sudo mv 00-installer-config.yaml 00-installer-config.yaml.bak
[sudo] password for donnie: 
donnie@ubuntu2004:/etc/netplan$
```

接下来，我将创建新的 `00-static-config.yaml` 文件，如下所示：

```
donnie@ubuntu2004:/etc/netplan$ sudo vim 00-static-config.yaml
donnie@ubuntu2004:/etc/netplan$
```

让我们将这个新文件调整为如下所示：

```
network:
  ethernets:
    enp0s3:
      addresses:
      - 192.168.0.50/24
      gateway4: 192.168.0.1
      nameservers:
        addresses:
        - 192.168.0.1
        - 8.8.8.8
        - 208.67.222.222
        - 208.67.220.220
  version: 2
```

好吧，我得承认，我稍微作弊了一下，将这个内容从另一个虚拟机上复制并粘贴过来。然后我修改了 IP 地址，并添加了另外两个 DNS 服务器的地址。顺便提一下，我在这里使用的 DNS 服务器地址是：

+   `192.168.0.1`：这是我的互联网网关路由器的地址。该路由器已配置为使用由我的 ISP（TDS Telecom）运行的 DNS 服务器。因此，`192.168.0.1` 地址并不是真正的 DNS 服务器。

+   `8.8.8.8`：这是 Google 的一个 DNS 服务器地址。

+   `208.67.222.222` 和 `208.67.220.220`：这些地址是由 OpenDNS 组织维护的 DNS 服务器地址。

所以，关于 `nameservers:` 地址，越多越好。如果一个 DNS 服务器宕机，我们只需要使用另一个，从而消除了单一网络故障点。

保存新文件后，你需要 *应用* 它，像这样：

```
donnie@ubuntu2004:/etc/netplan$ sudo netplan apply
```

请注意，如果你从远程终端执行此操作，可能永远看不到命令提示符返回，这会让你以为系统卡住了。其实并不是这样，而是因为如果你分配了与机器原始 IP 地址不同的地址，你会断开 SSH 连接。（事实上，这就是我刚才遇到的情况。）所以，实际上，你可能想从服务器的本地终端而不是远程执行这个操作。

当你使用 apply 操作时，你是在生成新的 networkd 配置并重新启动 networkd 服务。生成的 networkd 配置现在应该类似于这样：

```
donnie@ubuntu2004:/run/systemd/network$ cat 10-netplan-enp0s3.network 
[Match]
Name=enp0s3
[Network]
LinkLocalAddressing=ipv6
Address=192.168.0.50/24
Gateway=192.168.0.1
DNS=192.168.0.1
DNS=8.8.8.8
DNS=208.67.222.222
DNS=208.67.220.220
donnie@ubuntu2004:/run/systemd/network$
```

现在，让我们尝试一些不同的操作。假设我们的虚拟机有多个网络接口，我们希望确保这个网络配置始终应用于正确的接口。我们可以通过将这个配置分配给目标接口的 MAC 地址来实现这一点。首先，我们使用 `ip a` 来获取 MAC 地址，像这样：

```
donnie@ubuntu2004:~$ ip a
. . .
. . .
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:f2:c1:7a brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.50/24 brd 192.168.0.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fef2:c17a/64 scope link 
       valid_lft forever preferred_lft forever
. . .
. . .
```

如你所知，网络接口的硬件地址有很多种称呼。我们大多数人都叫它 *MAC 地址*。它也被称为 *物理地址*、*硬件地址*，或者像我们这里看到的，*link/ether* 地址。无论如何，我们复制那个地址，然后粘贴到 `.yaml` 文件中。在我们刚创建的配置文件的 `enp0s3:` 节点下，我们插入一个新的 `match:` 节点，里面包含所需网络接口的 `macaddress:` 属性。修改后的文件现在应该是这样的：

```
network:
  ethernets:
    enp0s3:
      match:
        macaddress: 08:00:27:f2:c1:7a
      addresses:
      - 192.168.0.50/24
      gateway4: 192.168.0.1
      nameservers:
        addresses:
        - 192.168.0.1
        - 8.8.8.8
        - 208.67.222.222
        - 208.67.220.220
  version: 2
```

这次，我们不使用 `apply` 操作，而是尝试使用 `try` 操作，像这样：

```
donnie@ubuntu2004:/etc/netplan$ sudo netplan try
Warning: Stopping systemd-networkd.service, but it can still be activated by:
  systemd-networkd.socket
Do you want to keep these settings?
Press ENTER before the timeout to accept the new configuration
Changes will revert in 110 seconds
Configuration accepted.
donnie@ubuntu2004:/etc/netplan$
```

`netplan try` 命令与 `netplan apply` 做的事情相同，不同之处在于它让你有机会在新配置不起作用时，恢复到旧的配置。

好吧，我已经向你展示了如何使用 Netplan 和 networkd 组合来设置网络的几个例子。当然，这里还有很多内容。如果你想查看一些更复杂的网络设置，最好的办法是查阅 `netplan` 的手册页。在手册页的底部，你会看到一些非常好的示例。

好的，我们继续前进，学习如何在 Red Hat 环境中使用 networkd。

# 在 RHEL 类型机器上理解 networkd 和 resolved

我们已经确定，所有 RHEL 类型的机器（例如我们的 AlmaLinux 机器）默认使用 NetworkManager。现在，假设我们有一台 AlmaLinux 服务器，并且需要 networkd 的附加功能。`systemd-networkd`包默认没有安装，并且不在普通的 Alma 软件仓库中。但是，它在第三方 EPEL 仓库中，因此我们可以通过以下命令来安装它：

```
[donnie@localhost ~]$ sudo dnf install epel-release
```

现在，我们可以安装`systemd-networkd`包，其中包括 networkd 和 resolved：

```
[donnie@localhost ~]$ sudo dnf install systemd-networkd
```

接下来，禁用 NetworkManager 并启用 networkd 和 resolved。请注意，我暂时不停止 NetworkManager，也不启动 networkd 和 resolved（因为我通过远程登录，不想断开网络连接。再者，我还没有创建 networkd 配置）：

```
[donnie@localhost ~]$ sudo systemctl disable NetworkManager
. . .
[donnie@localhost ~]$ sudo systemctl enable systemd-networkd systemd-resolved
```

在配置 networkd 时，我们不能使用`systemctl edit`命令，因为它会将`.network`文件创建在错误的位置。相反，我们将进入`/etc/systemd/network/`目录，并使用普通的文本编辑器。我们将此文件命名为`99-networkconfig.network`，并添加以下内容：

```
[Match]
Name=enp0s3
[Network]
DHCP=yes
IPv6AcceptRA=yes
```

networkd 的`.network`文件与其他 systemd 单元文件的设置方式相同。你不需要像处理 Netplan 的`.yaml`文件那样担心正确的缩进，只需将所有参数放入正确的部分即可。在`[Match]`部分，我们指定了网络适配器的名称。在`[Network]`部分，我们表示希望通过 DHCP 获取 IP 地址，并接受 IPv6 路由器广告。

下一步是摆脱静态的`/etc/resolv.conf`文件，并创建一个符号链接指向 resolved 生成的文件。我们可以通过这两个命令来实现：

```
[donnie@localhost ~]$ cd /etc
[donnie@localhost etc]$ sudo rm resolv.conf 
[sudo] password for donnie: 
[donnie@localhost etc]$ sudo ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf
[donnie@localhost etc]$
```

如果此时你执行`ls -l /etc/resolv.conf`命令，你会发现符号链接已损坏。原因是我们还没有启动`systemd-resolved.service`。因此，resolved 还没有在`/run/systemd/resolve/`目录中生成动态的`resolv.conf`文件。

最后的步骤是重启机器，然后测试网络是否正常运行。

现在，假设我们想将这台机器转换为静态配置，并且还想添加一些功能。让我们编辑`/etc/systemd/network/99-networkconfig.network`文件，并使其如下所示：

```
[Match]
Name=enp0s3
MACAddress=08:00:27:d2:fb:23
[Network]
Address=192.168.0.51/24
DNSSEC=yes
DNSOverTLS=opportunistic
Gateway=192.168.0.1
DNS=192.168.0.1
DNS=8.8.8.8
DNS=208.67.222.222
DNS=208.67.220.220
IPv6AcceptRA=yes
```

在`[Match]`部分，我添加了`MACAddress=`等号行，以确保此配置始终适用于此特定网络适配器。就像我之前做的那样，我通过执行`ip a`命令来获取 MAC 地址。

在`[Network]`部分，我为其分配了 IP 地址及其子网掩码、默认网关地址和四个 DNS 服务器地址。我还强制这台机器使用`DNSSEC`，并且在可用时使用`DNSOverTLS`。保存此文件后，重启机器或执行`sudo networkctl reload`命令。然后，验证网络是否正常工作。（请注意，在配置 networkd 时无需执行`daemon-reload`。）

在我们继续之前，我想谈谈你在这里看到的两个奇怪的 DNS 选项。`yes`，一定要彻底测试，以确保你能访问到所有需要访问的内容。到目前为止，这里运行得很好。让我感到惊讶的是，因为大多数网站仍然没有设置 DNSSEC 加密密钥。然而，公共 DNS 根服务器和顶级域名服务器已配置了 DNSSEC 密钥，因此我们至少可以验证它们的响应。

注

为了更好地了解 DNSSEC 在公共互联网中的使用范围，访问[`dnssec-analyzer.verisignlabs.com/`](https://dnssec-analyzer.verisignlabs.com/)，并输入一个网站的域名。这将显示一个简单的图表，展示哪些使用了 DNSSEC，哪些没有。

`opportunistic`，这允许机器在可用时使用它。（你的 IT 团队需要与公司管理层合作，确定**DNSOverTLS**的优点是否超过其缺点。）

当你在没有 Netplan 的情况下使用 networkd 时， `/run/systemd/network/`目录下不会有动态的`.network`文件，正如你在 Ubuntu 机器上看到的那样。然而，resolved 会在`/run/systemd/resolve/`目录中创建一个动态的`resolv.conf`文件。我们不需要去查看它，因为我们已经在`/etc/`目录中为它创建了一个符号链接。让我们来看看里面有什么内容：

```
[donnie@localhost etc]$ cat resolv.conf 
. . .
. . .
nameserver 192.168.0.1
nameserver 8.8.8.8
nameserver 208.67.222.222
# Too many DNS servers configured, the following entries may be ignored.
nameserver 208.67.220.220
[donnie@localhost etc]$
```

哎呀，看来 resolved 只想在其配置中看到最多三个 DNS 服务器。目前，这样就可以了。这将帮助我们稍后看到一些其他需要关注的内容。

好的，我已经给你展示了几个使用 networkd 和 resolved 的简单示例。如果你想真正震惊一下，可以打开 `systemd.network` 的手册页，看看里面的内容。（是的，就是 *systemd.network*，而不是 *network* 后面加上 *d*。）你可以创建一些非常复杂的设置，并且可以在手册页的底部查看一些示例。我特别感兴趣的是，使用 networkd 你可以做一些过去必须通过 `iptables` 或 `nftables` 完成的事情。在我看来，使用 networkd 做这些事会更简单一些。你还会看到，通过添加 `[DHCPSERVER]` 部分，networkd 可以作为一个简单的 DHCP 服务器工作。添加 `[CAN]` 部分允许你控制 `[BRIDGE]` 配置，从而为容器分配普通 IP 地址，这样外部世界就能访问它们，而不需要使用端口转发。嗯，关于使用 networkd 可以做的事情的列表相当长，我在这里无法涵盖所有内容。

最后，我想让你打开 Ubuntu 服务器机器上的 `netplan` 手册页。尽管 Netplan 应该是 networkd 的前端，但你会发现通过 Netplan 你能做的事情只是直接使用 networkd 时能够做的一部分。

现在我们已经看过了配置 networkd 和 resolved 的基础知识，让我们来看一对网络诊断工具。

# 使用 networkctl 和 resolvectl

在 Ubuntu 和 Alma 机器上，你可以尝试两个很酷的工具，它们可以帮助你查看网络配置的状态。要查看网络链接及其状态，可以使用 `networkctl` 或 `networkctl list`。(`list` 选项是默认的，因此你不必输入它。) 在 Alma 机器上，你应该能看到类似这样的内容：

```
[donnie@localhost ~]$ networkctl
IDX LINK     TYPE          OPERATIONAL   SETUP     
  1    lo           loopback      carrier                     unmanaged
  2    enp0s3   ether            routable                   configured
2 links listed.
[donnie@localhost ~]$
```

在这台机器上，我们只有两个链接。回环链接的 `OPERATIONAL` 状态显示为 `carrier`，这意味着该链接是可操作的，但不可路由。`SETUP` 显示回环链接是 `unmanaged`，这意味着我们无法重新配置它。`enp0s3` 链接是我们的正常以太网链接，显示为 `routable` 且已 `configured`。

在 Ubuntu 服务器机器上，情况有点更有趣。如图所示，这里有更多的链接：

```
donnie@ubuntu2004:~$ networkctl
IDX LINK                   TYPE       OPERATIONAL    SETUP     
  1    lo                         loopback   carrier                      unmanaged 
  2    enp0s3                 ether         routable                   configured
  3    docker0               bridge       no-carrier                 unmanaged 
  4    cali659bb8bc7b1 ether        degraded                   unmanaged 
  7    vxlan.calico          vxlan       routable                    unmanaged 
5 links listed.
donnie@ubuntu2004:~$
```

除了两个正常的链接外，还有三个是在安装 `docker` 软件包时创建的链接。`docker0` 链接是一个 `unmanaged` 桥接，目前处于 `no-carrier` `OPERATIONAL` 状态。我没有运行任何容器，所以没有人使用它。底部的两个链接是 Kubernetes 的链接，Kubernetes 是 Docker 容器的编排管理器。（*calico* 是来自 *Project Calico* 的引用，Project Calico 是维护该 Kubernetes 网络代码的团队。）`cali659bb8bc7b1` 链接被标记为 `degraded`，这意味着它处于在线状态，且有 `carrier`，但仅对链路本地地址有效。

`status` 选项显示与其关联的 IP 地址、默认网关地址以及我们希望使用的 DNS 服务器的地址。以下是在 Ubuntu 服务器机器上显示的内容：

```
donnie@ubuntu2004:~$ networkctl status
●   State: routable                                          
  Address: 192.168.0.50 on enp0s3                            
           172.17.0.1 on docker0                             
           10.1.89.0 on vxlan.calico                         
           fe80::a00:27ff:fef2:c17a on enp0s3                
           fe80::ecee:eeff:feee:eeee on cali659bb8bc7b1      
           fe80::648c:87ff:fe5f:94d5 on vxlan.calico         
  Gateway: 192.168.0.1 (Actiontec Electronics, Inc) on enp0s3
      DNS: 192.168.0.1                                       
           8.8.8.8                                           
           208.67.222.222                                    
           208.67.220.220
. . .
. . .
```

当然，我们已经了解了如何重新加载修改后的网络配置。在 Ubuntu 使用 Netplan 时，你可以执行 `sudo netplan apply` 或 `sudo netplan try`。在 Alma 机器上，你可以执行 `sudo networkctl reload`。

要查看 DNS 服务器信息，我们可以使用 `resolvectl`。这里需要注意的主要内容是输出被分成了几个部分。首先是 `Global` 部分，显示的是 `/etc/systemd/resolved.conf` 文件中的设置。在 Alma 机器上，这些设置如下所示：

```
[donnie@localhost ~]$ resolvectl
Global
       LLMNR setting: yes
MulticastDNS setting: yes
  DNSOverTLS setting: no
      DNSSEC setting: allow-downgrade
    DNSSEC supported: yes
          DNSSEC NTA: 10.in-addr.arpa
                      16.172.in-addr.arpa
. . .
. . .
```

在 `Global` 部分之后，每个链接都有自己的设置，这些设置可以覆盖 `Global` 设置。以下是 Alma 机器上 `enp0se` 链接的设置：

```
. . .
. . .
Link 2 (enp0s3)
      Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6
       LLMNR setting: yes
MulticastDNS setting: no
  DNSOverTLS setting: opportunistic
      DNSSEC setting: yes
    DNSSEC supported: yes
  Current DNS Server: 192.168.0.1
         DNS Servers: 192.168.0.1
                      8.8.8.8
                      208.67.222.222
                      208.67.220.220
```

仔细查看，你会看到一些 `Link` 设置覆盖了 `Global` 设置。之前我们看到，当前机器的 `resolv.conf` 文件有一个警告，提示我们列出了太多的 nameserver，resolved 可能会忽略第四个。但在这里我们看到所有四个 nameserver，因此一切正常。

networkctl 和 resolvectl 都有很多更多的选项。我会让你在 `networkctl` 和 `resolvectl` 的手册页中阅读它们。

接下来，我们简要了解一下 networkd 和 resolved 单元文件。

# 查看 networkd 和 resolved 单元文件

在我们离开之前，我希望你快速查看一下 networkd 和 resolved 的单元文件。以下是它们的列表：

```
[donnie@localhost system]$ pwd
/lib/systemd/system
[donnie@localhost system]$ ls -l *networkd*
-rw-r--r--. 1 root root 2023 Jun  6 22:26 systemd-networkd.service
-rw-r--r--. 1 root root  640 May 15 12:33 systemd-networkd.socket
-rw-r--r--. 1 root root  752 Jun  6 22:26 systemd-networkd-wait-online.service
[donnie@localhost system]$ ls -l *resolved*
-rw-r--r--. 1 root root 1668 Aug 10 17:19 systemd-resolved.service
[donnie@localhost system]$
```

我不会花时间为你一一追踪它们，因为现在你应该能够自己完成这项工作了。我的意思是，主要还是要查阅 `systemd.directives` 手册页，就像我们之前做过的那样。做完这些后，我们就可以结束这部分内容了。

# 总结

和往常一样，在本章中我们涵盖了很多内容。我们首先对比了 NetworkManager 与 systemd-networkd 和 systemd-resolved。接下来，我们看了如何处理 Ubuntu 上的 Netplan 工具。RHEL 类型的发行版默认使用 NetworkManager，而不使用 Netplan，因此我们查看了如何将 Alma 机器转换为使用 networkd 和 resolved。然后，我们使用了几个诊断工具，最后简要地查看了 networkd 和 resolved 的单元文件。

现在，是时候谈谈 *时间* 了，我们将在下一章讨论。到时见！

# 问题

回答以下问题，测试你对本章内容的理解：

1.  以下哪项说法是正确的？

    A. NetworkManager 更适合服务器，因为它可以快速在网络间切换。

    B. NetworkManager 更适合服务器，因为它更具多功能性。

    C. NetworkManager 更适合桌面和笔记本电脑，因为它可以快速在网络间切换。

    D. networkd 更适合桌面和笔记本电脑，因为它可以快速在网络间切换。

1.  判断正误：如果 Netplan `.yaml` 文件中没有`renderer:`行，系统将默认使用 NetworkManager。

1.  在一台 Ubuntu 服务器机器上，编辑完网络配置文件后，以下哪项操作你会执行？

    A. `sudo netplan reload`

    B. `sudo networkctl reload`

    C. `sudo netplan restart`

    D. `sudo networkctl restart`

    E. `sudo netplan apply`

1.  当`networkctl`命令显示一个链接为`degraded`时，这意味着什么？

    A. 链接处于离线状态。

    B. 链接在线，但未以全速运行。

    C. 链接在线，但仅对链路本地地址有效。

    D. 链接不可靠。

# 答案

1.  C

1.  错误

1.  E

1.  C

# 深入阅读

在 Ubuntu 上使用 Netplan 配置网络：

+   [`ubuntu.com/server/docs/network-configuration`](https://ubuntu.com/server/docs/network-configuration)

+   如何在 Ubuntu 20.04 上使用 Netplan 配置网络：

+   [`www.serverlab.ca/tutorials/linux/administration-linux/how-to-configure-networking-in-ubuntu-20-04-with-netplan/`](https://www.serverlab.ca/tutorials/linux/administration-linux/how-to-configure-networking-in-ubuntu-20-04-with-netplan/)

+   YAML 教程：

+   [`www.tutorialspoint.com/yaml/index.htm`](https://www.tutorialspoint.com/yaml/index.htm)

+   初学者的 YAML 教程：

+   [`www.redhat.com/sysadmin/yaml-beginners`](https://www.redhat.com/sysadmin/yaml-beginners)

+   `systemd-resolved`: Split DNS 介绍：

+   [`fedoramagazine.org/systemd-resolved-introduction-to-split-dns/`](https://fedoramagazine.org/systemd-resolved-introduction-to-split-dns/)

+   多播 DNS：小规模的名称解析：

+   [`www.ionos.com/digitalguide/server/know-how/multicast-dns/`](https://www.ionos.com/digitalguide/server/know-how/multicast-dns/)

+   通过 TLS 的 DNS：一种改进的安全概念：

+   [`www.ionos.com/digitalguide/server/security/dns-over-tls/`](https://www.ionos.com/digitalguide/server/security/dns-over-tls/)

+   Calico 项目：

+   [`www.tigera.io/project-calico/`](https://www.tigera.io/project-calico/)
