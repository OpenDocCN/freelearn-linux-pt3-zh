# 19

# Shell 脚本的可移植性

如我们稍后会看到，Linux、Unix 和类 Unix 操作系统有很多不同的 Shell 可用。但到目前为止，我们主要一直在使用`bash`。`bash`的一个大优点是它已经在大多数 Linux 发行版、macOS 和 OpenIndiana 上预装了。它通常不会在 BSD 类型的发行版上默认安装，但如果需要，你可以自行安装。

`bash`的一个大优点是它可以使用不同的脚本结构，这些结构能让脚本编写者的工作变得更轻松。`bash`的一个大缺点是许多这些`bash`结构在非`bash`的 Shell 中并不总是可用。如果你能在所有机器上安装`bash`，那这并不是一个大问题，但并不是每次都能做到这一点。（稍后我会解释为什么。）

在这一章中，我将展示一些你可能遇到的`bash`替代方案，以及如何让你的 Shell 脚本在这些 Shell 上运行。

本章内容包括：

+   在非 Linux 系统上运行`bash`

+   理解 POSIX 合规性

+   理解 Shell 之间的差异

+   理解 bashisms

+   测试脚本的 POSIX 合规性

介绍部分就到这里。现在，让我们深入探讨一下。

# 技术要求

我在这一章中使用了 Fedora、Debian 和 FreeBSD 虚拟机。如果你们中的任何人使用的是 Mac，也可以尝试在上面运行这些脚本。如果你愿意，像往常一样，你可以从 GitHub 获取脚本：

```
git clone https://github.com/PacktPublishing/The-Ultimate-Linux-Shell-Scripting-Guide.git 
```

# 在非 Linux 系统上运行 bash

在我们讨论`bash`的替代方案之前，让我们先谈谈在非 Linux 操作系统上使用`bash`。我的意思是，确保脚本在每个地方都使用相同的 Shell，是让你的 Shell 脚本具备移植性最简单的方式。

正如我在介绍中提到的，`bash`已经在大多数基于 Linux 的操作系统中安装好了，也包括 macOS 和 OpenIndiana。如果你想在 BSD 类型的发行版上使用`bash`，比如 FreeBSD 或 OpenBSD，你需要自己安装它。我已经在*第八章 基本 Shell 脚本构建*中展示了如何在 FreeBSD 上安装`bash`。为了提醒你，我再展示一遍：

```
donnie@freebsd14:~ $ sudo pkg install bash 
```

在任何其他 BSD 类型的发行版上安装`bash`同样简单，只是它们使用不同的包管理器。下面是各种 BSD 系统的命令表：

| **BSD 发行版** | **安装 bash 的命令** |
| --- | --- |
| FreeBSD | `sudo pkg install bash` |
| OpenBSD | `sudo pkg_add bash` |
| DragonflyBSD | `sudo pkg install bash` |
| NetBSD | `sudo pkgin install bash` |

表 19.1：在 BSD 发行版上安装 bash 的命令

现在，如果你在这些 BSD 发行版中的任何一个上运行带有`#!/bin/bash`的脚本，你会看到一个错误信息，类似下面这样：

```
donnie@freebsd14:~ $ ./ip.sh
-sh: ./ip.sh: not found
donnie@freebsd14:~ $ 
```

这个消息有点误导，因为它给人一种 shell 找不到脚本的印象。实际上，脚本找不到的是`bash`。这是因为，与在 Linux 和 OpenIndiana 中看到的不同，`bash`在 BSD 发行版的`/bin/`目录中并不存在。相反，BSD 发行版将`bash`放在`/usr/local/bin/`目录中。有几种方法可以解决这个问题，我们接下来会看到。

## 使用 env 设置 bash 环境

确保你的脚本总是能找到`bash`的第一种方法是将脚本中的`#!/bin/bash`行替换为：

```
#!/usr/bin/env bash 
```

这会导致脚本在用户的`PATH`环境中查找`bash`可执行文件，而不是在特定的硬编码位置查找。

我在*第三章，理解变量和管道*中向你展示了如何使用`env`命令查看在`bash`中设置的环境变量。在这种情况下，我使用`env`来指定我希望用来解释该脚本的 shell。

这是最简单的方法，但也存在一些潜在问题。

+   如果你为一个已安装多个版本的解释器指定路径，你将无法知道脚本会调用哪个版本。如果机器上安装了多个`bash`版本，你就无法知道`#!/usr/bin/env bash`会调用哪个版本。

+   使用`#!/usr/bin/env bash`可能会存在安全问题。正常的`bash`可执行文件总是安装在一个只有 root 权限的用户才能添加、删除或修改文件的目录中。但假设恶意黑客获得了你正常用户账户的访问权限。在这种情况下，他或她就不需要获取 root 权限，便可以在你的家目录中植入一个恶意的伪`bash`，并更改你的`PATH`环境变量，使得伪`bash`被调用，而不是实际的`bash`。使用硬编码的`#!/bin/bash`行可以避免这个问题。

+   使用`#!/usr/bin/env bash`时，你将无法在 shebang 行中调用任何`bash`选项。例如，假设你需要排查一个有问题的脚本，并且想以调试模式运行该脚本。使用`#!/bin/bash --debug`可以正常工作，但使用`#!/usr/bin/env bash --debug`则不行。原因在于，`env`命令只能识别一个选项参数，这个选项在此情况下是`bash`。(它永远不会看到`--debug`选项。)

暂时不用担心`--debug`选项的作用。我会在*第二十一章，调试 Shell 脚本*中向你展示更多关于调试脚本的内容。

尽管使用`#!/usr/bin/env bash`看起来是最简单的解决方案，但我更倾向于在可能的情况下避免使用它。所以，让我们看一下另一个稍微更安全和可靠的解决方案。

## 创建指向 bash 的符号链接

我推荐的解决方案是确保你的脚本在所有 BSD 发行版上都能找到`bash`，你只需要在`/bin/`目录下创建一个指向`bash`可执行文件的符号链接。下面是操作步骤：

```
donnie@freebsd14:~ $ which bash
/usr/local/bin/bash
donnie@freebsd14:~ $ sudo ln -s /usr/local/bin/bash /bin/bash
Password:
donnie@freebsd14:~ $ which bash
/bin/bash
donnie@freebsd14:~ $ 
```

使用`ln -s`命令创建符号链接时，你首先需要指定要链接的文件路径。然后，指定你想要创建的链接的路径和名称。注意，在我创建链接之后，`which`命令现在能在`/bin/`目录下找到`bash`，而不是在`/usr/local/bin/`目录下。那是因为`/bin/`在默认的`PATH`设置中排在`/usr/local/bin/`之前，正如你在这里看到的：

```
donnie@freebsd14:~ $ echo $PATH
/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin:/home/donnie/bin
donnie@freebsd14:~ $ 
```

你现在可以在你的 BSD 机器上使用`#!/bin/bash`的 shebang 行，就像在 Linux 机器上一样。

好的，如果你能在所有系统上安装`bash`，那这都没问题。但如果不能呢？如果你需要编写能够在`bash`以及其他 Shell 上运行的脚本呢？为了回答这个问题，我们首先需要了解一下叫做**POSIX**的东西。

# 理解 POSIX 合规性

在计算机的远古时代，没有所谓的标准化。为了让事情有意义，我们从 Unix 的出现开始讲述历史。

AT&T 在 1970 年代初期创造了 Unix，但直到 1983 年才被允许将其推向市场。这是由于美国政府在 1956 年对 AT&T 提起的反垄断法律案件。（我不了解这个案件的详细情况，所以请不要问。）但他们被允许将代码许可给其他厂商，以便他们可以销售自己的修改版实现。因此，多个不同的 Unix 实现相继出现，它们之间并不总是完全兼容。这些不同的实现包括**伯克利软件发行版**（**BSD**）、**微软 Xenix**和**SunOS**。1983 年，AT&T 终于被允许推销自己的**System V Unix**。更有趣的是，最终出现了多种不同的 Shell，供这些不同的 Unix 系统使用。最初是汤普森 Shell，几年后被 Bourne Shell 所取代。其他 Shell，如 C Shell 和 Korn Shell，紧随其后。最后，在 1990 年代初，Linus Torvalds 创建了 Linux 内核，它是 Unix 内核的干净重写。如今，基于 Linux 的操作系统基本上取代了基于 Unix 的系统，而 Bourne Again Shell（`bash`）是 Linux 系统中占主导地位的 Shell。

在 1980 年代，客户开始要求不同 Unix 厂商之间有某种程度的标准化。这最终导致了**IEEE 计算机学会**在 1988 年创建了**POSIX**标准。

**IEEE**代表**电气和电子工程师协会**。有趣的是，您在他们的网站上看到的仅仅是 IEEE，而没有解释它的全称。

POSIX 代表**可移植操作系统接口**。

最后，对于在美国以外的你们，AT&T 代表**美国电话电报公司**。

没错，微软曾经是一个 Unix 厂商。

POSIX 的主要目的是确保所有 Unix 实现中的 shell 和工具以相同的方式工作。例如，如果你知道如何在 Solaris/OpenIndiana 上使用 `ps` 工具，那么你也知道如何在 FreeBSD 上使用它。问题在于，虽然许多 Unix 实现及其默认 shell 符合 POSIX 标准，但有些仅部分符合。另一方面，大多数基于 Linux 的操作系统只是部分符合 POSIX 标准。这是因为许多 Linux 工具使用一些 Unix/Unix-like 系统没有的选项开关，而 `bash` 本身有一些 POSIX shell 无法提供的编程特性。例如，你可以在符合 POSIX 标准的 shell 中使用 `if [ -f /bin/someprogram ]` 这种构造来测试文件是否存在，但你不能使用 `if [[ -f /bin/someprogram ]]`。 （稍后我会详细解释这一点。）

POSIX 还定义了任何给定 Unix 实现必须包含哪些编程库。然而，这主要对实际的程序员有用，而非对 shell 脚本编写者。

公平地说，我承认如果你只在 Linux 服务器或工作站上工作，POSIX 合规性可能永远不会成为问题。但假设你正在处理一大批服务器，包含了 Unix、BSD 和 Linux 系统的混合配置。现在，假设这些 Unix 和 BSD 服务器中的一些仍在运行没有 `bash` 的旧版系统，你可能需要创建 POSIX 合规的 shell 脚本，以便在整个服务器群集上运行而不需要修改。此外，**物联网**（**IoT**）设备通常是资源非常有限的设备，可能无法运行 `bash`。相反，它们会使用更轻量级的工具，如 `ash` 或 `dash`。一般来说，这些轻量级的非 `bash` shell 会符合 POSIX 标准，且无法运行使用 `bash` 特有功能的脚本。

现在你已经对 POSIX 有了较好的理解，让我们来谈谈不同 shell 之间的差异。

# 理解 Shell 之间的差异

除了定义你想使用的特定 shell，例如 `/bin/bash` 或 `/bin/zsh`，你还可以定义通用的 `/bin/sh` shell，使你的脚本更具可移植性，如下所示：

```
#!/bin/sh 
```

这个通用的 `sh` shell 允许你在可能安装也可能没有安装 `bash` 的不同系统上运行脚本。但问题是，很多年前，`sh` 总是指 Bourne shell。如今，`sh` 在一些操作系统中仍然是 Bourne shell，但在其他操作系统中则完全不同。以下是它的工作原理：

+   在大多数 BSD 类型的系统上，如 FreeBSD 和 OpenBSD，`sh` 是传统的 Bourne shell。根据 `sh` 的手册页，它仅支持符合 POSIX 标准的功能，以及一些 BSD 扩展。

+   在 Red Hat 类型的系统上，`sh` 是一个符号链接，指向 `bash` 可执行文件。

+   在 Debian/Ubuntu 类型的系统上，`sh`是指向`dash`可执行文件的符号链接。`dash`代表**Debian Almquist Shell**，它是 Bourne shell 的一个更快、更轻量级的实现。

+   在 Alpine Linux 上，`sh`是一个指向`ash`的符号链接，`ash`是一个轻量级的 shell，内置在`busybox`可执行文件中。（在 Alpine Linux 上，默认情况下并未安装`bash`。）

+   在 OpenIndiana 上，OpenIndiana 是 Oracle Solaris 操作系统的一个**自由开源软件**分支，`sh`是指向`ksh93`的符号链接。这个 shell，也叫 Korn shell，在某种程度上与`bash`兼容。（它是由名为 David Korn 的人创建的，和任何蔬菜无关。）不过，OpenIndiana 的默认登录 shell 是`bash`。

+   在 macOS 上，`sh`指向`bash`可执行文件。有趣的是，`zsh`现在是 macOS 的默认用户登录 shell，但`bash`仍然安装并可以使用。

使用`#!/bin/sh`只在你小心地使脚本具有移植性时有效。如果你机器上的`sh`指向的是其他 shell，而不是`bash`，那么对于需要 `bash` 特有高级功能的脚本，它将无法工作。所以，如果你使用`sh`，请务必在不同的系统上测试，以确保一切如预期那样运行。

不过，在进入测试部分之前，让我们先了解一下 bashism 的概念，以及如何避免它们。

# 理解 Bash 特性

**bashism**指的是任何特定于`bash`的功能，而这些功能在其他 shell 中无法使用。让我们看几个例子。

## 使用可移植的测试

对于我们的第一个例子，试着在你的 Fedora 虚拟机上运行这个命令：

```
donnie@fedora:~$ [[ -x /bin/ls ]] && echo "This file is installed.";
This file is installed.
donnie@fedora:~$ 
```

这里，我正在测试`/bin/`目录中是否存在`ls`可执行文件。该文件存在，因此调用了`echo`命令。现在，让我们在 FreeBSD 虚拟机上运行相同的命令：

```
donnie@freebsd14:~ $ [[ -f /bin/ls ]] && echo "This file is installed."
-sh: [[: not found
donnie@freebsd14:~ $ 
```

这次我遇到了一个错误，因为 FreeBSD 上的默认用户登录 shell 是`sh`，而不是`bash`。问题在于 FreeBSD 中的`sh`实现不支持`[[. . .]]`构造。让我们看看是否能解决这个问题：

```
donnie@freebsd14:~ $ [ -f /bin/ls ] && echo "This file is installed."
This file is installed.
donnie@freebsd14:~ $ 
```

非常酷，这个方法有效。那么现在，你可能在想为什么有人会使用非 POSIX 的`[[. . .]]`构造，而不是更具移植性的`[. . .]`。嗯，主要是因为某些类型的测试无法与`[. . .]`一起使用。例如，让我们看一下`test-test-1.sh`脚本：

```
#!/bin/bash
if [[ "$1" == z* ]]; then
    echo "Pattern matched: "$1" starts with 'z'."
else
    echo "Pattern not matched: "$1" doesn't start with 'z'."
fi 
```

当我调用这个脚本时，我会提供一个单词作为`$1`位置参数。如果单词的第一个字母是“z”，那么模式将匹配。

在这个脚本中，`z*`是一个正则表达式。我们使用这个正则表达式来匹配所有以字母“z”开头的单词。每次在测试中使用正则表达式时，必须将其放在双中括号`[[. . .]]`构造内。

（我在*第九章，使用 grep、sed 和正则表达式过滤文本*中解释了正则表达式。）

下面是它的工作原理：

```
donnie@fedora:~$ ./test-test-1.sh zebra
Pattern matched: zebra starts with 'z'.
donnie@fedora:~$ ./test-test-1.sh donnie
Pattern not matched: donnie doesn't start with 'z'.
donnie@fedora:~$ 
```

现在，看一下`test-test-2.sh`脚本，它使用了`[. . .]`：

```
#!/bin/bash
if [ "$1" == z* ]; then
    echo "Pattern matched: "$1" starts with 'z'."
else
    echo "Pattern not matched: "$1" doesn't start with 'z'."
fi 
```

这个脚本与第一个脚本完全相同，唯一的不同是它使用了单括号进行测试，而不是双括号。下面是我运行该脚本时发生的情况：

```
donnie@fedora:~$ ./test-test-2.sh zebra
Pattern not matched: zebra doesn't start with 'z'.
donnie@fedora:~$ 
```

你会看到，这个测试在单括号下不起作用。因为单括号构造不识别正则表达式的使用。但是，如果在某些`sh`实现中双括号不起作用，那么我该如何使这个脚本具有可移植性呢？好吧，这里有一个解决方案：

```
#!/bin/sh
if [ "$(echo $1 | cut -c1)" = z ]; then
    echo "Pattern matched: "$1" starts with 'z'."
else
    echo "Pattern not matched: "$1" doesn't start with 'z'."
fi 
```

我并没有使用`z*`正则表达式来匹配任何以字母“z”开头的单词，而是将单词传递给`cut -c1`命令，以便提取首字母。我还将测试中的`==`改为`=`，因为`==`也被认为是 bash 语法。

`==`实际上在 FreeBSD 的`sh`实现中有效，但可能在其他实现中不起作用。例如，在 Debian/Ubuntu 类型的发行版中，`#!/bin/sh`调用的`dash`就不支持它。

在此过程中，我将 shebang 行更改为`#!/bin/sh`，因为我也想在 FreeBSD 上使用 Bourne shell 进行测试。那么，这能行吗？让我们在 Fedora 上试试，在 Fedora 上，`#!/bin/sh`指向的是`bash`：

```
donnie@fedora:~$ ./test-test-3.sh zebra
Pattern matched: zebra starts with 'z'.
donnie@fedora:~$ ./test-test-3.sh donnie
Pattern not matched: donnie doesn't start with 'z'.
donnie@fedora:~$ 
```

是的，在 Fedora 上运行正常。现在，让我们看看在 FreeBSD 上会发生什么：

```
donnie@freebsd14:~ $ ./test-test-3.sh zealot
Pattern matched: zealot starts with 'z'.
donnie@freebsd14:~ $ ./test-test-3.sh donnie
Pattern not matched: donnie doesn't start with 'z'.
donnie@freebsd14:~ $ 
```

确实，它运行得非常顺利。而且，作为记录，我还在一台使用`dash`的 Debian 机器上进行了测试，它也能正常工作。

现在，让我们来看一下下一个可移植性问题。

## 创建可移植的数组

有时，你需要在 Shell 脚本中创建和操作项目列表。在`bash`中，你可以像在 C、C++或 Java 等语言中一样创建变量数组。例如，让我们来看一下`ip-2.sh`脚本：

```
#!/bin/sh
echo "IP Addresses of intruder attempts"
declare -a ip
ip=( 192.168.3.78 192.168.3.4 192.168.3.9 )
echo "ip[0] is ${ip[0]}, the first item in the list."
echo "ip[2] is ${ip[2]}, the third item in the list."
echo "***********************************"
echo "The most dangerous intruder is ${ip[1]}, which is in ip[1]."
echo "***********************************"
echo "Here is the entire list of IP addresses in the array."
echo ${ip[*]} 
```

这个脚本与我在*第八章，基本 Shell 脚本构建*中展示的`ip.sh`脚本完全相同，唯一不同的是我将 shebang 行改为`#!/bin/sh`。为了提醒你，我使用了`declare -a`命令来创建`ip`数组，并用`ip=`行来填充数组。这应该能在我的 Fedora 机器上正常工作，因为 Fedora 的`sh`指向的是`bash`。让我们看看它是否能正常工作：

```
donnie@fedora:~$ ./ip-2.sh
IP Addresses of intruder attempts
ip[0] is 192.168.3.78, the first item in the list.
ip[2] is 192.168.3.9, the third item in the list.
***********************************
The most dangerous intruder is 192.168.3.4, which is in ip[1].
***********************************
Here is the entire list of IP addresses in the array.
192.168.3.78 192.168.3.4 192.168.3.9
donnie@fedora:~$ 
```

是的，它确实有效。

现在，让我们看看在 FreeBSD 上的表现如何：

```
donnie@freebsd14:~ $ ./ip-2.sh
IP Addresses of intruder attempts
./ip-2.sh: declare: not found
./ip-2.sh: 4: Syntax error: word unexpected (expecting ")")
donnie@freebsd14:~ $ 
```

这次的问题是，FreeBSD 上的 Bourne（`sh`）shell 不能使用变量数组。为了使脚本具备可移植性，我们需要找到一个解决方法。那么，让我们尝试这个`ip-3.sh`脚本：

```
#!/bin/sh
set 192.168.3.78 192.168.3.4 192.168.3.9
echo "$1 is the first item in the list."
echo "$3 is the third item in the list."
echo "**********************************"
echo "The most dangerous intruder is $2, which is the second item of the list."
echo "*********************************"
echo "Here is the entire list of IP addresses in the array:"
echo "$@" 
```

这次，我没有构建实际的数组，而是使用`set`命令创建了一个 IP 地址列表，并通过位置参数访问它。我知道，你可能认为位置参数仅仅用于在调用脚本时从命令行传递参数。但这里展示的其实是另一种使用位置参数的方法。大问题是，这能行吗？让我们看看它在 FreeBSD 机器上的效果：

```
donnie@freebsd14:~ $ ./ip-3.sh
192.168.3.78 is the first item in the list.
192.168.3.9 is the third item in the list.
**********************************
The most dangerous intruder is 192.168.3.4, which is the second item of the list.
*********************************
Here is the entire list of IP addresses in the array:
192.168.3.78 192.168.3.4 192.168.3.9
donnie@freebsd14:~ $ 
```

我们再次成功了。

请注意，在使用`set`和位置参数时，我们并没有创建一个真正的数组。我们只是*模拟*了一个数组。不过，管它的，反正有用，不是吗？

你也可以从文本文件中构建一个模拟数组，使用`cat`命令。首先，让我们创建`iplist.txt`文件，它看起来像这样：

```
donnie@fedora:~$ cat iplist.txt
192.168.0.12
192.168.0.16
192.168.0.222
192.168.0.3
donnie@fedora:~$ 
```

现在，创建`ip-4.sh`脚本，如下所示：

```
#!/bin/sh
set $(cat iplist.txt)
echo "$1 is the first item in the list."
echo "$3 is the third item in the list."
echo "********************************"
echo "The most dangerous intruder is $2, which is the second item of the list."
echo "*******************************"
echo "Here is the entire list of IP addresses in the array:"
echo "$@" 
```

这是我在 FreeBSD 上运行时的结果：

```
donnie@freebsd14:~ $ ./ip-4.sh
192.168.0.12 is the first item in the list.
192.168.0.222 is the third item in the list.
********************************
The most dangerous intruder is 192.168.0.16, which is the second item of the list.
*******************************
Here is the entire list of IP addresses in the array:
192.168.0.12 192.168.0.16 192.168.0.222 192.168.0.3
donnie@freebsd14:~ $ 
```

所以，看起来不错。

接下来，让我们处理一个到目前为止还没有成为问题的事情，但未来可能会成为问题的。那就是，`echo`的使用。

## 理解`echo`的可移植性问题

到目前为止，在我们的脚本中使用`echo`没有遇到任何问题。那只是因为我们没有使用`echo`的高级格式化功能。为了演示这一点，请在你的 Fedora 虚拟机上运行这个命令：

```
donnie@fedora:~$ echo -e "I want to fly \vto the moon."
I want to fly
                    to the moon.
donnie@fedora:~$ 
```

你在`\vto the moon`字符串中看到的`\v`插入了一个垂直制表符，导致输出被分成两行，第二行前面有制表符。

为了使用`\v`标签，我必须在`echo`命令中使用`-e`选项。现在，尝试不使用`-e`命令：

```
donnie@fedora:~$ echo "I want to fly \vto the moon."
I want to fly \vto the moon.
donnie@fedora:~$ 
```

你可以看到，在没有`-e`选项的情况下，`\v`并没有按预期插入垂直制表符。

`echo`的大问题是不同的非`bash`shell 处理其选项开关的方式不一致。例如，让我们在 Debian 机器上打开一个`dash`会话，看看它上面发生了什么：

```
donnie@debian12:~$ dash
$ echo -e "I want to fly \vto the moon."
-e I want to fly
                        to the moon.
$ echo "I want to fly \vto the moon."
I want to fly
                       to the moon.
$ 
```

你可以看到在`dash`中，`echo -e`在输出字符串的开头插入了`-e`。但是，如果我们省略`-e`，输出仍然正确显示。这是因为 POSIX 标准并没有为`echo`定义`-e`选项。相反，它只是允许`echo`默认识别反斜杠格式化选项，如`\v`。

保证脚本输出一致的最好方法是使用`printf`而不是`echo`。以下是在`dash`中显示的情况：

```
donnie@debian12:~$ dash
$ printf "%b\n" "I want to fly \vto the moon."
I want to fly
                    to the moon.
$ 
```

使用`printf`时，你需要将格式化选项放在要输出的字符串前面。在这种情况下，`%b`启用在输出字符串中使用反斜杠格式化选项，而`\n`表示在输出的末尾附加换行符。你会看到，我将这两个选项组合在`"%b\n"`构造中。酷的是，你可以在任何虚拟机上、任何 Shell 中运行这个命令，输出总是会一致的。

最后，让我们来看一下`ip-5.sh`脚本，它使用`printf`而不是`echo`：

```
#!/bin/sh
set 192.168.3.78 192.168.3.4 192.168.3.9
printf '%s\n' "$1 is the first item in the list."
printf '%s\n' "$3 is the third item in the list."
printf '%s\n' "**********************************"
printf '%b\n' "The most dangerous intruder is \v$2,\v which is the second item of the list."
printf '%s\n' "*********************************"
printf '%s\n' "Here is the entire list of IP addresses in the array:"
printf '%s\n' "$@" 
```

除了一个`printf`行之外，我在所有`printf`行中都使用了`"%s\n"`格式化选项，它的意思是打印指定的文本字符串，并在末尾添加换行符。在第四行`printf`中，我使用了`"%b\n"`选项，它表示允许在文本中使用反斜杠格式化选项。你会看到，我将位置参数`$2`用一对`\v`选项括起来，以便插入一对垂直制表符。运行脚本时，它的输出如下所示：

```
donnie@fedora:~$ ./ip-5.sh
192.168.3.78 is the first item in the list.
192.168.3.9 is the third item in the list.
**********************************
The most dangerous intruder is
                                                  192.168.3.4,
                                                                       which is the second item of the list.
*********************************
Here is the entire list of IP addresses in the array:
192.168.3.78
192.168.3.4
192.168.3.9
donnie@fedora:~$ 
```

酷的是，正如我之前所说，`printf`的输出在所有的 shell 中都会保持一致。

好了，现在你已经看到了 bashism 的一些例子，我们来看看如何测试我们的脚本是否符合 POSIX 标准。

# 测试脚本的 POSIX 合规性

在将 shell 脚本投入生产环境之前，测试它们始终非常重要。当你创建需要在多种操作系统和 shell 上运行的脚本时，这一点尤其重要。在这一部分，我们将介绍一些测试方法。

## 在符合 POSIX 标准的 Shell 上创建脚本

当你第一次开始编写脚本时，可能会想使用一个完全符合 POSIX 标准的解释器 shell。但是需要注意的是，一些符合 POSIX 标准的 shell 仍然允许你使用某些 bashism。这是因为 POSIX 定义了操作系统或 shell 必须满足的*最低*标准，并没有禁止添加扩展。例如，FreeBSD 上的 `sh` 允许使用以下两个 bashism：

+   使用 `echo -e` 输出。

+   使用 `==` 进行文本字符串比较。

目前，我还没有在 FreeBSD 上对 `sh` 进行广泛测试，以查看它到底允许多少 bashism。但是，至少允许这两个特性意味着我们不能仅凭它来确定我们的脚本是否能在整个网络上运行。

最符合 POSIX 标准的 shell 是 `dash`，它已预装在任何操作系统中。我已经提到过，如果在 Debian/Ubuntu 类型的系统上运行一个 `#!/bin/sh` 脚本，它会使用 `dash` 作为脚本解释器。好处是，如果你创建一个在 `dash` 上运行的脚本，它很可能也能在其他所有 shell 上正确运行。

嗯，差不多吧。记住我刚才给你演示的 `echo -e`。我告诉你在 `bash` 中，必须使用 `-e` 选项才能包含任何反斜杠格式化选项。我的意思是这个：

```
donnie@fedora:~$ echo -e "I want to fly \vto the moon."
I want to fly
                    to the moon.
donnie@fedora:~$ echo "I want to fly \vto the moon."
I want to fly \vto the moon.
donnie@fedora:~$ 
```

因此，在 `bash` 上，`\v` 格式选项除非使用带有 `-e` 开关的 `echo`，否则无法使用。而在 `dash` 上，正好相反，正如你在这里看到的：

```
donnie@debian12:~$ dash
$ echo -e "I want to fly \vto the moon."
-e I want to fly
                 to the moon.
$ echo "I want to fly \vto the moon."
I want to fly
              to the moon.
$ 
```

所以，如果你在脚本中使用 `echo` 命令，它们可能在 `dash` 上正常工作，但在 `bash` 上却不行。正如我之前提到的，最好的方法是使用 `printf` 替代 `echo`。除此之外，如果你确实创建了将在 `dash` 上运行的脚本，你还需要在 `bash` 上对它们进行测试。我不清楚有多少 POSIX 特性可以在 `dash` 上运行，但在 `bash` 上却不行。但是，我们确实知道这个 `echo -e` 问题。

**符合政策的普通 Shell**（`posh`）是一种比 `dash` 更严格符合 POSIX 标准的 shell。它的主要问题是，似乎只有 Debian/Ubuntu 类型的发行版在其仓库中包含它。另一方面，您几乎可以在任何 Linux 发行版上轻松安装 `dash`，以及一些类 Unix 发行版，如 FreeBSD。

无论如何，您可以在这里阅读更多关于`posh`的内容：

如何检查您的 shell 脚本的可移植性：[`people.redhat.com/~thuth/blog/general/2021/04/27/portable-shell.html`](https://people.redhat.com/~thuth/blog/general/2021/04/27/portable-shell.html)

说到测试，让我们看看如何进行测试。

## 使用 checkbashisms

这个很酷的`checkbashisms`工具会检查你的脚本，看看其中是否有可能在非`bash` shell 上无法运行的内容。但首先，你需要安装它。下面是安装的方法：

在 Fedora 上：

```
donnie@fedora:~$ sudo dnf install devscripts-checkbashisms 
```

在 Debian/Ubuntu 上：

```
donnie@debian12:~$ sudo apt install devscripts 
```

在 FreeBSD 上：

```
donnie@freebsd14:~ $ sudo pkg install checkbashisms 
```

在 macOS 上：

安装 Homebrew 系统，然后使用以下命令安装`checkbashisms`包：

```
macmini@MacMinis-Mac-mini ~ % brew install checkbashisms 
```

如果你想知道如何在 Mac 上安装 Homebrew 系统，请访问：[`brew.sh`](https://brew.sh)

`checkbashisms`的基本用法很简单。如果你想测试的脚本的 shebang 行是`#!/bin/sh`，那么只需输入：

```
donnie@fedora:~$ checkbashisms scriptname.sh 
```

默认情况下，`checkbashims`只检查包含`#!/bin/sh` shebang 行的脚本。如果你的脚本使用了其他作为 shebang 行的内容，例如`#!/bin/bash`，那么可以使用`-f`选项强制它检查该脚本，像这样：

```
donnie@fedora:~$ checkbashisms -f scriptbashname.sh 
```

对于第一个示例，让我们再看一下`ip-2.sh`脚本：

```
#!/bin/sh
echo "IP Addresses of intruder attempts"
declare -a ip
ip=( 192.168.3.78 192.168.3.4 192.168.3.9 )
echo "ip[0] is ${ip[0]}, the first item in the list."
echo "ip[2] is ${ip[2]}, the third item in the list."
echo "***********************************"
echo "The most dangerous intruder is ${ip[1]}, which is in ip[1]."
echo "***********************************"
echo "Here is the entire list of IP addresses in the array."
echo ${ip[*]} 
```

现在，让我们看看`checkbashisms`对此有什么看法：

```
donnie@fedora:~$ checkbashisms ip-2.sh
possible bashism in ip-2.sh line 3 (declare):
declare -a ip
possible bashism in ip-2.sh line 5 (bash arrays, ${name[0|*|@]}):
echo "ip[0] is ${ip[0]}, the first item in the list."
possible bashism in ip-2.sh line 6 (bash arrays, ${name[0|*|@]}):
echo "ip[2] is ${ip[2]}, the third item in the list."
possible bashism in ip-2.sh line 8 (bash arrays, ${name[0|*|@]}):
echo "The most dangerous intruder is ${ip[1]}, which is in ip[1]."
possible bashism in ip-2.sh line 11 (bash arrays, ${name[0|*|@]}):
echo ${ip[*]}
donnie@fedora:~$ 
```

嗯，这倒不奇怪，因为我们已经确定一些非`bash`的 shell 无法处理数组。而且，我已经展示过如何处理这个问题。

好的，这是你在`math6.sh`脚本中还没有见过的一个 bashism：

```
#!/bin/sh
start=0
limit=10
while [ "$start" -le "$limit" ]; do
  echo "$start... "
  start=$["$start"+1]
done 
```

这是一个 while 循环，输出数字 0 到 10，后面跟着一个字符串“三个点”，每个都在单独的一行。它的输出如下：

```
donnie@fedora:~$ ./math6.sh
0...
1...
2...
3...
4...
5...
6...
7...
8...
9...
10...
donnie@fedora:~$ 
```

在 Fedora 上，使用`bash`看起来一切正常。但是，它能在使用`dash`的 Debian 上工作吗？我们来看一下：

```
donnie@debian12:~$ ./math6.sh
0...
./math6.sh: 4: [: Illegal number: $[0+1]
donnie@debian12:~$ 
```

有什么问题吗？也许`checkbashisms`能告诉我们：

```
donnie@debian12:~$ checkbashisms math6.sh
possible bashism in math6.sh line 6 ('$[' should be '$(('):
  start=$["$start"+1]
donnie@debian12:~$ 
```

问题出在第 6 行。因为只有`bash`可以使用`[ . . . ]`结构进行整数运算。而其他 shell 则必须使用`(( . . . ))`结构。我们来在`math7.sh`脚本中修复这个问题：

```
#!/bin/sh
start=0
limit=10
while [ "$start" -le "$limit" ]; do
  echo "$start... "
  start=$(("$start"+1))
done 
```

那样应该会好很多，对吧？好吧，我们来看看：

```
donnie@debian12:~$ ./math7.sh
0...
./math7.sh: 6: arithmetic expression: expecting primary: ""0"+1"
donnie@debian12:~$ 
```

不，它仍然有问题。所以，让我们再给它一次`checkbashisms`扫描：

```
donnie@debian12:~$ checkbashisms math7.sh
donnie@debian12:~$ 
```

哇，怎么回事？`checkbashisms`扫描说我的代码是好的，但脚本在`dash`上运行时仍然明显有问题。怎么回事呢？经过一番实验，我发现把数学表达式中的变量名用一对引号括起来也是一个 bashism。但这是`checkbashisms`没捕捉到的。我们来在`math8.sh`脚本中修复这个问题：

```
#!/bin/sh
start=0
limit=10
while [ "$start" -le "$limit" ]; do
  echo "$start... "
  start=$(($start+1))
done 
```

好的，这次应该终于能行吗？请听鼓声！

```
donnie@debian12:~$ ./math8.sh
0...
1...
2...
3...
4...
5...
6...
7...
8...
9...
10...
donnie@debian12:~$ 
```

确实，现在它工作得非常顺利。

所以，你已经看到`checkbashisms`是一个很棒的工具，能帮助你发现问题。但正如人类发明的大多数东西一样，它并不完美。它能够标记许多有问题的、非 POSIX 的代码，但它也允许一些特定于`bash`的东西漏过。（如果能有一份`checkbashisms`漏掉的内容清单就好了，但目前没有。）

好的，让我们继续看下一个代码检查工具。

## 使用 shellcheck

`shellcheck`工具是另一个很棒的代码检查工具，也能检查 bashisms。它在大多数 Linux 和 BSD 发行版上都可以使用。下面是如何安装它：

在 Fedora 上：

```
donnie@fedora:~$ sudo dnf install ShellCheck 
```

在 Debian/Ubuntu 上：

```
donnie@debian12:~$ sudo apt install shellcheck 
```

在 FreeBSD 上：

```
donnie@freebsd14:~ $ sudo pkg install hs-ShellCheck 
```

在安装了 Homebrew 的 macOS 上：

```
macmini@MacMinis-Mac-mini ~ % brew install shellcheck 
```

为了演示这个问题，让我们回到 Debian 机器，扫描与 `checkbashisms` 一起扫描的相同脚本。我们从 `ip-2.sh` 脚本开始。以下是相关的输出部分：

```
donnie@debian12:~$ shellcheck ip-2.sh
In ip-2.sh line 4:
ip=( 192.168.3.78 192.168.3.4 192.168.3.9 )
   ^-- SC3030 (warning): In POSIX sh, arrays are undefined.
In ip-2.sh line 5:
echo "ip[0] is ${ip[0]}, the first item in the list."
               ^------^ SC3054 (warning): In POSIX sh, array references are undefined.
. . .
In ip-2.sh line 11:
echo ${ip[*]}
     ^------^ SC2048 (warning): Use "${array[@]}" (with quotes) to prevent whitespace problems.
     ^------^ SC3054 (warning): In POSIX sh, array references are undefined.
     ^------^ SC2086 (info): Double quote to prevent globbing and word splitting.
. . .
donnie@debian12:~$ 
```

你会看到，`shellcheck` 确实警告我们在 POSIX 兼容的脚本中不能使用数组。它还会提醒我们一些良好编程实践方面的问题，这些问题不一定是 POSIX 的问题。在这个案例中，它提醒我们应该将变量扩展构造 `${ip[0]}`（在这种情况下）用一对双引号括起来，以防止空白问题。当然，这在这里并不重要，因为 `${ip[0]}` 构造本身是我们无法使用的数组定义的一部分。

现在，让我们尝试一下 `math6.sh` 脚本：

```
donnie@debian12:~$ shellcheck math6.sh
In math6.sh line 6:
  start=$["$start"+1]
        ^-----------^ SC3007 (warning): In POSIX sh, $[..] in place of $((..)) is undefined.
        ^-----------^ SC2007 (style): Use $((..)) instead of deprecated $[..]
For more information:
  https://www.shellcheck.net/wiki/SC3007 -- In POSIX sh, $[..] in place of $(...
  https://www.shellcheck.net/wiki/SC2007 -- Use $((..)) instead of 
deprecated...
donnie@debian12:~$ 
```

正如预期的那样，`shellcheck` 检测到了一种非 POSIX 的数学运算方式。所以，这很好。

扫描 `math7.sh` 会得到以下结果：

```
donnie@debian12:~$ shellcheck math7.sh
donnie@debian12:~$ 
```

在这里，我们看到与 `checkbashisms` 一样的问题。也就是说，`math7.sh` 中数学构造内的变量名被一对双引号包围，就像你在 `start=$(("$start"+1))` 这一行中看到的那样。我们已经确认这种方式在 `bash` 上有效，但在 `dash` 上无效，然而 `shellcheck` 并没有将此标记为问题。

最后，让我们尝试一下 `math8.sh`。正如你所记得的，这就是我们最终在 `dash` 上能正常工作的脚本。它的样子是这样的：

```
donnie@debian12:~$ shellcheck math8.sh
In math8.sh line 6:
  start=$(($start+1))
           ^----^ SC2004 (style): $/${} is unnecessary on arithmetic variables.
For more information:
  https://www.shellcheck.net/wiki/SC2004 -- $/${} is unnecessary on arithmeti...
donnie@debian12:~$ 
```

在这种情况下，脚本在我们的 Debian/`dash` 机器上运行得非常顺利。在这里，`shellcheck` 提出了一个风格问题，这个问题不会影响脚本是否真正运行。如果你还记得 *第十一章，执行数学运算*，我曾经向你展示过，当回调一个在数学构造中的变量时，变量名前不需要加 `$`。我的意思是，如果你加了也没关系，但实际上并不需要。

### 使用 `-s` 选项指定 Shell

使用 `shellcheck` 的另一个好处是，你可以使用 `-s` 选项指定你想要用来测试脚本的 Shell。即使你的脚本中的 shebang 行是 `#!/bin/sh`，而且你机器上的 `sh` 指向的是非 `bash` 的 Shell，你仍然可以像这样使用 `bash` 进行测试：

```
donnie@debian12:~$ shellcheck -s bash ip-2.sh
In ip-2.sh line 11:
echo ${ip[*]}
     ^------^ SC2048 (warning): Use "${array[@]}" (with quotes) to prevent whitespace problems.
     ^------^ SC2086 (info): Double quote to prevent globbing and word splitting.
Did you mean:
echo "${ip[*]}"
For more information:
  https://www.shellcheck.net/wiki/SC2048 -- Use "${array[@]}" (with quotes) t...
  https://www.shellcheck.net/wiki/SC2086 -- Double quote to prevent globbing ...
donnie@debian12:~$ 
```

通过在 `bash` 上测试，我们不再收到关于不能使用数组的警告。这次我们收到的只是一个提醒，告诉我们应该将变量名用双引号括起来。

使用 `-s` 选项，你可以指定 `sh`、`bash`、`dash` 或 `ksh`。 （奇怪的是，它在 `zsh` 上无法使用。）

### 实践实验室 – 使用 -s 扫描函数库

你还可以使用 `-s` 选项扫描没有 shebang 行的函数库文件。要查看如何做，让我们扫描一下在 *第十四章，使用 awk-第一部分* 中最后遇到的 `sysinfo.lib` 文件。

本节中的脚本和库文件过大，无法在此处完全显示。因此，务必从 GitHub 仓库下载它们。

1.  首先，将`sysinfo.lib`文件复制到`sysinfo_posix.lib`，如下所示：

    ```
    donnie@debian12:~$ cp sysinfo.lib sysinfo_posix.lib
    donnie@debian12:~$ 
    ```

1.  然后，扫描它以查找`bash`，因为它最初只打算在`bash`上使用。以下是相关的输出：

    ```
    donnie@debian12:~$ shellcheck -s bash sysinfo_posix.lib
    In sysinfo_posix.lib line 19:
                if [ $(uname) = SunOS ]; then
                     ^------^ SC2046 (warning): Quote this to prevent word splitting.
    . . .
    In sysinfo_posix.lib line 48:
                    cd /Users
                    ^-------^ SC2164 (warning): Use 'cd ... || exit' or 'cd ... || return' in case cd fails.
    Did you mean:
                    cd /Users || exit
    . . .
    donnie@debian12:~$ 
    ```

1.  对于`bash`，`shellcheck`并没有发现任何阻碍的问题。但它确实建议了一些可以避免问题的改进。首先，它建议我用双引号将`uname`命令的替换包围起来，以防`uname`返回的文本字符串中包含空格或特殊字符，这些字符可能会被 Shell 误解。实际上，这在本脚本中永远不会成为问题，因为我们知道`uname`返回的永远是一个纯文本、仅包含字母的字符串。但我们还是做了修改。在这种情况下，`if [ $(uname) = SunOS ]; then`这一行将变成：

    ```
    if [ "$(uname)" = SunOS ]; then 
    ```

我们会对每个`uname`命令替换的出现位置做同样的处理。

1.  我们看到的第二个建议是，在脚本无法`cd`进入`/Users/`目录时插入一个优雅的退出机制。同样，我们知道在本脚本中这不会成为问题，因为我们知道指定的目录始终会存在。但我们还是做了修改。我们将那条`cd /Users`命令改成：

    ```
    cd /Users || return 
    ```

我们会对`open_files_users()`函数中的所有`cd`命令做同样的处理。这样，如果`cd`命令失败，脚本将继续运行，而不会中断这个函数。

1.  现在，假设我们希望使这个脚本具有可移植性，这样我们就不必在所有系统上都安装`bash`。如我之前提到的，`dash`是大多数 Linux 和 BSD 类型发行版中最符合 POSIX 标准的 Shell。所以，我们再次使用`-s dash`来扫描文件：

    ```
    donnie@debian12:~$ shellcheck -s dash sysinfo_posix.lib
    . . .
    . . .
    In sysinfo_posix.lib line 55:
               if [[ $user != "lost+found" ]] && [[ $user != "Shared" ]]; then
                  ^-------------------------^ SC3010 (error): In dash, [[ ]] is not supported.
                                                 ^---------------------^ SC3010 (error): In dash, [[ ]] is not supported.
    . . .
    . . .
    In sysinfo.lib line 71:
                       echo "${os:12}"
                             ^------^ SC3057 (error): In dash, string indexing is not supported.
    . . .
    . . .
    donnie@debian12:~$ 
    ```

1.  我们在之前的脚本中已经看到第一个问题。只是`[[. . .]]`结构在一些非`bash`的 Shell 中不被支持，比如`dash`。幸运的是，这里没有任何原因要求我们必须使用双中括号，所以我们可以直接将它们替换为单中括号。所以，我们将修改出问题的那一行：

    ```
    if [ $user != "lost+found" ] && [ $user != "Shared" ]; then 
    ```

1.  第二个问题是我们之前还没有遇到的。`"${os:12}"`类型的变量扩展是另一个`bash`特性，在 Bourne shell 或`dash`中无法使用。为了提醒你记忆，`os`变量是在文件末尾的`system_info()`函数中定义的。从命令行定义时，变量是这样的：

    ```
    donnie@debian12:~$ os=$(grep "PRETTY_NAME" /etc/os-release)
    donnie@debian12:~$ echo $os
    PRETTY_NAME="Debian GNU/Linux 12 (bookworm)"
    donnie@debian12:~$ 
    ```

但是，我们不希望`PRETTY_NAME=`部分出现在报告中。我们只想要操作系统的名称。这个`PRETTY_NAME=`字符串包含 12 个字符。在`bash`中，我们可以通过如下方式去掉这 12 个字符：

```
donnie@debian12:~$ echo "${os:12}"
"Debian GNU/Linux 12 (bookworm)"
donnie@debian12:~$ 
```

它在那里看起来不错，但看看我在`dash`会话中尝试时会发生什么：

```
donnie@debian12:~$ dash
$ os=$(grep "PRETTY_NAME" /etc/os-release)
$ echo "${os:12}"
dash: 2: Bad substitution
$ 
```

哎呀，这根本不起作用。幸运的是，这很容易修复。我们只需使用一种符合 POSIX 标准的变量扩展形式，看起来是这样的：

```
$ echo "${os#PRETTY_NAME=}"
"Debian GNU/Linux 12 (bookworm)"
$ 
```

我们不再指定要从文本字符串开头去除的字符数，而是直接指定要去除的文本部分。而且，我将 `:` 替换为了 `#`。简单吧？

这是我至今尚未弄明白的生活之谜之一。大多数 bashism 是作为一种更简单的方式来完成某些事情，或是为了让 `bash` 执行其他 shell 完全无法做到的任务。在这种情况下，`bash` 执行变量扩展的方式并不比 POSIX 的方式更简便。那么，为什么 `bash` 的开发者还要为我们提供这种新的非 POSIX 方式呢？你我猜测一样。

无论如何，你可以在这里阅读更多关于 POSIX 兼容的变量扩展的内容：

POSIX shell 备忘单：[`steinbaugh.com/posts/posix.html`](https://steinbaugh.com/posts/posix.html)

1.  接下来，我们需要扫描使用此函数库的脚本。首先，将 `system_info.sh` 复制为 `system_info_posix.sh`：

    ```
    donnie@debian12:~$ cp system_info.sh system_info_posix.sh
    donnie@debian12:~$ 
    ```

1.  现在，扫描 `system_info_posix.sh` 脚本以检查其是否与 `dash` 兼容：

    ```
    donnie@debian12:~$ shellcheck -s dash system_info.sh
    . . .
    . . .
    In system_info_posix.sh line 5:
    title="System Information for $HOSTNAME"
                                  ^-------^ SC3028 (error): In dash, HOSTNAME is not supported.
    In system_info_posix.sh line 31:
    if [[ -f /usr/local/bin/pandoc ]] || [[ -f /usr/bin/pandoc ]]; then
       ^----------------------------^ SC3010 (error): In dash, [[ ]] is not supported.
                                         ^----------------------^ SC3010 (error): In dash, [[ ]] is not supported.
    . . .
    . . .
    donnie@debian12:~$ 
    ```

我们首先看到的是，“在 dash 中，HOSTNAME 不受支持。”这听起来有点奇怪，因为我在这里做的只是调用一个环境变量的值。但它说这不起作用，因此我将其改为使用 `hostname` 命令的命令替换，像这样：

```
title="System Information for $(hostname)" 
```

下一个问题是我们之前看到过的那个老问题——双中括号。又是一个不需要双中括号的情况，所以我将其更改为单中括号，如下所示：

```
if [ -f /usr/local/bin/pandoc ] || [ -f /usr/bin/pandoc ]; then 
```

1.  接下来，我们将 `sysinfo_posix.lib` 文件复制到 `/usr/local/lib/` 目录中的正确位置：

    ```
    donnie@debian12:~$ sudo cp sysinfo_posix.lib /usr/local/lib/
    donnie@debian12:~$ 
    ```

1.  现在，编辑 `system_info_posix.sh` 脚本，使其加载新的函数库，并使用 `sh` 作为解释器。使脚本的顶部部分看起来像这样：

    ```
    #!/bin/sh
    # sysinfo_page - A script to produce an HTML file
    . /usr/local/lib/sysinfo_posix.lib 
    ```

1.  最后，将 `system_info_posix.sh` 脚本和 `sysinfo_posix.lib` 文件复制到其他操作系统的机器上。你应该会发现，这个脚本和函数库在其他 Linux 发行版、FreeBSD、OpenIndiana 和 macOS 上也能很好地工作。

实验结束

到此，我们已经介绍了 `checkbashisms` 和 `shellcheck`。我们还要介绍另一个有用的脚本检查工具吗？当然要，我们现在就来介绍。

## 使用 shall

`checkbashisms` 和 `shellcheck` 工具作为 **静态代码检查工具**。这意味着，这两个工具并不是实际运行脚本来验证它们是否有效，而是直接扫描代码以检测潜在问题。不过，有时使用 **动态代码检查器** 会更有帮助，动态检查器会实际运行代码以观察其结果。这时，`shall` 就派上用场了。

`shall` 是一个 `bash` 脚本，你可以从作者的 GitHub 仓库下载。最简单的安装方法是克隆该仓库，如下所示：

```
donnie@debian12:~$ git clone https://github.com/mklement0/shall.git 
```

接下来，进入 `shall/bin/` 目录，将 `shall` 脚本复制到 `/usr/local/bin/` 目录中，如下所示：

```
donnie@debian12:~$ cd shall/bin
donnie@debian12:~/shall/bin$ ls -l
total 28
-rwxr-xr-x. 1 donnie donnie 27212 May 27 18:20 shall
donnie@debian12:~/shall/bin$ sudo cp shall /usr/local/bin
donnie@debian12:~/shall/bin$ 
```

现在，这是最酷的部分。通过一个命令，`shall`可以测试你的脚本是否适用于`sh`、`bash`、`dash`、`zsh`和`ksh`。（不管你在脚本的 shebang 行中指定的是哪个解释器 Shell。）大多数 Linux 和 BSD 发行版的包仓库中都有这些不同的 Shell，所以你可以安装那些还没有安装的 Shell。

当然，`dash`和`bash`已经安装在我们的 Debian 机器上了，所以我们来安装`zsh`和`ksh`：

```
donnie@debian12:~$ sudo apt install zsh ksh 
```

让我们从测试我们的`ip-2.sh`脚本开始。输出太长，无法完全显示，所以我只会展示一些相关的部分。这里是顶部内容：

```
donnie@debian12:~$ shall ip-2.sh
✗ sh@ (-> dash)                           [0.00s]
  IP Addresses of intruder attempts
  ip-2.sh: 4: Syntax error: "(" unexpected 
```

第一个检查是针对`sh`的，在 Debian 中实际上意味着`dash`。你可以看到脚本无法运行，因为`dash`无法处理数组。接下来的检查是针对`bash`的，看起来是这样的：

```
✓ bash                                    [0.00s]
  IP Addresses of intruder attempts
  ip[0] is 192.168.3.78, the first item in the list.
  ip[2] is 192.168.3.9, the third item in the list.
  ***********************************
  The most dangerous intruder is 192.168.3.4, which is in ip[1].
  ***********************************
  Here is the entire list of IP addresses in the array.
  192.168.3.78 192.168.3.4 192.168.3.9 
```

后面的`zsh`和`ksh`检查看起来是一样的。在输出的最底部，你会看到这样的内容：

```
FAILED - 1 shell (sh) reports failure, 3 (bash, zsh, ksh) report success.
donnie@debian12:~$ 
```

这意味着我们的`ip-2.sh`脚本在`bash`、`zsh`和`ksh`上运行正常，但在`dash`上不能运行。

但是，尽管`shall`非常酷，它也并不完美。为了说明这一点，创建一个`fly.sh`脚本，像这样：

```
#!/bin/sh
#
echo -e "I want to fly to the \v moon."
printf "%b\n" "I want to fly to the \v moon." 
```

这只是一个简单的小脚本，演示了使用`echo -e`时在脚本中遇到的问题，正如我在几页前向你展示的那样。当我使用`shall`测试`fly.sh`时，会发生这样的情况：

```
donnie@debian12:~$ shall fly.sh
✓ sh@ (-> dash)                           [0.00s]
  -e I want to fly to the
                           	moon.
  I want to fly to the
                        	moon.
✓ bash                                    [0.00s]
  I want to fly to the
                        	moon.
  I want to fly to the
                        	moon.
. . .
OK - All 4 shells (sh, bash, zsh, ksh) report success.
donnie@debian12:~$ 
```

所以，`shall`表示脚本在所有四个 Shell 上都运行正确。但是，在输出的顶部，你会看到`-e`在`echo`输出中出现了`dash`，这就是我之前向你展示的问题。关键是，当你使用`shall`时，不要仅仅依赖输出底部状态行所显示的内容。查看所有 Shell 的输出，确保输出是你真正想看到的。

`shall`的唯一其他问题其实并不在`shall`本身，而是在`sh`。记住，`sh`在不同的 Linux、BSD 和 Unix 发行版上指向不同的 Shell。你可以在这些发行版上安装`shall`，只要你能够在它们上安装`bash`。然后，只需将`shall`脚本复制到你想要测试`sh`脚本的机器上。

`shall`脚本中嵌入了一个手册页面。要查看它，只需执行：

`shall --man`

好的，我想这一章就到此为止。让我们总结一下并继续下一章。

# 总结

如果你需要编写可以在各种 Linux、Unix 或类 Unix 操作系统上运行的脚本，脚本的可移植性非常重要。我首先向你展示了如何在各种 BSD 类发行版上安装 bash，并确保你的脚本能够在这些系统上找到`bash`。之后，我解释了 POSIX 标准及其必要性。然后，我展示了一些 bash 特性和一些可以测试脚本的实用工具。

在下一章，我们将讨论 Shell 脚本的安全性。到时候见。

# 问题

1.  对于 Shell 脚本编写者，遵循 POSIX 标准的最重要原因是什么？

    1.  为了确保脚本仅在 Linux 操作系统上运行。

    1.  确保脚本能够在各种 Linux、Unix 和类似 Unix 的操作系统上运行。

    1.  确保所有操作系统都运行`bash`。

    1.  确保脚本能够使用`bash`的高级功能。

1.  以下哪项是动态代码检查器？

    1.  `checkbashisms`

    1.  `shellcheck`

    1.  `shall`

    1.  `will`

1.  以下关于`sh`的说法哪项正确？

    1.  在每个操作系统上，`sh` 总是 Bourne shell。

    1.  在每个操作系统上，`sh` 都是一个指向`bash`的符号链接。

    1.  `sh` 在不同操作系统上代表不同的 shell。

1.  你想解决一个数学问题，并将其值赋给一个变量。你会使用以下哪种构造来确保最佳的可移植性？

    1.  `var=$(3*4)`

    1.  `var=$[3*4]`

    1.  `var=$[[3*4]]`

    1.  `var=$((3*4))`

# 进一步阅读

+   UNIX 与 Linux：它们有什么不同？: [`www.maketecheasier.com/unix-vs-linux-how-are-they-different/`](https://www.maketecheasier.com/unix-vs-linux-how-are-they-different/)

+   “#!/usr/bin/env bash”和“#!/usr/bin/bash”之间有什么区别？: [`stackoverflow.com/questions/16365130/what-is-the-difference-between-usr-bin-env-bash-and-usr-bin-bash`](https://stackoverflow.com/questions/16365130/what-is-the-difference-between-usr-bin-env-bash-and-usr-bin-bash)

+   编写不仅仅是 Bash 的 Bash 脚本：检查 Bashisms 和使用 Dash: [`www.bowmanjd.com/bash-not-bash-posix/`](https://www.bowmanjd.com/bash-not-bash-posix/)

+   简短的 POSIX 推广: [`www.usenix.org/system/files/login/articles/login_spring16_09_tomei.pdf`](https://www.usenix.org/system/files/login/articles/login_spring16_09_tomei.pdf)

+   POSIX 指南: [`www.baeldung.com/linux/posix`](https://www.baeldung.com/linux/posix)

+   如何测试 shell 脚本的 POSIX 合规性？: [`unix.stackexchange.com/questions/48786/how-can-i-test-for-posix-compliance-of-shell-scripts`](https://unix.stackexchange.com/questions/48786/how-can-i-test-for-posix-compliance-of-shell-scripts)

+   使 Unix Shell 脚本符合 POSIX 标准: [`stackoverflow.com/questions/40916071/making-unix-shell-scripts-posix-compliant`](https://stackoverflow.com/questions/40916071/making-unix-shell-scripts-posix-compliant)

+   Rich 的 sh（POSIX shell）技巧: [`www.etalabs.net/sh_tricks.html`](http://www.etalabs.net/sh_tricks.html)

+   Dash–ArchWiki: [`wiki.archlinux.org/title/Dash`](https://wiki.archlinux.org/title/Dash)

+   POSIX shell 备忘单: [`steinbaugh.com/posts/posix.html`](https://steinbaugh.com/posts/posix.html)

+   是否有符合 POSIX.2 最小要求的 shell？: [`stackoverflow.com/questions/11376975/is-there-a-minimally-posix-2-compliant-shell`](https://stackoverflow.com/questions/11376975/is-there-a-minimally-posix-2-compliant-shell)

# 答案

1.  b

1.  c

1.  c

1.  d

# 加入我们的 Discord 社区！

和其他用户、Linux 专家以及作者本人一起阅读这本书。

提出问题，为其他读者提供解决方案，通过“问我任何问题”环节与作者聊天，等等。扫描二维码或访问链接加入社区。

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code10596186092701843.png)
