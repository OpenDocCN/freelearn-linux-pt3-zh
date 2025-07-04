# 2

# 使用与不使用自动化魔法进行安装

虽然你可以从 USB 或 ISO 镜像手动安装 Oracle Linux，但在企业级别上，更常见的是自动化安装。本章最后的*附录 – kickstart 选项*涵盖了手动安装，适合偶尔需要安装的情况。

大多数数据中心安装使用自动化方法，因为这样可以快速扩展。

本章提供了不同的方案，帮助构建一个可操作的**预启动执行**（**PXE**；发音为*pixie*）系统。

PXE 启动是由英特尔开发的一项技术。它允许具有适当 ISO（或操作系统 RPM 包）的系统从网络启动，下载服务器上的所有必需文件。虽然存在更旧的方法，如**引导协议**（**BOOTP**），但行业已经迅速采用**PXE 启动**作为最常见的标准。虽然 PXE 启动最初是为英特尔系统设计的，但 ARM 系统也可以使用 TFTP 进行启动。为了使 PXE 启动过程正常工作，通常需要在网络上配置 DHCP 服务器和 PXE 服务器。该过程的高级概述可以通过以下图表看到：

![图 2.1 – PXE 启动过程](img/B18349_02_01.jpg)

图 2.1 – PXE 启动过程

以下是每个步骤的高级描述：

1.  **PXE 客户端**将从可用的**DHCP 服务器**请求 IP 地址。

1.  网络上第一个响应客户端的 DHCP 服务器将为其提供 IP 地址、网络掩码、DNS 信息、默认路由和其他所需的网络参数。

1.  **PXE 客户端**将进行启动服务发现，查找**PXE 服务器**。可选地，DHCP 服务器可以配置为指向 PXE 服务器以进行启动。

1.  PXE 服务器将网络引导程序和文件下载位置的 IP 信息发送给客户端。这使得客户端可以从网络启动，安装操作系统。

1.  PXE 客户端根据需要从**HTTPS 服务器**请求媒体（RPM 包和控制文件）。HTTPS 并不是唯一需要的协议；NFS 和 FTP 服务器也可以使用，尽管 HTTPS 是最常见的方法。PXE 系统还通过 kickstart 文件将*kickstart*参数发送给客户端。这些参数包含配置系统存储、网络和要安装的软件包的具体信息。

1.  媒体服务器将请求的文件发送到 PXE 客户端，从而允许操作系统安装。

在本示例中配置 PXE 服务器时，需要三个组件：

+   **DHCP 服务器**：它不仅为服务器分配 IP 地址，还分配启动时所需的重要参数，支持 PXE 启动。

+   **简易文件传输协议守护进程**（**TFTPD**）：该服务作为 TFTP 服务器工作，允许客户端运行启动加载程序，从而启动操作系统安装。

+   **超文本传输协议守护进程**（**HTTPD**）：这是一个 Web 服务器，用于托管安装所需的配置文件和 RPM 包。

注意

还有一些其他选项，例如使用 `dnsmasq` 作为一个结合的服务器，它同时充当 DHCP 服务器、TFTP 服务器（最常见的是 TFTPD）和 DNS 转发器。然而，在许多企业环境中，往往需要单独的系统。例如，网络中可能已经使用了 Microsoft DHCP 服务器。

在这个例子中，有两台逻辑服务器运行着三个不同的服务，第一台系统上运行 DHCP 服务器和 TFTP 服务器。第二台系统是 PXE 服务器，也将作为 HTTP 服务器，负责提供 **Red Hat 包管理器**（**RPM**）文件和安装所需的控制文件。两台系统都是多网卡的，并将在它们的辅助接口上运行 DHCP 和 PXE。系统还在辅助接口上设置了静态 IP 地址。

本章将涵盖以下主要食谱，这些组成了 PXE 的盛宴：

+   使用 kickstart 自动化操作系统安装选项

+   DHCP 服务器 – 不仅仅是 IP

+   设置 Web 服务器 – HTTPD

+   启动服务 – TFTPD

+   回顾过去 – 手动安装

注意

多网卡服务器是指同时连接多个网络的系统。这通常在企业数据中心中进行，用以隔离系统，同时允许一些受控系统传递数据。也可以在单一网络上设置 PXE 服务器。

# 技术要求

创建新的 kickstart 文件的简单方法是通过构建一个新的操作系统，因为此过程会自动创建文件，之后可以根据需要进行修改。PXE 环境还需要额外的系统或虚拟机来充当 DHCP 服务器和 HTTPS 服务器。开始之前，首先需要下载 ISO 镜像。推荐使用如 Oracle VM VirtualBox 之类的桌面虚拟化产品。

本书中许多食谱的相关配置文件可以在 GitHub 上找到，网址为 [`github.com/PacktPublishing/Oracle-Linux-Cookbook`](https://github.com/PacktPublishing/Oracle-Linux-Cookbook)。

# 使用 kickstart 自动化操作系统安装选项

**Kickstart** 文件是自动化安装过程中的重要组成部分。该文件包含自动安装和配置操作系统所需的所有信息。Kickstart 文件是一个简单的文本文件，包含了实现无人值守安装的指令。

## 准备工作

要使用 kickstart 文件，您需要一个地方来构建系统，例如 Oracle VM VirtualBox。该系统可用于创建您的第一个 kickstart 文件，也可以在将其移入 PXE 环境之前测试新创建的 kickstart 文件。

注意

Anaconda 是 Oracle Linux 安装程序。它可以通过使用 kickstart 文件进行自动化安装，或者通过图形界面或文本安装界面进行交互式操作。

## 如何操作……

虽然你可以手动创建一个文件，但从 Anaconda 在安装过程中创建的文件开始会更加容易。这个 kickstart 文件在手动安装时自动创建，并保存在新系统的 `/root/anaconda-ks.cfg` 中。这个文件可以很容易地编辑并用于未来的构建。我们通过将文件分解为更小的部分来查看它。这个系统是图形安装。其他选项包括文本或命令行：

注

kickstart 有许多选项，详见*附录 –* *kickstart 选项*。

```
#version=OL8
# Use graphical install
graphical
```

用于安装 Oracle Linux 的仓库是从 ISO 文件以环回挂载的目录。`baseurl` 也可以是 HTTPS 服务器、NFS 位置或 FTP 服务器。例如，`--baseurl=https://pxe.m57.local/ol8` 将告诉 kickstart 从 Web 服务器中提取 RPM 包：

```
repo --name="AppStream" --baseurl=file:///run/install/sources/mount-0000-cdrom/AppStream
```

软件包是所选的软件包。以 `@` 符号开头的名称是包组，没有 `@` 的行是单独的软件包。此外，`^` 符号用于仅启用安装包组中所需的包。如果没有设置 `^`，则包组中的所有包都会被安装。在此示例中，安装了 `graphical-server-environment` 包组 RPM 和 `kexec-tools` 包：

```
%packages
@^graphical-server-environment
kexec-tools
%end
```

这是选择默认键盘和语言的地方：

```
# Keyboard layouts
keyboard --xlayouts='us'
# System language
lang en_US.UTF-8
```

这是新服务器的主机名。理想情况下，它应该是系统的**完全限定域名**（**FQDN**）：

```
# Network information
network  --hostname=ol802.lab.m57.local
```

用于选择一个 CD-ROM 或挂载在文件系统上的 CD-ROM 作为安装介质：

```
# Use CDROM installation media
cdrom
```

用于在第一次启动时运行设置代理：

```
# Run the Setup Agent on first boot
firstboot --enable
```

在这里，选择了安装磁盘，以及逻辑卷管理器和文件系统的配置：

```
ignoredisk --only-use=sda
# Partition clearing information
clearpart --none --initlabel
# Disk partitioning information
part /boot --fstype="xfs" --ondisk=sda --size=1024
part pv.116 --fstype="lvmpv" --ondisk=sda --size=80868
volgroup ol --pesize=4096 pv.116
logvol /home --fstype="xfs" --size=5120 --name=home --vgname=ol
logvol / --fstype="xfs" --size=51200 --name=root --vgname=ol
logvol /u01 --fstype="xfs" --size=10240 --name=u01 --vgname=ol
logvol /var --fstype="xfs" --size=5120 --name=var --vgname=ol
logvol swap --fstype="swap" --size=4056 --name=swap --vgname=ol
logvol /var/log --fstype="xfs" --size=5120 --name=var_log --vgname=ol
```

这是服务器的时区：

```
# System timezone
timezone America/New_York --isUtc
```

可以为新系统设置加密的 root 密码。通常，密码是加密的，但你也可以选择使用 `--plaintext` 选项设置明文密码。你也可以通过运行 `python3 -c 'import crypt,getpass;pw=getpass.getpass();print(crypt.crypt(pw) if (pw==getpass.getpass("Confirm: ")) else exit())'` Python 命令生成加密密码：

```
# Root password
rootpw --iscrypted $6$W5fC.GDUSIVXPDS7$zQFm49tGCtRbfyAd/0f57QcuPZYtOB/gobgN2oKNG
zqrseiNtm7QqkthCcdiNxGJhzLSIQpNyxRQXEPJPuaYM.
```

启用 kdump，允许内核将崩溃转储保存到设备中以供故障排除：

```
%addon com_redhat_kdump --enable --reserve-mb='auto'
%end
```

这是 root、用户和 **Linux 统一密钥设置**（**LUKS**）密码策略：

```
%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
```

你可以根据需要编辑文件。在使用新文件之前，可以通过 `ksvalidator` 命令进行测试。该命令是 `pykickstart` 包的一部分，可通过 `dnf install -y pykickstart` 安装。

软件包安装完成后，运行 `ksvalidator $FILE` 测试新的 kickstart 文件。带有错误的示例将如下所示，其中第 3 行将安装定义为 `xgraphical`，这是一个未知选项：

```
[root@localhost ~]# ksvalidator anaconda-ks.cfg
The following problem occurred on line 3 of the kickstart file:
Unknown command: xgraphical
```

解决错误并重新运行验证。在此示例中，`graphical` 是正确的参数。清理后的验证返回空值，结果如下：

![图 2.2 – 清理 ksvalidator](img/B18349_02_02.jpg)

图 2.2 – 清理 ksvalidator

### Kickstart 选项！

现在你对 Kickstart 文件有了基本的了解，可以开始添加新选项。这些选项可以帮助你自动化安装过程，并且让你拥有更多控制权。选项详细介绍请参见*附录 –* *kickstart 选项*。

## 它是如何工作的……

你可以通过手动添加`ks`参数到`ks=PATH_TO_FILE`，启动任何系统，其中`PATH_TO_FILE`是 Kickstart 文件的 URL 路径。通常，这个文件放在 HTTP 服务器上，客户端可以访问。

# DHCP 服务器 – 不仅仅是分配 IP

DHCP 服务器将运行一个标准的 DHCP 服务器，该服务器可以在 Oracle Linux 8 中使用。DHCP 的安装非常简单。

## 准备工作

要设置 DHCP 服务器，你需要一台运行 Oracle Linux 8 的虚拟机，并且它需要与将要安装的虚拟机处于同一子网。

## 如何操作……

使用`dnf install -y dhcp-sever`命令安装 DHCP 服务器。以下截图展示了完整的过程：

![图 2.3 – DHCP 服务器安装](img/B18349_02_03.jpg)

图 2.3 – DHCP 服务器安装

安装完成后，通过防火墙启用 DHCP 流量（如下面截图所示），使用`firewall-cmd --add-service=dhcp --permanent`和`firewall-cmd --reload`命令：

![图 2.4 – DHCP 防火墙命令](img/B18349_02_04.jpg)

图 2.4 – DHCP 防火墙命令

DHCP 服务器在`/usr/share/doc/dhcp-server`目录下有一个示例的 DHCP 配置文件。这个文件包含 IPv4 的示例配置`/usr/share/doc/dhcp-server/dhcpd.conf.example`和 IPv6 的示例配置`/usr/share/doc/dhcp-server/dhcpd6.conf.example`。对于这个 DHCP 服务器，我们将使用 IPv4，并将示例配置文件复制到`/etc/dhcp/dhcpd.conf`。该目录下已有一个配置文件，但默认情况下它几乎是空的，没有任何配置，可以按照以下截图进行替换：

![图 2.5 – 复制 dhcp.conf](img/B18349_02_05.jpg)

图 2.5 – 复制 dhcp.conf

接下来，需要更新配置文件，修改 IP 地址范围，并且通过`option ntp-servers 192.168.200.1`选项将 NTP 服务器添加到配置中，该选项将 NTP 服务器设置为`192.168.200.1`。虽然配置 NTP 服务器是可选的，但强烈推荐这么做。此外，当 PXE 服务器*不是*DHCP 服务器时，你需要在子网中添加“next server”参数，这将指示 DHCP 客户端从 PXE 服务器的 TFTP 进行启动。

注意

如果你正在使用现有的 DHCP 服务器，请确保添加`next-server`参数，指向包含 TFTP 服务器的 PXE 服务器。

由于 TFTP 服务器在 PXE 服务器上运行，因此 DHCP 服务器需要将“next server”选项设置为 PXE 服务器的 IP 地址。以下是一个完整的示例配置文件：

```
# dhcpd.conf
#
# These options are common to all networks
option domain-name "lab.m57.local";
option domain-name-servers 192.168.56.10;
default-lease-time 600;
max-lease-time 7200;
# If the DHCP server is the main server for the network, this should be set authoritative;
# This sets where logs are sent. By default they are sent to syslog. You can reconfigure sysylog to send to a different location as needed.
log-facility local7;
subnet 192.168.56.0 netmask 255.255.255.0 {
  range dynamic-bootp 192.168.56.100 192.168.56.150 ;
  option broadcast-address  192.168.56.255;
  option routers 192.168.56.1;
  option domain-name-servers 192.168.56.10 ;
  option domain-name "lab.m57.local";
  option ntp-servers 192.168.200.1;
   next-server pxe.lab.m57.local;
}
# Hosts which require special configuration options can be listed here, with the host details. This could be used to hard-code an IP address to a host, or define host-specific parameters like a non-0standard vmunix filename or a fixed IP address.
host fixedhost {
  hardware ethernet 0:0:aa:bb:cc:dd;
  filename "vmunix.fixedhost";
  server-name "fixedhost.m57.local;
}
# When using a fixed IP address, it is more common to assign the host an IP address outside of the DHCP managed IP address range. set.
host fantasia {
  hardware ethernet 08:01:02:03:04:05;
  fixed-address fixedhost2.m57.local;
}
```

接下来，我们需要测试 DHCP 服务器。为此，使用`dnf install -y nmap`命令安装`nmap`程序。

Nmap 是一个功能强大的工具，可以用于许多用途。它最为人知的是作为端口扫描工具，允许用户扫描网络，寻找响应 ping 的服务器，使用命令 `nmap -sP 192.168.200.1/28`，如下所示：

![图 2.6 – Nmap ping 扫描](img/B18349_02_06.jpg)

图 2.6 – Nmap ping 扫描

它还可以用于识别特定服务器的操作系统和开放端口，使用 `nmap` `192.168.200.34 -O` 命令：

![图 2.7 – Nmap ID 服务器](img/B18349_02_07.jpg)

图 2.7 – Nmap ID 服务器

虽然 Nmap 是一个强大的网络扫描工具，但它也可以用于许多额外的任务，包括检查 DHCP 服务器是否正确响应。这可以通过 `broadcast-dhcp-discover` 脚本实现，并通过 `nmnap ––script` 选项使用。此外，如果你的系统有多个接口，可以使用 `-e` 选项指定扫描所使用的网络接口。在这个环境中，DHCP 服务器通过 `nmap --script broadcast-dhcp-discover -e` `enp0s8` 命令进行验证：

![图 2.8 – Nmap DHCP 发现](img/B18349_02_08.jpg)

图 2.8 – Nmap DHCP 发现

我们现在已经了解了 DHCP 服务器的工作原理，接下来是 Apache HTTPD 服务器。

# 设置 web 服务器 – HTTPD

PXE 服务器的下一步是安装 web 服务器。在本例中，我们将使用 Apache。其他 web 服务器也可以使用，但 Apache 是最常见的之一。

## 准备工作

要设置一个 HTTP 服务器，你需要在环境中运行一个 Oracle Linux 8 虚拟机。服务器的 IP 地址应能路由到虚拟机使用的子网，并且防火墙端口应允许 HTTP/HTTPS 流量。理想情况下，你还应有足够的空间存储多个 ISO 文件。通常，20-30 GB 的数据空间就足够了。

## 如何操作…

启动 Apache HTTPD 服务器的安装很简单；只需运行 `yum install -``y httpd`。

一旦安装了 web 服务器，将完整的 ISO 文件复制到服务器的任意目录。在我们的示例中，`OracleLinux-R8-U5-x86_64-dvd.iso` 被复制到 `/root` 目录，如 *图 2.9* 所示。

![图 2.9 – Oracle Linux ISO 位于 /root](img/B18349_02_09.jpg)

图 2.9 – Oracle Linux ISO 位于 /root 目录

下一步是将 ISO 文件挂载到 `docroot` 目录下。对于 Apache 的标准安装，`docroot` 位于 `/var/www/html`，因此创建了一个名为 `OL8` 的目录。当 ISO 挂载后，使用回环挂载方法会更高效。这使得 ISO 文件可以作为普通文件系统挂载。接下来，使用回环挂载方法，将 ISO 文件挂载到 `/var/www/html/OL8` 作为挂载点，使用以下命令：

```
mount -o ro,loop /root/OracleLinux-R8-U5-x86_64-dvd.iso /var/www/html/OL8
```

挂载后，使用 `df` 命令可以显示挂载的文件系统。整个命令链可以在以下截图中看到，*图 2.10*。

![图 2.10 – ISO 已挂载](img/B18349_02_10.jpg)

图 2.10 – ISO 已挂载

现在文件系统已挂载，我们来打开防火墙中的 `80` 和 `443` 端口。可以通过 `firewall-cmd` 命令轻松完成，添加 `http` 和 `https` 端口，并将其作为永久更改保存到防火墙中。一旦端口打开，防火墙规则将被重新加载。以下三个命令执行这些任务：

```
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --reload
```

现在防火墙已打开，Apache 已安装，并且 ISO 已通过环回设备挂载，我们可以通过 `systemctl start` `httpd` 命令启动 Apache 服务器。

你可以通过将浏览器指向服务器的 IP 地址和 `OL8` 目录来验证一切是否按预期工作，如下截图所示。这将显示 ISO 的内容，包括发布说明和 `BaseOS` 目录下的 RPM 文件，如 *图 2.11* 所示。

![图 2.11 – 可通过 httpd 访问的 ISO 文件](img/B18349_02_11.jpg)

图 2.11 – 可通过 httpd 访问的 ISO 文件

# 启动服务 – TFTPD

**TFTP** 用于通过网络传输简单文件。它最常用于加载操作系统的初始引导程序，或嵌入式设备和旧硬件的固件更新。在 PXE 服务器的上下文中，TFTP 系统用于初始引导程序。

## 准备工作

要设置 TFTP 服务器，你需要在你的环境中运行一个 Oracle Linux 8 虚拟机。服务器应与正在构建的系统使用相同的网络子网。理想情况下，你还应有足够的空间来存放几个引导文件。通常，5 GB 的数据空间就足够了。

## 如何操作…

安装 TFTP 很简单。运行 `dnf install -y tftp-server` 命令，如 *图 2.12* 所示。

![图 2.12 – TFTP 安装](img/B18349_02_12.jpg)

图 2.12 – TFTP 安装

安装完成后，我们需要为 TFTP 打开防火墙并重新加载它：

```
firewall-cmd --add-service=tftp --permanent
firewall-cmd --reload
```

这些命令的输出如以下截图所示：

![图 2.13 – TFTP 防火墙](img/B18349_02_13.jpg)

图 2.13 – TFTP 防火墙

安装和打开防火墙后，我们需要通过安装引导程序、准备 Oracle Linux 8 TFTP 位置并创建启动菜单来准备系统。

注意

本示例使用的是基于 BIOS 的主机。如果你想使用 UEFI 主机，你需要安装 `grub2-efi` 并配置 UEFI 特定的参数。Oracle 文档可以在 [`docs.oracle.com/en/operating-systems/oracle-linux/8/install/install-CreatingaNetworkInstallationSetup.html#uefi-clients`](https://docs.oracle.com/en/operating-systems/oracle-linux/8/install/install-CreatingaNetworkInstallationSetup.html#uefi-clients) 找到。

为了安装 BIOS 安装的引导程序，我们将安装 `syslinux` 包。`syslinux` 包包括用于网络启动的引导程序（`PXELINUX`）、Linux（`ext2`/`ext3`/`ext4`）或 `btrfs` 文件系统（`EXTLINUX`）、MS-DOS FAT 文件系统（`SYSLINUX`）和可启动的 El Torito CD-ROM（`ISOLINUX`）。对于网络启动，我们将使用 `PXELINUX`：

![图 2.14 – Syslinux 安装](img/B18349_02_14.jpg)

图 2.14 – Syslinux 安装

接下来，我们将复制启动镜像文件 `pxelinux.0`，并将文件复制到 `/var/lib/tftpboot`：

```
cp /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/
```

然后，我们将在 `tftpboot` 目录下创建一个 Oracle Linux 8 启动目录：

```
mkdir /var/lib/tftpboot/ol8
```

现在，我们需要将之前挂载的 ISO 文件中的 PXE 启动文件复制到新的 Oracle Linux 8 启动目录：

```
cp /var/www/html/OL8/images/pxeboot/* /var/lib/tftpboot/ol8/
```

接下来，我们需要一个启动菜单。幸运的是，我们可以复制示例并快速启动。我们只需要菜单文件，因此使用以下命令复制文件就能顺利进行：

```
cp -v /usr/share/syslinux/{ldlinux.c32,libcom32.c32,libutil.c32,menu.c32,vesamenu.c32} /var/lib/tftpboot/
```

快完成了。为了创建一个 PXS 配置文件目录并构建默认菜单，请使用以下命令：

```
mkdir /var/lib/tftpboot/pxelinux.cfg
```

我们最终可以安装一个配置文件。将这个示例配置文件复制到 `/var/lib/tftpboot/pxelinux.cfg/default`：

```
default linux-auto
prompt 1
timeout 60
display boot.msg
label linux-auto
  menu label ^Install OL8 in Graphical Mode using the kickstart file
  menu default
  kernel ol8/vmlinuz
  append initrd=ol8/initrd.img ip=dhcp inst.repo=http://pxe.lab.m57.local/ol8 ks=http://pxe.lab.m57.local/ks_files/ol8ks.cfg
label linux-manual
  menu label ^Install OL8 in Graphical Mode with manual input
  menu default
  kernel ol8/vmlinuz
  append initrd=ol8/initrd.img ip=dhcp inst.repo=http://pxe.lab.m57.local/ol8
label rescue
  menu label ^Rescue installed system
  kernel ol8/vmlinuz
  append initrd=ol8/initrd.img rescue
label local
  menu label Boot from ^local drive
  localboot 0xffff
```

在这个示例中，默认安装将是 `linux-auto`，它将在 60 秒内自动启动，除非用户手动选择以下选项之一：

+   `linux-auto`：这是默认选项，将使用 kickstart 参数安装 Oracle Linux

+   `linux-manual`：这将启动一个传统的 Oracle Linux 安装，提示用户手动选择 Anaconda 中的所有选项

+   `rescue`：这将以救援模式启动

+   `local`：这将从现有的本地磁盘启动

你可以根据需要轻松修改菜单，以满足你的具体需求。虽然 PXE 启动一开始可能让人感到棘手，但它提供了一个强大的工具来管理你的 Linux 安装。

## 它是如何工作的…

当系统启动时，它们会根据 DHCP 服务器的配置加载引导加载程序。然后，系统将从 TFTP 服务器启动，开始 PXE 过程。

# 曾几何时——手动安装

自动化安装很棒，但对于一些管理员来说，配置和维护支持自动化安装的基础设施的复杂性比手动安装更费力。这个指南将介绍手动安装的过程。

## 准备工作

为此，你需要一个安装源。在这个示例中，将使用 Oracle VM VirtualBox，但同样可以在裸机服务器或其他虚拟化平台上进行。你还需要安装介质。通常，ISO 文件就可以正常工作，但你也可以使用启动 ISO 文件和 HTTP 服务器、FTP 服务器，甚至是网络文件共享。

## 如何操作…

无论你是从 ISO、USB 驱动器，还是通过设置图形化安装选项的 kickstart 文件启动，过程都是一样的！

如果你是从仅有启动功能的镜像启动，你需要启用网络，并指向一个安装源。接下来的步骤将展示如何操作。如果你是从 ISO 或 USB 安装，跳到 *它是如何* *工作的…* 部分：

1.  当从启动镜像启动时，你需要选择**安装 Oracle Linux X**来开始安装过程。尽可能使用最新版的 Oracle Linux。在安装过程中，你会看到第一个屏幕，你可以选择测试启动镜像或继续：

![图 2.15 – Linux 安装程序启动](img/B18349_02_15.jpg)

图 2.15 – Linux 安装程序启动

1.  接下来，选择你的语言：

![图 2.16 – 安装语言](img/B18349_02_16.jpg)

图 2.16 – 安装语言

1.  选择你的语言后，点击**继续**。接下来，你应该会看到**安装** **总结**部分：

![图 2.17 – 安装总结](img/B18349_02_17.jpg)

图 2.17 – 安装总结

目前，我们需要完成两个任务：

+   启用网络

+   设置**安装源**为 Oracle 的 yum 仓库

要启用网络，点击**网络与主机名**选项。

1.  在这里，你需要开启以太网，并更新主机名为服务器名称。网络将自动使用 DHCP 配置启动镜像。你也可以选择手动配置 IP 堆栈，点击**配置**选项。

![图 2.18 – 网络与主机名](img/B18349_02_18.jpg)

图 2.18 – 网络与主机名

1.  接下来，点击**应用**，然后点击**完成**。这将带你回到**安装总结**页面。接下来，我们需要选择**安装源**来设置安装文件的位置。

![图 2.19 – 安装源](img/B18349_02_19.jpg)

图 2.19 – 安装源

在这个示例中，我们使用 Oracle 公共 YUM 服务器作为安装源。你可以根据需要选择性地添加其他仓库，以及在需要时设置代理设置。你也可以为 NFS 服务器路径设置 `FTP://` 补丁。一旦你设置了路径，点击**完成**。

1.  这将带你回到**安装总结**页面。等待几分钟以加载仓库索引。加载完成后，你应该能看到**安装源**选项，以及**软件选择**选项。

![图 2.20 – 选择安装源](img/B18349_02_20.jpg)

图 2.20 – 选择安装源

手动网络启动与 ISO 或 USB 启动基本相同。

## 它的工作方式……

手动启动过程相对简单。一旦启动，在**安装总结**屏幕上，你有许多选项：

+   **本地化**：在这里你可以设置与国家相关的选项，如语言、键盘类型、时间等：

    +   **键盘**：这让你可以设置使用的键盘类型。你可以使用非美国键盘。

    +   **语言支持**：这设置所使用的语言。

    +   **时间和日期**：这设置系统时间，以及将用于自动同步时间的**网络时间协议**（**NTP**）服务器。

+   **软件**：这设置了软件源以及要安装的软件组或软件包：

    +   **安装源**：这设置了安装的软件源和软件包

    +   **软件选择**：此项设置将安装哪些软件组和包。

+   **系统**：这些是系统级参数：

    +   **安装目标**：这是安装的目标位置，包括初始的文件系统布局。

    +   **KDUMP**：这允许你设置 kdump，这是一个内核崩溃转储工具，将系统信息写入磁盘，以便稍后分析。

    +   **网络与主机名**：在这里你可以设置主机名和网络信息。

    +   **安全策略**：此项启用更安全的安装，允许你使用符合流行安全标准（如 STIG 和 PCI-DSS）要求的系统。

+   **用户设置**：在这里你可以设置用户、组和密码：

    +   **根密码**：此项设置初始的 root 密码。

通常，首先设置的是网络。这对于网络安装以及设置 NTP 进行时间同步至关重要：

1.  要设置网络，请选择**网络与主机名**：

![图 2.21 – 设置主机名和 IP 信息](img/B18349_02_21.jpg)

图 2.21 – 设置主机名和 IP 信息

默认情况下，网络端口未启用，且主机名未设置。你可以手动设置主机名。设置主机名时，别忘了使用 FQDN。它应包括名称和域名。启用网络时，系统会自动获取 DHCP 的 IP 信息。如果你想手动配置此信息，请点击**配置…**：

![图 2.22 – 访问手动网络设置](img/B18349_02_22.jpg)

图 2.22 – 访问手动网络设置

1.  这将带你进入网络详细信息页面，你可以手动设置 MTU、IP 地址和其他与网卡相关的设置。如果需要，你也可以禁用 IPv6。

![图 2.23 – 手动网络设置](img/B18349_02_23.jpg)

图 2.23 – 手动网络设置

1.  网络设置完成后，你可以使用网络资源，如 NTP 服务器，同步时钟。为此，在**时间和日期**下，点击齿轮图标：

![图 2.24 – 访问 NTP 设置](img/B18349_02_24.jpg)

图 2.24 – 访问 NTP 设置

1.  然后，你可以添加 NTP 服务器。默认情况下，使用公共服务器，但如果你运行本地 NTP 服务器，确保在此添加它们。

![图 2.25 – NTP 设置](img/B18349_02_25.jpg)

图 2.25 – NTP 设置

1.  接下来，根据需要可以在**安装源**下添加其他软件源。在进行网络安装时，这项操作已经完成。

    你还可以设置要安装的 RPM 组以及任何附加软件。这可以在**软件选择**下进行：

![图 2.26 – 软件选择](img/B18349_02_26.jpg)

图 2.26 – 软件选择

1.  在设置了**基本环境**并添加了任何附加软件之后，下一步是设置安装目标。你可以通过点击**安装目标**来完成这项工作。

    从这里，你应该能看到系统可用的物理和逻辑驱动器。默认设置为将**存储配置**设置为**自动**，但我们可以通过选择自定义配置来探索一种不同的启动磁盘结构。一旦选择了**自定义**，点击**完成**按钮。

![图 2.27 – 安装目标](img/B18349_02_27.jpg)

图 2.27 – 安装目标

1.  从这里，你可以选择默认的卷管理策略，并手动创建新的布局。大多数系统将使用 LVM 安装，因为这使你能够在以后调整分区大小。

![图 2.28 – 手动分区](img/B18349_02_28.jpg)

图 2.28 – 手动分区

别忘了点击**完成**并接受所有对布局的更改。

1.  需要注意的是，如果你从基于 UEK 的安装启动，现在可以选择使用 Btrfs 作为根文件系统。

![图 2.29 – Btrfs 分区](img/B18349_02_29.jpg)

图 2.29 – Btrfs 分区

Btrfs 看起来会有些不同，因为系统将卷管理器和文件系统的角色结合成一个系统。这意味着在安装时，所有文件系统将显示出所有可用的空间。安装操作系统后，可以更新这些空间，并为每个需要限制的目录设置配额。有关详细信息，请参阅 *第四章*中的 Btrfs 配方。

你现在可以通过给 root 设置一个密码来完成这一步……但还有一个基本任务，那就是为系统应用安全政策。安全政策允许安装程序预先配置系统，以遵循设定的政策。这对于安装支持诸如 HIPPA、PCI DSS（用于信用卡处理）或 STIG（用于公共部门工作负载）等工作负载的系统非常有帮助。

1.  要设置一个政策，请从**安装总结**中选择**安全政策**：

![图 2.30 – 安全政策](img/B18349_02_30.jpg)

图 2.30 – 安全政策

1.  一旦你知道要使用的政策，选择它并点击**选择配置文件**：

![图 2.31 – 应用安全政策](img/B18349_02_31.jpg)

图 2.31 – 应用安全政策

现在你将看到政策以及所有正在对系统进行的更改的列表。你还会看到一些可能需要对文件系统布局进行更改的地方。像 STIG 这样的政策中就要求 `/var` 和 `/var/log` 必须是独立的文件系统。

完成后，从**安装总结**中点击**开始安装**以开始安装。

# 附录 – kickstart 选项

以下表格列出了控制安装的最常见选项及其语法：

| **选项名称** | `authselect` |  |  |
| --- | --- | --- | --- |
| **描述** | 它使用 `authselect` 命令配置系统的身份验证选项。`authselect` 选项与 `authselect` 命令相同，直接传递给该命令。 |
| **选项** | 查看`authselect`命令。 |
| **选项名称** | `autostep` |  |  |
| **描述** | 此选项不推荐用于生产环境。它更像是一个故障排除命令，使安装程序逐步显示每个屏幕，每个屏幕显示的时间较短。 |
| **选项** | `--autoscreenshot` 将自动截图每个安装屏幕。图像会在安装过程中存储在 `/tmp/anaconda-screenshots/`，安装完成后移动到 `/root/anaconda-screenshots`。这对于故障排除非常有帮助，因为如果您错过了必需的 kickstart 选项，安装未能自动运行，您将会看到缺少数据的屏幕，并可以手动输入缺失的配置数据。然后，点击继续时，屏幕会自动捕获您手动输入的数据。 |
| **选项名称** | `cmdline` |  |  |
| **描述** | 当使用此选项时，安装将在非交互式命令行模式下运行。Anaconda 的任何输入提示将停止安装。请确保先使用 `autostep` 选项进行测试。 |
| **选项** | 无 |
| **选项名称** | `eula` |  |  |
| **描述** | 如果使用此选项，则需要 `–-agreed` 选项。此选项允许您自动接受 **最终用户许可协议**（**EULA**）。使用此选项时，安装程序在系统第一次重启时将不会提示接受 EULA。这对于使用 PXE 启动作为自动扩展策略的一部分的环境非常有用。 |
| **选项** | `--agreed` 强制接受 EULA（最终用户许可协议），必须使用此选项。如果未使用该选项，EULA 参数将被忽略。 |
| **选项名称** | `firewall` |  |  |
| **描述** | 该选项配置已安装系统上的防火墙。 |
| **选项** |

+   `--enabled` 或 `–enable` 是必需的。它启用或禁用防火墙。

+   `--disabled` 或 `–disable` 是必需的。如果禁用，`iptables` 规则将不会配置。

+   `--trust` 设置受信网络设备，允许所有流量通过防火墙。要配置多个设备，请分别列出每个设备项，不使用逗号，例如 `–trust eth0 –``trust eth8`。

+   `--incoming` 添加以下每个标签以允许这些常见服务的入站流量：

    +   `–``ssh`

    +   `–``smtp`

    +   `–``http`

    +   `–``ftp`

+   `--port=` 指定一个特定的端口以允许访问服务器。可以使用 `/etc/service` 中的端口来指定特定的 `端口:协议` 组合。多个端口之间使用逗号分隔。例如 `ldap:tcp, 1521:tcp`。

+   `--service=` 提供了一种简便的方法来启用特定服务，如 CUPS，或需要打开多个 UDP/TCP 端口的服务。这为打开这些端口提供了一种易于阅读的格式。

+   `--use-system-defaults` 不配置防火墙。如果指定了其他选项，则它们将被忽略。

|  |

| **选项名称** | `firstboot` |  |  |
| --- | --- | --- | --- |
| **描述** | 此选项可以禁用在系统启动时运行设置应用程序。如果启用，请确保已安装 `initial-setup.rpm`（如果需要图形界面，则为 `initial-setup-gui.rpm`）。此选项默认禁用。 |
| **选项** |

+   `--enable` 或 `–enabled` 会在系统第一次启动时启动设置程序。

+   `--disable` 或 `–disabled` 不会在系统第一次启动时启动设置程序。

+   `--reconfig` 启用设置程序在启动时以重新配置模式启动。这使得可以设置网络配置、root 密码、时区、语言、鼠标和键盘选项，除了默认选项之外。

|  |  |  |  |

| **选项名称** | `graphical` |  |  |
| --- | --- | --- | --- |
| **描述** | 如果没有设置，`graphical` 命令是默认选项。安装将继续进行完全图形化安装。 |
| **选项** | `--non-interactive` 在完全非交互模式下执行安装。此模式将在需要用户交互时终止安装。 |
| **选项名称** | `group` |  |  |
| **描述** | 此选项在系统上创建一个用户组。 |
| **选项** |

+   `--name=` 是必需的，并设置组的名称。

+   `--gid=` 是一个可选参数，将设置组的 GID。

|

| **选项名称** | `halt` |  |  |
| --- | --- | --- | --- |
| **描述** | 如果使用，安装完成后系统将停止。这是默认的完成方法，如果没有设置其他方法。其他完成方法包括 `poweroff`、`reboot` 和 `shutdown`。 |
| **选项** | 无 |
| **选项名称** | `logging` |  |  |
| **描述** | 这对于故障排除 PXE 安装非常有用。它控制安装过程中安装程序的日志记录。它不会配置正在安装的系统的日志记录。 |
| **选项** |

+   `--host=` 是 syslog 主机，日志会发送到该主机。

+   `--port=` 是用于远程 syslog 主机的 syslog 端口。如果未设置，则使用默认端口（UDP 514）。

+   `--level=` 指定发送到设备 `tty3` 的最小消息级别。无论此级别如何，所有消息仍会发送到日志主机和文件。有效值包括 `critical`、`debug`、`error`、`info` 和 `warning`。

|

| **语法** | `logging [--host=host] [--port=port] [--level=critical | debug | error | info]` |
| --- | --- | --- | --- | --- |
| **选项名称** | `poweroff` |  |  |
| **描述** | 使用时，安装完成后会执行关机并关闭系统电源。 |
| **选项** | 无 |
| **选项名称** | `reboot` |  |  |
| **描述** | 安装完成后重新启动系统。 |
| **选项** | `--eject` 在重新启动系统之前弹出可引导介质（DVD、USB 等）。 |
| **选项名称** | `repo` |  |  |
| **描述** | 启用额外的 DNF/YUM 仓库作为安装源。每个仓库需要单独的一行。 |
| **选项** |

+   `--name=` 是必需的，它设置仓库的名称和 URL。如果名称与现有仓库冲突，将被忽略。格式为 `--name=repoid [--baseurl=url&#124;--mirrorlist=url&#124;--metalink=url] [options]`。以下是可以添加到 URL 的附加选项。作为警告，yum 仓库配置文件中可以使用的变量不受支持。但是，你可以使用 `$releasever` 和 `$basearch`：

    +   `--baseurl`：这是仓库的 URL

    +   `--mirrorlist`：这是仓库的镜像列表

    +   `--metalink`：这是仓库的 `metalink` URL

+   `--proxy=[protocol://][username[:password]@]host[:port]` 设置仓库的代理。

+   `--noverifyssl` 禁用 HTTPS 服务器的 SSL 验证。

|

| **选项名称** | `selinux` |  |  |
| --- | --- | --- | --- |
| **描述** | 用于配置 `selinux` 的初始策略和状态。默认策略为强制执行。 |
| **选项** |

+   `--enforcing` 启用 SELinux，并将策略设置为强制执行。这是默认设置。

+   `--permissive` 启用 SELinux，并将策略设置为宽松。这允许警告输出到控制台的 syslog，但不强制执行策略。

+   `--disabled` 禁用 SELinux。

|

| **选项名称** | `shutdown` |  |  |
| --- | --- | --- | --- |
| **描述** | 安装完成后，在系统上运行 `shutdown` 命令。 |
| **选项** | 无 |
| **选项名称** | `sshkey` |  |  |
| **描述** | 将 SSH 密钥添加到指定用户的授权密钥文件中。 |
| **选项** | `--username` 是必需的。其用法为：`sshkey --``username=user KEY` |
| **选项名称** | `text` |  |  |
| **描述** | 使用时，安装将以文本模式运行。默认情况下，安装程序使用 GUI 安装方法。只能使用文本/图形模式中的一种选项。 |
| **选项** | `--non-interactive` 以非交互模式进行安装。如果需要用户操作，该模式将终止安装。 |
| **选项名称** | `url` |  |  |
| **描述** | 使用时，它将使用为安装文件指定的 URL，该文件来自 ISO。URL 可以格式化为 FTP、HTTP 或 HTTPS。 |
| **选项** |

+   `--url` 是必需的。这是用于源文件的 URL。以下是语法示例：

    [`http://server/direcort`](http://server/direcort)`ftp://username:password@server/path`

+   `--mirrorlist` 是一个可选参数，用于设置用于安装的镜像 URL。

+   `--proxy` 是一个可选参数，用于在从 HTTP/HTTPS 或 FTP 服务器安装时设置代理。

+   `--noverifyssl` 是一个可选参数，禁用使用 HTTPS 服务器时的 SSL 验证。

+   `--metalink` 是一个可选参数，用于设置用于安装的 ULN/metalink URL。变量替换会对 `$releasever` 和 `$basearch` 进行。

|

| **选项名称** | `user` |  |  |
| --- | --- | --- | --- |
| **描述** | 在系统上创建一个新用户 |
| **选项** |

+   `--name` 是一个必需的参数，表示用户名。

+   `--uid` 是用户的**用户** **ID**（**UID**）。

+   `--gid` 是用户的**组** **ID**（**GID**）。

+   `--groups` 是一个以逗号分隔的用户应属于的组名列表。

+   `--homedir` 是用户的主目录。

+   `--lock` 如果使用，将默认锁定新用户的帐户。

+   `--password` 是新用户的密码。

+   `--iscrypted`，如果使用，意味着密码参数是加密后的密码。你可以从运行中的系统的 `/etc/shadow` 文件中获取。

+   `--plaintext`，如果使用，意味着密码参数是明文密码。

+   `--shell` 是用户默认 shell 的路径。

|

表 2.1 – Kickstart 选项
