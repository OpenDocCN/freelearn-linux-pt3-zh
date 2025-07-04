

# 一起走向云端

在虚拟化方面，我们已经走过了很长的路。从在数百个大学校园内将虚拟切片和条带（虚拟化的前身）DNA 测序仪分割到最先进的虚拟农场，能够托管数千个**虚拟机**（**VMs**）和数 PB 的存储，它们有一个共同点：复杂性。但这真的必须如此复杂吗？当我想到虚拟化时，我会想到四个资源：计算、内存、存储和网络。它们相互交织，共同构成了这个网络。没有计算或内存，你无法托管工作负载；没有存储，你无法托管或存储应用程序或工作负载组件；没有网络，你的工作负载无法流动。想象一下，你的车就是你的工作负载，高速公路是你的网络，乘客是你的计算和内存，行李厢就是存储。

在本章中，我们将演示如何在本地环境以及公共云中充分利用 Oracle Linux，诸如**Oracle Cloud Infrastructure**（**OCI**）或**Amazon Web Services**（**AWS**）。我们将演练在本地构建一个定制的 Oracle Linux 8.x 环境，并将其作为镜像迁移到 OCI，以及迁移到其他**云服务提供商**（**CSPs**），包括作为 AMI 的 AWS。*云原生*是一个热门词汇，也是一个颠覆性技术。*使用 Portainer 管理 Docker Desktop*的配方将演示如何创建一个 Portainer 环境，以管理围绕一个应用程序的容器生态系统。

本章将涵盖以下内容：

+   云中的 Oracle Linux

+   使用 Portainer 让 Docker Desktop 更易于管理

+   使用 VirtualBox 将虚拟机迁移到 OCI

+   对于其他人，我们将为 AWS 创建自定义镜像

# 技术要求

一般来说，在这种情况下，使用带**图形用户界面**（**GUI**）的操作系统会更为简便。也就是说，即使最初没有使用 GUI 来构建服务器，你仍然可以安装 GUI。以下是添加和配置 GUI 的详细步骤：

1.  安装 GUI 软件包：

    ```
    $ (sudo if not root) dnf groupinstall "Server with GUI"
    ```

1.  设置 GUI 为默认目标：

    ```
    $ (sudo if not root) systemctl set-default graphical.target
    ```

1.  设置 GUI 为默认启动源：

    ```
    $ (sudo if not root) ln -sf /lib/systemd/system/runlevel5.target /etc/systemd/system/default.target
    ```

1.  重启服务器：

    ```
    $ (sudo if not root) reboot (or shutdown –r now)
    ```

# 云中的 Oracle Linux

我们可以列出所有的 CSP 并详细分析它们的优缺点，但归根结底，每个 CSP 都有其主要竞争力。Oracle Linux 所做的是在此基础上进行扩展，使得产品更强大、更具多样性。就像任何其他（开源）Linux 发行版一样，Oracle Linux 是一个多功能且强大的操作系统。就 Oracle Linux 而言，操作系统的设计兼容 RHEL 的元素，不是为了与之竞争，而是为了添加功能，同时为用户提供超越 RHEL 的 Oracle 功能。

Oracle Linux 的灵活性，类似于 RHEL，使其不仅仅是一个操作系统。Oracle Linux 扩展了操作系统的功能，最大化了围绕虚拟化的 Linux 工具集，通过 **Oracle Linux 虚拟化管理器**（**OLVM**）进入虚拟机管理、跨平台的云功能，以及自动化等方面。Oracle 的目标是提供一个 RHEL 操作系统的功能，但又超越这一点，将操作系统与全球业务网络中的工具集连接，以推动业务成功。现在，经过前面的铺垫，才是 Oracle Linux 的真正力量与多功能性，其中一些内容在本章开头已提到。本部分讲述了哪些 CSP 与 Oracle Linux 最兼容；然而，了解操作系统的功能及其运作方式，对于理解哪些 CSP 与 Oracle Linux 更加兼容并最大化其能力是至关重要的。

Oracle 支持并推广适用于业务关键环境的开源技术。Oracle 是 Linux 基金会、Eclipse 基金会和 Java 社区流程的创始成员，并为开源项目做出了重大贡献。将 Oracle 软件与开源技术结合，能够提供高性能、可靠性和数据安全，同时降低计算成本。

下图提供了对比 Oracle Linux 的“免费”版本（唯一收费组件是支持）与 RHEL 加上扩展功能的成本效益分析，如前文所述。

![图 13.1 – RHEL 与 Oracle Linux](img/B18349_13_001.jpg)

图 13.1 – RHEL 与 Oracle Linux

和任何 Linux 发行版一样，每个 CSP（云服务提供商）都有自己的 Linux 平台版本。例如，AWS 有 **Amazon Linux 2**（**AL2**），Azure 提供与任何基于 RHEL 或 Ubuntu 的平台兼容的服务，而 Oracle 自然也有 Oracle Linux。Ubuntu 和 Oracle Linux、AL2 等之间有许多不同之处，但我们今天不讨论这些差异。归根结底，所有 CSP 都可以运行 Linux 发行版，但并非所有都针对 Oracle Linux 做了优化。使用 AWS 和 Azure 时，你可以选择购买 Oracle 的技术支持（**BYOS**，即购买自己的支持），支持内容包括访问补丁、升级和操作系统支持，或者在没有技术支持的情况下免费使用这些服务。使用 OCI Premier 服务时，支持是免费的。下图显示了主要的 CSP 及其对 Oracle Linux 的不同采用方式。

![图 13.2 – 云选项](img/B18349_13_002.jpg)

图 13.2 – 云选项

谁是最美的？我们来看看 AWS 提供了哪些 Linux 发行版。我们将简要介绍 AWS EC2 镜像构建器，以展示 Linux 在 AWS 及其他环境中的多功能性。稍后在本章中，这将用于 *使用 VirtualBox 将虚拟机导入 OCI* 这一部分。

## 准备就绪

首先，你需要有一个 AWS 账户，无论是个人账户还是通过你的组织，关键是你拥有正确的授权来访问账户并构建基础设施。在这种情况下，我选择了**Root 用户**，因为这是我的个人账户，并且我拥有这些权限。

![图 13.3 – Root 登录](img/B18349_13_003.jpg)

图 13.3 – Root 登录

输入用户类型后，你需要提供密码并继续完成身份验证选项。我已启用**多重身份认证**（**MFA**），并强烈建议使用此方法来保护你的账户。如果启用了 MFA，接下来你将看到捕获验证码的步骤，如下图所示。注意，虽然还有其他双因素认证选项，但使用 Google Authenticator 是最佳实践。当然，也可以使用其他应用，如 Microsoft、RSA 等。

![图 13.4 – MFA](img/B18349_13_004.jpg)

图 13.4 – MFA

完成登录和 MFA 部分后，你将进入 AWS 登陆页面。你需要进入 EC2，AWS 计算资源（CPU，RAM），以了解可用的 Linux 版本及其选择，超出标准 Linux 镜像（AMI）的默认选项。请注意，虽然某些版本未列出，但并不意味着不可用：你可以通过 Marketplace 或自定义镜像找到其他选项。

![图 13.5 – 计算](img/B18349_13_005.jpg)

图 13.5 – 计算

一旦进入 AWS EC2 世界，最简单的方式是查看 **EC2 镜像构建器**。镜像构建器将复杂的镜像世界转化为更容易理解的概念，帮助你将技术需求转化为执行：

![图 13.6 – 镜像构建器](img/B18349_13_006.jpg)

图 13.6 – 镜像构建器

进入 **镜像构建器** 后，你需要进入 **Amazon Marketplace** 下的 **镜像产品**。这是 AWS 提供的其他所有服务的入口，将直接带你进入镜像产品页面。

![图 13.7 – 镜像产品](img/B18349_13_007.jpg)

图 13.7 – 镜像产品

注意

选择 Oracle AMI 时需要小心。有些第三方会收取费用使用 AMI，但不会提供 Oracle 技术支持。它们中的一些可能会非常昂贵，每年花费数百美元。

如前所述，你不会在**AWS Marketplace**标签下找到官方的 Oracle Linux 镜像。请按照*如何操作...*部分中的说明，访问官方的 Oracle Linux 镜像。

![图 13.8 – 在 Marketplace 中搜索 Oracle Linux](img/B18349_13_008.jpg)

图 13.8 – 在 Marketplace 中搜索 Oracle Linux

如果你像前面的截图那样在 AWS Marketplace 中搜索`oracle linux`，你将只看到其他组织构建并通过 AWS Marketplace 公开提供的 Oracle Linux 发行版。*这些不是官方的 Oracle Linux 镜像。* 本章稍后我们将介绍如何在 Oracle VirtualBox 中构建虚拟机，并通过 AWS 将该镜像转换为 AMI。

## 如何操作…

以下是找到 Oracle AMI 的步骤：

1.  在浏览器中打开[`console.aws.amazon.com/ec2/`](https://console.aws.amazon.com/ec2/)。

![图 13.9 – 在镜像下拉菜单中查找 AMI](img/B18349_13_009.jpg)

图 13.9 – 在镜像下拉菜单中查找 AMI

1.  在左侧导航面板中，选择**AMIs**，然后从弹出的窗口中选择**公共镜像**：

![图 13.10 – 公共镜像](img/B18349_13_010.jpg)

图 13.10 – 公共镜像

1.  使用 ID `131827586825`按所有者筛选。该 ID 与官方 Oracle 镜像相关，只会显示由 Oracle 创建的官方 Oracle Linux AMI：

![图 13.11 – Oracle AMI](img/B18349_13_011.jpg)

图 13.11 – Oracle AMI

1.  选择你希望使用的官方 Oracle AMI。选择 AMI 后，直接启动该 AMI 以继续配置资源和操作系统：

![图 13.12 – 实例启动](img/B18349_13_012.jpg)

图 13.12 – 实例启动

1.  启动后，这将引导你进入操作系统镜像和虚拟机构建页面（见下图）。在这里，你为虚拟机添加一个合适的名称，并选择虚拟机的操作系统和网络密钥对，如下所示的实例启动窗口所示。

![图 13.13 – 实例配置](img/B18349_13_013.jpg)

图 13.13 – 实例配置

1.  一旦你从选定的 AMI 启动了新的虚拟机，你将看到启动状态：

![图 13.14 – 启动状态](img/B18349_13_014.jpg)

图 13.14 – 启动状态

1.  一旦你的新 Oracle Linux 虚拟机创建完成，你可以在 OCI 的**计算** | **EC2** | **镜像**选项卡中查看该虚拟机。然后，点击实例 ID 并复制公共 IP 地址。

![图 13.15 – 公共 IP](img/B18349_13_015.jpg)

图 13.15 – 公共 IP

1.  打开 PuTTY 或终端，具体取决于你是使用 Windows 机器还是 Mac，然后使用 OPC 作为用户通过`ssh`连接到你的机器（这是默认的 Oracle 用户；这些术语是可以更改的，但这里我们不讨论）。第一次运行`ssh`命令时，将会永久添加密钥对到已知主机的身份验证列表中：

    ```
    % ssh opc@18.116.203.168
    The authenticity of host '18.116.203.168 (18.116.203.168)' can't be established.
    ED25519 key fingerprint is SHA256:HS1eSJOOs3slMxDU41nYfDkxMQ7uocISC21lxLnQJTk.
    This key is not known by any other names
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added '18.116.203.168' (ED25519) to the list of known hosts.
    ```

# 使用 Portainer 使 Docker Desktop 更易于管理

Portainer 是一个容器管理平台，如本章介绍中所提到的。Portainer 最初是作为 Docker 的用户界面设计的，但现在它也支持其他几个容器环境。Portainer 拥有超过 100 万用户，并在 GitHub 上有着显著的存在。Portainer 提供两个版本：免费且开源的 **Community Edition** (**CE**) 和收费的 **Enterprise Edition** (**EE**)。在本食谱中，我们将使用免费和开源版本。

如果你更喜欢通过图形界面（GUI）管理容器，Portainer 是一个极好的选择。尽管命令行界面（CLI）和 API 端点对于开发很有用，但 GUI 往往更适合用于管理生产应用。通过 Portainer，你可以轻松监控多个端点，并授予团队成员访问共享部署环境的权限。下图展示了 Portainer 如何与 Docker 的容器镜像对齐，以及 Portainer 如何简化镜像管理。Portainer 提供了一个 Docker CLI 上的用户界面，使得无论是普通用户还是资深工程师，都能更轻松地管理容器。

![图 13.16 – Portainer 容器管理](img/B18349_13_016.jpg)

图 13.16 – Portainer 容器管理

Portainer 通常部署在自己的容器内。如果你使用的是 Docker，本指南将帮助你启动和运行 Portainer。然而，如果你更喜欢使用 Kubernetes，也可以使用官方的 Helm 图表直接运行 Portainer。以下是设置 Portainer 所需步骤的简要概述：

1.  安装 Docker。

1.  创建并配置 Portainer 的容器。

1.  打开必要的端口并相应地配置 OCI。

1.  登录 Portainer 界面并设置 UI。

以下部分将详细解释每个步骤。继续之前，请确保已经安装 Docker。如果你使用的是 Windows 或 Mac 计算机，可以下载并安装最新版本的 Docker Desktop 安装程序。Linux 用户可以试用 Docker Desktop for Linux 的测试版，或者使用以下说明安装 Docker Engine。对于本指南，我们将在 OCI 云中创建一个 Oracle Linux 虚拟机，并在该虚拟机上安装 Portainer，而不是在本地环境中安装。

## 准备工作

需要进行一些初步工作。如前所述，Docker 是一个容器管理环境。在本食谱中，Portainer 将位于 Docker 容器中，因此我们需要安装并配置 Docker。你将在本章稍后创建镜像时再次使用其中一些技能。实际上，我们在这里做的基础工作将帮助你贯穿整个章节，因为这些技能为管理 Oracle Linux 环境打下了基础，无论它们是在云中还是本地。

![图 13.17 – Portainer 虚拟机创建](img/B18349_13_017.jpg)

图 13.17 – Portainer 虚拟机创建

登录 Portainer 虚拟机所需的信息位于前面显示的镜像数据中。记住，就像前面在创建 OCI 虚拟机时提到的那样，需要创建并添加公钥。正如我们之前所看到的，这是你能够登录到这个环境的唯一方式。

如图所示，使用默认的 OPC 用户登录到虚拟机：

```
% ssh opc@132.226.58.24
Activate the web console with: systemctl enable --now cockpit.socket
Last login: Mon Aug 21 14:15:13 2023 from 107.200.172.229
[opc@portainer2 ~]$
```

登录后，你需要更新仓库。与本书中其他食谱类似，你需要超级用户权限。由于输出内容较长，更新的输出已被截断：

```
[opc@portainer2 ~]$ sudo su -
Last login: Mon Aug 21 15:28:41 GMT 2023 on pts/0
[root@portainer2 ~]# yum upgrade -y && yum update -y
=======================================================================================================
Package                           Arch      Version                             Repository            Size
=======================================================================================================
Upgrading:
 cloud-init                        noarch    22.1-8.0.4.el8_8.1                ol8_appstream      1.1 M
 iscsi-initiator-utils             x86_64    6.2.1.4-8.git095f59c.0.1.el8_8    ol8_baseos_latest  382 k
 iscsi-initiator-utils-iscsiuio    x86_64    6.2.1.4-8.git095f59c.0.1.el8_8    ol8_baseos_latest  102 k
 microcode_ctl                     x86_64    4:20230214-2.0.3.el8              ol8_baseos_latest  7.7 M
 systemd                           x86_64    239-74.0.4.el8_8.3                ol8_baseos_latest  3.6 M
 systemd-libs                      x86_64    239-74.0.4.el8_8.3                ol8_baseos_latest  1.1 M
 systemd-pam                       x86_64    239-74.0.4.el8_8.3                ol8_baseos_latest  506 k
 systemd-udev                      x86_64    239-74.0.4.el8_8.3                ol8_baseos_latest  1.6 M
```

更新仓库后，我们准备好开始安装 Docker 并开启我们的容器之旅。为了安装 Docker，我们需要安装 ZIP 工具以解压我们已下载的所需文件。为此，我们将使用 `dnf` 命令：

```
[root@portainer2 ~]# dnf install -y dnf-utils zip unzip
```

安装 ZIP 工具后，我们需要启用 Docker 仓库，以便下载 Docker。以下是启用仓库并将其添加到仓库列表的命令：

```
[root@portainer2 ~]# dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
```

注意

我们需要消除一个漏洞，该漏洞允许攻击者通过执行任何类型的容器中的 root 命令来覆盖主机的 `runc` 二进制文件。攻击者可以利用这个漏洞，通过使用攻击者控制的镜像启动新容器，或者如果他们之前拥有写入权限，还可以渗透到现有容器中：

**[root@portainer2 ~]# dnf remove -****y runc**

使用以下 `dnf` 命令安装 Docker。由于输出内容较长，已被截断：

```
[root@portainer2 ~]# dnf install -y docker-ce --nobest
Docker CE Stable – x86_64                                                   1.3 MB/s |  49 kB     00:00
Dependencies resolved.
 Package                     Arch     Version                                     Repository           Size
Installing:
 docker-ce                   x86_64   3:24.0.5-1.el8                            docker-ce-stable   24 M
…
[Truncated due to length]
...
Complete!
```

一旦 Docker 安装完成，我们可以继续启用 Docker 服务并启动 Docker 引擎。为了启用 Docker 服务，我们需要使用 `systemctl`，如下所示：

```
[root@portainer2 ~]# systemctl enable docker.service
Created symlink /etc/systemd/system/multi-user.target.wants/docker.service → /usr/lib/systemd/system/docker.service.
```

一旦服务启用，我们可以继续启动 Docker 服务/Docker 引擎。我们将再次使用 `systemctl` 调用 Docker 引擎启动：

```
[root@portainer2 ~]# systemctl start docker.service
```

启动服务后，我们需要检查服务的状态，以确保 Docker 已正确启动且服务正在运行且健康：

```
[root@portainer2 ~]# systemctl status docker.service
● docker.service - Docker Application Container Engine
   Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-08-21 15:50:49 GMT; 1min 19s ago
     Docs: https://docs.docker.com
 Main PID: 76180 (dockerd)
    Tasks: 9
   Memory: 28.2M
   CGroup: /system.slice/docker.service
           └─76180 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
Aug 21 15:50:48 portainer2 systemd[1]: Starting Docker Application Container Engine...
...
Truncated due to length lines 1-20/20 (END)
...
```

现在我们的 Docker 引擎已启动并运行，我们想查看正在运行的 Docker 版本，以及其健康状态和所有其他与 Docker 引擎相关的数据：

```
[root@portainer2 ~]# docker info
Client: Docker Engine - Community
 Version:    24.0.5
 Context:    default
 Debug Mode: false
 Plugins:
  buildx: Docker Buildx (Docker Inc.)
    Version:  v0.11.2
    Path:     /usr/libexec/docker/cli-plugins/docker-buildx
  compose: Docker Compose (Docker Inc.)
    Version:  v2.20.2
    Path:     /usr/libexec/docker/cli-plugins/docker-compose
Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 24.0.5
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Using metacopy: false
  Native Overlay Diff: false
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Cgroup Version: 1
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc io.containerd.runc.v2
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 8165feabfdfe38c65b599c4993d227328c231fca
 runc version: v1.1.8-0-g82f18fe
 init version: de40ad0
 Security Options:
  seccomp
   Profile: builtin
 Kernel Version: 5.15.0-103.114.4.el8uek.x86_64
 Operating System: Oracle Linux Server 8.8
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 15.32GiB
 Name: portainer2
 ID: 597d584f-bf1c-4cda-aa22-8ca2c28f33c0
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

由于 Portainer 需要更高版本的 Python 环境，我们需要确保安装了正确的版本。Python 是安装的前提条件。以下命令的输出由于内容较长，已被截断：

```
yum install -y libffi libffi-devel openssl-devel python3 python3-pip python3-devel
Last metadata expiration check: 0:13:58 ago on Mon 21 Aug 2023 03:43:32 PM GMT.
Package libffi-3.1-24.el8.x86_64 is already installed.
Package openssl-devel-1:1.1.1k-9.el8_7.x86_64 is already installed.
Package python36-3.6.8-38.module+el8.5.0+20329+5c5719bc.x86_64 is already installed.
Package python3-pip-9.0.3-22.el8.noarch is already installed.
Dependencies resolved.
============================================================================================================
 Package                   Arch      Version                                      Repository            Size
============================================================================================================
Installing:
Complete!
```

安装 Python 时，我们使用 PIP，这是 Python 的标准包管理器。PIP 允许你管理和安装所有非标准 Python 库中的包。安装 Python 是 Portainer 安装的基础要求。

你需要退出 root 用户来安装此版本的 Python：

```
[opc@portainer2 ~]$ pip install --upgrade --ignore-installed pip setuptools
Defaulting to user installation because normal site-packages is not writeable
Collecting pip
  Downloading pip-21.3.1-py3-none-any.whl (1.7 MB)
     |████████████████████████████████| 1.7 MB 38.3 MB/s
Collecting setuptools
  Downloading setuptools-59.6.0-py3-none-any.whl (952 kB)
     |████████████████████████████████| 952 kB 78.6 MB/s
Installing collected packages: setuptools, pip
Successfully installed pip-21.3.1 setuptools-59.6.0
```

你需要再次以 root 身份登录来安装和更新 yum 包。Yum 需要 root 权限来执行，所以你需要使用 `sudo` 或 `su` 切换到 root 用户。

更新 Python 安装后，我们需要安装并更新 Docker 和 Portainer 的 Python 库要求：

```
[opc@portainer2 ~]$ sudo su -
Last login: Mon Aug 21 15:33:33 GMT 2023 on pts/0
[root@portainer2 ~]# yum install -y libffi libffi-devel openssl-devel python3 python3-pip python3-devel
```

你需要退出 root 账户以安装`docker-compose`，并使用 Python pip 来执行。Python pip 是安装任何 Python 包的标准方法，是以安全方式安装这些包的首选方法：

```
[root@portainer2 ~]# exit
logout
[opc@portainer2 ~]$ pip3 install docker-compose
```

## 如何操作……

1.  现在我们准备开始安装和配置 Portainer 环境。我们首先需要为 Portainer 创建一个卷。我们通过告诉 Docker 创建该卷来完成此操作：

    ```
    [opc@portainer2 ~]$ sudo docker volume create portainer_data
    portainer_data
    ```

1.  创建卷后，我们安装 Portainer 容器并暴露 Portainer UI 将使用的端口，通过该端口我们将访问 UI：

    ```
    [opc@portainer2 ~]$ sudo docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
    Unable to find image 'portainer/portainer-ce:latest' locally
    latest: Pulling from portainer/portainer-ce
    772227786281: Pull complete
    96fd13befc87: Pull complete
    5171176db7f2: Pull complete
    a143fdc4fa02: Pull complete
    b622730c7bdc: Pull complete
    c1cad9f5200f: Pull complete
    d8a77b01f2cb: Pull complete
    0d4d8543f764: Pull complete
    c6fd0bcf10c9: Pull complete
    889200668c1c: Pull complete
    4f4fb700ef54: Pull complete
    Digest: sha256:94c3056dbe879f3a3df06b427713392a0962924914f5c2fc 557de3797f59f926
    docker ps to ascertain which containers are running and their health, within Docker Engine.
    ```

![图 13.18 – Docker 容器运行](img/B18349_13_018.jpg)

图 13.18 – Docker 容器运行

1.  一旦 Portainer 安装完成，并且`docker ps`显示 Portainer 已经启动，我们需要确保 OCI 已配置为允许端口`9000`可见。为了暴露这个端口，我们需要告诉 OCI 我们正在使用的**虚拟云网络**（**VCN**），以允许该端口开放。

![图 13.19 – 安全列表导航](img/B18349_13_019.jpg)

图 13.19 – 安全列表导航

当我们导航到相关的安全列表后，我们将创建一个如以下截图所示的规则。

![图 13.20 – VCN 安全列表规则创建](img/B18349_13_020.jpg)

图 13.20 – VCN 安全列表规则创建

一旦规则创建完成，我们可以看到端口已暴露在 OCI 之外。

![图 13.21 – 安全列表规则](img/B18349_13_021.jpg)

图 13.21 – 安全列表规则

我们可以打开浏览器并访问 Portainer UI。为了访问该 UI，我们需要两个信息：虚拟机的公网 IP 和端口。以下截图显示了端口`9000`，IP 地址与我们使用`ssh`登录虚拟机时使用的 IP 相同。

你也可以在 OCI 控制台找到该 IP。通过**计算**菜单导航到你的虚拟机，然后进入**实例**，这将带你到 OCI 控制台菜单中所有已创建的虚拟机。然后，复制虚拟机的 IP 地址，如下图所示。

![图 13.22 – 虚拟机的 IP](img/B18349_13_022.jpg)

图 13.22 – 虚拟机的 IP

1.  获得地址后，打开浏览器并以`https://<IP>:9000`的形式输入该信息。在我们的例子中，这是`https://132.226.58.24:9000`。

    注意：如果你遇到过时的页面，可能需要刷新 Portainer 环境，像下面这样：

![图 13.23 – Portainer 登录页面](img/B18349_13_023.jpg)

图 13.23 – Portainer 登录页面

1.  为了刷新，你需要通过运行`docker`的`restart`命令来重启 Portainer 环境：

    ```
    [opc@portainer2 ~]$ sudo docker restart portainer
    portainer
    ```

1.  重启 Portainer 后，你可以返回浏览器并刷新页面。现在应该能够进入 Portainer 欢迎页面。

![图 13.24 – Portainer 初始配置](img/B18349_13_024.jpg)

图 13.24 – Portainer 初始配置

1.  Portainer 将要求你设置一个 12 位字母数字密码以保护环境安全。

![图 13.25 – Portainer 密码创建](img/B18349_13_025.jpg)

图 13.25 – Portainer 密码创建

现在，你的工作 Portainer 环境正在 OCI 中的 VM 中的 Docker 容器内运行。

# 使用 VirtualBox 将 VM 跳板到 OCI

工作时间从早上 9 点到下午 5 点？

在许多这些步骤中的一个共同因素是将工作负载从一个环境移动到另一个环境。使用通用格式（例如**开放虚拟化格式**/**开放虚拟化设备**（**OVF**/**OVA**）），可以在不同的虚拟化主机环境之间完成不同类型的虚拟化（例如 VMware、KVM 和 Xen）。基本上，这将组件放入一个大多数虚拟化主机环境可以理解的中心化可理解格式中。图像可以在虚拟化格式之间进行传输和交换。在这种情况下，还需要一些拼图块，如网络、存储和连接的驱动器数量限制。

想象一个世界，在这个世界中，任何工作负载（无论是基于应用程序还是数据库的）都可以在整个项目生命周期中与你同在，并且可以在不同的虚拟化农场或云服务提供商之间转移。你可以在你的不同环境中携带这些创建的环境。

VirtualBox 作为主机存在于你的系统上，并被归类为“类型 2”虚拟化监视程序，允许你在主机环境上托管多个客户机。你的主机环境，VirtualBox，拥有大企业产品所期望的每个虚拟农场的方面，只是将其分解为更小、更简单的部分。计算和内存方面从主机环境（你的计算机）中划分出来。存储组件由你的本地硬盘部分或外部存储环境（如外部连接的 NVMe 驱动器）组成。最后，网络由桥接组件组成，这些虚拟网络位于你的物理接口之上。

虚拟机和操作系统，哪个先来？

每一次进化都从一个基础开始。在我们考虑将虚拟机从 VirtualBox 迁移到 OCI 之前，我们必须准备好这个虚拟机进行迁移。在 VirtualBox 的情况下，建立云环境（如 OCI）中可以迁移的虚拟机之前，只需要完成一些预先工作。我们将在这个步骤中详细介绍这些先决条件。以下图解释了配置、安装和迁移 VM 到云服务提供商的阶段。

![图 13.26 – 虚拟机配置阶段](img/B18349_13_026.jpg)

图 13.26 – 虚拟机配置阶段

## 准备工作

准备操作系统与建房子前的基础设置没有什么不同。我发现最好在 VirtualBox 用户界面配置操作系统，然后在网络设置并建立通信后使用`ssh`，这意味着你可以连接到虚拟机。

在准备将 VM 镜像导入 OCI 作为镜像时，有五个阶段。如下所示：

1.  **接口名称**：这涉及标准化，以确保接口名称被主机环境识别。这可以转化为主机系统如何实现网络连接。

1.  **接口配置**：在这里，我们确认没有显示 MAC 地址，及其他预配置任务。重复的 MAC 地址会导致额外的复杂性，这使得它在配置中成为一个关键点。

1.  **GRUB(2)配置**：这涉及准备内核。

1.  **串口控制台配置**：这是导入和使用过程中最常见的错误之一。没有这个配置，OCI 环境将在导入时失败，或者虚拟机无法启动。

1.  **重启操作系统**；即重新启动你的机器。

以下图示展示了操作系统从启动到重启的配置阶段：

![图 13.27 – 操作系统配置阶段](img/B18349_13_027.jpg)

图 13.27 – 操作系统配置阶段

注意更改后的接口名称：

```
[root@small-ol8 ~]# dmesg | grep -i eth
[    4.702620] e1000 0000:00:03.0 eth0: (PCI:33MHz:32-bit) 08:00:27:8d:0f:64
[    4.702628] e1000 0000:00:03.0 eth0: Intel(R) PRO/1000 Network Connection
[    5.371103] e1000 0000:00:08.0 eth1: (PCI:33MHz:32-bit) 08:00:27:89:87:98
[    5.371132] e1000 0000:00:08.0 eth1: Intel(R) PRO/1000 Network Connection
[    5.373296] e1000 0000:00:08.0 enp0s8: renamed from eth1
[    5.374971] e1000 0000:00:03.0 enp0s3: renamed from eth0
```

我们需要在内核上执行以下代码块中显示的更改，重命名接口以进行匹配。这将需要 GRUB(2)，因为 GRUB 是内核和操作系统之间的中介。现在我们有了当前的接口名称，那么接下来怎么办呢？我们这里做的事情是将接口重命名为更合适的名称，如`net0`或`Eth0`。因此，我们需要移动（`mv`）或重命名接口文件名：

```
[root@small-ol8 network-scripts]# mv /etc/sysconfig/network-scripts/ifcfg-enp0s3 /etc/sysconfig/network-scripts/ifcfg-net0
[root@small-ol8 network-scripts]# mv /etc/sysconfig/network-scripts/ifcfg-enp0s8 /etc/sysconfig/network-scripts/ifcfg-net1
```

重命名接口后，我们需要在配置文件中更改相应的条目以匹配文件名。在以下代码中，我们重命名了`net0`的配置：

```
[root@small-ol8 network-scripts]# vi /etc/sysconfig/network-scripts/ifcfg-net0
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=enp0s3
UUID=e9e58f08-3fcc-4a43-b605-797a6bc9998f
DEVICE=enp0s3
ONBOOT=yes
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=net0
#UUID=e9e58f08-3fcc-4a43-b605-797a6bc9998f
DEVICE=net0
ONBOOT=yes
```

我们在这里做的与之前相同，不过这次我们更改的是`net1`：

```
[root@small-ol8 network-scripts]# vi /etc/sysconfig/network-scripts/ifcfg-net1
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=enp0s8
UUID=eb2e5ed7-ec13-4f76-a0bf-48e7bcf45e84
DEVICE=enp0s8
ONBOOT=yes
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=eui64
NAME=net1
#UUID=eb2e5ed7-ec13-4f76-a0bf-48e7bcf45e84
DEVICE=net1
ONBOOT=yes
```

在下面的代码块中，我们使用`ip a`来识别接口，并确保我们的更改已生效：

```
[root@small-ol8 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: net0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:8d:0f:64 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.5/24 brd 10.0.2.255 scope global dynamic noprefixroute net0
       valid_lft 459sec preferred_lft 459sec
    inet6 fe80::a00:27ff:fe8d:f64/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: net1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:89:87:98 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.5/24 brd 192.168.56.255 scope global dynamic noprefixroute net0
       valid_lft 85059sec preferred_lft 85059sec
    inet6 fe80::a00:27ff:fe89:8798/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 52:54:00:9a:de:c3 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
[root@small-ol8 ~]#
```

GRUB 配置有什么作用？它改变了操作系统的启动方式和加载内核的方式。例如，我们来看看改变串口连接详情的作用。（我们有点跑题了；不过，了解 OCI 在最坏情况下的工作方式是很重要的。）假设你无法连接到你的服务器，网络中断了，但服务器却是绿色的并且在线。你可以使用串口连接作为后门。以下`console`编辑指示操作系统允许与虚拟机的带外连接：

```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true
~
~
~
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL="serial console"
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="console=tty1 console=ttyS0,115200"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true
GRUB_SERIAL_COMMAND="serial --unit=0 –speed=115200"
~
~
~
```

以下命令重新加载 GRUB。由于我们不能直接编辑 GRUB，所以通过配置文件来编辑它，并使用此文件执行 GRUB 上的修改。这告诉操作系统我们已准备好重新加载并输入了新的数据：

```
[root@small-ol8 ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
```

由于我们编辑了 GRUB 并进行了其他一些操作系统更改，重新开始是个不错的选择：

```
[root@small-ol8 network-scripts]# reboot
```

完成这些配置更改可能会让人感到困惑。归根结底，我们是在缩小 OCI 创建的实例和我们在此食谱中配置的实例之间的差异。GRUB 更改是为了使环境准备好在 OCI 租户中运行。

你需要在主机机器上配置几个步骤，以便实现 VirtualBox 和 OCI 之间的连接。VirtualBox 与 OCI 之间的通信通过**应用程序编程接口**（**API**）实现。此连接需要创建公钥和私钥，并将公钥上传到 OCI。

VirtualBox 和 OCI 之间的身份验证过程包含多个步骤。以下是组件及其 OCID/输出，它们是与 OCI 连接所必需的。

+   OCI 账户：

    +   租户

    +   分区

    +   VCN

    +   存储桶

    +   API 密钥

+   VirtualBox —— 确保创建并保存所需的密钥，如下所示：

    +   API 密钥

+   主机操作系统（你的计算机）——确保创建并保存所需的密钥，如下所示：

    +   API 密钥

私钥和公钥是从一个环境共享信息到另一个环境的关键信息。公钥可以共享，并用作加密方法来保护数据包。私钥保留在本地，不会被共享，仅由该用户使用，用于解密数据包。当公钥和私钥一起使用时，它们形成一个锁箱，在传输过程中保护数据包在两个系统之间流动的安全性。`openssl` 被用来生成公钥和私钥。

![图 13.28 – OpenSSL 密钥创建](img/B18349_13_028.jpg)

图 13.28 – OpenSSL 密钥创建

在创建上述密钥后，你需要更改权限，使得该密钥只能由你使用。我们将使用`chmod`来配置这些权限，允许该密钥由用户使用。

![图 13.29 – OpenSSL 密钥权限](img/B18349_13_029.jpg)

图 13.29 – OpenSSL 密钥权限

在创建私钥之后，我们需要创建公钥。这个公钥将上传到 OCI，以便 VirtualBox 和 OCI 进行通信。我们使用私钥来创建公钥，从而使两个环境能够互相通信。

![图 13.30 – 公钥创建](img/B18349_13_030.jpg)

图 13.30 – 公钥创建

下一步是从 OCI 云账户中收集必要的数据，以便输入到 VirtualBox 配置中。所输入的信息将帮助 VirtualBox 通过 OCI API 集成你的 OCI 租户和分区，通过你的 VCN。可以将其想象为在 VirtualBox 和 OCI 之间架设一条高速公路。公钥和私钥就像是这条公路的钥匙，允许数据包在高速公路上穿行。

## 如何操作……

以下步骤将介绍如何上传密钥并配置 VirtualBox 与 OCI 的通信：

1.  首先，就像任何其他云服务提供商（CSP）一样，我们需要登录到我们的账户。

![图 13.31 – 登录到你的账户](img/B18349_13_031.jpg)

图 13.31 – 登录到你的账户

现在我们就等着 OCI 登录我们了……

![图 13.32 – 登录中](img/B18349_13_032.jpg)

图 13.32 – 登录中

1.  登录到你的账户后，你就准备好登录到租户了。这是操作的起点，你将开始看到自己的努力得到回报。我们从识别租户开始，然后继续进行。

![图 13.33 – 选择并登录以验证身份提供者](img/B18349_13_033.jpg)

图 13.33 – 选择并登录以验证身份提供者

1.  一旦登录并进入租户，你需要的大部分数据将会出现在用户设置中，并且与公钥上传相关联。密钥本身会生成一个指纹，这个指纹是 OCI 与 VirtualBox 之间的通道。

    你将在 OCI 门户的右侧看到你的用户图标。点击它将带你进入包含账户、租户和个人数据的区域。这里也是你配置密钥访问权限的地方。

![图 13.34 – OCI 配置文件](img/B18349_13_034.jpg)

图 13.34 – OCI 配置文件

展开该菜单后，你将看到**用户设置**。这是你访问 OCI 的所有用户设置的入口。

![图 13.35 – 用户设置](img/B18349_13_035.jpg)

图 13.35 – 用户设置

1.  **API 密钥**部分将允许你输入用于访问的密钥值。

![图 13.36 – 管理员权限和 API 密钥](img/B18349_13_036.jpg)

图 13.36 – 管理员权限和 API 密钥

1.  在下面的截图中，你可以看到所有当前上传到 OCI 的密钥，显示的是你的特定用户。如果点击**添加 API 密钥**，将会弹出添加密钥的选项。

![图 13.37 – API 密钥](img/B18349_13_037.jpg)

图 13.37 – API 密钥

1.  选择**选择公钥文件**，然后从本地环境中选择文件。记得我们在上一节中创建了一个公钥。我们将在这里选择那个公钥。

![图 13.38 – 公钥](img/B18349_13_038.jpg)

图 13.38 – 公钥

1.  一旦选择了密钥并将其上传到 OCI，你将在屏幕上看到该密钥。点击**添加**：

![图 13.39 – 已提交的已添加密钥](img/B18349_13_039.jpg)

图 13.39 – 已提交的已添加密钥

1.  现在我们已经完成了密钥导入，我们来到了可以开始配置访问权限的阶段。在你正式添加密钥后，将会显示一个配置界面。这个配置文件就是你所需要的一切，除了一个我们稍后会讨论的**分区 OCID**，用于配置 VirtualBox/OCI 接口。

![图 13.40 – 显示的配置](img/B18349_13_040.jpg)

图 13.40 – 显示的配置

复制**配置文件预览**下的信息。你将需要这些信息来配置 VirtualBox 与 OCI 的通信。

1.  现在我们回到 VirtualBox。接近图钉图标的地方会有一个菜单栏。这个菜单是 VirtualBox 的核心，允许您更改从 vdisk（如以下截图所示）到网络配置，当然也包括与云相关的设置。展开 **Cloud** 菜单项以访问 OCI 设置：

![图 13.41 – VirtualBox OCI 云配置设置](img/B18349_13_041.jpg)

图 13.41 – VirtualBox OCI 云配置设置

1.  选择此选项后，您需要输入一个名称。我将输入 `OCI`，因为我们正在使用 OCI。

![图 13.42 – 配置名称](img/B18349_13_042.jpg)

图 13.42 – 配置名称

1.  现在，在 **Oracle Cloud Infrastructure** 下，您会看到您的新配置文件 **OCI**。您可以点击 **属性**，这将允许您配置该 OCI 配置文件。

![图 13.43 – 名称已提交](img/B18349_13_043.jpg)

图 13.43 – 名称已提交

1.  这时，打开另一个窗口中的 OCI 门户，或者将变量复制粘贴到记事本中会非常方便。您需要我们在几节前的 *图 13.40* 中讲解的所有信息。

1.  您需要复制的第一个 OCID 是 `user` 的：

![图 13.44 – 用户 OCID](img/B18349_13_044.jpg)

图 13.44 – 用户 OCID

1.  将该 OCID 粘贴到 OCI 配置文件中的 **user** 属性中，如前面的截图所示，再粘贴到 VirtualBox 中。希望您现在能明白我们要做什么。

![图 13.45 – 配置用户 OCID](img/B18349_13_045.jpg)

图 13.45 – 配置用户 OCID

1.  如之前所示，这次复制 `tenancy` 的 OCID 并返回到 VirtualBox。

![图 13.46 – 租户 OCID](img/B18349_13_046.jpg)

图 13.46 – 租户 OCID

1.  再次，如之前所示，将租户 OCID 粘贴到 **tenancy** 字段中。

![图 13.47 – 配置租户 OCID](img/B18349_13_047.jpg)

图 13.47 – 配置租户 OCID

1.  返回到您的记事本或 OCI 门户并复制 `region` 值，在本例中是 `us-asburn-1`，如下截图所示。记住，所有这些很快就会变得清晰。基本上，我们正在指示 VirtualBox 去哪里，如何到达那里，并使用哪些区域、租户、用户和密钥来进行访问。

![图 13.48 – 区域](img/B18349_13_048.jpg)

图 13.48 – 区域

1.  如之前所示，将区域粘贴到正确的字段中。

![图 13.49 – 配置区域](img/B18349_13_049.jpg)

图 13.49 – 配置区域

1.  接下来，您需要进入 OCI 控制台或记事本，并复制 `fingerprint` 值以输入到 VirtualBox 中。

![图 13.50 – Fingerprint](img/B18349_13_050.jpg)

图 13.50 – Fingerprint

1.  再次将该 fingerprint 值粘贴到相应字段中。

![图 13.51 – 配置 fingerprint](img/B18349_13_051.jpg)

图 13.51 – 配置 fingerprint

1.  最后一个条目是你创建的私钥的位置。这个位置不在 OCI 中，而是在你本地系统中创建它的地方。记住，私钥是本地密钥，不应与他人共享。你需要记录该路径并返回 VirtualBox。

![图 13.52 – API 密钥位置](img/B18349_13_052.jpg)

图 13.52 – API 密钥位置

1.  如前所示，将路径粘贴到相应的字段中。

![图 13.53 – 配置密钥位置](img/B18349_13_053.jpg)

图 13.53 – 配置密钥位置

1.  最后，你需要回到 OCI 门户并找到 **隔离区** 菜单。我发现简单搜索是定位此选项的最简便方法，但如果你想直接前往，可以在 **身份** 下找到它。

![图 13.54 – 隔离区](img/B18349_13_054.jpg)

图 13.54 – 隔离区

1.  展开你想使用的隔离区并复制 OCID，将其输入到 VirtualBox OCI 配置中。

![图 13.55 – 隔离区选择](img/B18349_13_055.jpg)

图 13.55 – 隔离区选择

1.  如前所示，将该 OCID 粘贴到相应字段中。

![图 13.56 – 配置隔离区](img/B18349_13_056.jpg)

图 13.56 – 配置隔离区

1.  在输入所有数据后，你应该能看到 OCID 的区域、用户、租户、密钥文件位置、隔离区和指纹属性都已填充。

![图 13.57 – 完成的配置](img/B18349_13_057.jpg)

图 13.57 – 完成的配置

1.  一旦确认一切已完成，应用设置。你会看到新配置文件出现；我们将其命名为 **OCI**。

![图 13.58 – VirtualBox OCI](img/B18349_13_058.jpg)

图 13.58 – VirtualBox OCI

1.  如果你展开该配置文件，你会看到租户现在是空的。如果你之前在该租户下部署了基础设施，你会看到那些资源显示在这里，前提是它们运行在与配置文件中输入的 OCID 相同的隔离区。

![图 13.59 – 扩展的 OCI](img/B18349_13_059.jpg)

图 13.59 – 扩展的 OCI

现在你已经准备好开始移动我们在本节中创建的虚拟机：

1.  导出虚拟机到 OCI 的第一步，一旦配置好 VirtualBox OCI 配置文件，就是在 VirtualBox 主欢迎菜单中点击 **导出**。

![图 13.60 – 导出](img/B18349_13_060.jpg)

图 13.60 – 导出

1.  接下来，你需要选择要导出到 OCI 的虚拟机，如下截图所示——在此例中，虚拟机名称为 **small-ol8.8**。这将根据你为虚拟机命名的不同而有所变化。

![图 13.61 – 镜像选择](img/B18349_13_061.jpg)

图 13.61 – 镜像选择

1.  选择虚拟机后，你必须配置导出虚拟机的格式。我们将扩展 `.ova` 文件扩展名。

![图 13.62 – 镜像格式配置](img/B18349_13_062.jpg)

图 13.62 – 镜像格式配置

1.  如前所示，选择**Oracle Cloud Infrastructure**作为格式，选择**OCI**作为 VirtualBox 配置文件，正如下面截图中的红色高亮所示。

![图 13.63 – 配置文件](img/B18349_13_063.jpg)

图 13.63 – 配置文件

1.  你需要确保所有变量都是正确的，如以下截图所示，并确认你有一个可用的存储桶：

![图 13.64 – 镜像输出](img/B18349_13_064.jpg)

图 13.64 – 镜像输出

1.  你需要展开`VM.Optimized.Flex`，它不会被 OCI 识别。我在以下图中识别了一个可用且轻量的形状，它是可用的。请按照以下图片选择形状：

![图 13.65 – 形状选择](img/B18349_13_065.jpg)

图 13.65 – 形状选择

1.  如前所示，展开此部分将为你提供多个形状选择。需要注意的是，形状是预设的大小，反映了一个固定数量的资源。在这种情况下，我们选择**VM.Standard.E2.1**：

![图 13.66 – 镜像形状](img/B18349_13_066.jpg)

图 13.66 – 镜像形状

注意

`VM.Optimized3.Flex` 不是一个被认可的形状，无法导入。你可以在虚拟机部署后随时更改形状。

1.  一旦选择了合适的形状，完成配置导出，然后点击**完成**以执行迁移到 OCI。

![图 13.67 – 镜像形状提交](img/B18349_13_067.jpg)

图 13.67 – 镜像形状提交

1.  确保你的镜像形状已经识别并且选择了正确的形状：

![图 13.68 – 镜像写入](img/B18349_13_068.jpg)

图 13.68 – 镜像写入

1.  一旦导出完成，你将看到你的虚拟机在 OCI 中运行，并且你可以登录到你的门户，看到该虚拟机在你的 OCI 租户中运行。

![图 13.69 – 镜像运行](img/B18349_13_069.jpg)

图 13.69 – 镜像运行

如前面的截图所示，镜像现在作为虚拟机在 OCI 中运行。

# 对于其他人来说，让我们为 AWS 创建自定义镜像

看起来非常简单：构建镜像，导出镜像，然后导入镜像（洗净，冲洗，再重复）。这个概念并不比这更复杂。然而，还是有一些步骤需要走，而且其中有些步骤是有陷阱的。

你首先需要完成设置和配置 AWS CLI 的步骤。类似于 OCI CLI，AWS 的 CLI 是一个直接通过命令行执行 AWS API 的工具。安装并配置 CLI 后，我们将通过设置 S3 存储桶、配置存储桶以及更改访问权限以允许上传的步骤。下图展示了本教程中各个阶段及其触点。AWS CLI 是在分阶段和执行过程中与图形用户界面共同使用的核心工具。AWS 中的存储由访问控制和身份验证管理，这使得我们能够创建、存储和操作存储的对象和存储桶。

![图 13.70 – AWS CLI 存储连接](img/B18349_13_070.jpg)

图 13.70 – AWS CLI 存储连接

## 准备就绪

在开始使用 CLI 之前，我们必须下载并安装必要的包。对于本教程，我使用的是 Mac。你可以轻松地访问 AWS 并下载适用于 Windows 或 Linux 的 CLI。不管怎样，一旦 CLI 安装完成，命令的使用没有区别，唯一不同的是位置和一些操作系统特定的更改，这些更改仅适用于 Windows：

```
root# curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
```

下载完包后，你需要安装 CLI。执行安装包时，如下所示：

```
root# installer -pkg ./AWSCLIV2.pkg -target /
```

在这种情况下，我们将 Python 版本升级到与 AWS CLI 所要求的版本匹配。你可以通过在终端中运行`which`命令来检查 AWS CLI 的安装位置，再使用`--version`命令查看 AWS CLI 和 Python 的版本：

```
root# which aws
/usr/local/bin/aws
root# aws --version
aws-cli/2.7.28 Python/3.9.11 Darwin/21.6.0 exe/x86_64
```

安装好 AWS CLI 之后，我们将继续配置 Oracle Linux 操作系统。接下来，我们会用我们的 AWS 账户信息和数据中心位置来配置 AWS CLI：

![图 13.71 – 创建一个新的虚拟机](img/B18349_13_077.jpg)

图 13.71 – 创建一个新的虚拟机

你需要像前面章节那样填写所有字段。确保通过引用你创建的磁盘（`/Users/USER/Desktop/<NAME>`）来命名你的虚拟机。这样可以帮助你稍后保持文件的整洁。你可以随时返回 VirtualBox 中的虚拟机并找到关联的介质，但现在命名磁盘会更容易。

![图 13.72 – ISO 选择](img/B18349_13_078.jpg)

图 13.72 – ISO 选择

如果你记得，在前一部分中，我们已分配了 CPU 和内存。就像在其他任何构建环境中一样，我们需要指定、安装和配置资源。无论它们是虚拟的还是物理的，我们都在安装资源。在这种情况下，我们为这台虚拟机配置了 4 GB 的 RAM 和两个 vCPU。记住，vCPU 相当于一个处理器线程。所以，这里我们安装了两个 vCPU，相当于一个物理核心（两个线程（vCPU）=一个处理器核心）。

![图 13.73 – 资源](img/B18349_13_079.jpg)

图 13.73 – 资源

配置完 CPU 和内存资源后，我们准备选择要附加到虚拟机的介质（磁盘）。在这种情况下，我们不会选择默认的创建磁盘选项，而是使用现有的磁盘。记得，我们已经创建了自己的`.vmdk`格式磁盘文件，所以在这里选择使用现有的虚拟硬盘文件。

![图 13.74 – 创建或使用现有磁盘的选项](img/B18349_13_080.jpg)

图 13.74 – 创建或使用现有磁盘的选项

记得选择与你的虚拟机名称匹配的驱动器（假设你之前已经创建了），或者创建一个新的驱动器。注意，`.vmdk`扩展名必须再次使用，才能被识别。

![图 13.75 – 磁盘选择](img/B18349_13_081.jpg)

图 13.75 – 磁盘选择

和之前的步骤一样，我们在 **摘要** 屏幕上确认我们的选择，并通过选择 **完成** 执行配置。我们不会深入讲解构建过程，因为我们有其他教程可以引导你配置操作系统。

![图 13.76 – 配置总结](img/B18349_13_082.jpg)

图 13.76 – 配置总结

一旦我们构建了 Linux 操作系统，就可以通过配置我们的 AWS 环境来执行操作，包括设置 CLI 上传并在创建后执行我们的 AMI（亚马逊机器镜像），从 OVA 文件导入。记住，在前面的步骤中，我们在 VirtualBox 中构建了一个虚拟机，并从该 OVA 镜像创建了一个 OCI。我们在这里也会做同样的事情，从 VirtualBox 导出并导入到 AWS。这里的区别在于，我们没有一个复杂的 VirtualBox 环境可供操作。我们将使用 AWS CLI 将该 OVA 镜像导入到我们的 S3 环境中。然而，如前所述，为了做到这一点，我们需要配置连接并确保其到位。我们将一步步展示如何在 AWS 上进行这些设置，并迁移 VirtualBox 到 AWS，看看这个过程需要哪些步骤。

在导出 OVA 文件时，你需要开始准备 AWS CLI 配置。

## 如何操作…

你可以通过两种方式上传导出的 OVA，一种是通过命令行，另一种是通过 S3 控制台。无论哪种方式，实际上都是将文件上传到 AWS S3，方式并不重要。

通过 S3 控制台上传是一个相当简单的过程：

1.  登录到你的 AWS 控制台，如下所示：

![图 13.77 – 以 root 用户登录](img/B18349_13_083.jpg)

图 13.77 – 以 root 用户登录

1.  登录后，导航到 S3。

![图 13.78 – AWS 菜单](img/B18349_13_084.jpg)

图 13.78 – AWS 菜单

如果 S3 不在你最近访问的列表中，可以在搜索栏中导航到 `S3`。

![图 13.79 – S3 存储](img/B18349_13_085.jpg)

图 13.79 – S3 存储

1.  导航到 S3 后，你需要为上传的 OVA 创建一个存储桶。存储桶本质上是对象存储，是存储图片等内容的理想位置。

![图 13.80 – S3 存储桶](img/B18349_13_086.jpg)

图 13.80 – S3 存储桶

1.  选择 **创建存储桶** 后，你需要为存储桶命名并配置其他方面的内容。

![图 13.81 – 存储桶名称和区域](img/B18349_13_087.jpg)

图 13.81 – 存储桶名称和区域

1.  命名后，根据 AWS 文档选择存储桶应创建的区域。记得在与你操作的区域相同的区域创建存储桶，并且在这个练习中使用该区域。在这个例子中，我们将在美国东部 2 - 俄亥俄地区部署。

1.  我们在这里不会启用任何 ACLs。

![图 13.82 – ACLs](img/B18349_13_088.jpg)

图 13.82 – ACLs

1.  如前所述，关于 AWS 访问控制，我们将取消勾选 **阻止所有公共访问**，并允许直接公共（即外部）访问 S3 存储桶。

注意

这通常不是最佳实践，我们这样做只是为了测试目的。

![图 13.83 – 公共访问](img/B18349_13_089.jpg)

图 13.83 – 公共访问

1.  我们将保持禁用版本管理（默认设置），如以下截图所示，因为这是一个一次性桶，不会永久保留。当您在多个版本的项目生命周期中工作时，您可能会启用版本管理，例如图像的多个版本。

![图 13.84 – 版本管理](img/B18349_13_090.jpg)

图 13.84 – 版本管理

1.  由于我们保持操作的基础性，我们不会关联任何标签。

![图 13.85 – 标签](img/B18349_13_091.jpg)

图 13.85 – 标签

1.  我们将使用标准的服务器端加密，使用 S3 托管的密钥。

![图 13.86 – 加密](img/B18349_13_092.jpg)

图 13.86 – 加密

1.  同样忽略高级选项，直接创建桶。

![图 13.87 – 桶显示](img/B18349_13_093.jpg)

图 13.87 – 桶显示

1.  转到新创建的`vb-aws-us-east-2`桶，点击**上传**。

![图 13.88 – 对象上传](img/B18349_13_094.jpg)

图 13.88 – 对象上传

1.  点击**添加文件**。这将允许您从本地文件系统中选择正确的 OVA 文件上传到桶中。

![图 13.89 – 添加文件](img/B18349_13_095.jpg)

图 13.89 – 添加文件

1.  我们将选择在本教程开始时创建的 OVA。

![图 13.90 – 文件选择](img/B18349_13_096.jpg)

图 13.90 – 文件选择

1.  一旦您添加了文件，进行上传。

![图 13.91 – 文件目标](img/B18349_13_097.jpg)

图 13.91 – 文件目标

1.  状态栏将显示进度，**摘要**部分将显示 S3 中的目标位置，如下图所示。

![图 13.92 – 上传](img/B18349_13_098.jpg)

图 13.92 – 上传

注意

如果您离开此状态页面，您将无法返回，所以请停留在页面上，直到上传完成。

1.  上传完成后——或者，如果您喜欢多任务处理，可以在此期间打开另一个 AWS 控制台会话——您需要为刚创建的桶设置角色和访问权限。转到**身份与访问管理**（**IAM**）以访问角色数据。

1.  在**角色**下，您可以创建自定义角色以用于访问、认证或其他与 IAM 相关的标准。

![图 13.93 – AWS 角色](img/B18349_13_0931.jpg)

图 13.93 – AWS 角色

我们将通过命令行配置角色和权限。我们必须创建一个 IAM 用户和密钥，因为没有访问权限或认证，CLI 将无法操作。然后，我们可以配置 CLI 与我们的账户连接。

要配置并将用户与该访问密钥关联，您需要返回到 IAM，接下来的步骤将引导您完成。

![图 13.94 – 安全、身份与合规菜单](img/B18349_13_100.jpg)

图 13.94 – 安全、身份与合规菜单

1.  一旦你在搜索栏中展开了`IAM`，为了帮助你熟悉控制台，我们将详细说明获取此选项的具体步骤。

![图 13.95 – IAM](img/B18349_13_101.jpg)

图 13.95 – IAM

1.  为了访问和创建密钥，我们必须了解密钥的用途以及它如何与用户相关联。所以，让我们导航到**用户**。我们将按以下步骤创建密钥。

![图 13.96 – 用户](img/B18349_13_102.jpg)

图 13.96 – 用户

1.  你需要按照以下截图所示创建用户，你可以通过进入`user-admin`来做到这一点：

![图 13.97 – 用户名](img/B18349_13_103.jpg)

图 13.97 – 用户名

1.  现在我们已经命名了用户，就像任何其他系统一样，我们需要定义用户，将他们加入一个组，并分配权限。

![图 13.98 – 用户定义](img/B18349_13_104.jpg)

图 13.98 – 用户定义

1.  我们将把此用户分配到管理员组，然后点击**下一步**继续。你可以通过创建一个与您的政策或组织政策更为贴切的组来定义权限。创建用户后，我们可以在此生成一个密钥，正如以下截图所示，附带密钥。以下步骤将演示如何操作。

![图 13.99 – 凭证](img/B18349_13_105.jpg)

图 13.99 – 凭证

1.  **安全凭证**选项卡是创建访问密钥的地方。

![图 13.100 – CLI 访问控制和凭证创建](img/B18349_13_106.jpg)

图 13.100 – CLI 访问控制和凭证创建

这个访问密钥是专门为 CLI 创建的，因此我们需要选择这个选项，如前面的截图所示，然后点击**下一步**。我们在这个步骤中不会创建任何标签，所以可以跳过这一步，直接创建访问密钥。

注意

下载`.csv`文件（凭证文档），或通过点击密钥旁边的两个方框来显示并复制密钥。这是唯一可以复制这些密钥的地方，无论是写下来还是粘贴到记事本中。**一旦你关闭此页面，将无法恢复这些信息。** 你可以重新创建并重新配置 CLI 而不丢失数据，但这会是很多不必要的工作，所以请确保写下密钥。

![图 13.101 – 密钥](img/B18349_13_107.jpg)

图 13.101 – 密钥

确保复制密钥，如下所示——你将看到不同的密钥：`eZ5mZxzGhEYM0AfyBznLDqIT6TMsUz//AO2DMQst`。

1.  一旦你创建了密钥并记录下了秘密访问密钥，访问密钥将变为有效。

![图 13.102 – 访问密钥](img/B18349_13_1021.jpg)

图 13.102 – 访问密钥

1.  在添加了访问密钥后，你将能够在 CLI 中运行`aws configure`：

    ```
    root# aws configure
    AWS Access Key ID [****************AZHA]: AKIA3HX45UYOK7HIAZHA
    AWS Secret Access Key [****************MQst]:
    Default region name [us-east-2]: us-east-2
    Default output format [json]: json
    ```

注意

如果你将`JSON`写成全大写字母，这将导致失败。它不会报告失败，但当你运行命令时，会在 API 中失败。

1.  配置 CLI 后，你可以执行以下命令将你的镜像从 OVA 转换为 AMI：

    ```
    root# aws ec2 import-image --description "VM Image" --disk-containers Format=ova,UserBucket="{S3Bucket=vb-aws-us-east-2,S3Key=vb-aws.ova}"
    {
        "Description": "VM Image",
        "ImportTaskId": "import-ami-0908bf8d72b678cdf",
        "Progress": "1",
        "SnapshotDetails": [
            {
                "DiskImageSize": 0.0,
                "Format": "OVA",
                "UserBucket": {
                    "S3Bucket": "vb-aws-us-east-2",
                    "S3Key": "vb-aws.ova"
                }
            }
        ],
        "Status": "active",
        "StatusMessage": "pending"
    }
    ```

1.  以下命令将确认转换状态：

注意

`import-ami-xxxxxxxx` 每次执行转换时都会变化。

```
aws ec2 describe-import-image-tasks --import-task-ids import-ami-<AMI-ID>
```

我已经执行了这个命令并多次运行，以展示每个过程步骤。完成以下步骤后，你将能够在你的 AWS 租户中使用新创建的 AMI。在执行导入任务时，我刷新了页面以展示所有版本的更新。

1.  运行以下命令查看导入状态：

    ```
    root# aws ec2 describe-import-image-tasks --import-task-ids import-ami-0908bf8d72b678cdf
    {
        "ImportImageTasks": [
            {
                "Description": "VM Image",
                "ImportTaskId": "import-ami-0908bf8d72b678cdf",
                "Progress": "19",
                "SnapshotDetails": [
                    {
                        "DiskImageSize": 8211946496.0,
                        "Format": "VMDK",
                        "Status": "active",
                        "UserBucket": {
                            "S3Bucket": "vb-aws-us-east-2",
                            "S3Key": "vb-aws.ova"
                        }
                    }
                ],
                "Status": "active",
                "StatusMessage": "converting",
                "Tags": []
            }
        ]
    active: The import task has started
    ```

1.  `converting`: 镜像正在转换为 AMI

1.  `updating`: 正在更新 AMI 导入

1.  `validating`: 正在验证导入

1.  `validated`: 验证已完成

1.  `completed`: AMI 准备好使用

1.  `deleted`: 导入任务已取消

当 AMI 完成时，以下将是最终状态：

```
{
    "ImportImageTasks": [
        {
            "Architecture": "x86_64",
            "Description": "VM Image",
            "ImageId": "ami-055ce6a1f65862de6",
            "ImportTaskId": "import-ami-0908bf8d72b678cdf",
            "LicenseType": "BYOL",
            "Platform": "Linux",
            "SnapshotDetails": [
                {
                    "DeviceName": "/dev/sda1",
                    "DiskImageSize": 8211946496.0,
                    "Format": "VMDK",
                    "SnapshotId": "snap-08a2c2a985caf8a6c",
                    "Status": "completed",
                    "UserBucket": {
                        "S3Bucket": "vb-aws-us-east-2",
                        "S3Key": "vb-aws.ova"
                    }
                }
            ],
            "Status": "completed",
            "Tags": [],
            "BootMode": "legacy_bios"
        }
    ]
}
```

1.  处理完成后，你将能够重新登录 AWS 控制台并导航到你的 EC2 环境。为了启动一个新的虚拟机并选择你的 AMI 作为模板，请点击**启动实例**。

![图 13.103 – 启动实例](img/B18349_13_109.jpg)

图 13.103 – 启动实例

1.  一旦进入启动菜单，你可以深入浏览其他可用的 AMI。

![图 13.104 – 浏览 AMI](img/B18349_13_110.jpg)

图 13.104 – 浏览 AMI

1.  这将带你进入自定义的 AMI（如前所示在*图 13.7，镜像产品*中），以及市场。市场是你可以与社区分享自己创建的 AMI，或者使用他人上传的 AMI 的地方。

![图 13.105 – 创建的 AMI](img/B18349_13_111.jpg)

图 13.105 – 创建的 AMI

1.  如前所示，你将看到在本指南中创建的可用 AMI。同时，顺便提一下，除了市场中提供的 AMI，你还可以找到许多操作系统，包括 Oracle Linux。

1.  如果你导航到新创建的 AMI，你会看到你可以选择该镜像作为 AMI 来执行并创建虚拟机。

![图 13.106 – 选择 AMI](img/B18349_13_1061.jpg)

图 13.106 – 选择 AMI

通过选择 AMI，你将该 AMI 附加为镜像，并执行从所选镜像（AMI）创建虚拟机的操作。

![图 13.107 – 实例启动配置](img/B18349_13_113.jpg)

图 13.107 – 实例启动配置

1.  在选择 AMI 并执行启动后，你将进入标准的 AWS 虚拟机创建流程，包括创建密钥对、命名密钥对并下载密钥。你也可以事先创建这些密钥。

![图 13.108 – 密钥对](img/B18349_13_108.jpg)

图 13.108 – 密钥对

1.  一旦创建了密钥对，私钥将下载到你的本地机器上，你可以继续启动虚拟机。

![图 13.109 – 成功状态](img/B18349_13_115.jpg)

图 13.109 – 成功状态

1.  你可以通过点击实例 ID 来导航到你的虚拟机。进入实例后，你将能够捕获 IP 并登录到你的新虚拟机。

![图 13.110 – 公共 IP](img/B18349_13_116.jpg)

图 13.110 – 公共 IP

1.  要登录到机器，你可以使用在创建机器时生成的密钥对或密码；两者都可以使用：

    ```
    root# ssh oracle@3.135.211.36
    oracle@3.135.211.36's password:
    Activate the web console with: systemctl enable --now cockpit.socket
    Last login: Sat Aug 12 01:45:46 2023 from 107.200.172.229
    ```
