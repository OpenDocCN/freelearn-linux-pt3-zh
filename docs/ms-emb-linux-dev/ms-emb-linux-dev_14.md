

# 第十一章：与设备驱动程序接口

内核设备驱动程序是将底层硬件暴露给系统其余部分的机制。作为嵌入式系统开发人员，你需要了解这些设备驱动程序如何融入整体架构，以及如何从用户空间程序访问它们。你的系统可能会有一些新颖的硬件，你需要找出一种访问它们的方法。在许多情况下，你会发现有现成的设备驱动程序可以使用，而你无需编写任何内核代码就能实现你想要的功能。例如，你可以通过`sysfs`中的文件操作 GPIO 引脚和 LED，并且可以使用库来访问包括**SPI**（**串行外设接口**）和**I2C**（**集成电路之间接口**）在内的串行总线。

有许多地方可以了解如何编写设备驱动程序，但很少有地方会告诉你为什么要编写设备驱动程序，以及你在编写时的选择。这正是我想在这里讨论的内容。然而，请记住，这不是一本专门讲解编写内核设备驱动程序的书籍，这里提供的信息是帮助你了解相关领域，但不一定能让你在其中扎根。很多优秀的书籍和博客文章可以帮助你编写设备驱动程序，其中一些将在本章末的*进一步学习*部分列出。

本章将涵盖以下主题：

+   设备驱动程序的作用

+   字符设备

+   块设备

+   网络设备

+   在运行时了解驱动程序

+   寻找合适的设备驱动程序

+   用户空间中的设备驱动程序

+   编写内核设备驱动程序

+   发现硬件配置

# 技术要求

要跟随示例进行操作，请确保你有以下设备：

+   基于 Linux 的主机系统

+   一张 microSD 卡读卡器和卡

+   BeaglePlay

+   一种能够提供 3A 电流的 5V USB-C 电源

+   一根以太网电缆和一个有可用端口的路由器，用于网络连接

本章使用的代码可以在本书 GitHub 仓库的章节文件夹中找到：[`github.com/PacktPublishing/Mastering-Embedded-Linux-Development/tree/main/Chapter11`](https://github.com/PacktPublishing/Mastering-Embedded-Linux-Development/tree/main/Chapter11)。

# 设备驱动程序的作用

正如我在*第四章*中提到的，内核的一个功能是封装计算机系统的众多硬件接口，并以一致的方式向用户空间程序呈现它们。内核有一些框架，旨在简化设备驱动程序的编写，设备驱动程序是连接上层内核和下层硬件的代码。设备驱动程序可以控制物理设备，例如 UART 或 MMC 控制器，或者它可以代表虚拟设备，如空设备(`/dev/null`)或 RAM 磁盘。一个驱动程序可能控制多个相同类型的设备。

内核设备驱动代码以较高的特权级别运行，和其余的内核一样。它可以完全访问处理器地址空间和硬件寄存器。它能够处理中断和 DMA 传输。它还可以利用复杂的内核基础设施来进行同步和内存管理。然而，你应该注意，这也有缺点；如果驱动程序有问题，可能会导致系统崩溃。

因此，有一个原则，设备驱动程序应该尽可能简单，仅仅为应用程序提供信息（实际决策由应用程序做出）。你经常会听到这个被表达为*内核中没有策略*。制定管理系统整体行为的策略是用户空间的责任。例如，响应外部事件（如插入新 USB 设备）来加载内核模块的任务是`udev`用户空间程序的责任，而不是内核的。内核只是提供了一种加载内核模块的手段。

在 Linux 中，设备驱动程序主要有三种类型：

+   **字符设备**：这是为无缓冲 I/O 提供的，具有丰富的功能范围，且在应用代码和驱动程序之间有一个薄层。在实现自定义设备驱动时，它是首选。

+   **块设备**：这是为大容量存储设备的块 I/O 设计的接口。它有一层厚厚的缓冲层，旨在使磁盘读写尽可能快，这使得它不适用于其他用途。

+   **网络设备**：这类似于块设备，但用于传输和接收网络数据包，而不是磁盘块。

还有第四种类型，它以伪文件系统中的一组文件呈现。例如，你可能通过`/sys/class/gpio`中的一组文件访问 GPIO 驱动程序，正如我将在本章后面描述的那样。让我们从更详细地了解这三种基本设备类型开始。

# 字符设备

字符设备在用户空间通过一个叫做**设备节点**的特殊文件来标识。这个文件名通过与之关联的主次设备号映射到一个设备驱动程序。广义上说，**主设备号**将设备节点映射到特定的设备驱动，而**次设备号**则告诉驱动程序访问的是哪个接口。例如，Arm Versatile PB 的第一个串口设备节点命名为`/dev/ttyAMA0`，主设备号为`204`，次设备号为`64`。第二个串口的设备节点主设备号相同，但次设备号是`65`。我们可以在目录列表中看到所有四个串口的设备号：

```
# ls -l /dev/ttyAMA*
crw-rw---- 1 root root 204, 64    Jan  1 1970 /dev/ttyAMA0
crw-rw---- 1 root root 204, 65    Jan  1 1970 /dev/ttyAMA1
crw-rw---- 1 root root 204, 66    Jan  1 1970 /dev/ttyAMA2
crw-rw---- 1 root root 204, 67    Jan  1 1970 /dev/ttyAMA3 
```

标准主次编号的列表可以在内核文档 `Documentation/admin-guide/devices.txt` 中找到。这个列表更新不频繁，且不包括前面提到的 `ttyAMA` 设备。然而，如果你查看内核源代码中的 `drivers/tty/serial/amba-pl011.c`，你将看到主次编号的声明位置：

```
#define SERIAL_AMBA_MAJOR 204
#define SERIAL_AMBA_MINOR 64 
```

如果设备有多个实例，例如 `ttyAMA` 驱动程序，设备节点的命名约定是使用基准名称（`ttyAMA`）并附加实例号，范围从 `0` 到 `3`。

正如我在 *第五章* 中提到的，设备节点可以通过几种方式创建：

+   `devtmpfs`：当设备驱动程序使用驱动程序提供的基准名称（`ttyAMA`）和实例号注册一个新设备接口时，会创建设备节点。

+   `udev` 或 `mdev`（没有 `devtmpfs`）：这些与 `devtmpfs` 基本相同，只是用户空间的守护程序需要从 `sysfs` 中提取设备名称并创建节点。

+   `mknod`：如果你使用静态设备节点，它们是通过 `mknod` 手动创建的。

你可能从我这里使用的数字中得到这样的印象：主次编号都是 8 位数字，范围从 0 到 255。事实上，主编号是 12 位的，允许有效的主编号范围从 1 到 4095，而次编号是 20 位的，范围从 0 到 1,048,575。

当你打开一个字符设备节点时，内核会检查主次编号是否属于字符设备驱动程序已注册的范围。如果是，内核将调用驱动程序；否则，`open(2)` 调用会失败。设备驱动程序可以提取次编号以确定使用哪个硬件接口。

要编写一个访问设备驱动程序的程序，你需要了解它的工作原理。换句话说，设备驱动程序不同于文件：你对它所做的操作会改变设备的状态。一个简单的例子是 `urandom` 伪随机数生成器，每次读取时都会返回一组随机数据字节。

这是一个实现此功能的程序：

```
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
int main(void)
{
    int f;
    unsigned int rnd;
    int n;
f = open("/dev/urandom", O_RDONLY);
    if (f < 0) {
        perror("Failed to open urandom");
        return 1;
    }
    n = read(f, &rnd, sizeof(rnd));
    if (n != sizeof(rnd)) {
        perror("Problem reading urandom");
        return 1;
    }
    printf("Random number = 0x%x\n", rnd);
    close(f);
    return 0;
} 
```

你可以在 `MELD/Chapter11/meta-device-drivers/recipes-local/read-urandom` 目录下找到该程序的完整源代码和 BitBake 配方。

Unix 驱动程序模型的优点是，一旦我们知道有一个名为 `urandom` 的设备，每次从中读取时，它都会返回一组新的伪随机数据，因此我们无需了解关于它的其他信息。我们可以直接使用标准函数，如 `open(2)`、`read(2)` 和 `close(2)`。

**提示**

你也可以使用流 I/O 函数，如 `fopen(3)`、`fread(3)` 和 `fclose(3)`，但这些函数中隐含的缓冲机制往往会导致意外的行为。例如，`fwrite(3)` 通常只写入用户空间缓冲区，而不是设备。你需要调用 `fflush(3)` 强制将缓冲区写出。因此，最好不要在调用设备驱动程序时使用流 I/O 函数。

大多数设备驱动程序使用字符接口。大容量存储设备是一个显著的例外。读取和写入磁盘需要使用块接口以获得最大速度。

# 块设备

块设备还与一个设备节点相关联，该节点也具有主设备号和次设备号。

**提示**

尽管字符设备和块设备是通过主设备号和次设备号来标识的，但它们位于不同的命名空间中。主设备号为 4 的字符驱动程序与主设备号为 4 的块驱动程序没有任何关系。

对于块设备，主设备号用于标识设备驱动程序，次设备号用于标识分区。让我们来看一下 BeaglePlay 上的 MMC 驱动程序：

```
# ls -l /dev/mmcblk*
brw-rw---- 1 root disk 179,   0 Aug  7 13:25 /dev/mmcblk0
brw-rw---- 1 root disk 179, 256 Aug  7 13:25 /dev/mmcblk0boot0
brw-rw---- 1 root disk 179, 512 Aug  7 13:25 /dev/mmcblk0boot1
brw-rw---- 1 root disk 179,   1 Aug  7 13:25 /dev/mmcblk0p1
brw-rw---- 1 root disk 179,   2 Aug  7 13:25 /dev/mmcblk0p2
brw-rw---- 1 root disk 236,   0 Aug  7 13:25 /dev/mmcblk0rpmb
brw-rw---- 1 root disk 179, 768 Feb  4 09:42 /dev/mmcblk1
brw-rw---- 1 root disk 179, 769 Feb  4 09:42 /dev/mmcblk1p1
brw-rw---- 1 root disk 179, 770 Feb  4 09:42 /dev/mmcblk1p2 
```

在这里，`mmcblk0` 是 eMMC 芯片，它有两个分区，而 `mmcblk1` 是 microSD 卡槽，卡槽上也有一张带有两个分区的卡。MMC 块设备驱动的主设备号是 `179`（你可以在 `devices.txt` 文件中查找）。次设备号用于标识不同的物理 MMC 设备以及该设备上存储介质的分区。在 MMC 驱动程序的情况下，每个设备的次设备号范围为八个：`0` 到 `7` 的次设备号用于第一个设备，`8` 到 `15` 用于第二个设备，以此类推。在每个范围内，第一个次设备号表示整个设备的原始扇区，其他次设备号表示最多七个分区。在 BeaglePlay 的 eMMC 芯片上，有两个 4 MB 的内存区域被保留用于引导加载程序。这两个区域分别表示为 `mmcblk0boot0` 和 `mmcblk0boot1`，其次设备号分别为 `256` 和 `512`。

另一个例子是，你可能熟悉名为 `sd` 的 SCSI 磁盘驱动程序，它用于控制使用 SCSI 命令集的一系列磁盘，包括 SCSI、SATA、USB 大容量存储和 **通用闪存存储** (**UFS**) 。它的主设备号是 `8`，每个接口或磁盘有 `16` 个次设备号范围。

从 `0` 到 `15` 的次设备号用于第一个接口，设备节点名称为 `sda` 到 `sda15`；从 `16` 到 `31` 的次设备号用于第二个磁盘，设备节点名称为 `sdb` 到 `sdb15`；以此类推。这一过程一直持续到第 16 个磁盘，从 `240` 到 `255`，设备节点名称为 `sdp`。由于 SCSI 磁盘非常流行，因此为它们保留了其他主设备号，但在这里我们不需要担心这个。

MMC 和 SCSI 块驱动程序都期望在磁盘的开始处找到分区表。分区表可以通过 `fdisk`、`sfidsk` 和 `parted` 等工具创建。

用户空间程序可以通过设备节点直接打开和与块设备交互。尽管如此，这并不是一个常见的操作，通常只用于执行管理操作，例如创建分区、使用文件系统格式化分区和挂载。一旦文件系统被挂载，您通过该文件系统中的文件间接与块设备交互。

大多数块设备都将有一个有效的内核驱动程序，因此我们很少需要编写自己的驱动程序。网络设备也是如此。就像文件系统抽象了块设备的细节一样，网络堆栈消除了直接与网络设备交互的需求。

# 网络设备

网络设备不通过设备节点访问，也没有主要和次要编号。相反，内核基于字符串和实例号为网络设备分配名称。以下是网络驱动程序注册接口的示例方式：

```
my_netdev = alloc_netdev(0, "net%d", NET_NAME_UNKNOWN, netdev_setup);
ret = register_netdev(my_netdev); 
```

这将在第一次调用时创建名为`net0`的网络设备，第二次调用时创建`net1`，依此类推。更常见的名称包括`lo`、`eth0`、`enp2s0`、`wlan0`和`wlp1s0`。请注意，这是它启动时的名称；设备管理器如`udev`可能会稍后将其更改为其他名称。

通常，网络接口名称仅在使用诸如`ip`等实用程序配置网络地址和路由时使用。之后，您通过打开套接字间接与网络驱动程序交互，让网络层决定如何将其路由到正确的接口。

然而，通过创建套接字并使用`include/linux/sockios.h`中列出的`ioctl`命令，可以从用户空间直接访问网络设备。以下是使用`SIOCGIFHWADDR`查询网络驱动程序硬件（MAC）地址的程序示例：

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/ioctl.h>
#include <linux/sockios.h>
#include <net/if.h>
int main(int argc, char *argv[])
{
    int s;
    int ret;
    struct ifreq ifr;
    if (argc != 2) {
        printf("Usage %s [network interface]\n", argv[0]);
        return 1;
    }
    s = socket(PF_INET, SOCK_DGRAM, 0);
    if (s < 0) {
        perror("socket");
        return 1;
    }
    strcpy(ifr.ifr_name, argv[1]);
    ret = ioctl(s, SIOCGIFHWADDR, &ifr);
    if (ret < 0) {
        perror("ioctl");
        return 1;
    }
  for (int i = 0; i < 6; i++) {
        printf("%02x:", (unsigned char)ifr.ifr_hwaddr.sa_data[i]);
    }
    printf("\n");
    close(s);
    return 0;
} 
```

您可以通过读取`MELD/Chapter11/meta-device-drivers/recipes-local/show-mac-address`目录中的完整源代码和 BitBake 配方来查找此程序的完整源代码和 BitBake 配方。`show-mac-address`程序将网络接口名称作为参数。打开套接字后，我们将接口名称复制到结构体中，并将该结构体传递到套接字上的`ioctl`调用中，然后打印出结果的 MAC 地址。

现在我们知道设备驱动程序的三个类别，如何列出系统中使用的不同驱动程序呢？

# 在运行时查找驱动程序

一旦您有一个运行中的 Linux 系统，了解已加载的设备驱动程序及其状态非常有用。通过阅读`/proc`和`/sys`中的文件，您可以获取很多信息。

通过读取`/proc/devices`列出当前加载和活动的字符和块设备驱动程序：

```
$ cat /proc/devices
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
<…> 
```

对于每个驱动程序，你可以看到主设备号和基本名称。然而，这并不能告诉你每个驱动程序连接了多少设备。它只显示了`ttyAMA`，但并未给出它是否连接了四个实际的串口。我会在后面讨论`sysfs`时再回到这个问题。

网络设备不出现在这个列表中，因为它们没有设备节点。相反，你可以使用`ip`工具获取网络设备的列表：

```
# ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq qlen 1000
    link/ether 34:08:e1:85:07:d9 brd ff:ff:ff:ff:ff:ff
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop qlen 1000
    link/ether 7a:1f:d8:46:36:b1 brd ff:ff:ff:ff:ff:ff 
```

你还可以通过著名的`lsusb`和`lspci`命令分别查看连接到 USB 或 PCI 总线的设备。关于它们的信息可以在各自的手册页中找到，也有大量在线指南，因此我在这里就不再详细描述了。

真正有趣的信息在`sysfs`中，这也是我们接下来要讨论的话题。

## 从 sysfs 获取信息

你可以从细致的角度定义`sysfs`为内核对象、属性和关系的表示。一个内核对象是一个**目录**，一个属性是一个**文件**，而一个关系是一个从一个对象到另一个对象的**符号链接**。从更实用的角度来看，由于 Linux 设备驱动模型将所有设备和驱动程序表示为内核对象，你可以通过查看`/sys`来看到内核对系统的视图：

```
# ls /sys
block     class     devices   fs        module
bus       dev       firmware  kernel    power 
```

在探索设备和驱动程序信息的过程中，我将关注这三个目录：`devices`、`class`和`block`。

### 设备 – /sys/devices

这是内核在启动后发现的设备以及它们如何相互连接的视图。它按系统总线在顶层进行组织，因此你看到的内容会因系统而异。以下是 QEMU 仿真中的 Arm Versatile 板的示例：

```
# ls /sys/devices
platform  software  system   tracepoint virtual 
```

所有系统中都有三个目录：

+   `system/`：包含系统核心的设备，包括 CPU 和时钟。

+   `virtual/`：包含基于内存的设备。你会在`virtual/mem`中找到像`/dev/null`、`/dev/random`和`/dev/zero`这样的内存设备。你还会在`virtual/net`中找到`lo`回环设备。

+   `platform/`：这是一个包含所有未通过常规硬件总线连接的设备的目录。在嵌入式设备上，几乎所有的东西都可能属于这个目录。

其他设备出现在与实际系统总线相对应的目录中。例如，如果存在 PCI 根总线，它会显示为`pci0000:00`。

浏览这个层次结构相当困难，因为它需要一些对系统拓扑的了解，并且路径名变得相当长且难以记住。为了简化操作，`/sys/class`和`/sys/block`提供了两种不同的设备视图。

### 驱动程序 – /sys/class

这是按照设备驱动类型展示的视图。换句话说，这是一个软件视图，而非硬件视图。每个子目录代表一种驱动类，并由驱动框架中的一个组件实现。例如，UART 设备由 `tty` 层管理，因此你会在 `/sys/class/tty` 目录下找到它们。同样，你可以在 `/sys/class/net` 找到网络设备，在 `/sys/class/input` 找到输入设备，如键盘、触摸屏和鼠标，等等。

每个子目录中都有一个符号链接，指向该类型设备在 `/sys/device` 中的表示。

让我们来看一下 Versatile PB 上的串行端口。我们可以看到它们共有四个：

```
# ls -d /sys/class/tty/ttyAMA*
/sys/class/tty/ttyAMA0    /sys/class/tty/ttyAMA2
/sys/class/tty/ttyAMA1    /sys/class/tty/ttyAMA3 
```

每个目录都是与设备接口实例关联的内核对象的表示。查看这些目录中的某一个，我们可以看到该对象的属性（以文件形式表示）以及与其他对象的关系（通过链接表示）：

```
# ls /sys/class/tty/ttyAMA0
close_delay       flags             line            uartclk
closing_wait      io_type           port            uevent
custom_divisor    iomem_base        power           xmit_fifo_size
dev               iomem_reg_shift   subsystem
device            irq               type 
```

名为 `device` 的链接指向该设备的硬件对象。名为 `subsystem` 的链接指向 `/sys/class/tty` 中的父子系统。其余的目录项是属性。某些属性特定于串行端口，如 `xmit_fifo_size`，而其他一些则适用于多种类型的设备，例如 `irq`（中断号）和 `dev`（设备号）。有些属性文件是可写的，允许你在运行时调整驱动程序的参数。

`dev` 属性尤其有趣。如果你查看它的值，你会发现以下内容：

```
# cat /sys/class/tty/ttyAMA0/dev
204:64 
```

这些是设备的主次号。这个属性是在驱动程序注册接口时创建的。正是通过这个文件，`udev` 和 `mdev` 才能找到设备驱动的主次号。

### 块设备驱动 – /sys/block

还有一个关于设备模型的重要视图：你可以在 `/sys/block` 中找到的块设备驱动视图。这里有每个块设备的子目录。以下是来自 BeaglePlay 的一部分：

```
# ls /sys/block
loop0   loop4   mmcblk0        ram0    ram12   ram2   ram6
loop1   loop5   mmcblk1        ram1    ram13   ram3   ram7
loop2   loop6   mmcblk0boot0   ram10   ram14   ram4   ram8
loop3   loop7   mmcblk0boot1   ram11   ram15   ram5   ram9 
```

如果你查看 `mmcblk0` 目录，这是该开发板上的 eMMC 芯片，你将看到接口的属性和其中的分区信息：

```
# ls /sys/block/mmcblk0
alignment_offset   events             holders       mmcblk0p2  ro
bdi                events_async       inflight      mq         size
capability         events_poll_msecs  integrity     power      slaves
dev                ext_range          mmcblk0boot0  queue      stat
device             force_ro           mmcblk0boot1  range      subsystem
discard_alignment  hidden             mmcblk0p1     removable  uevent 
```

总结来说，你可以通过阅读 `sysfs` 来了解很多关于系统中设备（硬件）和驱动程序（软件）的信息。

# 寻找合适的设备驱动

一个典型的嵌入式开发板通常基于制造商提供的参考设计，并做出一些更改以使其适用于特定的应用。随参考板一起提供的 BSP 应该支持该板上的所有外设。然后你可以定制设计，可能是通过 I2C 连接的温度传感器、通过 GPIO 引脚连接的灯光和按钮、通过 MIPI 接口连接的显示面板，或者其他许多东西。你的任务是创建一个定制的内核来控制所有这些外设，但你该从哪里开始寻找支持这些外设的设备驱动呢？

最明显的查询途径是查看制造商网站上的驱动支持页面，或者直接向他们询问。根据我的经验，这种方式很少能得到你想要的结果。硬件制造商通常对 Linux 不太熟悉，他们往往会提供误导性的信息。可能他们提供的是二进制格式的专有驱动，或者是与你当前内核版本不匹配的源代码。因此，尽管如此，你可以尝试这种方式。就我个人而言，我总是会尽量寻找适合当前任务的开源驱动程序。

你的内核中可能已经有支持：主线 Linux 中有成千上万的驱动程序，厂商内核中也有许多厂商特定的驱动程序。首先运行`make menuconfig`（或`xconfig`），并搜索产品名称或编号。如果没有找到完全匹配的项，尝试进行更通用的搜索，考虑到大多数驱动程序支持同一家族的多个产品。接下来，尝试在`drivers`目录中查找代码（`grep`是你的好帮手）。

如果你仍然没有找到驱动程序，可以尝试在线搜索并在相关论坛中询问，看是否有适用于较新版本 Linux 的驱动程序。如果找到了，应该认真考虑更新 BSP，以使用更新的内核。有时，这样做并不实际，因此你可能需要考虑将驱动程序回移植到当前内核。如果内核版本相似，移植可能比较简单，但如果版本相差超过 12 到 18 个月，那么代码变化可能非常大，你可能需要重写驱动程序的一部分来将其与当前内核集成。如果这些方法都失败了，你将不得不自己编写缺失的内核驱动程序。然而，这并不总是必要的。我们将在下一节中探讨一种替代方法。

# 用户空间中的设备驱动程序

在开始编写设备驱动程序之前，先停下来思考一下它是否真的必要。对于许多常见类型的设备，已经有通用的设备驱动程序，允许你直接在用户空间与硬件交互，而无需编写一行内核代码。用户空间代码当然更容易编写和调试。而且它不受 GPL 的约束，尽管我认为这并不是采取这种方式的好理由。

这些驱动程序大致分为两类：一类是通过`sysfs`中的文件进行控制的驱动，包括 GPIO 和 LED，另一类是通过设备节点暴露通用接口的串行总线，如 I2C。

让我们为 BeaglePlay 构建一个包含一些示例的 Yocto 镜像：

1.  导航到你克隆 Yocto 的目录的上一层：

    ```
    $ cd ~ 
    ```

1.  从本书的 Git 仓库中复制 meta-device-driver 层：

    ```
    $ cp -a MELD/Chapter11/meta-device-drivers . 
    ```

1.  为 BeaglePlay 设置你的 BitBake 工作环境：

    ```
    $ source poky/oe-init-build-env build-beagleplay 
    ```

1.  这将设置一系列环境变量，并将你带回到`build-beagleplay`目录，这是你在*第六章*的*层*部分中填充的目录。如果你已经删除了之前的工作，可以重复该练习，添加自己的`meta-nova`层，并为 BeaglePlay 构建`core-image-minimal`。

1.  移除`meta-nova`层：

    ```
    $ bitbake-layers remove-layer ../meta-nova 
    ```

1.  添加`meta-device-drivers`层：

    ```
    $ bitbake-layers add-layer ../meta-device-drivers 
    ```

1.  确认你的层结构是否设置正确：

    ```
    $ bitbake-layers show-layers
    NOTE: Starting bitbake server...
    layer                 path                                       priority
    =========================================================================
    core                /home/frank/poky/meta                        5
    yocto               /home/frank/poky/meta-poky                   5
    yoctobsp            /home/frank/poky/meta-yocto-bsp              5
    arm-toolchain       /home/frank/meta-arm/meta-arm-toolchain      5
    meta-arm            /home/frank/meta-arm/meta-arm                5
    meta-ti-bsp         /home/frank/meta-ti/meta-ti-bsp              6
    device-drivers      /home/frank/meta-device-drivers              6 
    ```

1.  修改`conf/local.conf`，使示例程序和虚拟驱动程序被安装：

    ```
    IMAGE_INSTALL:append = " read-urandom show-mac-address gpio-int i2c-eeprom-read dummy-driver" 
    ```

1.  在内核中启用传统的`/sys/class/gpio`接口：

    ```
    $ bitbake -c menuconfig virtual/kernel 
    ```

1.  确保启用了`CONFIG_EXPERT`、`CONFIG_GPIO_SYSFS`、`CONFIG_DEBUG_FS`和`CONFIG_DEBUG_FS_ALLOW_ALL`选项。确保禁用了`CONFIG_KEYBOARD_GPIO`选项。

1.  在内核中启用`/sys/class/leds`接口，确保启用了`CONFIG_LEDS_CLASS`、`CONFIG_LEDS_GPIO`和`CONFIG_LEDS_TRIGGER_TIMER`选项。

1.  保存修改后的内核`.config`并退出`menuconfig`。

1.  构建`core-image-minimal`：

    ```
    $ bitbake core-image-minimal 
    ```

使用 balenaEtcher 将完成的镜像写入 microSD 卡，将 microSD 插入 BeaglePlay 并启动，如*第六章*中的*运行 BeaglePlay 目标*部分所述。

## GPIO

**通用输入/输出**（**GPIO**）是最简单的数字接口形式，因为它为你提供对单个硬件引脚的直接访问，每个引脚可以处于两种状态之一：高或低。在大多数情况下，你可以将 GPIO 引脚配置为输入或输出。你甚至可以使用一组 GPIO 引脚，通过软件操作每个比特，创建更高级别的接口，如 I2C 或 SPI，这种技术称为**位接入**。主要的限制是软件循环的速度和准确性，以及你愿意为此分配的 CPU 周期数。通常，除非你配置实时内核，否则很难达到毫秒级别以下的定时器准确度，正如我们将在*第二十一章*中看到的那样。GPIO 的更常见用例是读取按键和数字传感器，以及控制 LED、马达和继电器。

大多数 SoC 将大量 GPIO 位组装在一起，通常每个寄存器 32 位。在芯片上的 GPIO 位通过一个称为**引脚复用**（**pin mux**）的多路复用器路由到芯片封装上的 GPIO 引脚。通过 I2C 或 SPI 总线连接的电源管理芯片和专用 GPIO 扩展器上，可能还会有额外的 GPIO 引脚。所有这些多样性都由一个内核子系统`gpiolib`处理，`gpiolib`实际上不是一个库，而是 GPIO 驱动程序用来以一致方式公开 I/O 的基础设施。有关`gpiolib`实现的详细信息，可以在内核源代码的`Documentation/driver-api/gpio/`中找到，驱动程序本身的代码位于`drivers/gpio/`。

应用程序可以通过 `/sys/class/gpio/` 目录中的文件与 `gpiolib` 进行交互。以下是在像 BeaglePlay 这样的典型嵌入式板上看到的内容：

```
# ls /sys/class/gpio
export       gpiochip512  gpiochip515  gpiochip539  gpiochip631  unexport 
```

从 `gpiochip512` 到 `gpiochip631` 命名的目录表示四个 GPIO 寄存器，每个寄存器有一个可变数量的 GPIO 位。如果你查看其中一个 `gpiochip` 目录，你将看到以下内容：

```
# ls /sys/class/gpio/gpiochip512
base       device     label      ngpio      power      subsystem  uevent 
```

名为 `base` 的文件包含寄存器中第一个 GPIO 引脚的编号，而 `ngpio` 包含寄存器中的位数。在这个例子中，`gpiochip512/base` 是 `512`，`gpiochip512/ngpio` 是 `3`，这告诉你它包含 GPIO 位 `512` 到 `514`。一个寄存器中的最后一个 GPIO 和下一个寄存器中的第一个 GPIO 之间可能会有间隙。

要从用户空间控制 GPIO 位，首先需要从内核空间导出它，你可以通过将 GPIO 编号写入 `/sys/class/gpio/export` 来实现。这个示例展示了 GPIO 640 的过程，它与 BeaglePlay 上的 mikroBUS 连接器的 INT 引脚相连：

```
# echo 640 > /sys/class/gpio/export
# ls /sys/class/gpio
export       gpiochip512  gpiochip539  unexport
gpio640      gpiochip515  gpiochip631 
```

现在，有一个新的 `gpio640` 目录，其中包含你需要控制引脚的文件。

**重要提示**

如果 GPIO 位已被内核占用，你将无法以这种方式导出它：

```
# echo 640 > /sys/class/gpio/export
bash: echo: write error: Device or resource busy 
```

`gpio640` 目录包含以下文件：

```
# ls /sys/class/gpio/gpio640
active_low  direction   power       uevent
device      edge        subsystem   value 
```

引脚开始时是一个有效的输入，用于 mikroBUS 连接器的 INT（中断）引脚。要将 GPIO 转换为输出，请将 `out` 写入 `direction` 文件。`value` 文件包含引脚的当前状态，`0` 表示低电平，`1` 表示高电平。如果它是输出，你可以通过写入 `0` 或 `1` 来改变状态。有时硬件中的低电平和高电平意义会被反转（硬件工程师喜欢做这种事情），因此写入 `1` 到 `active_low` 会反转 `value` 的意义，使低电压显示为 `1`，高电压显示为 `0`。

相反，你可以通过将 GPIO 编号写入 `/sys/class/gpio/unexport` 来从用户空间移除 GPIO 控制，就像你为导出所做的那样。

### 处理来自 GPIO 的中断

在许多情况下，GPIO 输入可以配置为在状态变化时生成中断。这使你可以等待中断，而不是在低效的软件循环中轮询。如果 GPIO 位能够生成中断，则会存在一个名为 `edge` 的文件。它的初始值为 `none`，表示不生成中断。要启用中断，你可以将其设置为以下值之一：

+   `rising`：在上升沿触发中断。

+   `falling`：在下降沿触发中断。

+   `both`：在上升沿和下降沿触发中断。

+   `none`：没有中断（默认）。

要确定 BeaglePlay 上的 USR 按钮分配到哪个 GPIO：

```
# cat /sys/kernel/debug/gpio | grep USR_BUTTON
 gpio-557 (USR_BUTTON          |sysfs               ) in  hi IRQ 
```

如果你想等待 GPIO 557（USR 按钮）上的下降沿，你必须首先启用中断：

```
# echo 557 > /sys/class/gpio/export
# echo falling > /sys/class/gpio/gpio557/edge 
```

这是一个等待来自 GPIO 的中断的程序：

```
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/epoll.h>
#include <sys/types.h>
int main(int argc, char *argv[])
{
    int ep;
    int f;
    struct epoll_event ev, events;
    char value[4];
    int ret;
    int n;
    ep = epoll_create(1);
    if (ep == -1) {
        perror("Can't create epoll");
        return 1;
    }
    f = open("/sys/class/gpio/gpio557/value", O_RDONLY | O_NONBLOCK);
    if (f == -1) {
        perror("Can't open gpio557");
        return 1;
    }
    n = read(f, &value, sizeof(value));
    if (n > 0) {
        printf("Initial value value=%c\n", value[0]);
        lseek(f, 0, SEEK_SET);
    }
    ev.events = EPOLLPRI;
    ev.data.fd = f;
    ret = epoll_ctl(ep, EPOLL_CTL_ADD, f, &ev);
    while (1) {
        printf("Waiting\n");
        ret = epoll_wait(ep, &events, 1, -1);
        if (ret > 0) {
            n = read(f, &value, sizeof(value));
            printf("Button pressed: value=%c\n", value[0]);
            lseek(f, 0, SEEK_SET);
        }
    }
    return 0;
} 
```

下面是`gpio-int`程序的工作原理。首先，调用`epoll_create`创建`epoll`通知功能。接下来，打开 GPIO 并读取其初始值。调用`epoll_ctl`将 GPIO 的文件描述符注册为`POLLPRI`事件。最后，使用`epoll_wait`函数等待中断。当你按下 BeaglePlay 上的 USR 按钮时，程序将打印出`Button pressed:`，后跟从 GPIO 读取的字节数和值。

虽然我们本可以使用`select`和`poll`来处理中断，但与其他两个系统调用不同，`epoll`的性能在监视的文件描述符数量增加时不会迅速下降。

这个程序的完整源代码，以及 BitBake 配方和 GPIO 配置脚本，可以在`MELD/Chapter11/meta-device-drivers/recipes-local/gpio-int`目录下找到。

与 GPIO 一样，LED 通过`sysfs`访问。然而，它的接口有明显不同。

## LED 灯

LED 通常通过 GPIO 引脚进行控制，但还有一个内核子系统提供了更专门的控制，特别是为此目的。`leds`内核子系统增加了设置亮度的功能（如果 LED 具备此能力），并且能够处理以其他方式连接的 LED，而不仅仅是简单的 GPIO 引脚。它可以配置为在事件发生时触发 LED 点亮，例如块设备访问或心跳，用以显示设备正在工作。你需要在内核中启用`CONFIG_LEDS_CLASS`选项，并配置适合你的 LED 触发动作。更多信息请参阅`Documentation/leds/`，驱动程序位于`drivers/leds/`。

与 GPIO 一样，LED 通过`/sys/class/leds/`目录下的`sysfs`接口进行控制。在 BeaglePlay 中，用户 LED 的名称以`：function`的形式编码在设备树中，如下所示：

```
# ls /sys/class/leds
:cpu            :heartbeat      :wlan           mmc1::
:disk-activity  :lan            mmc0::          mmc2:: 
```

现在，我们可以查看其中一个 LED 的属性：

```
# cd /sys/class/leds/\:heartbeat
# ls
brightness      invert          power           trigger
device          max_brightness  subsystem       uevent 
```

请注意，路径中的冒号需要通过反斜杠进行转义，这是 shell 的要求。

`brightness`文件控制 LED 的亮度，值可以在`0`（关闭）和`max_brightness`（完全开启）之间。如果 LED 不支持中间亮度，则任何非零值都将其点亮。名为`trigger`的文件列出了触发 LED 点亮的事件。触发器的列表依赖于实现。以下是一个示例：

```
# cat trigger
none kbd-scrolllock kbd-numlock <…> disk-write [heartbeat] cpu <…> 
```

当前选择的触发器显示在方括号中。你可以通过将其他触发器写入文件来更改它。如果你想完全通过亮度控制 LED，请选择`none`。如果设置触发器为`timer`，将会出现两个额外的文件，允许你设置开关时间，单位为毫秒：

```
# echo timer > trigger
# ls
brightness      delay_on        max_brightness  subsystem       uevent
delay_off       device          power           trigger
# cat delay_on
500
# cat delay_off
500 
```

如果 LED 具有芯片内定时器硬件，闪烁操作将在不打断 CPU 的情况下进行。

## I2C

I2C 是一种简单的低速两线总线，常见于嵌入式板上。它通常用于访问不在 SoC 上的外设，如显示控制器、摄像头传感器、GPIO 扩展器等。还有一个相关标准叫做 **系统管理总线**（**SMBus**），它通常用于 PC 上访问温度和电压传感器。SMBus 是 I2C 的一个子集。

I2C 是一种主从协议，主设备是 SoC 上的一个或多个主控制器。每个从设备有一个由制造商分配的 7 位地址（请参考数据手册），允许每条总线最多有 128 个节点，但其中 16 个已被保留，因此实际允许最多 112 个节点。主设备可以与其中一个从设备发起读写事务。通常，第一个字节用于指定从设备上的寄存器，而剩余的字节是从该寄存器读取或写入的数据。

每个主控制器有一个设备节点。该 SoC 有五个设备节点：

```
# ls -l /dev/i2c*
crw-rw---- 1 root gpio 89, 0  Aug  7 13:25 /dev/i2c-0
crw-rw---- 1 root gpio 89, 1  Aug  7 13:25 /dev/i2c-1
crw-rw---- 1 root gpio 89, 2  Aug  7 13:25 /dev/i2c-2
crw-rw---- 1 root gpio 89, 3  Aug  7 13:25 /dev/i2c-3
crw-rw---- 1 root gpio 89, 5  Aug  7 13:25 /dev/i2c-5 
```

设备接口提供了一系列 `ioctl` 命令，用于查询主控制器并将读写命令发送到 I2C 从设备。有一个名为 `i2c-tools` 的软件包，使用该接口提供基本的命令行工具与 I2C 设备进行交互。以下是这些工具：

+   `i2cdetect`：列出 I2C 适配器并探测总线。

+   `i2cdump`：从 I2C 外设的所有寄存器中转储数据。

+   `i2cget`：从 I2C 从设备读取数据。

+   `i2cset`：向 I2C 从设备写入数据。

`i2c-tools` 包在 Buildroot 和 Yocto 项目中都有提供，也在大多数主流发行版中可用。编写一个用户空间程序与设备通信是直接的，只要你知道从设备的地址和协议。以下示例展示了如何从 FT24C32A-ELR-T EEPROM 中读取前四个字节，该 EEPROM 被安装在 BeaglePlay 上的 I2C 总线 0。该 EEPROM 的从设备地址是 `0x50`。

以下是一个程序代码，它从 I2C 地址读取前四个字节：

```
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/ioctl.h>
#include <linux/i2c-dev.h>
#define I2C_ADDRESS 0x50
int main(void)
{
    int f;
    int n;
    char buf[10];
    /* Open the adapter and set the address of the I2C device */
    f = open("/dev/i2c-0", O_RDWR);
    /* Set the address of the i2c slave device */
    ioctl(f, I2C_SLAVE, I2C_ADDRESS);
    /* Set the 16-bit address to read from to 0 */
    buf[0] = 0; /* address byte 1 */
    buf[1] = 0; /* address byte 2 */
    n = write(f, buf, 2);
/* Now read 4 bytes from that address */
    n = read(f, buf, 4);
    printf("0x%x 0x%x 0x%x 0x%x\n", buf[0], buf[1], buf[2], buf[3]);
    close(f);
    return 0;
} 
```

这个 `i2c-eeprom-read` 程序在 BeaglePlay 上执行时会打印 `0xaa 0x55 0x33 0x33`。这四个字节的序列是 EEPROM 的魔术数字。该程序的完整源代码和 BitBake 配方可以在 `MELD/Chapter11/meta-device-drivers/recipes-local/i2c-eeprom-read` 目录下找到。

请注意，I2C 总线另一端的设备可能是小端（little-endian）或大端（big-endian）。小端和大端指的是数据字内字节的顺序。一个 32 位的数据字包含四个字节。小端表示最低有效字节位于索引 0，最高有效字节位于索引 3。相反，大端表示最高有效字节位于索引 0，最低有效字节位于索引 3。大端也被称为 *网络字节序*，对应于网络协议中字节通过网络传输的顺序。

此程序类似于`i2cget`，但要读取的地址和寄存器字节都是硬编码的，而不是作为参数传递。我们可以使用`i2cdetect`来发现 I2C 总线上任何外围设备的地址。使用`i2cdetect`可能会使 I2C 外围设备处于不良状态或锁定总线，因此使用后最好重启。外围设备的数据手册告诉我们寄存器映射的内容。有了这些信息，我们可以使用`i2cset`通过 I2C 写入其寄存器。这些 I2C 命令可以轻松地转换为用于与外围设备交互的 C 函数库。

**重要提示**

有关 Linux 实现的更多 I2C 信息，请参阅`Documentation/i2c/dev-interface.rst`。主控制器驱动程序位于`drivers/i2c/busses/`中。

另一种流行的通信协议是**SPI**，它使用 4 线总线。

## SPI

SPI 总线类似于 I2C，但速度更快，可高达数十 MHz。该接口使用四根线分别发送和接收线，允许全双工操作。总线上的每个芯片都通过专用的芯片选择线选中。它通常用于连接触摸屏传感器、显示控制器和串行 NOR 闪存设备。

与 I2C 类似，SPI 也是一种主从协议，大多数 SoC 都实现了一个或多个主机控制器。可以通过`CONFIG_SPI_SPIDEV`内核配置启用通用 SPI 设备驱动程序。这会为每个 SPI 控制器创建一个设备节点，允许您从用户空间访问 SPI 芯片。设备节点命名为`spidev<bus>.<chip select>`：

```
# ls -l /dev/spi*
crw-rw---- 1 root root 153, 0  Jan  1 00:29 /dev/spidev1.0 
```

有关使用`spidev`接口的示例，请参阅`Documentation/spi/`中的示例代码。

到目前为止，我们看到的设备驱动程序都在 Linux 内核中得到了长期的上游支持。因为所有这些设备驱动程序都是通用的（GPIO、LED、I2C 和 SPI），所以从用户空间访问它们很简单。在某些情况下，您可能会遇到一个硬件设备缺乏兼容的内核设备驱动程序。这种硬件可能是您产品的核心（如 LiDAR、SDR 等）。在 SoC 和该硬件之间可能还有一个 FPGA。在这种情况下，您可能别无选择，只能编写自己的内核模块。

# 编写内核设备驱动程序

最终，当你耗尽所有先前的用户空间选项时，你会发现自己不得不编写一个设备驱动程序来访问附加到设备的硬件。字符驱动是最灵活的，应该能满足你 90%的需求；如果你正在处理网络接口，则使用网络驱动；块设备驱动则用于大容量存储。编写内核驱动的任务非常复杂，超出了本书的范围。书末有一些参考资料，能帮助你继续前进。在这一部分，我将概述与驱动程序交互的选项——这是一个通常不被涵盖的话题——并向你展示字符设备驱动的基本骨架。

## 设计字符驱动接口

主要的字符驱动接口基于一个字节流，就像你在串行端口上看到的那样。然而，许多设备并不符合这个描述：例如，机器人臂的控制器需要能够移动和旋转每个关节的功能。幸运的是，除了 `read` 和 `write` 之外，还有其他方式与设备驱动程序进行通信：

+   `ioctl`：`ioctl` 函数允许你向驱动程序传递两个参数。这些参数可以有任何你想要的意义。根据惯例，第一个参数是一个命令，用来选择驱动程序中的某个功能，而第二个参数是指向一个结构体的指针，这个结构体用作输入和输出参数的容器。这就像一张空白画布，允许你设计任何你喜欢的程序接口。当驱动和应用程序紧密关联并由同一团队编写时，这种做法非常常见。然而，`ioctl` 在内核中已经被弃用，你会发现很难让任何新使用 `ioctl` 的驱动被上游接受。内核维护者不喜欢 `ioctl`，因为它让内核代码和应用代码过于依赖，且很难确保两者在不同的内核版本和架构间保持同步。

+   `sysfs`：这是当前首选的方法，一个很好的例子就是之前提到的 LED 接口。它的优势在于，文件命名只要具有描述性，便可以部分自我文档化。它还可以脚本化，因为文件的内容通常是文本字符串。另一方面，每个文件必须包含一个单一值的要求，使得如果你需要一次性更改多个值时，很难实现原子性。相对地，`ioctl` 通过一个函数调用将所有参数打包在一个结构体中传递。

+   `mmap`：你可以通过将内核内存映射到用户空间，直接访问内核缓冲区和硬件寄存器，从而绕过内核。你可能仍然需要一些内核代码来处理中断和 DMA。有一个封装了这一思想的子系统，称为 `uio`，即**用户 I/O**。更多的文档可以在 `Documentation/driver-api/uio-howto.rst` 中找到，示例驱动程序则在 `drivers/uio/` 目录下。

+   `sigio`：您可以使用名为`kill_fasync()`的内核函数从驱动程序发送信号，以通知应用程序某个事件，如输入准备就绪或接收到中断。按照惯例，使用名为`SIGIO`的信号，但也可以是任何信号。您可以在`drivers/uio/uio.c`和`drivers/char/rtc.c`中看到一些示例。主要问题是，在用户空间编写可靠的信号处理程序很困难，因此它仍然是一个不常使用的功能。

+   `debugfs`：这是另一个伪文件系统，将内核数据表示为文件和目录，类似于`proc`和`sysfs`。主要区别在于，`debugfs`不得包含系统正常运行所需的信息；它仅用于调试和追踪信息。它通过`mount -t debugfs debug /sys/kernel/debug`进行挂载。在`Documentation/filesystems/debugfs.rst`中有关于`debugfs`的详细描述。

+   `proc`：`proc`文件系统对于所有新代码已被弃用，除非它与进程有关，这是该文件系统最初的用途。然而，您可以使用`proc`发布您选择的任何信息。而且，与`sysfs`和`debugfs`不同，它对非 GPL 模块也可用。

+   `netlink`：这是一个套接字协议族。`AF_NETLINK`创建一个套接字，将内核空间与用户空间连接起来。最初创建它是为了让网络工具能够与 Linux 网络代码进行通信，以访问路由表和其他细节。它也被`udev`用来将事件从内核传递到`udev`守护进程。在一般的设备驱动程序中很少使用。

在内核源代码中有许多前述文件系统的示例，您可以为驱动程序代码设计非常有趣的接口。唯一的通用规则是*最小惊讶原则*。换句话说，使用您驱动程序的应用程序编写者应该发现一切都以逻辑的方式工作，没有任何奇怪或异常的地方。

## 设备驱动程序的构成

现在是时候通过查看一个简单设备驱动程序的代码来将一些线程联系起来了。

这是一个名为`dummy`的设备驱动程序的开头，它创建了四个可以通过`/dev/dummy0`到`/dev/dummy3`访问的设备：

```
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/device.h>
#define DEVICE_NAME "dummy"
#define MAJOR_NUM 42
#define NUM_DEVICES 4
static struct class *dummy_class; 
```

接下来，我们将定义字符设备接口的`dummy_open()`、`dummy_release()`、`dummy_read()`和`dummy_write()`函数：

```
static int dummy_open(struct inode *inode, struct file *file)
{
    pr_info("%s\n", __func__);
    return 0;
}
static int dummy_release(struct inode *inode, struct file *file)
{
    pr_info("%s\n", __func__);
    return 0;
}
static ssize_t dummy_read(struct file *file, char *buffer, size_t length, loff_t * offset)
{
    pr_info("%s %u\n", __func__, length);
    return 0;
}
static ssize_t dummy_write(struct file *file, const char *buffer, size_t length, loff_t * offset)
{
    pr_info("%s %u\n", __func__, length);
    return length;
} 
```

之后，我们需要初始化一个`file_operations`结构，并定义`dummy_init()`和`dummy_exit()`函数，这些函数在驱动程序加载和卸载时调用：

```
struct file_operations dummy_fops = {
    .open = dummy_open,
    .release = dummy_release,
    .read = dummy_read,
    .write = dummy_write,
};
int __init dummy_init(void)
{
    int ret;
    int i;
    printk("Dummy loaded\n");
    ret = register_chrdev(MAJOR_NUM, DEVICE_NAME, &dummy_fops);
    if (ret != 0){
        return ret;
    }
    dummy_class = class_create(DEVICE_NAME);
    for (int i = 0; i < NUM_DEVICES; i++) {
        device_create(dummy_class, NULL, MKDEV(MAJOR_NUM, i), NULL, "dummy%d", i);
    }
    return 0;
}
void __exit dummy_exit(void)
{
    int i;
    for (int i = 0; i < NUM_DEVICES; i++) {
        device_destroy(dummy_class, MKDEV(MAJOR_NUM, i));
    }
    class_destroy(dummy_class);
    unregister_chrdev(MAJOR_NUM, DEVICE_NAME);
    printk("Dummy unloaded\n");
} 
```

在代码的末尾，名为`module_init`和`module_exit`的宏指定了在模块加载和卸载时要调用的函数：

```
module_init(dummy_init);
module_exit(dummy_exit); 
```

最后的三个宏，命名为`MODULE_*`，添加了一些关于模块的基本信息：

```
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Chris Simmonds");
MODULE_DESCRIPTION("A dummy driver"); 
```

这些信息可以通过`modinfo`命令从编译后的内核模块中获取。完整的源代码以及该驱动程序的`Makefile`可以在`MELD/Chapter11/meta-device-drivers/recipes-kernel/dummy-driver`目录中找到。

当模块被加载时，会调用`dummy_init()`函数。它成为字符设备的时刻，是它调用`register_chrdev`并传递指向`struct file_operations`的指针，`struct file_operations`中包含了驱动程序实现的四个函数的指针。虽然`register_chrdev`告诉内核有一个主设备号为`42`的驱动程序，但它没有说明驱动程序的类别，因此不会在`/sys/class/`中创建条目。

如果`/sys/class/`中没有条目，设备管理器将无法创建设备节点。因此，接下来的几行代码创建了一个名为`dummy`的设备类，并创建了四个该类的设备，分别叫做`dummy0`到`dummy3`。结果是，在驱动程序初始化时，会创建`/sys/class/dummy/`目录，并包含子目录`dummy0`到`dummy3`。每个子目录中都有一个`dev`文件，包含设备的主设备号和次设备号。这就是设备管理器创建设备节点`/dev/dummy0`到`/dev/dummy3`所需要的全部内容。

`dummy_exit()`函数必须通过释放设备类和主设备号来释放`dummy_init()`所申请的资源。

该驱动程序的文件操作由`dummy_open()`、`dummy_read()`、`dummy_write()`和`dummy_release()`实现。当用户空间程序调用`open(2)`、`read(2)`、`write(2)`和`close(2)`时，会分别调用这些函数。它们仅打印内核消息，以便你可以看到它们是否被调用。你可以通过命令行使用`echo`命令来演示这一点：

```
# echo hello > /dev/dummy0
dummy_open
dummy_write 6
dummy_release 
```

在这种情况下，消息会出现是因为我已经登录到控制台，而内核消息默认会打印到控制台。如果你没有登录到控制台，仍然可以通过使用`dmesg`命令查看内核消息。

这个驱动程序的完整源代码不到 100 行，但足以说明设备节点与驱动代码之间的关联、设备类的创建以及数据在用户空间和内核空间之间的传输。接下来，你需要构建它。

## 编译内核模块

此时，你已经有了一些驱动程序代码，想要在目标系统上进行编译和测试。你可以将其复制到内核源代码树中，并修改 makefile 进行构建，或者你也可以将其作为模块进行树外编译。让我们从树外编译开始。

你将需要一个简单的`Makefile`，它利用内核构建系统来完成所有繁重的工作：

```
obj-m := dummy.o
SRC := $(shell pwd)
all:
        $(MAKE) -C $(KERNEL_SRC) M=$(SRC)
modules_install:
        $(MAKE) -C $(KERNEL_SRC) M=$(SRC) modules_install
clean:
        rm -f *.o *~ core .depend .*.cmd *.ko *.mod.c
        rm -f Module.markers Module.symvers modules.order
        rm -rf .tmp_versions Modules.symvers 
```

Yocto 将 `KERNEL_SRC` 设置为目标设备的内核目录，你将在该设备上运行该模块。`obj-m := dummy.o` 代码将调用内核构建规则，将 `dummy.c` 源文件转换为 `dummy.ko` 内核模块。我将在下一节向你展示如何加载内核模块。

**重要提示**

内核模块在内核发布版本和配置之间不具有二进制兼容性：该模块只能在它编译的内核上加载。

如果你想在内核源码树中构建一个驱动程序，过程非常简单。选择一个适合你驱动程序类型的目录。这个驱动程序是一个基本的字符设备，所以我会把`dummy.c`放在`drivers/char/`目录下。接着，编辑目录中的 makefile，并添加一行代码，无条件地构建这个驱动程序作为一个模块，像这样：

```
obj-m += dummy.o 
```

或者，你可以添加以下行来无条件地将其构建为内置模块：

```
obj-y += dummy.o 
```

如果你希望驱动程序是可选的，你可以在 `Kconfig` 文件中添加一个菜单选项，并且根据配置选项进行条件编译，就像我在 *第四章* 的 *理解内核配置* 部分描述的那样。

## 加载内核模块

你可以使用简单的 `modprobe`、`lsmod` 和 `rmmod` 命令来加载、列出和卸载模块。这里，它们正在加载和卸载虚拟驱动程序：

```
# modprobe dummy
# lsmod
Module                  Size  Used by
dummy                  12288  0
# rmmod dummy 
```

如果模块被放置在 `/lib/modules/<kernel release>` 的子目录中，你可以使用 `depmod -a` 命令创建一个模块依赖数据库，像这样：

```
# depmod -a
# ls /lib/modules/6.12.9-ti-g8906665ace32
modules.alias              modules.builtin.modinfo    modules.softdep
modules.alias.bin          modules.dep                modules.symbols
modules.builtin            modules.dep.bin            modules.symbols.bin
modules.builtin.alias.bin  modules.devname            updates
modules.builtin.bin        modules.order 
```

`modules.*` 文件中的信息由 `modprobe` 命令用于按名称而非完整路径定位模块。`modprobe` 还有许多其他功能，所有这些功能都在 `modprobe(8)` 手册页中有描述。

现在我们已经编写并加载了虚拟内核模块，那么我们如何让它与真实硬件交流？我们需要通过设备树或平台数据将我们的驱动程序绑定到该硬件上。发现硬件并将其与设备驱动程序绑定是下一节讨论的主题。

# 发现硬件配置

虚拟驱动程序展示了设备驱动程序的结构，但它缺乏与真实硬件的交互，因为它只操纵内存结构。设备驱动程序通常被编写来与硬件交互。其中一部分是能够首先发现硬件，记住它在不同配置中可能在不同地址。

在某些情况下，硬件本身提供信息。在可发现总线上的设备（如 PCI 或 USB）具有查询模式，返回资源需求和唯一标识符。内核将标识符及可能的其他特征与设备驱动程序进行匹配。

然而，大多数嵌入式板上的硬件模块没有这样的标识符。你必须以 **设备树** 的形式或称为 **平台数据** 的 C 结构形式提供信息。

在 Linux 的标准驱动模型中，设备驱动会在适当的子系统中注册自己：PCI、USB、开放固件（设备树）、平台设备等。注册包括一个标识符和一个回调函数，即 `probe` 函数，只有当硬件的 ID 与驱动的 ID 匹配时，`probe` 函数才会被调用。对于 PCI 和 USB，ID 是基于设备的厂商和产品 ID。而对于设备树和平台设备，ID 是一个名称（文本字符串）。

## 设备树

我在*第三章*中给你介绍了设备树。在这里，我想向你展示 Linux 设备驱动如何与这些信息连接。

举个例子，我将使用 Arm Versatile 板（`arch/arm/boot/dts/versatile-ab.dts`），该板定义了以太网适配器：

```
net@10010000 {
   compatible = "smsc,lan91c111";
   reg = <0x10010000 0x10000>;
   interrupts = <25>;
}; 
```

特别注意此节点的 `compatible` 属性。这个字符串值稍后会在以太网适配器的源代码中再次出现。我们将在*第十二章*中进一步学习设备树。

## 平台数据

在没有设备树支持的情况下，有一种回退方法，通过使用 C 结构体描述硬件，这种方法称为平台数据。

每个硬件组件都通过`struct platform_device`进行描述，该结构体包含一个名称和指向资源数组的指针。资源的类型由标志决定，标志包括以下内容：

+   `IORESOURCE_MEM`：这是一个内存区域的物理地址。

+   `IORESOURCE_IO`：这是 I/O 寄存器的物理地址或端口号。

+   `IORESOURCE_IRQ`：这是中断号。

这里是来自 `arch/arm/machversatile/core.c` 的以太网控制器平台数据示例，已进行编辑以提高可读性：

```
#define VERSATILE_ETH_BASE 0x10010000
#define IRQ_ETH 25
static struct resource smc91x_resources[] = {
[0] = {
   .start = VERSATILE_ETH_BASE,
   .end = VERSATILE_ETH_BASE + SZ_64K - 1,
   .flags = IORESOURCE_MEM,
 },
 [1] = {
   .start = IRQ_ETH,
   .end = IRQ_ETH,
   .flags = IORESOURCE_IRQ,
 },
};
static struct platform_device smc91x_device = {
  .name = "smc91x",
  .id = 0,
  .num_resources = ARRAY_SIZE(smc91x_resources),
  .resource = smc91x_resources,
}; 
```

它有一个 64 KB 的内存区域和一个中断。平台数据通常会在板卡初始化时注册到内核：

```
void __init versatile_init(void)
{
   platform_device_register(&versatile_flash_device);
   platform_device_register(&versatile_i2c_device);
   platform_device_register(&smc91x_device);
<…> 
```

这里展示的的平台数据在功能上与之前的设备树源代码等效，唯一不同的是 `name` 字段，它取代了 `compatible` 属性的位置。

## 硬件与设备驱动的连接

在前面的部分中，你已经看到如何使用设备树或平台数据描述以太网适配器。对应的驱动代码在 `drivers/net/ethernet/smsc/smc91x.c` 中，它同时支持设备树和平台数据。

这里是初始化代码，再次进行了编辑以提高可读性：

```
static const struct of_device_id smc91x_match[] = {
   { .compatible = "smsc,lan91c94", },
   { .compatible = "smsc,lan91c111", },
   {},
};
MODULE_DEVICE_TABLE(of, smc91x_match);
static struct platform_driver smc_driver = {
    .probe = smc_drv_probe,
    .remove = smc_drv_remove,
    .driver = {
        .name = "smc91x",
        .of_match_table = of_match_ptr(smc91x_match),
},
};
static int __init smc_driver_init(void)
{
    return platform_driver_register(&smc_driver);
}
static void __exit smc_driver_exit(void)
{
    platform_driver_unregister(&smc_driver);
}
module_init(smc_driver_init);
module_exit(smc_driver_exit); 
```

当驱动程序初始化时，它会调用 `platform_driver_register()`，并指向 `struct platform_driver`，其中包含一个指向 `probe` 函数的回调、驱动名称 `smc91x` 和指向 `struct of_device_id` 的指针。

如果该驱动已经由设备树配置，内核将查找设备树节点中的 `compatible` 属性与 compatible 结构元素指向的字符串之间的匹配。对于每个匹配，它都会调用 `probe` 函数。

另一方面，如果它是通过平台数据配置的，则`probe`函数会针对`driver.name`指向的字符串中的每个匹配项进行调用。

`probe`函数提取有关接口的信息：

```
static int smc_drv_probe(struct platform_device *pdev)
{
   struct smc91x_platdata *pd = dev_get_platdata(&pdev->dev);
   const struct of_device_id *match = NULL;
   struct resource *res, *ires;
   int irq;
   res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
   ires = platform_get_resource(pdev, IORESOURCE_IRQ, 0);
   <…>
   addr = ioremap(res->start, SMC_IO_EXTENT);
   irq = ires->start;
   <…>
} 
```

调用`platform_get_resource()`可以从设备树或平台数据中提取内存和`irq`信息。由驱动程序负责映射内存并安装中断处理程序。第三个参数（前两种情况中的`0`）在存在多个该类型资源时发挥作用。

```
register-io-width property:
```

```
match = of_match_device(of_match_ptr(smc91x_match), &pdev->dev);
if (match) {
   struct device_node *np = pdev->dev.of_node;
   u32 val;
   <…>
   of_property_read_u32(np, "reg-io-width", &val);
   <…>
} 
```

对于大多数驱动程序，具体的绑定信息可以在`Documentation/devicetree/bindings/`中找到。对于这个特定的驱动程序，信息存放在`Documentation/devicetree/bindings/net/smsc,lan9115.yaml`中。

这里需要记住的主要内容是，驱动程序应该注册一个`probe`函数，并提供足够的信息，以便内核在找到与已知硬件匹配的设备时能够调用`probe`函数。设备树描述的硬件与设备驱动程序之间的关联是通过`compatible`属性完成的。平台数据与驱动程序之间的关联则是通过名称完成的。

# 总结

设备驱动程序负责处理设备，通常是物理硬件，但有时也包括虚拟接口，并以一致且有用的方式将其呈现给用户空间。Linux 设备驱动程序分为三大类：字符设备、块设备和网络设备。三者中，字符设备接口最为灵活，因此也是最常见的。Linux 驱动程序适配于一个名为驱动模型（driver model）的框架，该框架通过`sysfs`对外暴露。几乎所有设备和驱动程序的状态都可以在`/sys/`中看到。

每个嵌入式系统都有其独特的硬件接口和要求。Linux 为大多数标准接口提供了驱动程序，通过选择正确的内核配置，您可以非常快速地获得一个可用的目标板。这时，剩下的就是那些非标准组件，您需要为它们添加自己的设备支持。

在某些情况下，您可以通过使用通用驱动程序来避开这个问题，例如 GPIO、I2C 和 SPI，然后编写用户空间代码来完成工作。我推荐将此作为起点，因为它可以让您在不编写内核代码的情况下熟悉硬件。编写内核驱动程序并不特别困难，但您需要小心编写代码，以免破坏系统的稳定性。

我已经讲解了如何编写内核驱动代码：如果您选择这条路，您不可避免地想知道如何检查它是否正常工作以及如何检测错误。我将在*第十九章*中讲解这个话题。

下一章展示了如何使用单板计算机和附加板进行快速原型设计的技巧。

# 进一步学习

+   *Linux 内核开发，第 3 版*，作者：Robert Love

+   *Linux 每周新闻* – [`lwn.net/Kernel`](https://lwn.net/Kernel%0D%0A)

+   *《Linux 上的异步 IO：select、poll 和 epoll》*，作者：Julia Evans – [`jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll/`](https://jvns.ca/blog/2017/06/03/async-io-on-linux--select--poll--and-epoll/%0D%0A)

+   *《Linux 必备设备驱动程序，第 1 版》*，作者：Sreekrishnan Venkateswaran

# 加入我们的社区，参与 Discord 讨论

加入我们社区的 Discord 空间，和作者及其他读者一起讨论： https://packt.link/embeddedsystems

![](img/QR_Code12308107448340296.png)
