

# 内存管理

本章涵盖与内存管理相关的问题，这是任何 Linux 系统中的重要话题，尤其是在嵌入式 Linux 中，系统内存通常是有限的。简要回顾虚拟内存后，我将向你展示如何衡量内存使用情况，如何检测内存分配问题，包括内存泄漏，以及当内存不足时会发生什么。你将需要了解可用的工具，从简单的工具如 `free` 和 `top` 到复杂的工具如 `mtrace` 和 Valgrind。

我们将学习内核空间和用户空间内存的区别，以及内核如何将物理内存页映射到进程的地址空间。然后，我们将定位并读取 `proc` 文件系统下各个进程的内存映射。我们将看到如何使用 `mmap` 系统调用将程序的内存映射到文件，这样它可以批量分配内存或与其他进程共享内存。在本章的后半部分，我们将使用 `ps` 来测量每个进程的内存使用情况，然后再使用更准确的工具，如 `smem` 和 `ps_mem`。

本章将涵盖以下主题：

+   虚拟内存基础

+   内核空间内存布局

+   用户空间内存布局

+   进程内存映射

+   内存管理

+   交换

+   使用 `mmap` 映射内存

+   我的应用程序使用了多少内存？

+   每个进程的内存使用

+   识别内存泄漏

+   内存不足

# 技术要求

为了跟随示例，确保你的 Linux 主机系统已安装 `gcc`、`make`、`top`、`procps`、`valgrind` 和 `smem`。

所有这些工具在大多数流行的 Linux 发行版（如 Ubuntu、Arch 等）上都可以使用。

本章中使用的代码可以在本书的 GitHub 仓库的章节文件夹中找到：[`github.com/PacktPublishing/Mastering-Embedded-Linux-Development/tree/main/Chapter18`](https://github.com/PacktPublishing/Mastering-Embedded-Linux-Development/tree/main/Chapter18)。

# 虚拟内存基础

总结一下，Linux 配置了 CPU 的 **内存管理单元** (**MMU**)，为正在运行的程序呈现一个从零开始并在 32 位处理器上以 `0xffffffff` 结束的虚拟地址空间。这个地址空间默认被划分为 4 KB 的页面。如果 4 KB 的页面对你的应用来说太小，你可以配置内核使用 **HugePages**，从而减少访问页面表项所需的系统资源，并增加 **转换后备缓冲区** (**TLB**) 的命中率。

Linux 将这个虚拟地址空间划分为供应用程序使用的区域，称为**用户空间**，以及供内核使用的区域，称为**内核空间**。两者之间的划分由一个名为`PAGE_OFFSET`的内核配置参数设置。在典型的 32 位嵌入式系统中，`PAGE_OFFSET`为`0xc0000000`，将低 3GB 分配给用户空间，顶端的 1GB 分配给内核空间。用户地址空间按进程分配，每个进程运行在自己的沙箱中，与其他进程隔离。内核地址空间对于所有进程都是相同的，因为只有一个内核。

这个虚拟地址空间中的页面通过 MMU 映射到物理地址，MMU 使用页表来执行映射。

每个虚拟内存页面可以按以下方式进行映射或未映射：

+   未映射，因此尝试访问这些地址将导致`SIGSEGV`。

+   映射到进程专用的物理内存页面。

+   映射到与其他进程共享的物理内存页面。

+   映射并共享，并设置**写时复制**（**CoW**）标志：写操作会被内核捕获，内核会复制该页面，并将其映射到进程中，取代原始页面，然后允许写操作发生。

+   映射到一个由内核使用的物理内存页面。

内核还可以将页面映射到保留的内存区域，例如，访问设备驱动程序中的寄存器和内存缓冲区。

一个显而易见的问题是：为什么要这样做，而不是像典型的实时操作系统那样直接引用物理内存？

虚拟内存有许多优势，以下是其中的一些描述：

+   无效的内存访问会被捕获，应用程序通过`SIGSEGV`收到警告。

+   进程在自己的内存空间中运行，彼此隔离。

+   通过共享公共代码和数据（例如在库中）高效使用内存。

+   通过添加交换文件增加物理内存的表面量，尽管在嵌入式目标上交换操作很少见。

这些是有力的论点，但我不得不承认也有一些缺点。确定应用程序的实际内存预算是很困难的，这也是本章的主要关注点之一。默认的分配策略是过度承诺，这会导致棘手的内存不足情况，稍后我会在*内存不足*一节中讨论。最后，内存管理代码在处理异常（页面错误）时引入的延迟，使得系统变得不那么确定，这对实时程序来说非常重要。我将在*第二十一章*中讲解这一点。

内存管理在内核空间和用户空间中是不同的。接下来的章节将描述这些基本区别以及你需要了解的内容。

# 内核空间内存布局

内核内存的管理方式非常简单。它不是按需分页的，这意味着每次使用`kmalloc()`或类似函数进行分配时，都会有实际的物理内存。内核内存从不被丢弃或分页出去。

一些架构在引导时会在内核日志消息中显示内存映射的摘要。这个跟踪来自一台 32 位的 Arm 设备（BeagleBone Black）：

```
Memory: 511MB = 511MB total
Memory: 505980k/505980k available, 18308k reserved, 0K highmem
Virtual kernel memory layout:
    vector  : 0xffff0000 - 0xffff1000   ( 4 kB)
    fixmap  : 0xfff00000 - 0xfffe0000   ( 896 kB)
    vmalloc : 0xe0800000 - 0xff000000   ( 488 MB)
    lowmem  : 0xc0000000 - 0xe0000000   ( 512 MB)
    pkmap   : 0xbfe00000 - 0xc0000000   ( 2 MB)
    modules : 0xbf800000 - 0xbfe00000   ( 6 MB)
      .text : 0xc0008000 - 0xc0763c90   (7536 kB)
      .init : 0xc0764000 - 0xc079f700   ( 238 kB)
      .data : 0xc07a0000 - 0xc0827240   ( 541 kB)
      .bss  : 0xc0827240 - 0xc089e940   ( 478 kB) 
```

505,980 KB 可用内存是内核开始执行时看到的自由内存量，但在内核开始进行动态分配之前。

内核空间内存的消费者包括以下内容：

+   内核本身，换句话说，是从引导时加载的内核映像文件中的代码和数据。这些内容在前面的内核日志中显示为`.text`、`.init`、`.data`和`.bss`段。`.init`段在内核初始化完成后会被释放。

+   通过 slab 分配器分配的内存，用于各种内核数据结构。这包括使用`kmalloc()`进行的分配。它们来自标记为**lowmem**的区域。

+   通过`vmalloc()`分配的内存，通常用于分配比`kmalloc()`可用内存更大的内存块。这些位于**vmalloc**区域。

+   设备驱动程序访问属于各种硬件部分的寄存器和内存的映射，您可以通过读取`/proc/iomem`来查看。这些也来自**vmalloc**区域，但由于它们映射到主系统内存之外的物理内存，因此不占用实际内存。

+   加载到标记为**modules**的区域中的内核模块。

+   其他未在任何地方跟踪的低级别分配。

现在我们知道了内核空间中内存的布局，让我们来看看内核实际使用了多少内存。

## 内核使用了多少内存？

不幸的是，关于内核使用多少内存这个问题，并没有确切的答案，但以下内容是我们能得到的最接近的答案。

首先，您可以通过之前显示的内核日志查看内核代码和数据占用的内存，或者使用`size`命令：

```
$ cd ~
$ cd build_arm64
$ aarch64-buildroot-linux-gnu-size vmlinux
   text     data    bss      dec  hex     filename
26412819 15636144 620032 42668995 28b13c3 vmlinux 
```

通常，与总内存量相比，内核为静态代码和数据段所占的内存较小。如果不是这种情况，您需要检查内核配置并删除不需要的组件。一个旨在构建小型内核的努力，称为**Linux 内核简化**，在项目停滞之前取得了良好的进展，Josh Triplett 的补丁最终在 2016 年从`linux-next`树中移除。现在，减少内核内存占用的最佳方法是**原地执行**（**XIP**），您可以通过牺牲 RAM 来换取闪存（[`lwn.net/Articles/748198/`](https://lwn.net/Articles/748198/)）。

您可以通过读取`/proc/meminfo`获取更多内存使用信息：

```
# cat /proc/meminfo
MemTotal:        1996796 kB
MemFree:         1917020 kB
MemAvailable:    1894044 kB
Buffers:            2444 kB
Cached:            11976 kB
SwapCached:            0 kB
Active:             9440 kB
Inactive:           8964 kB
Active(anon):         92 kB
Inactive(anon):     4096 kB
Active(file):       9348 kB
Inactive(file):     4868 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:             0 kB
SwapFree:              0 kB
Dirty:                 8 kB
Writeback:             0 kB
AnonPages:          4008 kB
Mapped:             6864 kB
Shmem:               200 kB
KReclaimable:       7412 kB
Slab:              20924 kB
SReclaimable:       7412 kB
SUnreclaim:        13512 kB
KernelStack:        1552 kB
PageTables:          540 kB
SecPageTables:         0 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:      998396 kB
Committed_AS:       7396 kB
VmallocTotal:   135288315904 kB
VmallocUsed:        4072 kB
VmallocChunk:          0 kB
<…> 
```

每个字段的描述可以在手册页`proc(5)`中找到。内核内存使用量是以下各项的总和：

+   `Slab`：由 slab 分配器分配的总内存

+   `KernelStack`：执行内核代码时使用的栈空间

+   `PageTables`：用于存储页表的内存

+   `VmallocUsed`：`vmalloc()`分配的内存

对于 slab 分配，你可以通过读取`/proc/slabinfo`获取更多信息。同样，`/proc/vmallocinfo`中有`**vmalloc**`区域的分配详细信息。在这两种情况下，你需要详细了解内核及其子系统，才能准确看到是哪个子系统进行的分配以及原因，这超出了本讨论的范围。

使用模块时，你可以通过`lsmod`查看代码和数据所占的内存空间：

```
# lsmod
Module          Size  Used by
g_multi        47670  2
libcomposite   14299  1 g_multi
mt7601Usta    601404  0 
```

这会留下低级分配，没有记录，这使得我们无法生成准确的内核空间内存使用统计。当我们将所有已知的内核和用户空间分配加起来时，这部分将表现为缺失的内存。

测量内核空间内存使用情况比较复杂。`/proc/meminfo`中的信息有些有限，而`/proc/slabinfo`和`/proc/vmallocinfo`提供的附加信息难以解读。用户空间通过进程内存映射提供了更好的内存使用可见性。

# 用户空间内存布局

Linux 对用户空间采用懒分配策略，只有当程序访问内存时，才会映射物理内存页。例如，使用`malloc(3)`分配 1 MB 的缓冲区时，返回的是指向内存地址块的指针，但并没有实际的物理内存。在页表条目中设置了一个标志，使得任何读或写访问都会被内核拦截。这就是**页面错误**。只有此时，内核才会尝试找到一页物理内存并将其添加到进程的页表映射中。让我们通过一个来自`MELD/Chapter18/pagefault-demo`的简单程序来演示这一点：

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/resource.h>
#define BUFFER_SIZE (1024 * 1024)
void print_pgfaults(void)
{
    int ret;
    struct rusage usage;
    ret = getrusage(RUSAGE_SELF, &usage);
    if (ret == -1) {
        perror("getrusage");
    } else {
        printf("Major page faults %ld\n", usage.ru_majflt);
        printf("Minor page faults %ld\n", usage.ru_minflt);
    }
}
int main(int argc, char *argv[])
{
    unsigned char *p;
    printf("Initial state\n");
    print_pgfaults();
    p = malloc(BUFFER_SIZE);
    printf("After malloc\n");
    print_pgfaults();
    memset(p, 0x42, BUFFER_SIZE);
    printf("After memset\n");
    print_pgfaults();
    memset(p, 0x42, BUFFER_SIZE);
    printf("After 2nd memset\n");
    print_pgfaults();
    return 0;
} 
```

当你运行它时，你将看到如下输出：

```
Initial state
Major page faults 0
Minor page faults 172
After malloc
Major page faults 0
Minor page faults 186
After memset
Major page faults 0
Minor page faults 442
After 2nd memset
Major page faults 0
Minor page faults 442 
```

在初始化程序环境后，遇到了 172 个小型页面错误，在调用`getrusage(2)`时又遇到了 14 个（这些数字会根据架构和你使用的 C 库版本有所不同）。重要的是填充内存时的增加：442 - 186 = 256。缓冲区为 1 MB，即 256 页。第二次调用`memset(3)`没有任何影响，因为所有页面现在已经映射。

如你所见，当内核截获对尚未映射的页面的访问时，会生成页面错误。事实上，有两种类型的页面错误：`轻微`和`严重`。在轻微错误中，内核只需找到一个物理内存页面并将其映射到进程地址空间，如前面的代码所示。严重页面错误发生在虚拟内存映射到文件时，例如使用`mmap(2)`，我稍后会描述。读取此内存意味着内核不仅需要找到一个内存页面并将其映射进来，还需要用文件中的数据填充它。因此，严重错误在时间和系统资源方面的成本要高得多。

虽然`getrusage(2)`提供了有关进程中的轻微和严重页面错误的有用指标，但有时我们真正想看到的是进程的总体内存映射。

# 进程内存映射

每个在用户空间中运行的进程都有一个可以检查的进程映射。这些内存映射告诉我们程序如何分配内存以及它链接了哪些共享库。你可以通过`proc`文件系统查看进程的内存映射。以下是`init`进程（PID 1）的映射：

```
# cat /proc/1/maps
aaaaaf830000-aaaaaf83a000 r-xp 00000000 b3:62 397   /sbin/init.sysvinit
aaaaaf84f000-aaaaaf850000 r--p 0000f000 b3:62 397   /sbin/init.sysvinit
aaaaaf850000-aaaaaf851000 rw-p 00010000 b3:62 397   /sbin/init.sysvinit
aaaae9d63000-aaaae9d84000 rw-p 00000000 00:00 0     [heap]
ffff7ffb0000-ffff8013b000 r-xp 00000000 b3:62 309   /lib/libc.so.6
ffff8013b000-ffff8014d000 ---p 0018b000 b3:62 309   /lib/libc.so.6
ffff8014d000-ffff80150000 r--p 0018d000 b3:62 309   /lib/libc.so.6
ffff80150000-ffff80152000 rw-p 00190000 b3:62 309   /lib/libc.so.6
ffff80152000-ffff8015e000 rw-p 00000000 00:00 0
ffff8016c000-ffff80193000 r-xp 00000000 b3:62 304   /lib/ld-linux-aarch64.so.1
ffff801a4000-ffff801a6000 rw-p 00000000 00:00 0
ffff801a6000-ffff801a8000 r--p 00000000 00:00 0     [vvar]
ffff801a8000-ffff801aa000 r-xp 00000000 00:00 0     [vdso]
ffff801aa000-ffff801ac000 r--p 0002e000 b3:62 304   /lib/ld-linux-aarch64.so.1
ffff801ac000-ffff801ae000 rw-p 00030000 b3:62 304   /lib/ld-linux-aarch64.so.1
ffffd73ca000-ffffd73eb000 rw-p 00000000 00:00 0     [stack] 
```

前两列显示每个映射的起始和结束虚拟地址，以及权限。权限在此处显示：

+   `r`: 读取

+   `w`: 写入

+   `x`: 执行

+   `s`: 共享

+   `p`: 私有（写时复制）

如果映射与文件相关联，则文件名出现在最后一列，第三、四和五列包含文件起始处的偏移量、块设备号和文件的 inode。大多数映射指向程序本身和它链接的库。有两个区域，程序可以在其中分配内存，分别标记为`[heap]`和`[stack]`。使用`malloc`分配的内存来自前者（除了非常大的分配，我们稍后会提到）；堆栈上的分配来自后者。两个区域的最大大小由进程的`ulimit`控制：

+   **堆**: `ulimit -d`，默认为无限制

+   **堆栈**: `ulimit -s`，默认为 8 MB

超出限制的分配会被`SIGSEGV`拒绝。

当内存不足时，内核可能决定丢弃那些映射到文件且为只读的页面。如果该页面再次被访问，将导致严重页面错误，并从文件中重新读取。

# 交换

交换的理念是保留一些存储空间，内核可以将未映射到文件的内存页面放入其中，从而释放出内存供其他用途。它通过交换文件的大小增加了物理内存的有效大小。它并不是万能的：将页面复制到交换文件和从交换文件复制回来是有代价的，这在系统的实际内存不足以支撑其工作负载时变得显而易见，交换成为主要的活动。这有时被称为**磁盘抖动**。

在嵌入式设备上，交换通常不被使用，因为它不适用于闪存存储，频繁的写入会迅速磨损闪存。然而，你可能会考虑将交换设置为压缩 RAM（zram）。

## 将内存交换到压缩内存（zram）

**zram** 驱动程序创建基于 RAM 的块设备，命名为 `/dev/zram0`、`/dev/zram1` 等。写入这些设备的页面在存储之前会被压缩。压缩比在 30% 到 50% 之间，你可以期望空闲内存大约增加 10%，但这会增加更多的处理量并相应地增加功耗。

要启用 zram，请使用以下选项配置内核：

```
CONFIG_SWAP
CONFIG_CGROUP_MEM_RES_CTLR
CONFIG_CGROUP_MEM_RES_CTLR_SWAP
CONFIG_ZRAM 
```

然后，通过将以下内容添加到 `/etc/fstab` 中，在启动时挂载 zram：

```
/dev/zram0 none swap defaults zramsize=<size in bytes>, swapprio=<swap partition priority> 
```

你可以使用以下命令开启或关闭交换：

```
# swapon /dev/zram0
# swapoff /dev/zram0 
```

将内存交换到 zram 比交换到闪存存储更好，但这两种技术都不能替代足够的物理内存。

用户空间进程依赖内核管理虚拟内存。有时，程序需要比内核提供的更多的内存映射控制。有一个系统调用允许我们将内存映射到文件中，以便从用户空间进行更直接的访问。

# 使用 mmap 映射内存

一个进程从启动时就会有一定量的内存映射到程序文件的 **文本**（代码）和 **数据** 段中，并与它链接的共享库一起。它可以在运行时使用 `malloc(3)` 在堆上分配内存，并通过局部作用域变量以及使用 `alloca(3)` 分配的内存在栈上分配内存。它还可以在运行时使用 `dlopen(3)` 动态加载库。所有这些映射由内核处理。然而，进程也可以通过 `mmap(2)` 显式地操作其内存映射：

```
void *mmap(void *addr, size_t length, int prot, int flags,int fd, off_t offset); 
```

此函数从具有 `fd` 描述符的文件中映射 `length` 字节的内存，从文件中的 `offset` 位置开始，并返回指向映射的指针（如果成功）。由于底层硬件按页工作，`length` 会向上舍入到最接近的整页数。保护参数 `prot` 是读、写和执行权限的组合，`flags` 参数至少包含 `MAP_SHARED` 或 `MAP_PRIVATE`。还有许多其他标志，详细说明可以参考 `mmap` 的手册页。

你可以使用 `mmap` 做很多事情。接下来的章节将展示其中的一些。

## 使用 mmap 分配私有内存

你可以通过在 `flags` 参数中设置 `MAP_ANONYMOUS` 并将文件描述符 `fd` 设置为 `-1` 来使用 `mmap` 分配私有内存区域。这类似于使用 `malloc` 从堆中分配内存，不同的是分配的内存是页对齐的，并且是页的倍数。分配的内存与库使用的内存区域相同。事实上，这个区域因此也被一些人称为 `mmap` 区域。

匿名映射更适合大块分配，因为它们不会将堆与内存块绑定，这样会增加碎片化的可能性。有趣的是，你会发现 `malloc`（至少在 `glibc` 中）在处理超过 128 KB 的请求时，会停止从堆中分配内存，并改用 `mmap`，因此在大多数情况下，直接使用 `malloc` 就是正确的做法。系统会选择最合适的方式来满足请求。

## 使用 mmap 来共享内存

正如我们在 *第十七章* 中看到的，POSIX 共享内存需要 `mmap` 来访问内存段。在这种情况下，你需要设置 `MAP_SHARED` 标志，并使用 `shm_open()` 的文件描述符：

```
int shm_fd;
char *shm_p;
shm_fd = shm_open("/myshm", O_CREAT | O_RDWR, 0666);
ftruncate(shm_fd, 65536);
shm_p = mmap(NULL, 65536, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0); 
```

另一个进程使用相同的调用、文件名、长度和标志来映射到该内存区域以进行共享。后续调用 `msync(2)` 控制何时将内存更新传递到底层文件。

通过 `mmap` 共享内存还提供了一种直接的方式来读写设备内存。

## 使用 mmap 访问设备内存

正如我在 *第十一章* 中提到的，驱动程序可能允许其设备节点进行内存映射，并与应用程序共享一些设备内存。具体实现取决于驱动程序。

一个例子是 Linux 帧缓冲区 `/dev/fb0`。像 Xilinx Zynq 系列这样的 FPGA 也可以通过 `mmap` 从 Linux 中作为内存访问。帧缓冲接口定义在 `/usr/include/linux/fb.h` 中，包括一个 `ioctl` 函数来获取显示的大小和每像素的位数。然后，你可以使用 `mmap` 请求视频驱动程序与应用程序共享帧缓冲区，并读取和写入像素：

```
int f;
int fb_size;
unsigned char *fb_mem;
f = open("/dev/fb0", O_RDWR);
/* Use ioctl FBIOGET_VSCREENINFO to find the display
 dimensions and calculate fb_size */
fb_mem = mmap(0, fb_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
/* read and write pixels through pointer fb_mem */ 
```

第二个例子是流媒体视频接口 **Video 4 Linux 2** (**V4L2**) ，它定义在 `/usr/include/linux/videodev2.h` 中。每个视频设备都有一个名为 `/dev/video<N>` 的节点，从 `/dev/video0` 开始。这里有一个 `ioctl` 函数来请求驱动程序分配一些视频缓冲区，你可以将其 `mmap` 到用户空间。然后，只需要循环使用这些缓冲区，并根据是播放还是捕获视频流来填充或清空它们。

现在我们已经讲解了内存布局和映射，让我们来看看内存使用情况，从如何衡量它开始。

# 我的应用程序使用了多少内存？

与内核空间一样，不同的用户空间内存分配、映射和共享方式使得回答这个看似简单的问题变得相当困难。

首先，你可以询问内核它认为可用的内存量，你可以通过 `free` 命令来做到这一点。以下是输出的典型示例：

```
 total        used   free shared buffers cached
Mem:   509016      504312   4704 0      26456   363860
-/+ buffers/cache: 113996 395020
Swap:       0           0      0 
```

乍一看，这看起来像是一个几乎没有内存的系统，只剩下 4,704 KB 的空闲内存，总共 509,016 KB：少于 1%。然而，请注意，26,456 KB 在缓冲区中，而令人震惊的 363,860 KB 在缓存中。Linux 认为空闲内存是浪费的内存；内核使用空闲内存用于缓冲区和缓存，并知道在需要时可以缩小它们。从测量中去除缓冲区和缓存提供真正的空闲内存，即 395,020 KB：总量的 77%。在使用`free`时，第二行标记为`-/+ buffers/cache`的数字是重要的。

您可以通过将数字写入 `/proc/sys/vm/drop_caches` 强制内核释放缓存。

```
# echo 3 > /proc/sys/vm/drop_caches 
```

这个数字实际上是一个位掩码，确定你想要释放的两种广义缓存类型之一：`1` 表示页面缓存，`2` 表示 dentry 和 inode 缓存的组合体。因为 `1` 和 `2` 是不同的位，写入 `3` 可以释放两种类型的缓存。

这些缓存的确切角色在这里并不特别重要，只要知道内核正在使用但可以在短时间内收回的内存即可。

`free`命令告诉我们正在使用多少内存以及剩余多少。它既不告诉我们哪些进程正在使用不可用内存，也不告诉我们使用了多少。要测量这一点，我们需要其他工具。

# 每个进程的内存使用情况

有几种指标可以衡量进程正在使用的内存量。我将从最容易获取的两种开始：**虚拟集大小**（**VSS**）和**常驻内存大小**（**RSS**），这两者在大多数`ps`和`top`命令的实现中都可以找到：

+   **VSS**：在`ps`命令中称为`VSZ`，在`top`中称为`VIRT`，这是由进程映射的所有内存区域的总和。由于虚拟内存的一部分仅在任何时候映射到物理内存中，这个数字的兴趣有限。

+   **RSS**：在`ps`中称为`RSS`，在`top`中称为`RES`，这是映射到物理内存页的内存总和。这更接近进程实际内存预算，但存在一个问题：如果将所有进程的 RSS 相加，将会高估正在使用的内存，因为某些页面将是共享的。

让我们更多地了解`top`和`ps`命令。

## 使用 top 和 ps

BusyBox 提供的`top`和`ps`版本提供的信息非常有限。以下示例使用来自`procps`包的完整版本。

这是使用自定义格式的`ps`命令的输出，包括`vsz`和`rss`：

```
# ps -eo pid,tid,class,rtprio,stat,vsz,rss,comm
    PID     TID CLS RTPRIO STAT    VSZ   RSS COMMAND
      1       1 TS       - Ss     4496  2652 systemd
    <…>
    205     205 TS       - Ss     4076  1296 systemd-journal
    228     228 TS       - Ss     2524  1396 udevd
    581     581 TS       - Ss     2880  1508 avahi-daemon
    584     584 TS       - Ss     2848  1512 dbus-daemon
    590     590 TS       - Ss     1332   680 acpid
    594     594 TS       - Ss     4600  1564 wpa_supplicant 
```

同样，`top`显示了每个进程的空闲内存和内存使用情况的摘要：

```
top - 21:17:52 up 10:04, 1 user, load average: 0.00, 0.01, 0.05
Tasks: 96 total, 1 running, 95 sleeping, 0 stopped, 0 zombie
%Cpu(s):  1.7 us,  2.2 sy,  0.0 ni, 95.9 id,  0.0 wa,  0.0 hi
KiB Mem : 509016 total, 278524 used, 230492 free,  25572 buffers
KiB Swap:      0 total,      0 used,      0 free, 170920 cached
PID USER PR NI   VIRT   RES   SHR S  %CPU  %MEM    TIME+ COMMAND
595 root 20  0  64920  9.8m  4048 S   0.0   2.0  0:01.09 node
866 root 20  0  28892  9152  3660 S   0.2   1.8  0:36.38 Xorg
<…> 
```

这些简单的命令使您了解内存使用情况，并在看到进程的 RSS 持续增加时提供了内存泄漏的第一个迹象。但是，它们在内存使用的绝对测量方面并不十分精确。

## 使用 smem

2009 年，Matt Mackall 开始研究如何对进程内存中的共享页面进行记账的问题，并增加了两个新的指标，分别为**唯一集合大小**（**USS**）和**比例集合大小**（**PSS**）：

+   **USS**：这是分配到物理内存并且唯一属于一个进程的内存量；它不会与其他进程共享。它是进程终止时会释放的内存量。

+   **PSS**：这会将共享页面在物理内存中被多进程映射时的记账分配给所有映射该页面的进程。例如，如果某个库代码区域有 12 页，并且被六个进程共享，那么每个进程的 PSS 将增加 2 页。因此，如果你将所有进程的 PSS 加起来，你将得到这些进程实际使用的内存量。换句话说，PSS 就是我们一直在寻找的数字。

关于 PSS 的信息可以在`/proc/<PID>/smaps`中找到，该文件包含每个映射的附加信息，显示在`/proc/<PID>/maps`中。下面是该文件的一部分，提供了关于`libc`代码段映射的信息：

```
ffffbd080000-ffffbd20b000 r-xp 00000000 b3:62 309 /lib/libc.so.6
Size:               1580 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
Rss:                1132 kB
Pss:                 112 kB
Pss_Dirty:             0 kB
Shared_Clean:       1132 kB
Shared_Dirty:          0 kB
Private_Clean:         0 kB
Private_Dirty:         0 kB
Referenced:         1132 kB
Anonymous:             0 kB
KSM:                   0 kB
LazyFree:              0 kB
AnonHugePages:         0 kB
ShmemPmdMapped:        0 kB
FilePmdMapped:         0 kB
Shared_Hugetlb:        0 kB
Private_Hugetlb:       0 kB
Swap:                  0 kB
SwapPss:               0 kB
Locked:                0 kB
THPeligible:           0
VmFlags: rd ex mr mw me 
```

注意，`Rss`为`1132 kB`，但由于它在多个其他进程之间共享，因此`Pss`只有`112 kB`。

有一个名为**smem**的工具，它将来自`smaps`文件的信息整合并以多种方式呈现，包括饼状图或条形图。`smem`的项目页面是[`www.selenic.com/smem/`](https://www.selenic.com/smem/)。它作为软件包在大多数桌面发行版中提供。然而，由于它是用 Python 编写的，在嵌入式目标系统上安装它需要一个 Python 环境，这对于仅仅是一个工具来说可能太麻烦。为了解决这个问题，有一个名为**smemcap**的小程序，它从目标系统的`/proc`捕获状态，并将其保存到一个可以在主机计算机上分析的 TAR 文件中。`smemcap`是 BusyBox 的一部分，但也可以从源代码编译。

如果你以`root`身份直接运行`smem`，你将看到这些结果：

```
# smem -t
PID User Command                     Swap   USS   PSS   RSS
  1 root init [5]                       0   136   267  1532
361 root /sbin/klogd -n                 0   104   273  1708
367 root /sbin/getty 38400 tty1         0   108   278  1788
369 root /sbin/getty -L 115200 ttyS2    0   108   278  1788
358 root /sbin/syslogd -n -O /var/lo    0   108   279  1728
306 root udhcpc -R -b -p /var/run/ud    0   168   284  1372
366 root /bin/sh /bin/start_getty 11    0   116   315  1736
383 root -sh                            0   220   506  2184
129 root /sbin/udevd -d                 0  1436  1517  2380
351 root sshd: /usr/sbin/sshd [liste    0   928  1893  3764
380 root sshd: root@pts/0               0  3816  4900  7160
387 root python3 /usr/bin/smem -t       0 11968 12136 13456
-----------------------------------------------------------
 12 1                                   0 19216 22926 40596 
```

从输出的最后一行可以看到，在这种情况下，总的 PSS 大约是 RSS 的一半。

如果你没有安装 Python，或者不想在目标系统上安装 Python，你可以使用`smemcap`捕获状态，再次以`root`身份运行：

```
# smemcap > smem-beagleplay-cap.tar 
```

然后，将 TAR 文件复制到主机上，使用`smem -t -S`读取，尽管这一次不需要以`root`身份运行命令：

```
$ smem -t -S smem-beagleplay-cap.tar 
```

输出与我们直接运行`smem`时获得的输出相同。

## 其他工具可供考虑

另一种显示 PSS 的方式是使用**ps_mem**（[`github.com/pixelb/ps_mem`](https://github.com/pixelb/ps_mem)），它以更简单的格式显示几乎相同的信息。它也是用 Python 编写的。

Android 也有一个工具，它显示每个进程的 USS 和 PSS 摘要，名为**procrank**，它可以通过一些小的修改交叉编译到嵌入式 Linux 上。你可以从[`github.com/csimmonds/procrank_linux`](https://github.com/csimmonds/procrank_linux)获取代码。

我们现在知道如何衡量每个进程的内存使用情况。假设我们使用刚才展示的工具来找到系统中占用内存最多的进程。那么我们该如何进一步深入这个进程，找出它出问题的地方呢？这就是下一节的内容。

# 识别内存泄漏

内存泄漏发生在内存被分配但在不再需要时没有释放。内存泄漏绝不仅仅是嵌入式系统的专属问题，但它成为一个问题，部分原因是目标设备本身内存有限，部分原因是它们通常长时间运行而不重启，这使得泄漏逐渐变成一个大水坑。

当你运行`free`或`top`并看到即使清空缓存，空闲内存仍然持续减少时，你就会意识到有内存泄漏，正如上一节所示。通过查看每个进程的 USS 和 RSS，你将能够识别泄漏的罪魁祸首。

有几种工具可以识别程序中的内存泄漏。我将介绍两种：`mtrace`和`valgrind`。

## mtrace

**mtrace**是`glibc`的一个组件，它跟踪对`malloc`、`free`及相关函数的调用，并识别在程序退出时未释放的内存区域。你需要在程序中调用`mtrace()`函数以开始跟踪，然后在运行时将路径名写入`MALLOC_TRACE`环境变量中，该变量指定了写入跟踪信息的文件路径。如果`MALLOC_TRACE`不存在或无法打开该文件，则`mtrace`钩子将无法安装。

虽然跟踪信息是以 ASCII 格式写入的，但通常使用`mtrace`命令来查看它。以下是一个使用`mtrace`的程序示例，来自`MELD/Chapter18/mtrace-example`：

```
#include <mcheck.h>
#include <stdlib.h>
#include <stdio.h>
int main(int argc, char *argv[])
{
    int j;
    mtrace();
    for (j = 0; j < 2; j++)
        malloc(100); /* Never freed:a memory leak */
    calloc(16, 16); /* Never freed:a memory leak */
    exit(EXIT_SUCCESS);
} 
```

以下是你在运行程序并查看跟踪时可能看到的内容：

```
$ export MALLOC_TRACE=mtrace.log
$ ./mtrace-example
$ mtrace mtrace-example mtrace.log
Memory not freed:
-----------------
           Address   Size      Caller
0x0000000001479460    0x64  at /home/chris/mtrace-example.c:12
0x00000000014794d0    0x64  at /home/chris/mtrace-example.c:12
0x0000000001479540   0x100  at /home/chris/mtrace-example.c:14 
```

不幸的是，`mtrace`不会告诉你程序运行期间的泄漏内存。它必须先终止才能报告。

## Valgrind

**Valgrind**是一个非常强大的工具，用于发现包括泄漏在内的内存问题。它的一个优势是你无需重新编译要检查的程序和库，尽管如果它们在编译时使用了`-g`选项，以便包含调试符号表，它的效果会更好。它通过在模拟环境中运行程序并在多个点截获执行来工作。这导致了 Valgrind 的一个大缺点，那就是程序运行速度远低于正常速度，这使得它在测试有实时约束的应用时效果较差。

**提示**

顺便说一句，Valgrind 的名字常被误读：Valgrind 常见 FAQ 中表示 "grind" 部分的发音应该是短音 *i*，就像 "grinned"（与 "tinned" 押韵），而不是 "grind"（与 "find" 押韵）。FAQ、文档和下载可以访问 [`valgrind.org`](https://valgrind.org)。

Valgrind 包含多个诊断工具：

+   `memcheck`：这是默认的工具，用于检测内存泄漏和一般的内存误用。

+   `cachegrind`：此工具计算处理器缓存命中率。

+   `callgrind`：此工具计算每个函数调用的开销。

+   `helgrind`：此工具突出了 Pthread API 的误用，包括潜在的死锁和竞争条件。

+   `DRD`：这是另一个 Pthread 分析工具。

+   `massif`：此工具分析堆和栈的使用情况。

你可以通过 `-tool` 选项选择你想使用的工具。Valgrind 支持主要的嵌入式平台：Arm（Cortex-A）、PowerPC、MIPS 和 x86 的 32 位和 64 位版本。它在 Yocto 项目和 Buildroot 中都有提供作为软件包。

要找到我们的内存泄漏，我们需要使用默认的 `memcheck` 工具，并使用 `-–leak-check=full` 选项打印出发现泄漏的代码行：

```
$ valgrind --leak-check=full ./mtrace-example
==3384686== Memcheck, a memory error detector
==3384686== Copyright (C) 2002-2022, and GNU GPL'd, by Julian Seward et al.
==3384686== Using Valgrind-3.22.0 and LibVEX; rerun with -h for copyright info
==3384686== Command: ./mtrace-example
==3384686==
==3384686==
==3384686== HEAP SUMMARY:
==3384686==     in use at exit: 456 bytes in 3 blocks
==3384686==   total heap usage: 3 allocs, 0 frees, 456 bytes allocated
==3384686==
==3384686== 200 bytes in 2 blocks are definitely lost in loss record 1 of 2
==3384686==    at 0x4846828: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==3384686==    by 0x1091D3: main (mtrace-example.c:12)
==3384686==
==3384686== 256 bytes in 1 blocks are definitely lost in loss record 2 of 2
==3384686==    at 0x484D953: calloc (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==3384686==    by 0x1091EC: main (mtrace-example.c:14)
==3384686==
==3384686== LEAK SUMMARY:
==3384686==    definitely lost: 456 bytes in 3 blocks
==3384686==    indirectly lost: 0 bytes in 0 blocks
==3384686==      possibly lost: 0 bytes in 0 blocks
==3384686==    still reachable: 0 bytes in 0 blocks
==3384686==         suppressed: 0 bytes in 0 blocks
==3384686==
==3384686== For lists of detected and suppressed errors, rerun with: -s
==3384686== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0) 
```

Valgrind 的输出显示在 `mtrace-example.c` 中发现了两个内存泄漏：第 12 行的 `malloc` 和第 14 行的 `calloc`。但是，程序中缺少与这两个内存分配配对的 `free` 调用。如果不加以检查，长时间运行的进程中的内存泄漏最终可能导致系统内存耗尽。

# 内存耗尽

标准的内存分配策略是**过度提交**，这意味着内核允许应用程序分配比物理内存更多的内存。大多数情况下，这种做法是有效的，因为应用程序请求的内存量通常超过其实际需求。这也有助于 `fork(2)` 的实现：复制一个大型程序是安全的，因为内存页与写时复制标志共享。在大多数情况下，`fork` 之后会调用 `exec` 函数，这会解除共享内存并加载新程序。

然而，总是存在某些工作负载可能导致一组进程试图同时请求它们已经被承诺的内存分配，从而需求超过实际可用的内存量。这就是**内存不足**（**OOM**）的情况。此时，唯一的办法是终止一些进程，直到问题解决。这是**内存不足杀手**的工作。

在我们深入之前，`/proc/sys/vm/overcommit_memory` 中有一个内核分配的调优参数，你可以将其设置为以下值：

+   `0`：启发式过度提交

+   `1`：始终过度提交；绝不检查

+   `2`：始终检查；绝不过度提交

选项 `0` 是默认设置，在大多数情况下是最佳选择。

选项 `1` 只在你运行的程序需要处理大型稀疏数组并分配大量内存但仅对其中一小部分进行写入时有用。在嵌入式系统的背景下，这种程序比较少见。

如果你担心内存耗尽，尤其是在任务或安全关键的应用中，选项 `2` 似乎是一个不错的选择。它将会失败大于提交限制的分配，而提交限制是交换空间大小加上总内存与过度提交比例的乘积。过度提交比例由 `/proc/sys/vm/overcommit_ratio` 控制，默认值为 50%。

举个例子，假设你有一台系统内存为 2 GB 的设备，并且你设置了一个非常保守的 25% 比例：

```
# echo 25 > /proc/sys/vm/overcommit_ratio
# grep -e MemTotal -e CommitLimit /proc/meminfo
MemTotal:        1996796 kB
CommitLimit:      499196 kB 
```

没有交换空间，因此提交限制是 `MemTotal` 的 25%，这也是预期的结果。

`/proc/meminfo` 中还有另一个重要变量，叫做 `Committed_AS`。它表示为了完成迄今为止所有分配所需的总内存。我在一个系统上找到了以下内容：

```
# grep -e MemTotal -e Committed_AS /proc/meminfo
MemTotal:        1996796 kB
Committed_AS:    2907335 kB 
```

换句话说，内核已经承诺分配的内存超出了可用内存。因此，将 `overcommit_memory` 设置为 `2` 将意味着所有分配都会失败，无论 `overcommit_ratio` 设置为多少。为了让系统正常工作，我必须要么安装双倍的内存，要么大幅减少正在运行的进程数量，而当时大约有 40 个进程。

在所有情况下，最终的防线是 `oom-killer`。它使用启发式方法计算每个进程的“坏度”得分，范围从 0 到 1000，然后终止得分最高的进程，直到释放出足够的空闲内存。你应该在内核日志中看到类似这样的内容：

```
[44510.490320] eatmem invoked oom-killer: gfp_mask=0x200da, order=0, oom_score_adj=0 
```

你可以通过执行 `echo f > /proc/sysrq-trigger` 强制触发一个 OOM 事件。

你可以通过向 `/proc/<PID>/oom_score_adj` 写入调整值来影响进程的坏度得分。`-1000` 的值意味着坏度得分永远不会大于零，因此进程永远不会被杀死；`+1000` 的值意味着它将始终大于 1000，因此进程将始终被杀死。

# 总结

在虚拟内存系统中，完全记录每一个字节的内存使用情况几乎是不可能的。不过，你可以通过 `free` 命令获得一个相当准确的空闲内存总量的数字，排除缓冲区和缓存所占用的内存。通过在一段时间内和不同的工作负载下监控它，你应该可以确认它会保持在一个给定的限制范围内。

当你想要调整内存使用或识别意外分配的来源时，有一些资源可以提供更详细的信息。对于内核空间，最有用的信息在 `/proc` 目录下：`meminfo`、`slabinfo` 和 `vmallocinfo`。

在获取用户空间的准确测量时，最佳的度量标准是 PSS，如 `smem` 和其他工具所示。对于内存调试，你可以使用像 `mtrace` 这样的简单追踪工具，或者使用较为重型的 Valgrind `memcheck` 工具。

如果你对 OOM（内存不足）情况的后果有所担忧，你可以通过`/proc/sys/vm/overcommit_memory`来微调分配机制，并且可以通过`oom_score_adj`参数控制特定进程被终止的可能性。

下一章将详细介绍如何使用 GNU 调试器调试用户空间和内核代码，以及你通过观察代码运行时能够获得的洞见，其中包括我在这里描述的内存管理功能。

# 进一步学习

+   *Linux 内核开发（第三版）*，作者：Robert Love

+   *Linux 系统编程（第二版）*，作者：Robert Love

+   *理解 Linux 虚拟内存管理*，作者：Mel Gorman – [`www.kernel.org/doc/gorman/pdf/understand.pdf`](https://www.kernel.org/doc/gorman/pdf/understand.pdf)

+   *Valgrind 3.3：GNU/Linux 应用程序的高级调试与性能分析*，作者：Julian Seward、Nicholas Nethercote 和 Josef Weidendorfer
