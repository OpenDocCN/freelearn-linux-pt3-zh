12. CentOS 8 网络管理

很难想象一个 CentOS 8 系统没有至少一个网络连接，更难想象这样一个孤立的系统有什么实际用途。事实上，CentOS 8 被设计为提供通过网络和互联网连接的企业级服务。学习如何管理 CentOS 8 系统的一个关键部分是学习如何配置和管理系统上安装的网络接口。

本章旨在概述 CentOS 8 上的网络管理，包括 NetworkManager 服务和工具，以及一些其他实用的工具。

12.1 NetworkManager 简介

NetworkManager 是一个服务和一组专门设计的工具，旨在简化 Linux 系统上的网络配置管理，是 CentOS 8 的默认网络管理服务。

除了在后台运行的服务外，NetworkManager 还包括以下工具：

•nmcli - 一个通过命令行与 NetworkManager 配合使用的工具。这个工具在没有图形环境访问的情况下非常有用，也可以在脚本中使用，以进行网络配置更改。

•nmtui - 一个基本的基于文本的用户界面，用于管理 NetworkManager。此工具可以在任何终端窗口中运行，并允许通过选择菜单和输入数据来进行更改。虽然在执行基本任务时有用，但 nmtui 缺少 nmcli 工具提供的许多功能。

•nm-connection-editor - 一个完整的图形管理工具，提供访问大多数 NetworkManager 配置选项的功能。

•GNOME 设置 - GNOME 桌面设置应用程序的网络屏幕允许执行基本的网络管理任务。

•Cockpit 网络设置 - Cockpit Web 界面的网络屏幕允许执行一系列网络管理任务。

尽管在 CentOS 8 系统上有多种方式管理网络环境，但本章将重点介绍 nmcli 命令。虽然图形工具在您访问桌面环境或启用了 Cockpit 时非常有用，但在只有命令行界面的情况下，理解命令行界面是至关重要的。此外，图形工具（包括 Cockpit）并不包括 nmcli 工具的所有功能。最后，一旦你对 NetworkManager 和 nmcli 有了一定的熟悉，这些技能可以轻松地转化为使用更直观的工具选项。而图形工具选项则不一定如此。例如，如果你只使用过 nm-connection-editor，那么使用 nmcli 会更为困难。

12.2 安装和启用 NetworkManager

NetworkManager 应该在大多数 CentOS 8 安装中默认安装。使用 rpm 命令查看是否需要安装：

# 第十一章：rpm -q NetworkManager

NetworkManager-1.14.0-14.el8.x86_64

如有必要，可以按以下方式安装软件包：

# dnf install NetworkManager

安装完软件包后，需要启用 NetworkManager 守护进程，以便每次系统启动时都能自动启动：

# systemctl enable NetworkManager

最后，启动服务并检查状态，以验证启动是否成功：

# systemctl start NetworkManager

# systemctl status NetworkManager

● NetworkManager.service - 网络管理器

已加载：已加载 (/usr/lib/systemd/system/NetworkManager.service; 已启用; vendor >

Drop-In: /usr/lib/systemd/system/NetworkManager.service.d

└─NetworkManager-ovs.conf

激活状态：激活（运行中）自 2019-04-09 10:07:22 EDT 起； 2 小时 48 分钟前

.

.

12.3 基本的 nmcli 命令

nmcli 命令已作为 NetworkManager 包的一部分安装，并且可以通过命令行使用以下语法执行：

# nmcli [选项] 对象 {命令 | 帮助}

在上述语法中，对象将是 general、networking、radio、connection、monitor、device 或 agent 中的一个，可以将其缩写为该词的几个字母（例如，connection 可缩写为 con，甚至只用字母 c）。例如，以下所有命令都会输出与设备对象相关的帮助信息：

# nmcli device help

# nmcli dev help

# nmcli d help

要检查系统上 NetworkManager 的整体状态，请使用以下命令：

# nmcli general status

状态 连接性 WIFI 硬件 WIFI WWAN 硬件 WWAN

已连接 完全启用 启用 启用 启用 启用

要检查系统上安装的设备状态，可以使用以下命令：

# nmcli dev status

设备 类型 状态 连接

eno1 以太网 已连接 eno1

wlp0s26u1u2 wifi 已连接 zoneone

virbr0 桥接 已连接 virbr0

lo 回环 未管理 --

virbr0-nic tun 未管理 --

输出结果还可以通过使用 -p（pretty）选项进行修改，使输出更具人性化：

# nmcli -p dev status

=====================

设备状态

=====================

设备 类型 状态 连接

-------------------------------------------------------------------

eno1 以太网 已连接 eno1

wlp0s26u1u2 wifi 已连接 zoneone

virbr0 桥接 已连接 virbr0

lo 回环 未管理 --

virbr0-nic tun 未管理 --

相反，可以使用 -t 选项使输出更简洁，适合自动处理：

# nmcli -t dev status

eno1:以太网:已连接:eno1

wlp0s26u1u2:wifi:已连接:emilyzone

virbr0:桥接:已连接:virbr0

lo:回环:未管理:

virbr0-nic:tun:未管理:

从状态输出中，我们可以看到系统已安装两台物理设备，一台为以太网，另一台为 Wi-Fi 设备。

桥接（virbr）条目是用于为虚拟机提供网络的虚拟设备（虚拟化技术概述章节将进行讨论，该章节名为“虚拟化技术概述”）。回环接口是一种特殊的虚拟设备，允许系统与自身通信，通常用于进行网络诊断。

在使用 NetworkManager 时，了解设备和连接之间的区别非常重要。如上所述，设备可以是物理或虚拟网络设备，而连接是设备连接到的网络配置。

以下命令显示了系统上配置的连接的信息:

# nmcli con show

名称 UUID 类型 设备

zoneone 2abecafa-4ea2-47f6-b20f-4fb0c0fd5e94 wifi wlp0s26u1u2

eno1 99d40009-6bb1-4182-baad-a103941c90ff ethernet eno1

virbr0 e13e9552-1765-42d1-b739-ff981668fbee bridge virbr0

zonetwo f940a2d1-8c18-4a9e-bf29-817702429b8a wifi --

zonethree fd65c5e5-3e92-4e9e-b924-1b0b07b70032 wifi --

从上述输出中，我们可以看到 Wi-Fi 设备（wlp0s26u1u2）连接到名为 zoneone 的无线网络，而以太网设备（eno1）连接到名为 eno1 的连接。除了 zoneone，NetworkManager 还列出了另外两个名为 zonetwo 和 zonethree 的 Wi-Fi 连接，目前都没有设备连接。

要查找分配给连接的 IP 地址，可以使用 ip 工具加上 address 选项:

# ip address

这也可以简写为:

# ip a

.

.

3: wlp0s26u1u2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000

link/ether 74:da:38:ee:be:50 brd ff:ff:ff:ff:ff:ff

inet 192.168.1.121/24 brd 192.168.1.255 scope global dynamic noprefixroute wlp0s26u1u2

valid_lft 57584 秒 preferred_lft 57584 秒

.

.

ip 命令将输出系统检测到的所有设备的信息。上述输出显示，Wi-Fi 设备已被分配 IP 地址 192.168.1.121。

如果我们只想列出活动连接，可以使用 nmcli 命令加上 -a 选项:

# nmcli con show -a

名称 UUID 类型 设备

emilyzone 2abecafa-4ea2-47f6-b20f-4fb0c0fd5e94 wifi wlp0s26u1u2

eno1 99d40009-6bb1-4182-baad-a103941c90ff ethernet eno1

virbr0 e13e9552-1765-42d1-b739-ff981668fbee bridge virbr0

要将 Wi-Fi 设备连接从 zoneone 切换到 zonetwo，可以运行以下命令:

# nmcli device wifi connect zonetwo -ask

密码:

-ask 标志会导致 nmcli 提示用户输入 Wi-Fi 网络的密码。要在命令行中包含 Wi-Fi 密码（特别是如果命令在脚本中执行时），请使用 password 选项:

# nmcli device wifi connect zonetwo password <password here>

nmcli 工具也可以用来扫描可用的 Wi-Fi 网络，如下所示:

# nmcli device wifi list

IN-USE SSID MODE CHAN RATE SIGNAL BARS SECURITY

zoneone Infra 6 195 Mbit/s 80 WPA2

* zonetwo Infra 11 130 Mbit/s 74 WPA1 WPA2

当前激活的连接可以通过以下方式停用：

# nmcli con down <连接名称>

同样，任何时候都可以重新激活一个非活动连接：

# nmcli con up <连接名称>

当连接被停用时，NetworkManager 会自动搜索另一个连接，激活该连接并将其分配给先前建立连接的设备。为了防止在这种情况下使用某个连接，可以通过以下方式禁用自动连接选项：

# nmcli con mod <连接名称> connection.autoconnect no

以下命令可用于获取有关特定连接的更多信息。这包括所有连接属性的当前值：

# nmcli con show eno1

connection.id: eno1

connection.uuid: 99d40009-6bb1-4182-baad-a103941c90ff

connection.stable-id: --

connection.type: 802-3-ethernet

connection.interface-name: eno1

connection.autoconnect: yes

connection.autoconnect-priority: 0

connection.autoconnect-retries: -1（默认值）

connection.multi-connect: 0（默认值）

connection.auth-retries: -1

connection.timestamp: 1554833695

connection.read-only: no

connection.permissions: --

connection.zone: --

connection.master: --

connection.slave-type: --

connection.autoconnect-slaves: -1（默认值）

.

.

.

所有这些属性都可以使用 nmcli 的 modify 选项进行修改，语法如下：

# nmcli con mod <连接名称> connection.<属性名称> <设置>

12.4 使用连接配置文件

到目前为止，我们已经探讨了如何使用连接，但尚未解释如何配置连接。连接的配置被称为连接配置文件，并存储在 /etc/sysconfig/network-scripts 目录中的一个文件中，该文件的内容可能如下所示：

# ls /etc/sysconfig/network-scripts

ifcfg-zoneone ifcfg-eno1 ifdown-ppp

ifcfg-zonethree ifcfg-zonetwo ifup-ppp keys-zonethree

keys-zoneone keys-zonetwo

每个以 ifcg- 为前缀的文件都是一个接口配置文件，包含相应连接的连接配置文件，而 key- 文件则包含 Wi-Fi 连接的密码。

例如，考虑 ifcfg-eno1 文件的内容：

TYPE=Ethernet

PROXY_METHOD=none

BROWSER_ONLY=no

DEFROUTE=yes

IPV4_FAILURE_FATAL=no

NAME=eno1

UUID=99d40009-6bb1-4182-baad-a103941c90ff

DEVICE=eno1

ONBOOT=yes

BOOTPROTO=dhcp

IPV6INIT=yes

IPV6_AUTOCONF=yes

IPV6_DEFROUTE=yes

IPV6_FAILURE_FATAL=no

IPV6_ADDR_GEN_MODE=stable-privacy

该文件包含有关连接的基本信息，包括设备类型（以太网）以及当前分配的设备（eno1），并且该连接将在系统启动时自动激活，并通过 DHCP 获取 IP 地址。通过修改此文件并指示 nmcli 重新加载连接配置文件，可以实现对连接配置文件的更改：

# nmcli con reload

新的连接配置文件也可以通过 nmcli 手动创建或自动生成。例如，假设系统中安装了新的网络设备。当发生这种情况时，NetworkManager 服务会检测到新硬件并为其创建一个设备。在以下示例中，新设备已被分配了名称 eno2：

# nmcli dev status

DEVICE TYPE STATE CONNECTION

en01 以太网已连接 eno1

eno2 以太网已连接 有线连接 1

NetworkManager 自动检测到设备，激活了它并将其分配到名为“有线连接 1”的连接上。这是一个默认连接，我们无法对其进行配置控制，因为在/etc/sysconfig/network-scripts 中没有该接口的配置文件。接下来的步骤是删除“有线连接 1”连接，并使用 nmcli 创建一个新的连接并将其分配给设备。删除连接的命令如下：

# nmcli con delete "有线连接 1"

接下来，可以使用 nmcli 创建一个新的连接配置文件，该配置文件可以使用静态 IP 地址，或者通过 DHCP 服务器获取动态 IP 地址。要创建一个名为 dyn_ip 的动态连接配置文件，可以使用以下命令：

# nmcli connection add type ethernet con-name dyn_ip ifname eno2

连接‘dyn_ip’（160d9e10-bbc8-439a-9c47-a2ec52990472）成功添加。

要创建一个新的连接配置文件，而不将其锁定到特定设备，只需在命令中省略 ifname 选项：

# nmcli connection add type ethernet con-name dyn_ip

创建连接后，一个名为 ifcg-dyn_ip 的文件将被添加到/etc/sysconfig/network-scripts 目录中。

或者，要创建一个名为 static_ip 的连接并分配静态 IP 地址（此例中为 192.168.1.200），可以使用以下命令：

# nmcli con add type ethernet con-name static_ip ifname eno0 ip4 192.168.1.200/24 gw4 192.168.1.1

连接‘static_ip’（3fccafb3-e761-4271-b310-ad0f28ee8606）成功添加。

对应的 ifcg-static_ip 文件内容如下所示：

TYPE=Ethernet

PROXY_METHOD=none

BROWSER_ONLY=no

BOOTPROTO=none

IPADDR=192.168.1.200

PREFIX=24

GATEWAY=192.168.1.1

DEFROUTE=yes

IPV4_FAILURE_FATAL=no

IPV6INIT=yes

IPV6_AUTOCONF=yes

IPV6_DEFROUTE=yes

IPV6_FAILURE_FATAL=no

IPV6_ADDR_GEN_MODE=stable-privacy

NAME=static_ip

UUID=3fccafb3-e761-4271-b310-ad0f28ee8606

DEVICE=eno2

ONBOOT=yes

添加新连接的命令可以稍微修改，以便同时分配 IPv4 和 IPv6 静态地址：

# nmcli con add type ethernet con-name static_ip ifname eno0 ip4 192.168.1.200/24 gw4 192.168.1.1 gw4 192.168.1.1 ip6 cabf::4532 gw6 2010:dfa::1

12.5 交互式编辑

除了使用 nmcli 带有命令行选项外，该工具还包括一个交互模式，可以用于创建和修改连接配置文件。例如，以下记录展示了使用交互模式创建一个名为 demo_con 的新以太网连接：

# nmcli con edit

有效的连接类型：6lowpan，802-11-olpc-mesh（olpc-mesh），802-11-wireless（wifi），802-3-ethernet（以太网），adsl，蓝牙，bond，桥接，cdma，dummy，generic，gsm，infiniband，ip-tunnel，macsec，macvlan，ovs-bridge，ovs-interface，ovs-port，pppoe，team，tun，vlan，vpn，vxlan，wimax，wpan，bond-slave，bridge-slave，team-slave

输入连接类型：以太网

===| nmcli 交互式连接编辑器 |===

添加一个新的 ‘802-3-ethernet’ 连接

输入 'help' 或 '?' 获取可用命令。

输入 'print' 显示所有连接属性。

输入 ‘describe [<setting>.<prop>]’ 查看详细属性描述。

您可以编辑以下设置：连接，802-3-ethernet（以太网），802-1x，dcb，sriov，ethtool，match，ipv4，ipv6，tc，代理

nmcli> 设置 connection.id demo_con

nmcli> 设置 connection.interface eno1

nmcli> 设置 connection.autoconnect yes

nmcli> 设置 ipv4.method 自动

nmcli> 设置 802-3-ethernet.mtu 自动

nmcli> 设置 ipv6.method 自动

nmcli> 保存

保存连接并设置 ‘autoconnect=yes’。这可能会导致连接立即激活。

您是否仍希望保存？（是/否）[是] 是

连接 ‘demo_con’ (cb837408-6c6f-4572-9548-4932f88b9275) 已成功保存。

nmcli> 退出

以下记录修改了先前创建的 static_ip 连接配置文件，使用不同于原始指定的静态 IP 地址：

# nmcli con 编辑 static_ip

===| nmcli 交互式连接编辑器 |===

编辑现有的 '802-3-ethernet' 连接：'static_ip'

输入 'help' 或 '?' 获取可用命令。

输入 'print' 显示所有连接属性。

输入 'describe [<setting>.<prop>]' 查看详细属性描述。

您可以编辑以下设置：连接，802-3-ethernet（以太网），802-1x，dcb，sriov，ethtool，match，ipv4，ipv6，tc，代理

nmcli> 打印 ipv4.addresses

ipv4.addresses: 192.168.1.200/24

nmcli> 设置 ipv4.addresses 192.168.1.201/24

nmcli> 保存

连接 'static_ip' (3fccafb3-e761-4271-b310-ad0f28ee8606) 已成功更新。

nmcli> 退出

修改现有连接后，请记得指示 NetworkManager 重新加载配置文件：

# nmcli con 重新加载

在使用交互模式时，了解有一个丰富的内置帮助系统可以帮助你学习如何使用该工具是非常有用的。帮助主题可以通过在 nmcli > 提示符下输入 help 或 ? 来访问：

nmcli> ?

------------------------------------------------------------------------------

---[ 主菜单 ]---

转到 [<setting> | <prop>] :: 转到某个设置或属性

移除 <setting>[.<prop>] | <prop> :: 移除设置或重置属性值

设置 [<setting>.<prop> <value>] :: 设置属性值

描述 [<setting>.<prop>] :: 描述属性

打印 [all | <setting>[.<prop>]] :: 打印连接

验证 [all | fix] :: 验证连接

保存 [persistent|temporary] :: 保存连接

激活 [<ifname>] [/<ap>|<nsp>] :: 激活连接

返回 :: 向上一层返回 (back)

help/? [<command>] :: 打印帮助信息

nmcli <conf-option> <value> :: nmcli 配置

quit :: 退出 nmcli

------------------------------------------------------------------------------

12.6 配置 NetworkManager 权限

除了简化 CentOS 8 上的网络管理，NetworkManager 还允许为连接指定权限。例如，以下命令将连接配置文件限制为 root 用户和名为 john 和 caitlyn 的用户帐户：

# nmcli con mod static_ip connection.permissions user:root,john,caitlyn

一旦 NetworkManager 重新加载了连接配置文件，静态 _ip 连接仅在至少有一个指定用户登录到系统的活跃会话时才会处于活动状态并可供其他用户访问。只要最后一个用户注销，连接将会断开并保持不活跃，直到其中一个用户重新登录。

此外，只有具有权限的用户才能更改连接状态或配置。

12.7 总结

CentOS 8 上的网络管理由 NetworkManager 服务处理。NetworkManager 将网络视为由网络接口设备和连接组成。网络设备可以是物理以太网或 Wi-Fi 设备，或是虚拟机客户机使用的虚拟设备。连接表示设备连接的网络，并通过连接配置文件进行配置。配置文件将定义连接是否具有静态或动态 IP 地址、网络使用的任何网关的 IP 地址，以及系统启动时是否自动建立连接。

NetworkManager 可以使用多种工具进行管理，包括 nmcli 和 nmtui 命令行工具、nm-connection-editor 图形工具以及 Cockpit 网络设置界面。通常，nmcli 命令行工具提供了最多的功能和灵活性。
