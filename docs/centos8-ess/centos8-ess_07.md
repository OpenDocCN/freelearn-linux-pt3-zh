8. 使用 CentOS 8 上的 Bash Shell

学习如何在 CentOS 8 和一般的 Linux 发行版中工作，重要的一部分是熟练使用 Shell 环境。虽然 Linux 提供的图形桌面环境（如 GNOME）为操作系统提供了一个用户友好的界面，但实际上，Shell 环境提供了比图形桌面工具更强大的功能、灵活性和自动化能力。Shell 环境还为在没有桌面环境时与操作系统交互提供了一种手段；这在使用基于服务器的操作系统（如 CentOS 8）或无法完全启动的损坏系统时是常见的情况。

因此，本章的目标是概述 CentOS 8 的默认 Shell 环境（特别是 Bash shell）。

8.1 什么是 Shell？

Shell 是一个交互式命令解释器环境，用户可以在提示符下输入命令或将命令写入脚本文件并执行。Shell 的起源可以追溯到 UNIX 操作系统的早期。事实上，在 Linux 的早期，在引入图形桌面环境之前，Shell 是用户与操作系统交互的唯一方式。

多种 shell 环境在多年间陆续开发出来。第一个广泛使用的 shell 是由 Stephen Bourne 在贝尔实验室编写的 Bourne shell。

另一个早期的创作是 C shell，它与 C 编程语言共享一些语法相似性，并引入了命令行编辑和历史记录等可用性增强功能。

Korn shell（由 David Korn 在贝尔实验室开发）基于 Bourne shell 和 C shell 提供的功能。

CentOS 8 的默认 Shell 是 Bash Shell（即 Bourne Again SHell 的缩写）。这个 Shell 最初作为 Bourne shell 的开源版本出现，由 Brian Fox 为 GNU 项目开发，并基于 Bourne shell 和 C shell 提供的功能。

8.2 访问 Shell

在 GNOME 桌面环境中，可以通过从顶部栏选择“Activities”选项，在搜索栏中输入“Terminal”，并点击 Terminal 图标来访问 Shell 提示符。

例如，当远程登录 CentOS 8 服务器时（使用 SSH），用户也会看到 Shell 提示符。关于如何使用 SSH 访问远程服务器的详细信息将在《在 CentOS 8 上配置基于 SSH 的认证》一章中介绍。当启动一个未安装桌面环境的基于服务器的系统时，用户在完成登录程序后，会立即进入 Shell 界面（无论是在物理控制台终端还是远程登录会话中）。

8.3 在提示符下输入命令

命令是在 shell 提示符下通过输入命令并按 Enter 键来执行的。虽然有些命令在执行时不会输出任何信息，但大多数命令在返回提示符之前会显示某种形式的输出。例如，ls 命令可以用来显示当前工作目录中的文件和目录：

$ ls

桌面 文档 下载 音乐 图片 公共 模板 视频

可用的命令要么是内置在 shell 中，要么存在于物理文件系统中。可以使用 which 命令来查找命令在文件系统中的位置。例如，要查找 ls 可执行文件在文件系统中的位置：

$ which ls

alias ls=’ls --color=auto’

/usr/bin/ls

显然，ls 命令位于 /usr/bin 目录下。还需注意，有一个别名已配置，这一主题将在本章后续部分讨论。使用 which 命令查找内置于 shell 中的命令的路径时，会显示一个消息，表明无法找到可执行文件。例如，尝试查找 history 命令的位置（该命令实际上是内置在 shell 中，而不是作为文件系统中的可执行文件存在）时，会得到类似如下的输出：

$ which history

/usr/bin/which: no history in (/home/demo/.local/bin:/home/demo/bin:/usr/share/Modules/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)

8.4 获取命令信息

许多 Linux shell 中可用的命令开始时可能看起来有些难以理解。要查找有关命令的详细信息及其使用方法，可以使用 man 命令并指定命令名称作为参数。例如，要了解更多关于 pwd 命令的信息：

$ man pwd

执行上述命令时，将显示 pwd 命令的详细描述。许多命令在使用 --help 命令行选项运行时，还会提供额外的信息：

$ wc --help

8.5 Bash 命令行编辑

早期的 shell 环境没有提供任何形式的行编辑功能。这意味着，如果你在输入一个长命令时发现开头有错误，你必须删除后面的所有字符，纠正错误后再重新输入其余的命令。幸运的是，Bash 提供了广泛的命令行编辑选项，如下表所示：

| 键盘快捷键 | 操作 |
| --- | --- |
| Ctrl-b 或 左箭头 | 将光标向后移动一个位置 |
| Ctrl-f 或 右箭头 | 将光标向前移动一个位置 |
| Delete | 删除当前光标下的字符 |
| Backspace | 删除光标左侧的字符 |
| Ctrl-_ | 撤销上一个更改（可以重复操作以撤销所有更改） |
| Ctrl-a | 将光标移到行首 |
| Ctrl-e | 将光标移到行末 |
| Meta-f 或 Esc 然后 f | 将光标向前移动一个单词 |
| Meta-b 或 Esc 然后 b | 将光标向后移动一个单词 |
| Ctrl-l | 清除屏幕上除当前命令外的所有内容 |
| Ctrl-k | 从当前位置删除到行尾 |
| Meta-d 或 Esc 然后 d | 删除到当前单词的末尾 |
| Meta-DEL 或 Esc 然后 DEL | 删除到当前单词的开始位置 |
| Ctrl-w | 从当前位置删除到前一个空白字符 |

表 8-1

8.6 Shell 历史记录的使用

除了命令行编辑功能外，Bash Shell 还提供了命令行历史记录支持。可以使用 history 命令查看先前执行的命令列表：

$ history

1 ps

2 ls

3 ls –l /

4 ls

5 man pwd

6 man apropos

此外，可以使用 Ctrl-p（或上箭头）和 Ctrl-n（或下箭头）来前后滚动查看之前输入的命令。当显示出历史记录中的所需命令时，按 Enter 键执行该命令。

另一个选项是输入 ‘!’ 字符，后跟要重复执行的命令的前几个字符，然后按 Enter 键。

8.7 文件名简写

许多 Shell 命令都需要一个或多个文件名作为参数。例如，要显示名为 list.txt 的文本文件的内容，可以使用 cat 命令，命令如下：

$ cat list.txt

同样，可以通过指定所有文件名作为参数，显示多个文本文件的内容：

$ cat list.txt list2.txt list3.txt list4.txt

与其输入每个文件名，模式匹配可以用来指定所有符合特定标准的文件。例如，‘*’ 通配符可以简化上述示例：

$ cat *.txt

上述命令将显示所有以 .txt 结尾的文件的内容。你还可以进一步限制，只显示以 list 开头，且以 .txt 结尾的文件：

$ cat list*.txt

可以使用 ‘?’ 字符指定单个字符匹配：

$ cat list?.txt

8.8 文件名和路径补全

与其输入整个文件名或路径，或者使用模式匹配来减少输入量，Shell 提供了文件名自动补全功能。要使用文件名自动补全，只需输入文件或路径名的前几个字符，然后按两次 Esc 键。Shell 将自动为你完成文件名，补全目录中与输入字符匹配的第一个文件或路径名。要获取可能的匹配项列表，输入前几个字符后按 Esc = 键。

8.9 输入和输出重定向

如前所述，许多 Shell 命令在执行时会输出信息。默认情况下，这些输出会发送到一个名为 stdout 的设备文件，它本质上就是 Shell 正在运行的终端窗口或控制台。相反，Shell 从名为 stdin 的设备文件获取输入，默认情况下它是键盘。

可以使用 ‘>’ 字符将命令的输出从标准输出（stdout）重定向到文件系统上的实际文件。例如，要将 ls 命令的输出重定向到名为 files.txt 的文件，所需的命令如下：

$ ls *.txt > files.txt

执行完成后，`files.txt` 将包含当前目录中文件的列表。类似地，文件的内容也可以作为命令的输入替代标准输入（stdin）。例如，要将文件内容重定向为命令的输入：

$ wc –l < files.txt

上面的命令将显示 `files.txt` 文件中包含的行数。

需要注意的是，‘>’ 重定向操作符会创建一个新文件，或者在使用时截断已有文件。为了向已有文件追加内容，可以使用 ‘>>’ 操作符：

$ ls *.dat >> files.txt

除了标准输出外，shell 还提供了标准错误输出（stderr）。当命令的输出被重定向到 `stdout` 时，任何由命令生成的错误信息会被重定向到 `stderr`。这意味着，如果 `stdout` 被重定向到文件，错误信息仍会显示在终端。这通常是期望的行为，尽管如果需要，`stderr` 也可以使用 ‘2>’ 操作符进行重定向：

$ ls dkjfnvkjdnf 2> errormsg

命令执行完毕后，名为 `dkjfnvkjdnf` 的文件无法找到的错误信息将被保存在 `errormsg` 文件中。

`stderr` 和 `stdout` 可以通过 `&>` 操作符重定向到同一个文件：

$ ls /etc dkjfnvkjdnf &> alloutput

执行完成后，`alloutput` 文件将包含 `/etc` 目录内容的列表，以及尝试列出一个不存在的文件时产生的错误信息。

8.10 在 Bash Shell 中使用管道

除了 I/O 重定向外，shell 还允许将一个命令的输出直接通过管道作为另一个命令的输入。管道操作是通过在命令行上的两个或多个命令之间插入 ‘|’ 字符来实现的。例如，要统计系统上正在运行的进程数量，可以将 `ps` 命令的输出通过管道传递给 `wc` 命令：

$ ps –ef | wc –l

在命令行上可以执行任意数量的管道操作。例如，要查找一个包含 "Smith" 名字的文件中行数：

$ cat namesfile | grep Smith | wc –l

8.11 配置别名

随着你对 shell 环境的熟练掌握，你可能会发现自己经常使用相同参数的命令。例如，你可能经常使用带有 `l` 和 `t` 选项的 `ls` 命令：

$ ls –lt

为了减少输入命令时的键入量，可以创建一个别名，将命令和参数映射到该别名。例如，要创建一个别名，使得输入字母 `l` 时执行 `ls –lt` 命令，可以使用以下语句：

$ alias l="ls –lt"

在命令提示符下输入 `l` 现在会执行原始语句。

8.12 环境变量

Shell 环境变量提供数据和配置设置的临时存储。Shell 本身会设置一些环境变量，用户可以更改这些变量，以修改 Shell 的行为。可以使用 env 命令获取当前定义的变量列表：

$ env

SSH_CONNECTION=192.168.0.19 61231 192.168.0.28 22

MODULES_RUN_QUARANTINE=LD_LIBRARY_PATH

LANG=en_US.UTF-8

HISTCONTROL=ignoredups

HOSTNAME=-pc.ebookfrenzy.com

XDG_SESSION_ID=15

MODULES_CMD=/usr/share/Modules/libexec/modulecmd.tcl

USER=demo

ENV=/usr/share/Modules/init/profile.sh

SELINUX_ROLE_REQUESTED=

PWD=/home/demo

HOME=/home/demo

SSH_CLIENT=192.168.0.19 61231 22

SELINUX_LEVEL_REQUESTED=

.

.

.

可能最有用的环境变量是 PATH。它定义了 Shell 在命令提示符下输入命令时，会搜索的目录及其顺序。新安装的 CentOS 8 系统上，用户账户的 PATH 环境变量可能会被配置如下：

$ echo $PATH

/home/demo/.local/bin:/home/demo/bin:/usr/share/Modules/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

另一个有用的变量是 HOME，它指定当前用户的主目录。例如，如果你希望 Shell 也在主目录中的 scripts 目录中查找命令，可以按如下方式修改 PATH 变量：

$ export PATH=$PATH:$HOME/scripts

可以使用 echo 命令显示现有环境变量的当前值：

$ echo $PATH

你可以使用 export 命令创建自己的环境变量。例如：

$ export DATAPATH=/data/files

一个有用的技巧是使用反引号（`）将命令的输出赋值给环境变量。例如，要将当前日期和时间赋值给名为 NOW 的环境变量：

$ export NOW=`date`

$ echo $NOW

2019 年 4 月 2 日 星期二 13:48:40 EDT

如果每次进入 Shell 环境时需要配置环境变量或别名设置，可以将它们添加到位于主目录中的 .bashrc 文件中。例如，以下示例 .bashrc 文件配置了 DATAPATH 环境变量和一个别名：

# 第八章：.bashrc

# 加载全局定义

if [ -f /etc/bashrc ]; then

. /etc/bashrc

fi

# 用户特定的环境

PATH="$HOME/.local/bin:$HOME/bin:$PATH"

export PATH

# 如果你不喜欢 systemctl 的自动分页功能，请取消注释以下行：

# export SYSTEMD_PAGER=

# 用户特定的别名和函数

export DATAPATH=/data/files

alias l="ls -lt"

8.13 编写 Shell 脚本

到目前为止，我们一直专注于 Bash shell 的交互性质。所谓交互性，指的是手动在提示符下逐一输入命令并执行。实际上，这只是 shell 能力的一小部分。可以说，shell 最强大的特性之一就是它可以创建 shell 脚本。Shell 脚本本质上是包含一系列语句的文本文件，可以在 shell 环境中执行这些语句以完成任务。除了能够执行命令，shell 还提供了许多编程构造，如 for 和 do 循环、if 语句等，这是你在脚本语言中通常会看到的功能。

不幸的是，关于 shell 脚本的详细概述超出了本章的范围。然而，有许多书籍和网站资源专门讲解 shell 脚本，它们远比我们在这里所能达到的更能公正地讲解这一主题。因此，在本节中，我们只会提供一个非常简单的 shell 脚本介绍。

创建 shell 脚本的第一步是创建一个文件（在本示例中，我们将其命名为 simple.sh），并将以下内容作为第一行添加：

#!/bin/sh

#! 被称为“shebang”，它是一种特殊的字符序列，表示接下来一项内容是执行脚本所需的解释器的路径（在此示例中是位于 /bin 的 sh 可执行文件）。如果你想使用的是其他解释器，比如 /bin/csh 或 /bin/ksh，这里同样可以写成这样的路径。

下一步是编写一个简单的脚本：

#!/bin/sh

for i in *

do

echo $i

done

这个脚本的作用是遍历当前目录中的所有文件，并显示每个文件的名称。可以通过将脚本名称作为参数传递给 sh 来执行它：

$ sh simple.sh

为了使文件可执行（从而无需将其传递给 sh 命令），可以使用 chmod 命令：

$ chmod +x simple.sh

一旦文件的执行位被设置，它就可以直接执行。例如：

$ ./simple.sh

8.14 总结

在《CentOS 8 Essentials》一章中，我们简要浏览了 Bash shell 环境。在图形化桌面环境的世界里，我们很容易忘记操作系统的真正力量和灵活性往往只有通过跳出用户友好的桌面界面，使用 shell 环境才能得到充分发挥。此外，当需要管理和维护没有安装桌面的服务器系统，或者在修复损坏到无法启动桌面或 Cockpit 接口的系统时，对 shell 的熟悉是必不可少的。

Shell 的能力远远超出了本章涵盖的范围。如果你是 Shell 的新手，我们强烈建议你寻找额外的资源。一旦熟悉了这些概念，你会很快发现，在终端窗口中使用 Shell 来执行许多任务比在桌面上浏览菜单和对话框要快得多。
