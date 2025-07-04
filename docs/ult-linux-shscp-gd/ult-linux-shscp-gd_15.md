# 15

# 使用 awk —— 第二部分

在本章中，我们将从不同的角度继续讨论 `awk`。在上一章中，我向你展示了如何创建可以在普通 shell 脚本中使用的单行 `awk` 命令。在本章中，我将向你展示如何用 `awk` 语言编写 `awk` 脚本。本章的内容包括：

+   基本的 `awk` 脚本构建

+   使用条件语句

+   使用 `while` 结构并设置变量

+   使用 for 循环和数组

+   使用浮动点数学和 `printf`

+   处理多行记录

如果你准备好了，让我们深入探讨。

# 技术要求

你可以使用 Fedora 或 Debian 虚拟机来执行这个操作。而且，和往常一样，你可以通过以下方式获取脚本：

```
git clone https://github.com/PacktPublishing/The-Ultimate-Linux-Shell-Scripting-Guide.git 
```

# 基本的 awk 脚本构建

让我们从你能想象的最简单的 `awk` 脚本开始，我们将其命名为 `awk_kernel1.awk`。它看起来像这样：

```
/kernel/ 
```

正如你可能已经猜到的，这个脚本会查看指定的文件，搜索包含文本字符串 `kernel` 的所有行。你已经知道，如果没有指定操作，`{print $0}` 是默认操作。因此，这个脚本将打印出包含指定文本字符串的每一行。

在实际的 `awk` 脚本中，不需要在每个命令前加上 `awk`，也不需要像在普通的 shell 脚本中嵌入 `awk` 命令时那样，用单引号括起来。我没有在这个脚本中加入 shebang 行，因此不需要设置可执行权限。相反，只需像这样调用脚本：

```
donnie@fedora:~$ sudo awk -f awk_kernel1.awk /var/log/messages
Jan 11 16:17:55 fedora kernel: audit: type=1334 audit(1705007875.578:35): prog-id=60 op=LOAD
Jan 11 16:18:00 fedora kernel: msr: Write to unrecognized MSR 0x17f by mcelog (pid: 856).
Jan 11 16:18:00 fedora kernel: msr: See https://git.kernel.org/pub/scm/linux/kernel/git/tip/tip.git/about for details.
. . .
. . .
Jan 11 17:15:28 fedora kernel: fwupdmgr[1779]: memfd_create() called without MFD_EXEC or MFD_NOEXEC_SEAL set
donnie@fedora:~$ 
```

当然，这样也可以。但你不觉得还是更希望有一个独立的、可执行的脚本吗？这其实很简单。只需添加 shebang 行，像这样：

```
#!/usr/bin/awk -f
/kernel/ 
```

然后，使脚本可执行，就像你处理普通的`bash`脚本一样。

有两件事我希望你注意到这个 shebang 行。首先，我使用的是 `/usr/bin/` 而不是 `/bin/` 作为 `awk` 可执行文件的路径。这是因为我想让这个脚本具备可移植性，这样它就可以在 Linux、Unix 以及像 FreeBSD 和 macOS 这样的 Unix-like 系统上运行。

你习惯在 shebang 行中看到的 `/bin/` 路径，实际上是一个遗留物，它来自较早的 Linux 系统。在当前的 Linux 系统中，`/bin/` 是指向 `/usr/bin/` 的符号链接。在旧版 Linux 系统中，`/bin/` 和 `/usr/bin/` 曾经是两个独立的目录，每个目录都包含各自的程序文件集。现在已经不是这样了。如今，你会发现在所有 Linux 系统中，`awk` 可执行文件都位于 `/usr/bin/` 中。

FreeBSD 仍然使用独立的 `/bin/` 和 `/usr/bin/` 目录，其中包含不同的程序文件集。但 `awk` 位于 `/usr/bin/` 中，并且在 `/bin/` 中没有它的符号链接。因此，只需使用 `#!/usr/bin/awk`，大多数操作系统就能正常运行。

第二件事要注意的是，我仍然需要使用`-f`选项来调用`awk`，这样`awk`才会读取程序文件。如果你省略了`-f`，脚本将无法运行。

现在你已经看到了`awk`脚本的基本结构，接下来我们来看看一些`awk`编程结构。

# 使用条件语句

你其实已经在使用`if`语句了，只是你可能没有意识到。因为你不需要明确声明它们。你在`awk_kernel1.awk`脚本中看到的简单`/kernel/`命令意味着，如果在某行中找到*kernel*字符串，就打印该行。然而，`awk`还提供了你在其他编程语言中会看到的完整编程结构。例如，我们来创建`awk_kernel2.awk`脚本，内容如下：

```
#!/usr/bin/awk -f
{
        if (/kernel/) {
                print $0
        }
} 
```

这和你在`bash`脚本中习惯看到的有所不同，因为在`awk`中，不需要使用`then`或`fi`语句。这是因为`awk`使用 C 语言语法来编写编程结构。所以，如果你习惯用 C 语言编程，那就高兴吧！

另外，注意如何需要将模式用一对括号括起来，以及如何必须用一对花括号将整个多行脚本括起来。无论如何，在运行脚本时，只需指定你的日志文件的名称和位置，像这样：

```
donnie@fedora:~$ sudo ./awk_kernel2.awk /var/log/messages 
```

现在，你可能会想，为什么有人会想输入额外的代码来创建一个完整的`if`语句结构，而仅仅输入`/kernel/`就能完成任务呢？原因是这样可以创建完整的`if. .else`结构，就像在`awk_kernel3.awk`脚本中的这个结构：

```
#!/usr/bin/awk -f
{
        if ($5 ~ /kernel/) {
                print "Kernel here in Field 5"
        }
        else if ($5 ~ /systemd/) {
                print "Systemd here in Field 5"
        }
        else {
                print "No kernel or systemd in Field 5"
        }
} 
```

现在，让我们看看每种类型的消息在日志文件中出现的次数：

```
donnie@fedora:~$ sudo ./awk_kernel3.awk /var/log/messages | sort | uniq -c
  25795 Kernel here in Field 5
  38580 No kernel or systemd in Field 5
  35506 Systemd here in Field 5
donnie@fedora:~$ 
```

很酷，它工作了。

对于我们的最后一个`if`技巧，让我们创建`awk_kernel4.awk`脚本，如下所示：

```
#!/usr/bin/awk -f
{
        if ($5 ~ /kernel/) {
                print "Kernel here in Field 5 on line " NR
        }
        else if ($5 ~ /systemd/) {
                print "Systemd here in Field 5 on line " NR
        }
        else {
                print "No kernel or systemd in Field 5 on line " NR
        }
} 
```

记录数（`NR`）内建变量会将行号和消息一起打印出来。输出会很多，你可能需要将其管道传送到`less`，像这样：

```
donnie@fedora:~$ sudo ./awk_kernel4.awk /var/log/messages | less 
```

下面是输出的示例：

```
Kernel here in Field 5 on line 468
Systemd here in Field 5 on line 469
Systemd here in Field 5 on line 470
No kernel or systemd in Field 5 on line 471
No kernel or systemd in Field 5 on line 472
No kernel or systemd in Field 5 on line 473
No kernel or systemd in Field 5 on line 474
No kernel or systemd in Field 5 on line 475
No kernel or systemd in Field 5 on line 476
Systemd here in Field 5 on line 477 
```

好的，我想你已经明白了。除了语法不同外，它其实和在普通`bash`脚本中使用`if`没有什么不同。所以，接下来我们继续。

# 使用`while`结构并设置变量

在这一部分，我将同时介绍两个新概念。你将看到如何使用`while`循环，以及如何使用`awk`编程变量。让我们从简单的开始。

## 求一行中的数字和

在这个场景中，我们有一个包含几行数字的文件。我们希望将每一行的数字相加，并显示每一行的总和。首先，创建输入文件并使其看起来像这样`numbers_fields.txt`文件：

```
38 87 389 3 3432
34 13
38976 38 198378 38 3
3878538 38
38
893 18 3 384 352 3892 10921 10 384
348 35 293
93 1 2
1 2 3 4 5 
```

这看起来是一个相当具有挑战性的任务，因为每一行的字段数量不同。但实际上，这个任务非常简单。下面是执行此任务的`add_fields.awk`脚本：

```
#!/usr/bin/awk -f
{
addend=1
sum=0
while (addend <= NF) {
sum = sum + $addend
addend++
       		      }
print "Line " NR " Sum is " sum
} 
```

我做的第一件事是初始化`addend`和`sum`变量。`addend`变量表示字段号。通过将其初始化为`1`，脚本将始终从每行的第一个字段开始。`sum`变量被初始化为`0`，这是显而易见的。`while (addend <= NF)`这一行使得`while`循环会一直执行，直到到达行中的最后一个字段。（内建变量`NF`保存当前行中的字段数。）在下一行，使用`$addend`与列出字段号（例如`$1`或`$2`）是一样的。因此，正如你所预期的，`$addend`返回的是给定字段中包含的值。通过使用变量代替硬编码的字段号，我们可以在下一行使用`addend++`命令来进到下一字段。（这种`variable++`结构会将变量的值增加 1，和 C 语言中一样。）

如果你对这个有些困惑，请允许我进一步说明。

与普通的 shell 脚本不同，在`awk`中，你不需要在变量名前加上`$`来引用其值。在`awk`中，`$`用于引用字段的编号。因此，在`awk`中给变量名前加`$`，意味着你引用的是分配给该变量的字段号。

在`while`循环结束后，脚本会输出其信息，并显示一行中所有数字的总和。然后，它返回脚本的开头，继续执行，直到文件中的所有行都被处理完。输出如下：

```
donnie@fedora:~$ ./add_fields.awk numbers_fields.txt
Line 1 Sum is 3949
Line 2 Sum is 47
Line 3 Sum is 237433
Line 4 Sum is 3878576
Line 5 Sum is 38
Line 6 Sum is 16857
Line 7 Sum is 676
Line 8 Sum is 96
Line 9 Sum is 15
donnie@fedora:~$ 
```

它能正常工作，一切都很顺利。现在，让我们做一些改进。添加一行代码，计算每行数字的平均值并格式化输出。将文件命名为`average_fields.awk`，并使其如下所示：

```
#!/usr/bin/awk -f
{
addend=1
sum=0
while (addend <= NF) {
        sum = sum + $addend
        addend++
}
print "Line " NR "\n\t" "Sum: " sum "\n\t" "Average: " sum/NF
} 
```

这非常酷，因为我可以在数学运算中使用`NF`内建变量。在这种情况下，我只是将每行的总和除以每行的字段数。输出如下：

```
donnie@fedora:~$ ./average_fields.awk numbers_fields.txt
Line 1
	Sum: 3949
	Average: 789.8
Line 2
	Sum: 47
	Average: 23.5
. . .
. . .
Line 9
	Sum: 15
	Average: 3
donnie@fedora:~$ 
```

如果你习惯了 C 语言编程，请理解`awk`和 C 在使用变量上的区别。与 C 不同，在`awk`中，你不需要在使用变量之前声明它们。然而，有时确实需要在使用之前将它们初始化为某个特定值。此外，与 C 不同的是，`awk`中只有一种变量类型。所有`awk`变量都是字符串类型，所有`awk`数学运算符会自动识别这些变量可能代表的数值。

接下来，让我们看看一些稍微复杂的内容。

## 查找 CPU 世代

让我们考虑另一种情况。你有一台老旧的服务器，搭载的是 AMD Opteron CPU。你刚刚尝试在其上安装 Red Hat Enterprise Linux 9（RHEL 9），但无法成功安装。问题可能出在哪里？

其实，就是因为你那颗可靠的 Opteron CPU 太旧了，这意味着它缺少了新一代 CPU 的一些功能。自从 AMD 在 2003 年推出第一款 64 位 x86 CPU 以来，AMD 和英特尔一直在为其新型号增加新的功能。2020 年，英特尔、AMD、Red Hat 和 SUSE 的代表们聚集在一起，定义了 x86_64 的四个代次。每个新的代次都包含上一代没有的功能。以下是每一代发布的时间：

+   第一代：2003 年

+   第二代：2009 年

+   第三代：2013 年发布的英特尔，2015 年发布的 AMD

+   第四代：2017 年发布的英特尔，2022 年发布的 AMD

为第一代 x86_64 CPU 创建的软件也可以在后三代 CPU 上正常运行。但是，如果你拥有更新的 CPU，你可以通过使用专门为其优化的软件来提升性能。当然，这也意味着它无法在旧版 CPU 上运行。因此，如果你仍在使用第一代 x86_64 CPU，你将无法运行 RHEL 9 或任何它的克隆版本。（有传言称，RHEL 10 将至少需要第三代 x86_64 CPU，预计将在 2025 年某个时候发布。你可以在*进一步阅读*部分找到关于这一传闻的链接。）其实这也没关系，因为 Red Hat 的目标客户是那些定期更新设备的大型企业。

这使得几乎不可能有任何设备仍在运行这些第一代机器。对于普通的桌面 Linux 用户来说，这并不会造成太大影响，因为很少有家庭用户会使用 RHEL 或其克隆版本，且大多数非 RHEL 发行版仍然支持旧机器。那么，如何知道你的 CPU 属于哪一代呢？很简单，写一个脚本。

为了了解这一过程的工作原理，可以查看`/proc/cpuinfo`文件，并滚动到`flags`部分。你看到的内容将取决于你机器中的 x86_64 CPU 属于哪一代。这里是我那台 2012 年款戴尔工作站上的 Intel Xeon CPU 的`flags`部分，它是第二代 x86_64 架构：

```
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx rdtscp lm constant_tsc rep_good nopl xtopology nonstop_tsc cpuid tsc_known_freq pni pclmulqdq ssse3 cx16 sse4_1 sse4_2 x2apic popcnt aes xsave avx hypervisor lahf_lm pti md_clear flush_l1d 
```

在我的那台 2009 年款的惠普老机器上，它配有一对第一代 Opteron CPU，`flags`部分如下所示：

```
flags           : fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm 3dnowext 3dnow constant_tsc rep_good nopl nonstop_tsc cpuid extd_apicid pni monitor cx16 popcnt lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw ibs skinit wdt hw_pstate vmmcall npt lbrv svm_lock nrip_save 
```

你可以看到，我的戴尔电脑上的第二代 CPU 拥有更新的功能，例如`sse4_1`、`sse4_2`，以及一些旧版 Opteron 所不具备的其他特性。在 Linux 系统中，你可以创建一个`bash`脚本或`awk`脚本，它会自动解析`/proc/cpuinfo`文件，从而确定你的 CPU 属于哪个代次。下面是我从 StackExchange 网站的一篇帖子中借来的`x86_64_check.awk`脚本：

```
#!/usr/bin/awk -f
BEGIN {
    while (!/flags/) if (getline < "/proc/cpuinfo" != 1) exit 1
    if (/cmov/&&/cx8/&&/fpu/&&/fxsr/&&/mmx/&&/syscall/&&/sse2/) level = 1
    if (level == 1 && /cx16/&&/lahf/&&/popcnt/&&/sse4_1/&&/sse4_2/&&/ssse3/) level = 2
    if (level == 2 && /avx/&&/avx2/&&/bmi1/&&/bmi2/&&/f16c/&&/fma/&&/abm/&&/movbe/&&/xsave/) level = 3
    if (level == 3 && /avx512f/&&/avx512bw/&&/avx512cd/&&/avx512dq/&&/avx512vl/) level = 4
    if (level > 0) { print "CPU supports x86-64-v" level; exit level + 1 }
    exit 1
} 
```

首先需要注意的是，整个脚本必须放在`BEGIN`块中。这是因为我们需要一次性处理整个`cpuinfo`文件，而不是像`awk`通常那样逐行处理。`BEGIN`块帮助我们完成这项任务。

然后，请注意这个脚本使用了不同风格的 `if` 结构。与我之前给你展示的 C 语言风格不同，作者将每个 `if` 结构单独放在一行。两种风格都可以正常工作，接下来我留给你决定你喜欢哪一种。总之，这里是它如何工作的详细说明。

+   在前四个 `if` 语句的末尾，`level` 变量会被设置为新的值。

+   第一个 `if` 语句查找第一代的功能，然后将 `1` 赋值给 `level`。

+   第二个 `if` 语句验证 CPU 是否包含一级功能，查找第二代 CPU 的另一组功能，然后将 `2` 赋值给 `level`。

+   该过程会重复执行第三和第四个 `if` 语句，用于检测第三代或第四代的 CPU。

+   最后，第五个 `if` 语句验证 `level` 的值是否大于 `0`，打印出消息，然后以 `level` 加 1 的值作为退出码退出脚本。

+   `exit 1` 行的作用是，如果脚本因任何原因无法正确运行，则会以退出码 `1` 退出脚本。（第四个 `if` 语句末尾的 `level + 1` 命令防止程序成功运行时返回 `1` 作为退出码。记住，`1` 通常是表示程序未正确运行的退出码。）

关于 `awk` 变量的一件有趣的事情是，它们都是字符串变量，这大大简化了编码过程。如果你将值 `1` 赋给 `level` 变量，该值会作为字符串存储，而不是整数。但是，当你使用变量进行数学运算时，`awk` 会自动识别字符串是否是数字，因此一切都会正常工作。另外，与 `bash` 不同，`awk` 原生支持浮点数运算。所以，你可以比在 `bash` 脚本或普通编程语言中更轻松、更快速地进行数学运算。为了演示这一点，让我们在我目前运行的两台工作站上运行这个脚本。这是我那台使用 Opteron 处理器的惠普机器上运行的情况：

```
donnie@opensuse:~> ./x86-64-level_check.awk
CPU supports x86-64-v1
donnie@opensuse:~> echo $?
2
donnie@opensuse:~> 
```

正如预期的那样，脚本显示这是一个第一代 x86_64 机器。`echo $?` 命令显示退出码，这是由脚本中的 `level + 1` 命令生成的。现在，这就是我那台使用 Xeon 处理器的戴尔机器上的情况：

```
donnie@fedora:~$ ./x86-64-level_check.awk
CPU supports x86-64-v2
donnie@fedora:~$ echo $?
3
donnie@fedora:~$ 
```

所以，是的，一切运行正常。

啊，不过等等，我们还没完，因为我还没有解释 `while` 结构。这个有点复杂，所以我把它留到最后讲。

我已经向你展示过，在 `/proc/cpuinfo` 文件中，你要查找的字符串在第一字段包含 `flags` 字符串的段落中。但是，`while (!/flags/)` 语句让人看起来像是我们*没有*在寻找 `flags` 段落。（记住，`!` 是取反运算符。）要理解发生了什么，查看整个 `cpuinfo` 文件，可以通过输入 `cat /proc/cpuinfo` 来实现。你会看到相同的信息会为你系统中的每个 CPU 核心打印一次。例如，我的戴尔工作站配备了一个八核 Xeon 处理器，启用了超线程技术，这意味着我总共有 16 个虚拟 CPU 核心。因此，运行 `cat /proc/cpuinfo` 会导致相同的 CPU 信息打印 16 次。你还会看到在 `flags` 段之后会打印出更多的信息段。要理解这个 `while` 循环的工作原理，可以运行以下单行的 `awk` 命令：

```
donnie@fedora:~$ awk 'BEGIN {while (!/flags/) if (getline <"/proc/cpuinfo" == 1) print $0}'
processor	: 0
vendor_id	: GenuineIntel
. . .
. . .
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid dca sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx lahf_lm epb pti ssbd ibrs ibpb stibp tpr_shadow flexpriority ept vpid xsaveopt dtherm ida arat pln pts vnmi md_clear flush_l1d
donnie@fedora:~$ 
```

`flags` 这一段实际上只是一个很长的单行，它会在你的终端和打印页面上换行。因此，`awk` 将 `flags` 行视为单个记录。这个命令中的 `while (!/flags/)` 语句使得 `getline` 会读取 `cpuinfo` 文件，直到遇到第一个 `flags` 字符串为止。这意味着只有第一个 CPU 核心的 CPU 信息会显示出来，任何在第一个 `flags` 行之后的信息都不会显示。（是的，看起来有点困惑，但仔细想一想就能明白。）

在原始脚本中，`while` 循环中的 `if (getline < "/proc/cpuinfo" != 1) exit 1` 做了两件事。首先，它使用 `!=1` 参数检查 `cpuinfo` 文件是否存在。如果 `!=1`，也就是相当于说 *不正确*，其结果为 *正确* 时，脚本会以退出码 `1` 退出。

如果 `!=1` 参数的结果为 *假*，即文件存在，那么 `getline`，一个内置的 `awk` 函数，会读取 `cpuinfo` 文件。这样，如果你在没有 `/proc/cpuinfo` 文件的操作系统（如 FreeBSD）上运行此脚本，它会优雅地退出。

现在，有些人可能更喜欢使用 C 风格的语法来让脚本更易读。幸运的是，这很容易做到。改变语法会使脚本变得太长，无法在这里完整展示，但我可以给你看一小段代码：

```
#!/usr/bin/awk -f
BEGIN {
        while (!/flags/) {
                if (getline < "/proc/cpuinfo" !=1) {
                        exit 1
                        }
                }
. . .
. . .
if (level > 0) {
                { print "CPU supports x86-64-v" level; exit level + 1 }
        }
        exit 1
} 
```

如果你想查看完整的转换脚本，只需从 GitHub 下载 `x86-64-level_check2.awk` 文件。

我认为我们已经完成了这个脚本。让我们看看如何使用 `for` 循环和数组。

# 使用 `for` 循环和数组

一些语言，如西班牙语和法语，具有阳性和阴性名词的概念。在这个演示中，我们将使用一组英文名词、它们在西班牙语中的对应词以及西班牙语名词的性别标注。

为什么一个法国姓氏的人在创建西班牙语单词列表？嗯，尽管我有法国血统，但我在高中选择学习西班牙语而非法语。所以，我会一些西班牙语，但我不会法语。（我知道，我有点怪。）此外，我也意识到西班牙语单词*camiόn*的最后一个音节上有一个重音符号。可惜的是，在英语键盘上插入重音符号并不容易，特别是在纯文本文件中，至少不使用`awk`脚本时，不会破坏其工作。

首先，创建`spanish_words.txt`文件，并使其如下所示：

```
ENGLISH:SPANISH:GENDER
cat:gato:M
table:mesa:F
bed:cama:F
bus:camion:M
house:casa:F 
```

如你所见，我们使用冒号作为字段分隔符，并用`M`或`F`来表示单词是阳性还是阴性。第一行是表头，因此我们在处理文件时需要考虑到这一点。

接下来，创建`masc-fem.awk`脚本，如下所示：

```
#!/usr/bin/awk -f
BEGIN {FS=":"}
NR==1 {next}
$3 == "M" {masc[$2]=$1}
$3 == "F" {fem[$2]=$1}
END {
        print "\nMasculine Nouns\n----";
                for (m in masc)
                        {print m "--" masc[m]; count++}
        print "\nFeminine Nouns\n----";
                for (f in fem)
                        {print f "--" fem[f]; count2++}
        print "\nThere are " count " masculine nouns and " count2 " feminine nouns."
} 
```

在`BEGIN`部分，我们设置了`:`作为字段分隔符。`NR == 1 {next}`这一行表示忽略第一行，直接跳到下一行。接下来的两行构建了`masc`和`fem`数组。任何第三列是`M`的行都会被放入`masc`数组，而任何第三列是`F`的行则会被放入`fem`数组。`END`部分包含在主体代码运行完成后执行的代码。两个`for`循环的工作原理与正常的 Shell 脚本`for`循环相同，只是我们现在使用的是 C 语言的语法。第一个循环打印出所有的阳性名词，并使用`count`变量统计阳性名词的总数。第二个循环对阴性名词执行相同的操作，只是它使用`count2`变量统计阴性名词的数量。运行脚本时，效果如下：

```
donnie@fedora:~$ ./masc-fem.awk spanish_words.txt
Masculine Nouns
----
gato--cat
camion--bus
Feminine Nouns
----
mesa--table
casa--house
cama--bed
There are 2 masculine nouns and 3 feminine nouns.
donnie@fedora:~$ 
```

就这样，完工了。简单吧？

接下来，让我们做一些浮点数学运算。

# 使用浮点数学和`printf`

由于某种奇怪的原因，你发现自己正在做一份需要跟踪与天气相关的统计数据的工作。你的老板刚刚发给你一个文本文件，里面包含了一系列温度。有些温度是摄氏温度，有些是华氏温度。你被分配的任务是将华氏温度转换为摄氏温度。你可以通过使用计算器手动转换每个华氏温度来完成这个任务，或者你可以编写一个脚本来自动化这个过程。你决定编写脚本会更容易一些。温度列表保存在`temps.txt`文件中，内容如下：

```
Temperature Celsius_or_Fahrenheit
32 F
212 F
-40 F
-14.7 F
24.6111 C
75.8 F
55.21 F
23.9444 C
29.8 F
104.34 F
98.6 F
23.1 C 
```

前三个温度是用来检查脚本是否正确工作的。我们知道 32 华氏度等于 0 摄氏度，212 华氏度等于 100 摄氏度，最后，-40 华氏度等于-40 摄氏度。如果脚本能正确转换这三个温度，我们可以合理地确信其余的温度也会被正确转换。

确保你没有在最后一行温度数据之后插入空行，否则脚本将在最后一行实际输出数据后插入无意义的信息。

有两种方法可以表达转换公式。这里是其中一种方法：

```
(($1-32)*5) / 9 
```

这是另一种方法：

```
($1-32)/1.8 
```

`$1` 代表保存原始华氏温度的字段。所以，我们首先从华氏值中减去 32。

两种公式都同样有效，因此我们使用哪一种其实并不重要。为了好玩，我会在 `fahrenheit_to_celsius.awk` 脚本中使用第一种方法，脚本内容如下：

```
#!/usr/bin/awk -f
NR==1 ; NR>1{print ($2=="F" ? (($1-32)*5) / 9 : $1)"\t\t Celsius"} 
```

如你所见，这只是一个简单的单行代码。`NR==1` 使得标题被打印出来，而 `NR>1` 确保转换只在包含实际数据的行上进行。`print` 操作中 `?` 和 `:` 的组合被称为**三元运算符**。如果第一个条件 (`$2=="F"`) 为真，那么字段 1 的原始值将被 `?` 和 `:` 之间的值替换。在这种情况下，新值是通过执行转换计算得出的。在每行的新温度值后，我们希望打印出一对制表符，后跟“Celsius”一词。运行脚本时的结果如下：

```
donnie@fedora:~$ ./fahrenheit_to_celsius.awk temps.txt
Temperature Celsius_or_Fahrenheit
0		 Celsius
100		 Celsius
-40		 Celsius
-25.9444		 Celsius
24.6111	 Celsius
24.3333	 Celsius
12.8944	 Celsius
23.9444	 Celsius
-1.22222		 Celsius
40.1889	 Celsius
37		 Celsius
23.1		 Celsius
donnie@fedora:~$ 
```

这看起来不太好，因为一些摄氏值比其他的要长，这导致第二列的那些行没有正确对齐。我们将通过使用 `printf` 替代 `print` 来修复这个问题。

`printf` 命令允许你以 `print` 无法做到的方式定制输出。在 `awk` 中它的工作方式与 C 中相同，所以再次让 C 程序员们高兴吧。以下是 `fahrenheit_to_celsius2.awk` 脚本中解决方案的工作方式：

```
#!/usr/bin/awk -f
NR==1; NR>1{printf("%-11s %s\n",($2=="F" ? (($1-32)*5) / 9 : $1), "Celsius")} 
```

`printf` 命令中的 `%` 符号代表格式化指令。让我们来看一下 `%-11s` 指令，它格式化每行的第一个字段。`-` 告诉 `printf` 左对齐输出。（默认情况下，输出是右对齐的。）`11s` 告诉 `printf` 为第一个字段分配 11 个空格。如果任何给定行的字符串长度小于 11 个字符，`printf` 会用足够的空白填充输出，直到达到差距。最后，`%s\n` 使得 `printf` 打印出指定的文本字符串作为第二个字段，后跟换行符。（与 `print` 不同，`printf` 不会自动在行末添加换行符。）无论如何，输出如下：

```
donnie@fedora:~$ ./fahrenheit_to_celsius2.awk temps.txt
Temperature Celsius_or_Fahrenheit
0           Celsius
100         Celsius
-40         Celsius
-25.9444    Celsius
24.6111     Celsius
24.3333     Celsius
12.8944     Celsius
23.9444     Celsius
-1.22222    Celsius
40.1889     Celsius
37          Celsius
23.1        Celsius
donnie@fedora:~$ 
```

是的，看起来好多了。但是，如果你不需要看到所有的小数位怎么办？很简单，只需要为第一个字段使用不同的格式化指令，正如你在 `fahrenheit_to_celsius3.awk` 脚本中看到的那样：

```
#!/usr/bin/awk -f
NR==1; NR>1{printf("%.2f %s\n",($2=="F" ? (($1-32)*5) / 9 : $1), "Celsius")} 
```

这里，我使用 `%.2f` 将输出格式化为一个浮点数，且小数点后只有两位数字。输出的结果如下：

```
donnie@fedora:~$ ./fahrenheit_to_celsius3.awk temps.txt
Temperature Celsius_or_Fahrenheit
0.00 Celsius
100.00 Celsius
-40.00 Celsius
-25.94 Celsius
24.61 Celsius
24.33 Celsius
12.89 Celsius
23.94 Celsius
-1.22 Celsius
40.19 Celsius
37.00 Celsius
23.10 Celsius
donnie@fedora:~$ 
```

唯一的小问题是，现在第二列对不齐了。你不能将两个指令用于单个字段，所以我们只能接受这样的格式。不过，这也没关系，因为如果你决定将输出重定向到文件中，依然可以将它导入到电子表格程序中。

通过稍微修改公式，你也可以将摄氏温度转换为华氏温度。以下是执行此操作的`celsius_to_fahrenheit2.awk`脚本：

```
#!/usr/bin/awk -f
NR==1; NR>1{printf("%-11s %s\n",($2=="C" ? (($1+32)*9) / 5 : $1),"Fahrenheit")} 
```

这是输出结果：

```
donnie@fedora:~$ ./celsius_to_fahrenheit2.awk temps.txt
Temperature Celsius_or_Fahrenheit
32            Fahrenheit
212           Fahrenheit
-40           Fahrenheit
-14.7         Fahrenheit
101.9         Fahrenheit
75.8          Fahrenheit
55.21         Fahrenheit
100.7         Fahrenheit
29.8          Fahrenheit
104.34        Fahrenheit
98.6          Fahrenheit
99.18         Fahrenheit
donnie@fedora:~$ 
```

接下来，假设我们不关心查看温度列表，只想看到平均温度。那么，以下是实现这一目标的`average_temp.awk`脚本：

```
#!/usr/bin/awk -f
BEGIN{temp_sum=0; total_records=0; print "Calculate the average temperature."}
$2=="F"{temp_sum += ($1-32) / 1.8; total_records += 1;}
$2=="C" {temp_sum += $1; total_records += 1}
END {average_temp = temp_sum/total_records; print "The average temperature is: \n\t " average_temp " Celsius."} 
```

这次，我决定使用从华氏度转换为摄氏度的替代公式。在两个`$2==`的行中，我使用了`+=`运算符来求和第一个字段中的温度，并递增`total_records`变量。以下是输出结果：

```
donnie@fedora:~$ ./average_temp.awk temps.txt
Calculate the average temperature.
The average temperature is:
	 18.2421 Celsius.
donnie@fedora:~$ 
```

为了验证它是否给出了准确的平均值，可以尝试修改`temps.txt`文件中的不同温度值，看看会发生什么。

关于这个话题最后要说的是，`awk`提供了全套数学运算符，并且有很多有用的数学函数。你可以通过查阅*进一步阅读*部分中的链接了解更多信息。

接下来，让我们看看如何处理多行记录。

# 处理多行记录

到目前为止，我们一直在使用`awk`解析每一行都是独立记录的文本文件。不过，有时你可能需要处理一些文件，其中每条记录跨越多行。例如，看看这个`inventory.txt`文件：

```
Kitchen spatula
$4.99
Housewares
Raincoat
$36.99
Clothing
On Sale!
Claw hammer
$7.99
Tools 
```

第一条和第三条记录各由三行组成，第二条记录由四行组成。每条记录由空行分隔。现在，假设我们需要将这些信息导入到电子表格中。由于多行记录的存在，这样做效果不好，所以我们需要找到一种简单的方法将其转换为适合电子表格格式的内容。再次使用`awk`来拯救我们！以下是帮助我们完成这一任务的`inventory.awk`脚本：

```
#!/usr/bin/awk -f
BEGIN {
        FS="\n"
        RS=""
        ORS=""
}
{
        count=1
        while (count<NF) {
                print $count "\t"
                count++
        }
        print $NF "\n"
} 
```

`BEGIN`块将换行符（`\n`）定义为字段分隔符，这意味着每一行都算作记录中的一个独立字段。记录分隔符（`RS`）和输出记录分隔符（`ORS`）都被定义为空值（`""`）。`RS`变量将空值解读为空白行，而`ORS`变量不会。在这种情况下，`ORS`被定义为空值只是为了防止`while`循环中的`print`命令在每个字段末尾添加换行符。

`while` 循环中没有什么是你没见过的。它只是使用 `count` 变量来保存正在处理的字段的编号。这个 `count` 变量的值在每次迭代 `while` 循环后都会增加 1。你可能会觉得奇怪，为什么循环条件定义为 `count<NF` 而不是 `count<=NF`。我的意思是，我们难道不想处理每个记录的最后一个字段吗？嗯，确实是的，我们通过 `while` 循环后的 `print $NF "\n"` 命令来处理最后一个字段。正如我所说，定义 `OFS` 为 `""` 可以防止 `print` 命令在每个字段的末尾添加换行符。因此，为了让每条记录在单独的一行上，我们必须为最后一个字段单独使用一个 `print` 命令，并指定它在行末添加换行符。无论如何，以下是我使用该脚本解析 `inventory.txt` 文件时的结果：

```
donnie@fedora:~$ ./inventory.awk inventory.txt
Kitchen spatula	$4.99		Housewares
Raincoat		$36.99		Clothing	On Sale!
Claw hammer		$7.99		Tools
donnie@fedora:~$ 
```

当然，你可以将输出重定向到 `.tsv` 文件中，这样你就可以在你喜欢的电子表格程序中打开它。如果你更愿意使用 `.csv` 文件，只需在 `while` 循环中的 `print $count "\t"` 行替换为 `print $count ","`。输出将如下所示：

```
donnie@fedora:~$ ./inventory.awk inventory.txt
Kitchen spatula,$4.99,Housewares
Raincoat,$36.99,Clothing,On Sale!
Claw hammer,$7.99,Tools
donnie@fedora:~$ 
```

这个脚本的优点是，无论每条记录中有多少字段都不重要。`NF` 内建变量跟踪字段的数量，`while` 循环根据记录的字段数量进行处理。

好的，我想这差不多就结束了我们对 `awk` 脚本的介绍。所以，让我们总结一下并继续前进。

# 总结

`awk` 脚本语言对于需要从纯文本文件中提取有意义数据的人来说极其有用。我从这一章开始时，首先向你展示了一个 `awk` 脚本的基本结构。接着，我展示了如何使用 `if` 和 `if..else` 创建条件命令，如何使用 `while` 循环，以及如何解析包含多行记录的文本文件。在演示中，我向你展示了各种 `awk` 脚本，这些脚本执行的任务是你在现实生活中可能遇到的类型。

不幸的是，我无法完全展现 `awk` 的所有内容。它是一个那种有完整书籍专门讨论的主题，所以我能做的最好就是让你对它产生兴趣。

在下一章中，我们将回到 Shell 脚本的主要主题，介绍几个工具，它们允许你为脚本创建用户界面。我们在那里见。

# 问题

1.  为了最佳的可移植性，你应该在 `awk` 脚本中使用以下哪一行 shebang？

    1.  `#!/bin/awk`

    1.  `#!/usr/bin/awk -f`

    1.  `#!/bin/awk -f`

    1.  `#!/usr/bin/awk`

1.  以下哪项陈述是正确的？

    1.  所有 `awk` 编程变量必须在使用之前声明。

    1.  `awk` 变量有多种类型，例如整数、浮点数和字符串。

    1.  所有 `awk` 编程变量都是字符串类型的变量。

    1.  在进行数学运算之前，你必须将字符串类型的变量转换为整数或浮点数类型。

1.  在`awk`脚本中，如何定义逗号为字段分隔符？

    1.  `-F=,`

    1.  `-F=","`

    1.  `FS=,`

    1.  . `FS=","`

1.  你有一个包含多行数字的文本文件，数字之间有空格。每一行的数字数量不同。你想计算每行的数字之和。当你编写一个`awk`脚本来计算这些和时，如何处理每行字段数量不同的问题？

    1.  创建一个变量数组来存储每行的字段数，并使用`for`循环构建该数组。

    1.  使用内建变量`NF`。

    1.  使用内建变量`NR`。

    1.  只有当每一行有相同数量的字段时，才能执行此操作。

1.  以下哪个`printf`指令可以确保浮点数总是显示四位小数？

    1.  `%.4f`

    1.  `%4`

    1.  `#4.f`

    1.  `#4`

# 进一步阅读

+   如何检查我的 CPU 是否支持 x86_v2？: [`unix.stackexchange.com/questions/631217/how-do-i-check-if-my-cpu-supports-x86-64-v2`](https://unix.stackexchange.com/questions/631217/how-do-i-check-if-my-cpu-supports-x86-64-v2)

+   探索 x86-64-v3 在 Red Hat Enterprise Linux 10 中的应用: [`developers.redhat.com/articles/2024/01/02/exploring-x86-64-v3-red-hat-enterprise-linux-10`](https://developers.redhat.com/articles/2024/01/02/exploring-x86-64-v3-red-hat-enterprise-linux-10)

+   x86_64 级别: `medium.com/@BetterIsHeather/x86-64-levels-944e92cd6d83`

+   《awk 命令如何让我成为 10 倍工程师》: [`youtu.be/FbSpuZVb164?si=ri9cnjBh1sxM_STz`](https://youtu.be/FbSpuZVb164?si=ri9cnjBh1sxM_STz)

+   三元运算符: [`www.tutorialspoint.com/awk/awk_ternary_operators.htm`](https://www.tutorialspoint.com/awk/awk_ternary_operators.htm)

+   printf 示例: [`www.gnu.org/software/gawk/manual/html_node/Printf-Examples.html`](https://www.gnu.org/software/gawk/manual/html_node/Printf-Examples.html)

+   使用 awk 进行数学运算: [`www.networkworld.com/article/942538/doing-math-with-awk.html`](https://www.networkworld.com/article/942538/doing-math-with-awk.html)

+   awk 中的数值函数: [`www.gnu.org/software/gawk/manual/html_node/Numeric-Functions.html`](https://www.gnu.org/software/gawk/manual/html_node/Numeric-Functions.html)

# 答案

1.  b

1.  c

1.  d

1.  b

1.  a

# 加入我们的 Discord 社区！

与其他用户、Linux 专家以及作者本人一起阅读这本书。

提问、为其他读者提供解决方案、通过“问我任何问题”环节与作者交流，等等。扫描二维码或访问链接加入社区。

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code10596186092701843.png)
