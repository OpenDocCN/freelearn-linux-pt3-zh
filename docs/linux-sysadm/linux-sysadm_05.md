

# 硬件发现

了解如何找出操作系统运行在哪些硬件上，以及有哪些外设连接，是所有系统管理员必须具备的技能——至少，每个管理员都需要知道有多少个 CPU 核心，以及可以分配给应用程序的内存有多少。在本章中，我们将学习如何使用原始内核接口和工具来检索并解释关于 CPU、USB 外设和存储设备的信息。我们还将介绍一些专门针对具有 SMBIOS/DMI 接口的 x86 机器的工具。

在本章中，我们将讨论以下内容：

+   如何发现 CPU 的数量、型号名称和特性

+   如何发现 PCI、USB 和 SCSI 外设

+   如何使用特定平台的工具从系统固件中检索详细的硬件信息

# 发现 CPU 型号和特性

中央处理器无疑是最重要的硬件组件之一，了解它的详细信息有很多原因。CPU 的型号名称或编号和频率是你首先查看的内容，用来了解机器的使用年限和整体性能。然而，实际上还有更多的细节信息常常是有用的。例如，CPU 核心的数量是很重要的，尤其是当你运行支持多个工作线程或进程的应用程序时（比如`make -j2`）。如果运行的进程数超过了 CPU 的数量，可能会导致应用程序变慢，因为一些进程最终需要等待可用的 CPU。因此，你可能需要运行较少的工作进程，以避免过度加载机器。

了解你的 CPU 是否支持特定的加速技术，如 AES-NI 或 Intel QuickAssist 也很重要。如果支持这些加速功能，启用后某些应用程序的性能可能会大幅提升。

## 不同平台上的特性发现

你需要记住的一点是，关于 CPU 信息的发现，主要是 CPU 本身的特性，而不是 Linux 的功能。如果 CPU 无法报告某些信息，那么通过软件来查找这些信息可能会非常困难，甚至不可能。

许多 ARM 和 MIPS 架构的 CPU 并不会报告它们的确切型号名称，因此唯一的办法可能就是打开机箱，查看芯片上的信息。

例如，流行的树莓派 4 单板计算机使用的是 ARM Cortex A72 CPU，但它并没有告诉你这一点。

```
pi@raspberrypi4   :~ $ cat /proc/cpuinfo
processor         : 0
model name        : ARMv7 Processor rev 3 (v7l)
BogoMIPS          : 108.00
Features          : half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer   : 0x41
CPU architecture  : 7
CPU variant       : 0x0
CPU part          : 0xd08
CPU revision      : 3
```

我们将重点关注由 AMD 和 Intel 制造的 x86 架构 CPU，因为它们仍然是嵌入式系统市场以外最常见的 CPU，而且它们具有最广泛的特性报告功能。

## /proc/cpuinfo 文件

当我们在*第四章*《进程与进程控制》中讨论进程控制时，我们谈到了`/proc`文件系统。这个文件系统是虚拟的——`/proc`下的文件并不实际存在于磁盘上，内核只是根据 Unix 哲学中的“**一切皆文件**”原则，像存储在文件中一样呈现这些信息。

除了有关运行进程的信息外，内核还使用`/proc`来获取关于 CPU 和内存的信息。包含 CPU 信息的文件名为`/proc/cpuinfo`，正如我们之前所看到的。

现在，让我们来看一下 Intel 机器上的那个文件：

```
$ cat /proc/cpuinfo
processor         : 0
vendor_id         : GenuineIntel
cpu family        : 6
model             : 44
model name        : Intel(R) Xeon(R) CPU            L5630  @ 2.13GHz
stepping          : 2
microcode         : 0x1a
cpu MHz           : 2128.000
cache size        : 12288 KB
physical id       : 0
siblings          : 1
core id           : 0
cpu cores         : 1
apicid            : 0
initial apicid    : 0
fpu               : yes
fpu_exception     : yes
cpuid level       : 11
wp                : yes
flags             : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts nopl xtopology tsc_reliable nonstop_tsc cpuid aperfmperf pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes hypervisor lahf_lm pti tsc_adjust dtherm ida arat
bugs              : cpu_meltdown spectre_v1 spectre_v2 spec_store_bypass l1tf mds swapgs itlb_multihit
bogomips          : 4256.00
clflush size      : 64
cache_alignment   : 64
address sizes     : 42 bits physical, 48 bits virtual
power management  :
processor         : 1
vendor_id         : GenuineIntel
cpu family        : 6
model             : 44
model name        : Intel(R) Xeon(R) CPU            L5630  @ 2.13GHz
stepping          : 2
microcode         : 0x1a
cpu MHz           : 2128.000
cache size        : 12288 KB
physical id       : 2
siblings          : 1
core id           : 0
cpu cores         : 1
apicid            : 2
initial apicid    : 2
fpu               : yes
fpu_exception     : yes
cpuid level       : 11
wp                : yes
flags             : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts nopl xtopology tsc_reliable nonstop_tsc cpuid aperfmperf pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes hypervisor lahf_lm pti tsc_adjust dtherm ida arat
bugs              : cpu_meltdown spectre_v1 spectre_v2 spec_store_bypass l1tf mds swapgs itlb_multihit
bogomips          : 4256.00
clflush size      : 64
cache_alignment   : 64
address sizes     : 42 bits physical, 48 bits virtual
power management  :
```

有些信息是显而易见的，不需要解释。型号是 Intel Xeon L5630，频率为 2.13 GHz，并且有 12 KB 的缓存。CPU 供应商名称看起来像`GenuineIntel`和`AuthenticAMD`，是因为内部供应商字符串是以三个 32 位值存储的，共 12 字节。注意`GenuineIntel`和`AuthenticAMD`都恰好是 12 个字符——它们被选中填充这 12 字节，以避免不同系统上对空字节的不同解释问题。

一个需要注意的字段是`bogomips`。它的名称暗示了**每秒百万条指令**（**MIPS**）的性能指标，但这个值并不是衡量整体性能的有用指标。Linux 内核用它进行内部校准，你不应该用它来比较不同 CPU 的性能。

`flags`字段的信息密度最高，也是最难解读的。标志只是 Flags 寄存器中的一位，这些位的含义差异很大。有些标志甚至没有在硬件 CPU 中设置，例如，超管标志，它表示系统在虚拟机上运行（但其缺失并不意味着什么，因为并非所有超管都设置此标志）。

许多标志指示是否存在某个特性，虽然解读它们的性质和重要性需要熟悉供应商的术语。例如，来自 Raspberry Pi 输出的 Neon 特性是一组适用于 ARM CPU 的**单指令多数据**（**SIMD**）指令，与 Intel 的 SSE 相当，常用于多媒体和科学应用。当不确定时，请查阅 CPU 供应商文档。

## 多处理器系统

市面上大多数 CPU 都包含多个 CPU 核心，而服务器主板通常有多个插槽。`cpuinfo`文件包含了确定 CPU 插槽和核心布局所需的所有信息，但也有一些注意事项。

你应该查看的字段以确定 CPU 插槽数量是`physical id`。注意，`physical id`值并不总是连续的，因此你不能仅仅查看最大值，而应该考虑所有存在的值。你可以通过将`cpuinfo`文件通过`sort`和`uniq`命令管道处理来查找唯一的 ID：

```
$ cat /proc/cpuinfo | grep "physical id" | sort | uniq
physical id    : 0
physical id    : 2
```

在运行在 x86 硬件上的虚拟机中，所有超管程序呈现给它们的 CPU 都会显示得像是在不同的物理插槽中。例如，如果你有一台主机，配备单个四核 CPU，并创建了一个有两个虚拟 CPU 的虚拟机，它会看起来像是两个单核 CPU 位于不同的插槽中。

在许多非 x86 架构上，如 ARM，所有的 CPU 看起来都像是不同的物理 CPU，无论它们是不同的芯片，还是同一芯片上的不同核心。

在裸机 x86 机器上，你可以找到两个不同物理 ID 的`cpuinfo`条目，并进一步检查它们，以找出每个插槽中的核心数量。

请考虑这台运行 Linux 系统的笔记本电脑的`cpuinfo`条目：

```
processor         : 7
vendor_id         : GenuineIntel
cpu family        : 6
model             : 140
model name        : 11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
stepping          : 1
microcode         : 0xa4
cpu MHz           : 1029.707
cache size        : 8192 KB
physical id       : 0
siblings          : 8
core id           : 3
cpu cores         : 4
apicid            : 7
initial apicid    : 7
fpu               : yes
fpu_exception     : yes
cpuid level       : 27
wp                : yes
flags             : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb cat_l2 invpcid_single cdp_l2 ssbd ibrs ibpb stibp ibrs_enhanced tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid rdt_a avx512f avx512dq rdseed adx smap avx512ifma clflushopt clwb intel_pt avx512cd sha_ni avx512bw avx512vl xsaveopt xsavec xgetbv1 xsaves split_lock_detect dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp hwp_pkg_req avx512vbmi umip pku ospke avx512_vbmi2 gfni vaes vpclmulqdq avx512_vnni avx512_bitalg avx512_vpopcntdq rdpid movdiri movdir64b fsrm avx512_vp2intersect md_clear ibt flush_l1d arch_capabilities
vmx flags    : vnmi preemption_timer posted_intr invvpid ept_x_only ept_ad ept_1gb flexpriority apicv tsc_offset vtpr mtf vapic ept vpid unrestricted_guest vapic_reg vid ple pml ept_mode_based_exec tsc_scaling
bugs              : spectre_v1 spectre_v2 spec_store_bypass swapgs eibrs_pbrsb
bogomips          : 4838.40
clflush size      : 64
cache_alignment   : 64
address sizes     : 39 bits physical, 48 bits virtual
power management  :
```

从`cpu cores`字段中可以看出，它是一个四核 CPU。

请注意，条目中有`processor: 7`，并且`siblings`字段设置为`8`。看起来系统有八个 CPU，即使它显然只有一个物理 CPU 芯片（所有条目都有`physical id: 0`），而每个条目中报告的核心数为`4`。

这是由*同时多线程*技术（如 AMD SMT 和 Intel 超线程技术）引起的。这些技术允许单个 CPU 核心维持多个执行线程的状态，从而加速某些应用程序。如果每个核心同时支持两个线程，那么操作系统看起来就像机器的 CPU 核心数量是实际数量的两倍。因此，仅通过查看最高的`/proc/cpuinfo`条目编号，无法确定物理核心的数量，因此你需要更仔细地检查它。

GNU `coreutils`包中有一个名为`nproc`的工具，它的作用是输出系统中 CPU 核心的数量，如果你需要了解 CPU 的数量，它可以简化你的工作——例如，确定一个应用程序需要生成多少个工作进程或线程。然而，它没有考虑到同时多线程技术（SMT），并且如果启用了 SMT 技术，它会打印出虚拟核心的数量。如果你的应用程序需要物理核心的数量，不应依赖`nproc`的输出。以下是该机器上`nproc`输出的样子：

```
$ nproc
8
```

## 高级 CPU 发现工具

如你所见，读取`/proc/cpuinfo`文件可能是一项繁琐且复杂的任务。为此，人们创建了旨在简化这项任务的工具。该领域中最流行的工具是来自`util-linux`包的`lscpu`。

它提供了一个更易读的输出：

```
$ lscpu
Architecture         : x86_64
CPU op-mode(s)       : 32-bit, 64-bit
Byte Order           : Little Endian
Address sizes        : 42 bits physical, 48 bits virtual
CPU(s)               : 2
On-line CPU(s) list  : 0,1
Thread(s) per core   : 1
Core(s) per socket   : 1
Socket(s)            : 2
NUMA node(s)         : 1
Vendor ID            : GenuineIntel
CPU family           : 6
Model                : 44
Model name           : Intel(R) Xeon(R) CPU            L5630  @ 2.13GHz
Stepping             : 2
CPU MHz              : 2128.000
BogoMIPS             : 4256.00
Hypervisor vendor    : VMware
Virtualization type  : full
L1d cache            : 32K
L1i cache            : 32K
L2 cache             : 256K
L3 cache             : 12288K
NUMA node0 CPU(s)    : 0,1
Flags                : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts mmx fxsr sse sse2 syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts nopl xtopology tsc_reliable nonstop_tsc cpuid aperfmperf pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes hypervisor lahf_lm pti tsc_adjust dtherm ida arat
```

另一个优点是，你可以通过运行`lscpu --json`获得机器可读的 JSON 输出，并通过脚本分析它。

但请注意，`lscpu`没有考虑同时多线程问题（至少在版本 2.38 中是这样），如果启用了 AMD SMT 或 Intel 超线程技术，它会报告两倍的 CPU 数量。

如你所见，你可以直接从`/proc`文件系统获取关于 CPU 的许多信息，或者使用高层工具简化这个过程。然而，你应该始终记住一些解释的细微差别，比如核心数量可能被同时多线程技术膨胀的问题。

# 内存发现

发现内存的数量往往比发现 CPU 特性更为实际和重要。它是计划应用程序部署、选择交换分区大小以及估计是否需要安装更多内存时所必需的。

然而，内核提供的内存发现接口并不像 CPU 特征发现接口那样丰富。例如，单独通过内核接口无法得知系统有多少内存插槽，有多少已被使用，以及安装在这些插槽中的内存条的大小。至少在某些架构上，可以通过固件而不是内核获得这些信息，正如我们稍后在*dmidecode*部分中看到的那样。

此外，来自内核的信息可能会误导那些不熟悉 Linux 内核约定的初学者。首先，让我们看一下这些信息，然后讨论如何解读它们。

首先，我们将查看来自`procps-ng`包的`free`工具的输出。该工具有`-m`选项，可以以 MB 为单位显示内存量，而不是以 KiB 为单位，这样更容易阅读。

![图 5.1 – free –m 输出](img/B18575_05_01.jpg)

图 5.1 – free –m 输出

表面上看，它的输出似乎是自描述的。这里的总内存量大约是 16GB，其中`13849`MB 已被使用。然而，内核如何使用内存并非简单的问题。并非所有声明为已使用的内存都被用户应用程序使用——内核也使用内存来缓存磁盘数据，以加速输入输出操作。系统中未被任何进程使用的内存越多，Linux 用来缓存的内存就越多。当没有足够的完全未使用的内存分配给请求它的进程时，内核会驱逐一些缓存数据以释放空间。因此，即使系统运行的应用程序非常少，`used`列中显示的内存量也可能非常高，这可能给人一种系统内存不足的印象，实际上，应用程序仍有大量内存可用。用于磁盘缓存的内存量在`buff/cache`列中，而可供应用程序使用的内存量在`available`列中。

关于内存消耗的原始信息可以在`/proc/meminfo`文件中找到。`free`工具的输出是该文件信息的总结。

# 发现 PCI 设备

许多外设都连接到 PCI 总线。如今，这通常意味着 **PCI Express** (**PCI-e**) 而非更旧的 PCI 或 PCI-x 总线，但从软件的角度来看，它们都是 PCI 设备，不论它们连接的是哪一种总线。

内核通过 `/sys/class/pci_bus` 层次结构暴露这些信息，但手动读取这些文件将是一项非常耗时的工作，和 `/proc/cpuinfo` 不同，因此实际上，人们总是使用工具来处理它。最流行的工具是来自 `pciutils` 包的 `lspci`。

这是一个示例输出：

```
$ sudo lspci
00:00.0 Host bridge: Intel Corporation 11th Gen Core Processor Host Bridge/DRAM Registers (rev 01)
00:02.0 VGA compatible controller: Intel Corporation TigerLake-LP GT2 [Iris Xe Graphics] (rev 01)
00:04.0 Signal processing controller: Intel Corporation TigerLake-LP Dynamic Tuning Processor Participant (rev 01)
00:06.0 PCI bridge: Intel Corporation 11th Gen Core Processor PCIe Controller (rev 01)
00:0a.0 Signal processing controller: Intel Corporation Tigerlake Telemetry Aggregator Driver (rev 01)
00:14.0 USB controller: Intel Corporation Tiger Lake-LP USB 3.2 Gen 2x1 xHCI Host Controller (rev 20)
00:14.2 RAM memory: Intel Corporation Tiger Lake-LP Shared SRAM (rev 20)
00:14.3 Network controller: Intel Corporation Wi-Fi 6 AX201 (rev 20)
00:15.0 Serial bus controller: Intel Corporation Tiger Lake-LP Serial IO I2C Controller #0 (rev 20)
00:15.1 Serial bus controller: Intel Corporation Tiger Lake-LP Serial IO I2C Controller #1 (rev 20)
00:16.0 Communication controller: Intel Corporation Tiger Lake-LP Management Engine Interface (rev 20)
00:17.0 SATA controller: Intel Corporation Tiger Lake-LP SATA Controller (rev 20)
00:1c.0 PCI bridge: Intel Corporation Device a0bc (rev 20)
00:1d.0 PCI bridge: Intel Corporation Tiger Lake-LP PCI Express Root Port #9 (rev 20)
00:1f.0 ISA bridge: Intel Corporation Tiger Lake-LP LPC Controller (rev 20)
00:1f.4 SMBus: Intel Corporation Tiger Lake-LP SMBus Controller (rev 20)
00:1f.5 Serial bus controller: Intel Corporation Tiger Lake-LP SPI Controller (rev 20)
01:00.0 3D controller: NVIDIA Corporation GP108M [GeForce MX330] (rev a1)
02:00.0 Non-Volatile memory controller: Samsung Electronics Co Ltd NVMe SSD Controller 980
03:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 15)
```

完全访问 PCI 总线信息仅对 `root` 用户开放，因此你应该使用 `sudo` 来运行 `lspci`。PCI 设备类别是标准化的，所以该工具不仅会告诉你设备的厂商和型号，还会告诉你它们的功能，如以太网控制器或 SATA 控制器。

其中一些名称可能会有些误导——例如，一个兼容 VGA 的控制器可能没有实际的 VGA 端口；如今，它几乎总是 DVI、HDMI 或 Thunderbolt/DisplayPort。

# 发现 USB 设备

要发现 USB 设备，可以使用来自 `usbutils` 包的 `lsusb` 工具。这个命令不需要 `root` 权限。以下是它的输出可能的样子：

```
$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 005: ID 1bcf:2b98 Sunplus Innovation Technology Inc. Integrated_Webcam_HD
Bus 001 Device 036: ID 0b0e:0300 GN Netcom Jabra EVOLVE 20 MS
Bus 001 Device 035: ID 05ac:12a8 Apple, Inc. iPhone 5/5C/5S/6/SE
Bus 001 Device 006: ID 8087:0aaa Intel Corp. Bluetooth 9460/9560 Jefferson Peak (JfP)
Bus 001 Device 002: ID 047d:1020 Kensington Expert Mouse Trackball
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

尽管 USB 总线规范也包括标准化的设备类别，但 `lsusb` 默认并不会显示它们。这样做的一个原因是一个 USB 设备可能实现多个功能。例如，智能手机可以作为一个大容量存储设备（类似于 USB 闪存盘）用于常规文件传输，也可以作为数字相机，通过 **图片传输协议** (**PTP**) 获取其中的照片。

你可以通过运行带有附加选项的 `lsusb` 来获取设备的详细信息。在之前的例子中，运行 `lsusb -s 2 -v` 会显示关于设备 `002`（一个 Kensington 滚珠轨迹球）的信息。

请注意，如果设备出现在 `lsusb` 的输出中，并不总是意味着它连接到了 USB 端口。板载设备也可能连接到 USB 总线。在前面的例子中，蓝牙控制器是笔记本电脑中的一项内部设备。

# 发现存储设备

存储设备可以连接到不同的总线，因此发现它们可能比较复杂。一个有用的命令是来自 `util-linux` 包的 `lsblk`（其名称代表列出块设备），可以用来发现所有看起来像存储设备的设备：

```
$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
zram0  251:0     0  959M  0 disk [SWAP]
vda    252:0     0   25G  0 disk
├─vda1 252:1     0    1M  0 part
├─vda2 252:2     0  500M  0 part /boot
├─vda3 252:3     0  100M  0 part /boot/efi
├─vda4 252:4     0    4M  0 part
└─vda5 252:5     0 24.4G  0 part /home
                                 /
```

需要注意的是，它会显示虚拟设备以及物理设备。例如，如果你通过虚拟循环设备挂载一个 ISO 镜像，它将作为存储设备显示——因为从用户的角度来看，它确实是一个存储设备：

```
$ sudo mount -t iso9660 -o ro,loop /tmp/some_image.iso /mnt/iso/
$ lsblk
NAME                                           MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
loop0                                            7:0     0   1.7G  1 loop  /mnt/tmp
├─loop0p1                                      259:4     0     30K  1 part
└─loop0p2                                      259:5     0   4.3M  1 part
```

如果你想发现物理设备，可以尝试使用同样通常叫做 `lsscsi` 的包中的 `lsscsi` 命令：

```
$ lsscsi
[N:0:5:1]     disk     PM991a NVMe Samsung 512GB__1           /dev/nvme0n1
```

一个令人困惑的部分是，原始并行 SCSI 总线的*协议*发现了许多新应用，并且仍然被广泛使用，尽管其原始硬件实现已经被更新的总线所替代。该工具的输出中会显示许多设备，包括 SATA、**串行连接 SCSI**（**SAS**）和 NVMe 驱动器，以及 USB 大容量存储设备。

相反，像 KVM 和 Xen 虚拟机中的 VirtIO 驱动器这样的半虚拟设备将不会出现在 `lsscsi` 输出中。总的来说，您可能需要依赖 `lsusb`、`lsscsi` 和 `lsblk` 的组合来获取完整的情况。一个优点是，这些命令都不需要 `root` 权限。关于 `lsblk` 的另一个好处是，您可以运行 `lsblk --json` 来获取机器可读的输出，并将其加载到脚本中。

# 高级发现工具

除了这些特定于某个总线或设备类型的工具外，还有一些工具可以帮助您发现系统中所有存在的硬件。

## dmidecode

在 x86 系统上，您可以使用名为 `dmidecode` 的程序通过桌面管理接口（因此得名）从固件（BIOS/UEFI）中检索和查看信息。该接口也称为 SMBIOS。由于它是特定于 x86 PC 固件标准的，因此在其他架构的机器上（如 ARM 或 MIPS）无法使用，但在 x86 笔记本电脑、工作站和服务器上，它可以帮助您发现无法通过其他方式获得的信息，例如 RAM 插槽的数量。

一个缺点是，您需要 `root` 权限才能运行它。另一个需要注意的事项是，它会产生大量输出。由于完整的输出内容太长，不可能在书中重现，但作为参考，下面是它的输出开头部分（在此案例中是在 VMware 虚拟机中）：

```
$ sudo dmidecode
# dmidecode 3.2
Getting SMBIOS data from sysfs.
SMBIOS 2.4 present.
556 structures occupying 27938 bytes.
Table at 0x000E0010.
Handle 0x0000, DMI type 0, 24 bytes
BIOS Information
    Vendor: Phoenix Technologies LTD
    Version: 6.00
    Release Date: 09/21/2015
    Address: 0xE99E0
    Runtime Size: 91680 bytes
    ROM Size: 64 kB
    Characteristics:
         ISA is supported
         PCI is supported
         PC Card (PCMCIA) is supported
         PNP is supported
         APM is supported
         BIOS is upgradeable
         BIOS shadowing is allowed
         ESCD support is available
         Boot from CD is supported
         Selectable boot is supported
         EDD is supported
         Print screen service is supported (int 5h)
         8042 keyboard services are supported (int 9h)
         Serial services are supported (int 14h)
         Printer services are supported (int 17h)
         CGA/mono video services are supported (int 10h)
         ACPI is supported
         Smart battery is supported
         BIOS boot specification is supported
         Function key-initiated network boot is supported
         Targeted content distribution is supported
    BIOS Revision: 4.6
    Firmware Revision: 0.0
Handle 0x0001, DMI type 1, 27 bytes
System Information
    Manufacturer: VMware, Inc.
    Product Name: VMware Virtual Platform
    Version: None
    Serial Number: VMware-56 4d 47 fd 6f 6f 20 30-f8 85 d6 08 bf 09 fe b5
    UUID: 564d47fd-6f6f-2030-f885-d608bf09feb5
    Wake-up Type: Power Switch
    SKU Number: Not Specified
    Family: Not Specified
Handle 0x0002, DMI type 2, 15 bytes
Base Board Information
    Manufacturer: Intel Corporation
    Product Name: 440BX Desktop Reference Platform
    Version: None
    Serial Number: None
    Asset Tag: Not Specified
    Features: None
    Location In Chassis: Not Specified
    Chassis Handle: 0x0000
    Type: Unknown
    Contained Object Handles: 0
```

正如您在此输出中看到的，机箱和主板都有精确的型号名称。与例如 USB 这样的设备不同，USB 协议本身包含允许设备向主机报告其名称和功能的特性，而查询主板名称没有跨平台的方式，但至少在大多数 x86 机器上，这些信息可以通过 DMI 接口获得。

## lshw

另一个提供全面硬件报告的工具是 `lshw`（LiSt HardWare）。与 `dmidecode` 类似，它也可以使用 DMI 接口作为信息来源，但它尝试支持更多的硬件平台及其平台特定的硬件发现接口。

一个缺点是，流行的发行版默认没有安装该工具，因此您需要手动从软件库中安装它。它还需要 `root` 权限才能运行，因为它依赖于特权访问 DMI。

这是一个在 DigitalOcean 云平台的 KVM 虚拟机中运行的样本输出。它的输出也非常大，因此我们只包括其开头部分：

```
$ sudo lshw
my-host
     description: Computer
     product: Droplet
     vendor: DigitalOcean
     version: 20171212
     serial: 293265963
     width: 64 bits
     capabilities: smbios-2.4 dmi-2.4 vsyscall32
     configuration: boot=normal family=DigitalOcean_Droplet uuid=AE220510-4AD6-4B66-B49F-9D103C60BA5A
  *-core
        description: Motherboard
        physical id: 0
      *-firmware
           description: BIOS
           vendor: DigitalOcean
           physical id: 0
           version: 20171212
           date: 12/12/2017
           size: 96KiB
      *-cpu
           description: CPU
           product: DO-Regular
           vendor: Intel Corp.
           physical id: 401
           bus info: cpu@0
           slot: CPU 1
           size: 2GHz
           capacity: 2GHz
           width: 64 bits
           capabilities: fpu fpu_exception wp vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ss syscall nx rdtscp x86-64 constant_tsc arch_perfmon rep_good nopl cpuid tsc_known_freq pni pclmulqdq vmx ssse3 fma cx16 pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand hypervisor lahf_lm abm cpuid_fault invpcid_single pti ssbd ibrs ibpb tpr_shadow vnmi flexpriority ept vpid ept_ad fsgsbase tsc_adjust bmi1 avx2 smep bmi2 erms invpcid xsaveopt md_clear
      *-memory
           description: System Memory
           physical id: 1000
           size: 1GiB
           capacity: 1GiB
           capabilities: ecc
           configuration: errordetection=multi-bit-ecc
         *-bank
              description: DIMM RAM
              physical id: 0
              slot: DIMM 0
              size: 1GiB
              width: 64 bits
      *-pci
           description: Host bridge
           product: 440FX - 82441FX PMC [Natoma]
           vendor: Intel Corporation
           physical id: 100
           bus info: pci@0000:00:00.0
           version: 02
           width: 32 bits
           clock: 33MHz
         *-isa
              description: ISA bridge
              product: 82371SB PIIX3 ISA [Natoma/Triton II]
              vendor: Intel Corporation
              physical id: 1
              bus info: pci@0000:00:01.0
              version: 00
              width: 32 bits
              clock: 33MHz
              capabilities: isa
              configuration: latency=0
         *-ide
              description: IDE interface
              product: 82371SB PIIX3 IDE [Natoma/Triton II]
              vendor: Intel Corporation
              physical id: 1.1
              bus info: pci@0000:00:01.1
              version: 00
              width: 32 bits
              clock: 33MHz
              capabilities: ide isa_compat_mode bus_master
              configuration: driver=ata_piix latency=0
              resources: irq:0 ioport:1f0(size=8) ioport:3f6 ioport:170(size=8) ioport:376 ioport:c160(size=16)
         *-usb
              description: USB controller
              product: 82371SB PIIX3 USB [Natoma/Triton II]
              vendor: Intel Corporation
              physical id: 1.2
              bus info: pci@0000:00:01.2
              version: 01
              width: 32 bits
              clock: 33MHz
              capabilities: uhci bus_master
              configuration: driver=uhci_hcd latency=0
              resources: irq:11 ioport:c0c0(size=32)
            *-usbhost
                 product: UHCI Host Controller
                 vendor: Linux 5.16.18-200.fc35.x86_64 uhci_hcd
                 physical id: 1
                 bus info: usb@1
                 logical name: usb1
                 version: 5.16
                 capabilities: usb-1.10
                 configuration: driver=hub slots=2 speed=12Mbit/s
```

如你所见，输出包括一些只能从 DMI 获取的信息（例如主板型号），以及我们在 `lsusb` 和其他工具的输出中看到的信息。从这个意义上讲，如果你想要一个详细和完整的所有安装硬件概览，`lshw` 可以替代它们。

# 总结

在本章中，我们了解到 Linux 内核可以收集有关系统硬件的大量信息并提供给用户。在紧急情况下，可以直接从内核中检索所有这些信息，方法是使用 `/proc` 和 `/sys` 文件系统，并读取如 `/proc/cpuinfo` 等文件。

然而，高级工具如 `lscpu`、`lsscsi` 和 `lsusb` 可以使检索和分析信息变得更加容易。

还有一些平台特定的工具，例如用于 x86 PC 的 `dmidecode`，可以帮助你获取其他方法无法获取的更详细信息，例如内存插槽的数量。

在下一章，我们将学习如何配置基本系统设置。

# 第二部分：配置和修改 Linux 系统

本书的第二部分专注于管理独立的 Linux 系统。一旦你熟悉了与系统的交互，下一步就是学习如何配置它。在本部分，你将学习如何配置基本设置，例如系统主机名，如何创建和管理用户和组，如何从软件包文件或远程仓库安装附加软件，以及如何设置和调试网络连接与存储设备。

本部分包含以下章节：

+   *第六章*，*基本系统设置*

+   *第七章*，*用户与组管理*

+   *第八章*，*软件安装与软件包仓库*

+   *第九章*，*网络配置与故障排除*

+   *第十章*，*存储管理*
