# *第十章*：使用 Xen 安全模块与 FLASK

在 *第九章* *安全虚拟化* 中，我们看到 libvirt 能够基于 SELinux 域和类别分配，对几个支持的虚拟机监控器应用 sVirt 保护措施。Xen 作为另一种流行的开源虚拟机监控器，也受 libvirt 支持，但更常见的做法是单独使用 Xen，而不依赖 libvirt。

Xen 本身有一个名为 **Xen 安全模块**（**XSM**）的安全框架，类似于 **Linux 安全模块**（**LSM**），以及一个名为 XSM-FLASK 的访问控制系统，这是基于 SELinux 的安全框架。我们将了解 Xen 如何使用 XSM，如何构建支持 XSM 的 Xen，最后，如何将策略应用于 Xen 域。

在本章中，我们将讨论以下主要主题：

+   理解 Xen 和 XSM

+   运行支持 XSM 的 Xen

+   应用自定义 XSM 策略

# 技术要求

查看以下视频，了解代码演示：[`bit.ly/3kcCePl`](https://bit.ly/3kcCePl)

# 理解 Xen 和 XSM

Xen 项目是一个由 Linux 基金会维护的项目，负责维护 Xen 虚拟机监控器（hypervisor）。虽然 Xen 项目管理多个与安全性和虚拟化相关的软件，但我们的重点是 Xen 虚拟机监控器。

## 介绍 Xen 虚拟机监控器

Xen 虚拟机监控器直接运行在硬件之上，位于各种虚拟机和硬件之间。与在 Linux 中作为进程运行以提供虚拟化功能的 QEMU 或 KVM 不同，Xen 工作得更加独立。因此，管理员不会将运行的实例视为独立的进程。相反，他们需要依赖 Xen 命令和 API 来获取更多信息并与 Xen 虚拟机监控器交互。

重要提示

与 libvirt 类似，Xen 虚拟机监控器使用 *domain* 这个术语来指代其虚拟机。由于我们在 SELinux 中经常使用 *domain* 来表示正在运行的进程的 SELinux 类型，因此也表示正在运行的虚拟机的 SELinux 类型，我们将在可能的情况下使用 *guest*（来宾）一词。然而，也会有一些与 Xen 相关的术语，我们需要保留 *domain* 术语。

Xen 总是至少定义了一个虚拟来宾，称为 `xend`。管理员通过 dom0 来创建和操作运行在 Xen 中的虚拟来宾。这些常规来宾是非特权的，因此简称为 **domU**——**非特权域**。

当管理员启动 Xen 主机时，他们会启动 Xen 的 *dom0* 实例，然后通过该实例与 Xen 进行进一步的交互。Linux 内核已经支持在 *dom0* 和 *domU* 中运行很长时间了（自 Linux 内核 3.0 起，包括完整的支持，如后端驱动程序）。

让我们使用现有的 Linux 部署来安装 Xen，并使用这个现有的部署作为 Xen 的 dom0 来宾。

## 安装 Xen

虽然许多 Linux 发行版开箱即用提供 Xen 支持，但这些发行版很可能不支持 XSM（我们将在*启用 XSM 的 Xen 运行*部分启用它）。因此，在不急于调整预构建的 Xen 环境的情况下，我们希望直接从 Xen 项目发布的源代码开始构建它。

在我们开始使用 Xen 之前，更不用说启用它的 XSM 支持，我们首先需要确保正在运行一个启用 Xen 的 Linux 内核。

### 使用启用 Xen 的 Linux 内核

系统上的 Linux 内核必须支持（至少）在 dom0 客户机中运行。没有此支持，dom0 客户机不仅无法与 Xen 虚拟机监控程序交互，还无法启动 Xen 虚拟机监控程序本身（Xen 启用内核需要在启动自己作为 dom0 客户机之前引导 Xen 虚拟机监控程序）。

如果你构建自己的 Linux 内核，必须根据[`wiki.xenproject.org/wiki/Mainline_Linux_Kernel_Configs`](https://wiki.xenproject.org/wiki/Mainline_Linux_Kernel_Configs)中的文档配置内核。一些 Linux 发行版提供了更深入的构建说明（如 Gentoo 的[`wiki.gentoo.org/wiki/Xen`](https://wiki.gentoo.org/wiki/Xen)）。然而，在 CentOS 中，当前的最后版本缺少开箱即用的 Xen 支持（因为 CentOS 更专注于 libvirt 及其相关技术的虚拟化支持）。

幸运的是，社区提供了维护良好的 Linux 内核版本，这些版本包括 Xen 支持，通过`kernel-ml`包提供。让我们安装这个内核包：

1.  启用**企业 Linux 仓库**（**ELRepo**），该仓库引入了其他几个社区驱动的仓库：

    ```
    # yum install elrepo-release
    ```

1.  安装`kernel-ml`包，它将安装最新的 Linux 内核，并带有包括 Xen 支持的配置。我们同时启用`elrepo-kernel`仓库，该仓库提供了这个包：

    ```
    # yum install --enablerepo=elrepo-kernel kernel-ml
    ```

1.  通常，Linux 启动加载程序会重新配置以包含这些新内核。如果没有，或者你想确保内核被正确检测，可以使用以下命令重新生成**统一引导加载程序**（**GRUB2**）配置文件：

    ```
    # grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

    当然，如果你的系统使用的是不同的启动加载程序，则需要遵循不同的说明。请参考你的 Linux 发行版文档，了解如何配置启动加载程序。

1.  使用新安装的内核重启系统：

    ```
    # reboot
    ```

如果一切顺利，你现在应该在运行一个兼容 Xen 的内核。当然，这并不意味着 Xen 已启用，而只是表示如果需要，内核能够支持 Xen。接下来，让我们继续构建 Xen 虚拟机监控程序及相关工具。

### 从源代码构建 Xen

Xen 虚拟机监控程序和工具依赖于各种程序和库，而在从源代码构建 Xen 时，并非所有工具和库都能正确检测为依赖项。

让我们首先安装这些依赖项：

1.  启用`PowerTools`仓库：

    ```
    # dnf config-manger --set-enabled PowerTools
    ```

1.  安装 CentOS 仓库支持的依赖项：

    ```
    # yum install gcc xz-devel python36-devel acpica-tools uuid-devel ncurses-devel glib2-devel pixman-devel yajl yajl-devel zlib-devel transfig pandoc perl-Pod-Html git glibc-devel.i686 patch libuuid-devel
    ```

1.  安装 `dev86` 包。写这篇文档时，CentOS 8 中还没有该包，因此我们使用 CentOS 7 中的版本：

    ```
    # yum install https://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/d/dev86-0.16.21-2.el7.x86_64.rpm
    ```

现在已安装好依赖项，让我们下载最新的 Xen 并进行构建：

1.  访问 [`xenproject.org/downloads/`](https://xenproject.org/downloads/) 并转到最新的 Xen Project 发行版。

1.  在页面底部，下载最新的归档文件。

1.  在系统上解压下载的归档文件：

    ```
    $ tar xvf xen-4.13.1.tar.gz
    ```

1.  进入解压后的归档文件所在的目录：

    ```
    $ cd xen-4.13.1
    ```

1.  配置本地系统的源。在此阶段，不需要传递特定的参数：

    ```
    $ ./configure
    ```

1.  构建 Xen 虚拟机监控器及相关工具：

    ```
    $ make world
    ```

1.  在系统上安装 Xen 虚拟机监控器和工具：

    ```
    # make install
    ```

1.  重新配置引导加载程序。此操作应自动检测 Xen 二进制文件并添加必要的引导加载项：

    ```
    # grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1.  配置系统以支持安装在 `/usr/local/lib` 中的库：

    ```
    # echo "/usr/local/lib" > /etc/ld.so.conf.d/local-xen.conf
    # ldconfig
    ```

1.  为 `/usr/local` 中的子目录创建等价规则，以便正确应用 SELinux 文件上下文：

    ```
    # semanage fcontext -a -e /usr/local/bin /usr/bin
    # semanage fcontext -a -e /usr/local/sbin /usr/sbin
    ```

1.  重新标记 `/usr/local` 中的文件：

    ```
    # restorecon -RvF /usr/local
    ```

1.  这些步骤的结果是 Xen 已经准备好在系统上启动。然而，默认情况下，引导加载程序不会使用启用 Xen 的内核，因此在重启时，选择正确的条目非常重要。该条目的标题将包含 *with Xen hypervisor*：

    ```
    # reboot
    ```

1.  在重启进入启用 Xen 的系统后，我们需要做的就是启动 Xen 守护进程：

    ```
    # systemctl start xencommons
    # systemctl start xendomains
    # systemctl start xendriverdomain
    # systemctl start xen-watchdog
    ```

1.  为了验证一切按预期工作，列出当前正在运行的来宾：

    ```
    Domain-0, which is the guest you just executed the xl list command in.
    ```

1.  通过确保之前启动的守护进程在启动时重新启动，来完成安装：

    ```
    # systemctl enable xencommons
    # systemctl enable xendomains
    # systemctl enable xendriverdomain
    # systemctl enable xen-watchdog
    ```

在我们继续进行 XSM 操作之前，让我们先在 Xen 中创建一个来宾（作为 domU），这样我们可以在稍后的 *使用 XSM 标签* 部分将策略与之关联。

## 创建一个无特权的来宾

当 Xen 虚拟机监控器启用时，我们与 Xen 交互的操作系统被称为 dom0，它是 Xen 支持的（唯一的）特权来宾。其他来宾是无特权的，我们希望通过 XSM 进一步隔离和保护这些来宾之间的交互和它们采取的操作。

首先，我们创建一个简单的无特权来宾，与特权的 dom0 来宾一起运行。在这个例子中，我们使用 Alpine Linux，但您可以轻松地将其替换为其他发行版或操作系统。这个示例将使用 **ParaVirtualized**（**PV**）来宾方式，但 Xen 也支持 **硬件虚拟化机**（**HVM**）来宾：

1.  下载 Alpine Linux 发行版的 ISO 镜像，因为该发行版在低内存消耗和较低的（虚拟）磁盘大小要求方面更为优化。当然，如果您的系统能够处理，您也可以选择其他发行版。我们选择从 [`www.alpinelinux.org/downloads/`](https://www.alpinelinux.org/downloads/) 下载适用于虚拟系统的优化版，并将 ISO 存储在系统的 `/srv/data` 中。

1.  将 ISO 挂载到系统上，以便在接下来的步骤中创建一个没有特权的客体时使用它的可引导内核：

    ```
    # mount -o loop -t iso9660 /srv/data/alpine-virt-3.8.0-x86_64.iso /media/cdrom
    ```

1.  创建一个映像文件，该文件将用作虚拟客体的启动磁盘：

    ```
    # dd if=/dev/zero of=/srv/data/a1.img bs=1M count=3000
    ```

1.  接下来，为虚拟客体创建一个配置文件。我们将文件命名为 `a1.cfg` 并将其放置在 `/etc/xen` 目录下：

    ```
    # Alpine Linux PV DomU
    # Kernel paths for install
    kernel = "/media/cdrom/boot/vmlinuz-virt"
    ramdisk = "/media/cdrom/boot/initramfs-virt"
    extra = "modules=loop,squashfs console=hvc0"
    # Path to HDD and ISO file
    disk = [
      'format=raw, vdev=xvda, access=w, target=/srv/data/a1.img',
      'format=raw, vdev=xvdc, access=r, devtype=cdrom, target=/srv/data/alpine-virt-3.8.0-x86_64.iso'
    ]
    # DomU settings
    memory = 512
    name = "alpine-a1"
    vcpus = 1
    maxvcpus = 1
    ```

1.  使用 `xl create` 命令启动虚拟客体：

    ```
    -c option will immediately show the console to interact with, allowing you to initiate and complete the installation of the operating system in the guest. 
    ```

1.  当客体需要重启时，使用 shutdown 命令，并编辑配置文件。删除与 ISO 相关的行，防止客体再次启动到安装环境。

1.  要再次启动客体，请再次使用 `xl create` 命令。如果客体安装完成且不再需要访问控制台，请删除 `-c` 选项：

    ```
    # xl create -f /etc/xen/xa1.cfg
    ```

1.  我们可以通过 `xl list` 确认虚拟客体是否正在运行：

    ```
    # xl list
    Name           ID     Mem VCPUs       State      Time(s)
    Domain-0        0    7836     4      r-----         99.4
    alpina-a1       1     128     1      -b----          2.5
    ```

在 Xen 中，客体通过 `create` 子命令启动，并通过 `shutdown`（优雅关机）或 `destroy` 子命令关闭。

完成这些步骤后，我们现在拥有一个正在运行的 Xen 安装和一个运行中的客体。接下来，是时候了解 Xen 在安全方面能为我们提供什么了。

## 了解 Xen 安全模块

在 *第一章*，*基础的 SELinux 概念* 中，我们了解到 SELinux 是通过一个名为 **Linux 安全模块**（**LSM**）的 Linux 子系统来实现的。Xen 借用了这一思想，并采用了类似的安全措施。

通过 **Xen 安全模块**（**XSM**），Xen 使得可以定义和控制 Xen 客体之间、以及 Xen 客体和 Xen 虚拟机监控程序之间的行为。与 Linux 内核不同，虽然 Linux 内核中有多个强制访问控制框架可以插入到 LSM 子系统中，但 Xen 当前仅提供一个可用的 XSM 模块，名为 **XSM-FLASK**。

**FLASK** 代表 **Flux Advanced Security Kernel**，它是 SELinux 用于自己访问控制表达式的安全架构和方法。通过 XSM-FLASK，开发人员和管理员可以执行以下操作：

+   在客体之间定义权限和细粒度的访问控制

+   为原本没有特权的客体定义有限的特权提升

+   在策略级别上控制客体的直接硬件和设备访问

+   限制并审计由特权客体执行的活动

虽然 XSM-FLASK 使用类似 SELinux 的命名约定（甚至使用 SELinux 构建工具来构建策略），但与 XSM-FLASK 相关的设置是独立于 SELinux 的。如果 dom0 启用了 SELinux（没有理由不启用），其策略与 XSM-FLASK 策略无关。

XSM-FLASK 使用的标签也不会显示在来宾内部运行的常规 Linux 命令中（因此也包括 dom0）。由于运行中的来宾不会作为系统中的进程显示，它们根本没有 SELinux 标签，只有 XSM-FLASK 标签（如果启用）。因此，Xen 无法从 sVirt 方法中受益，如*第九章*《安全虚拟化》中所述。

# 运行启用 XSM 的 Xen

从常规的 Xen 部署切换到启用 XSM 的 Xen 部署，只需重新构建 Xen 并启用 XSM 支持，然后重启系统。Xen 自带一个现成的策略，可以直接应用，我们将在 XSM 过程中使用它。

## 重新构建带有 XSM 支持的 Xen

让我们在系统上重新构建带有 XSM 支持的 Xen 虚拟机监控程序和工具：

1.  通过在`build`目录（我们的示例是`xen-4.13.1`）内运行`make clean`命令清理之前的构建：

    ```
    $ make clean
    ```

1.  在`build`目录内，进入`xen`目录：

    ```
    $ cd xen
    ```

1.  使用`make menuconfig`启动 Xen 配置：

    ```
    $ make menuconfig
    ```

1.  进入 XSM 设置并启用与 XSM 相关的参数：

    ```
    Common Features --->
      [*] Xen Security Modules support
      [*]   FLux Advanced Security Kernel support
    [*]     Compile Xen with a built-in FLAS security 
              policy
      [*]   SILO support
    Default XSM implementation (FLux Advanced 
            Security Kernel)
    ```

1.  返回主构建目录（我们的示例中是`xen-4.13.1`）：

    ```
    $ cd ..
    ```

1.  重新构建 Xen 虚拟机监控程序和工具：

    ```
    $ ./configure
    $ make world
    ```

1.  将更新的 Xen 构建安装到系统中：

    ```
    /boot (named xenpolicy-4.13.1).
    ```

1.  使用新的 Xen 构建重新配置引导加载程序，确保 XSM 策略也与之一起加载：

    ```
    # grub2-mkconfig -o /boot/grub2/grub.cfg
    ```

1.  重启系统：

    ```
    # reboot
    ```

1.  一旦重新启动，我们可以通过查询 Xen 以获取与运行中的来宾相关的标签，来验证 XSM 策略是否已加载并生效：

    ```
    # xl list -Z
    Name               ID   ...     Security Label
    Domain-0            0   ...   system_u:system_r:dom0_t
    alpina-a1           1   ...   system_u:system_r:domU_t
    ```

如果`xl list`命令在给定`-Z`参数时列出安全标签，那么 Xen 正在以 XSM 策略运行。我们来看这些标签的使用位置。

## 使用 XSM 标签

当 Xen 以 XSM 支持启动并且默认策略已启用时，来宾可以使用以下类型：

+   `dom0_t` 类型保留给特权来宾使用。

+   `domU_t` 类型是用于非特权来宾的默认类型。

+   `isolated_domU_t` 类型用于分配给不应与其他非特权来宾交互的非特权来宾，仅能与特权 dom0 进行交互。

+   `prot_domU_t` 类型用于那些如果 XSM 策略布尔值`prot_doms_locked`被设置时将被阻止启动的来宾。

+   `nomigrate_t` 类型应用于不允许从一个 Xen 主机迁移到另一个 Xen 主机的来宾。内部机制防止 dom0 来宾在启动后访问该来宾的内存。

XSM 策略中还有一些其他类型不可用于常规来宾：

+   `dm_dom_t` 类型分配给设备模型来宾。这是一个特殊的特权来宾，表示为 HVM 类型来宾虚拟化的硬件，而不危及 dom0。

+   `xenstore_t` 类型分配给`xenstore`存根来宾。这是一个特殊的特权来宾，支持非特权来宾访问其虚拟化资源，而不危及 dom0。

+   `nic_dev_t`类型分配给可以在直通模式下使用的硬件设备（这意味着 domU 客户机可以直接与这些硬件设备交互，而无需通过特权客户机）。

这些占位域（在 Xen 中被称为**占位域**或**stubdoms**）是 Xen 进一步增强安全性的方式，因为无法被阻止的特权操作与 dom0 的隔离性增强。如果在任何时候这些特权服务中的安全漏洞被利用，它们不一定会影响 dom0，并且通过适当的 XSM 策略，甚至可以完全缓解。

将其中一个标签分配给客户机，只需编辑客户机在`/etc/xen`中的配置文件，并添加`seclabel`配置参数：

```
seclabel = 'system_u:system_r:isolated_domU_t'
```

一旦配置并重启（使用`xl create`），新标签将在查询运行中的客户机时可见：

```
# xl list -Z
Name               ID   ...   Security Label
Domain-0            0   ... system_u:system_r:dom0_t
alpina-a1           1   ... system_u:system_r:isolated_domU_t
```

为客户机应用正确的标签是最常见的用例（因为它有效地处理了我们期望从 XSM 实现中获得的访问控制和保护措施），但也支持其他操作。

## 操作 XSM

与 SELinux 一样，可以执行几项活动来进一步操作 XSM 子系统或当前活动策略。

### 定义状态，范围从禁用到强制执行

当 Xen 启动时，我们可以添加一个名为`flask`的内核参数，可以设置为以下值之一：

+   当`flask=enforcing`时，确保 XSM 处于活动状态，强制执行其客体与资源之间的策略，并且强制执行是即时的（没有延迟激活）。

+   当`flask=permissive`时，XSM 将加载策略，但不会强制执行策略中设定的规则。这显然是为了开发目的，行为类似于 SELinux 的宽容模式。

+   当`flask=late`时，XSM 在加载策略之前不会强制执行任何访问控制，加载策略后才会开始强制执行。这允许管理员启动时启用 XSM，但只有在管理员认为准备好时，才加载并应用策略。

+   当`flask=disabled`时，XSM 不会强制执行任何访问控制，也不会加载策略。

该参数可以在启动时直接设置（从引导加载程序）或通过系统中的引导加载程序配置进行设置。例如，使用 GRUB2 时，我们可以编辑`/etc/default/grub`并添加或修改以下参数：

```
GRUB_CMDLINE_XEN_DEFAULT="flask=enforcing"
```

别忘了重新生成 GRUB2 配置文件：

```
# grub2-mkconfig -o /boot/grub2/grub.cfg
```

与 SELinux 一样，我们也可以通过命令行操作 XSM 的状态。使用`xl getenforce`，我们可以查询当前状态：

```
# xl getenforce
Enforcing
```

可以使用`xl setenforce`命令切换到另一种状态：

```
# xl setenforce permissive
```

这些命令与 dom0 中的 SELinux 配置无关：将 Xen 从宽容模式切换到强制模式或反之，特定于 Xen，不会影响 dom0 中的 SELinux 设置。

### 查询 XSM 日志

像 SELinux 一样，XSM 也使用 AVC 日志记录来向管理员提供关于它所做决定的反馈。使用`xl dmesg`，我们可以查询这些日志信息（以及其他 Xen 输出日志）：

```
# xl dmesg
...
(XEN) avc: granted { setenforce } for domid=0 scontext=system_u:system_r:dom0_t tcontext=system_u:system_r:security_t tclass=security
```

并非所有已授权的操作都会被记录，但被拒绝的操作总会导致 AVC 条目的生成。这些 AVC 条目本身格式与 SELinux 的 AVC 条目完全相同，使得管理员可以使用 SELinux 工具，如`audit2allow`，来生成 XSM 策略。

### 使用 XSM 布尔值

Xen 启用的默认策略有两个布尔值可以切换：

+   `guest_writeconsole`布尔值，默认为 1（开启），允许来宾访问并写入 Xen 控制台。

+   `prot_doms_locked`布尔值，默认为 0（关闭），如果启用，将禁止`prot_domU_t`来宾启动。

虽然`xl`命令没有可用的子命令来查询和设置 XSM 布尔值，但系统中安装了另外两个命令来完成此任务——`flask-get-bool`和`flask-set-bool`：

+   使用`flask-get-bool`，我们可以查询布尔值的当前状态，或者列出所有布尔值及其当前值：

    ```
    # flask-get-bool -a
    guest_writeconsole: 1
    prot_doms_locked: 0
    ```

+   `flask-set-bool`命令用于切换布尔值：

    ```
    # flask-set-bool prot_doms_locked 1
    ```

这与 SELinux 的`getsebool`和`setsebool`命令非常相似。

### 查询 XSM 策略

XSM 策略文件（`xenpolicy-4.13.1`）与 SELinux 策略文件非常相似。因此，我们可以使用 SELinux 工具查询此文件并了解更多关于策略的信息：

+   使用`seinfo`，我们可以查询有关策略的统计信息，查看支持的类、已启用的约束以及更多内容。唯一会失败的查询是列出策略中支持的类型：

    ```
    $ seinfo --all ./xenpolicy-4.13.1
    ```

+   使用`sesearch`，我们可以查询 XSM 策略规则本身，例如列出所有允许规则：

    ```
    $ sesearch -A ./xenpolicy-4.13.1
    ```

当我们在*第十三章*中讨论分析 SELinux 策略时，*分析策略行为*，我们将熟悉其他可以用于分析 XSM 策略文件的工具。

### 标记硬件资源

使用`flask-label-pci`命令，管理员可以为指定的 PCI 设备打上给定的标签。这种方法允许管理员为无特权的来宾标记某些设备，以便进行直通访问。

例如，要将地址为`3:2:0`的 PCI 设备标记为`nic_dev_t`类型，可以使用以下命令：

```
# flask-label-pci 0000:03:02.0 system_u:object_r:nic_dev_t
```

如你从名称中可能猜到的，这种类型最初是为网络设备的直通访问而定义的，但也可以用于其他 PCI 硬件。

# 应用自定义 XSM 策略

Xen 还允许管理员构建和使用他们自己的自定义策略。

Xen 的默认策略位于 Xen 构建目录中的`tools/flask/policy`目录下。例如，dom0 来宾的策略规则位于`modules/dom0.te`中。

重要说明

调整 Xen XSM 策略超出了本章的范围。你可以在 *第十五章* *使用参考策略* 中找到有关如何使用参考策略风格方法创建 SELinux 策略的说明。Xen XSM 策略基于这种风格。

构建自定义策略的方法是更新这些文件（更新前请先备份），然后重新构建策略本身：

```
$ make
```

策略构建的结果是一个新的 `xenpolicy-4.13.1` 文件。这个文件可以通过 `xl loadpolicy` 命令直接加载：

```
# xl loadpolicy /path/to/xenpolicy-4.13.1
```

这个命令类似于 `flask-loadpolicy` 命令：

```
# flask-loadpolicy /path/to/xenpolicy-4.13.1
```

如果在测试后，策略被认为已经准备好用于持续使用，可以将其复制到 `/boot` 目录，这样它将在下次启动时自动加载。

# 总结

Xen 虚拟机监控器与 QEMU 和 KVM 虚拟机监控器有很大不同，后者在 libvirt 中更常用。Xen 的 SELinux 支持与 sVirt 也不同，因为 SELinux 子系统只能在 Xen 客户操作系统内部启用，且 SELinux 无法看到其他客户操作系统。

Xen 通过实现自己的 SELinux 副本作为 XSM-FLASK 解决了这个问题，并在其工具中集成了对 XSM-FLASK 标签的适当支持。在本章中，我们学习了如何为 Xen 客户操作系统应用我们自己的类型，切换 XSM 状态，切换 XSM 布尔值，甚至如何构建并加载我们自己的 XSM-FLASK 策略。

在下一章，我们将讨论容器工作负载以及 SELinux 如何帮助管理员进一步强化和保护容器运行时。我们将看到如何将 sVirt 应用于容器运行时，以及工具如何处理 SELinux 支持。

# 问题

1.  为什么常规的 SELinux 子系统不管理 Xen 客户操作系统？

1.  Xen 客户操作系统如何分配标签？

1.  处理 XSM 标签的常见 Xen 命令有哪些？

1.  管理员如何加载自定义策略进行测试？
