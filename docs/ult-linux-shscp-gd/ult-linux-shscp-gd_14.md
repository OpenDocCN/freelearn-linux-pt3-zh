

# 第十四章：使用 awk – 第一部分

在本章中，我将向您展示一些关于 `awk` 的知识。它是一个具有悠久历史的编程环境，追溯到 1970 年代，当时由 Alfred Aho、Peter Weinberger 和 Brian Kernighan 发明，用于早期的 Unix 操作系统。

您可以通过多种方式使用 `awk`。它是一个完整的编程语言，因此您可以用它编写非常复杂的独立程序。您还可以创建简单的 `awk` 命令，既可以从命令行运行，也可以在常规的 shell 脚本中运行。`awk` 内容丰富，已有专门的书籍介绍它。本章的目标是向您展示如何在常规的 shell 脚本中使用 `awk`。

本章中的主题包括：

+   介绍 `awk`

+   理解模式与操作

+   从文本文件获取输入

+   从命令获取输入

如果你准备好使用 `awk`，那我们就开始吧。

# 介绍 awk

`awk` 是一个模式扫描和文本处理工具，您可以使用它来自动化生成报告和数据库的过程。凭借其内置的数学功能，您还可以使用它对列状数字数据的文本文件执行电子表格操作。术语 `awk` 来自其创造者的名字：Aho、Weinberger 和 Kernighan。原始版本现在被称为“旧版 `awk`”。更新的实现版本，如 `nawk` 和 `gawk`，具有更多功能，且更易于使用。

您使用的 `awk` 版本取决于您运行的操作系统。大多数 Linux 操作系统运行的是 `gawk`，它是 `awk` 的 GNU 实现。通常会有一个指向 `gawk` 可执行文件的 `awk` 符号链接，就像我在 Fedora 工作站上看到的那样：

```
[donnie@fedora ~]$ awk --version
GNU Awk 5.1.1, API: 3.1 (GNU MPFR 4.1.1-p1, GNU MP 6.2.1)
Copyright (C) 1989, 1991-2021 Free Software Foundation.
This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 3 of the License, or
(at your option) any later version.
. . .
. . .
[donnie@fedora ~]$ ls -l /bin/awk
lrwxrwxrwx. 1 root root 4 Jan 18  2023 /bin/awk -> gawk
[donnie@fedora ~]$ 
```

一个显著的例外是 Alpine Linux，它默认使用内置在 `busybox` 可执行文件中的轻量级 `awk` 实现。不过，`gawk` 和另一种 `awk` 实现 `mawk` 都可以从 Alpine 仓库安装。（我找不到 `mawk` 为什么被称为 `mawk` 的确切答案。不过，它的作者是 Michael Brennan，所以我猜它代表着“Michael 的 `awk`”。）

Unix 和类 Unix 操作系统，如 macOS、OpenIndiana 和各种 BSD 发行版，使用的是 `nawk`，即“新 `awk`”的简称。您有时会看到它被称为“唯一真正的 `awk`”，部分原因是 `awk` 原作者之一 Brian Kernighan 是它的维护者之一。不过，`gawk` 可在 FreeBSD 和 OpenIndiana 上安装，而 `mawk` 则可在 FreeBSD 上安装。

那么，这些不同的 `awk` 实现之间有什么区别呢？这里有一个简要的概述：

+   `busybox`：`busybox`中内置的`awk`实现非常轻量，非常适合低资源的嵌入式系统。这也是它成为 Alpine Linux（一个也在嵌入式系统中非常流行的系统）默认选择的原因。然而要注意，它可能并不总是具备你在复杂`awk`命令或脚本中所需的功能。

+   `nawk`：正如我之前提到的，`nawk`是大多数 Unix 和类 Unix 系统（如 FreeBSD、OpenIndiana 和 macOS）上的默认选择。但你在这些系统上找到的可执行文件通常是`awk`，而不是`nawk`。

+   `mawk`：这是`awk`的一个更快的版本，由 Mike Brennan 创建。

+   `gawk`：这个实现具有其他实现所没有的功能。一个重要的增强功能是**国际化和本地化**能力，这有助于为不同语言和地区创建软件。它还包括 TCP/IP 网络功能和改进的正则表达式处理功能。

除非我另有说明，否则我将向你展示在`nawk`和`gawk`上都能使用的编码技巧。现在，让我们开始吧。

# 理解模式和动作

`awk`**模式**只是文本或正则表达式，用于执行**操作**的依据。数据源可以是普通文本文件，也可以是另一个程序的输出。首先，让我们将`/etc/passwd`文件的全部内容输出到屏幕上，如下所示：

```
[donnie@fedora ~]$ awk '{print $0}' /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
. . .
. . .
donnie:x:1000:1000:Donald A. Tevault:/home/donnie:/bin/bash
systemd-coredump:x:986:986:systemd Core Dumper:/:/usr/sbin/nologin
systemd-timesync:x:985:985:systemd Time Synchronization:/:/usr/sbin/nologin
clamupdate:x:984:983:Clamav database update user:/var/lib/clamav:/sbin/nologin
setroubleshoot:x:983:979:SELinux troubleshoot server:/var/lib/setroubleshoot:/usr/sbin/nologin
[donnie@fedora ~]$ 
```

在这个命令中，`{print $0}`部分是操作，它必须被一对单引号包围。`$`用于指定要打印的字段。在这个例子中，`$0`会打印出所有字段。指定模式将导致只有匹配该模式的行被打印。由于这次没有指定模式，所以我让每一行都被打印。现在，假设我只想显示包含特定用户名的行。我只需添加模式，如下所示：

```
[donnie@fedora ~]$ awk '/donnie/ {print $0}' /etc/passwd
donnie:x:1000:1000:Donald A. Tevault:/home/donnie:/bin/bash
[donnie@fedora ~]$ 
```

模式需要被一对斜杠包围，而这对斜杠需要放在单引号中，这些单引号也同时包围着操作部分。

`awk`操作的是被称为`records`的信息组。默认情况下，文本文件中的每一行就是一个记录。现在，我们暂时只处理这个。不过，你也可以有多行记录的文件，这需要使用特殊技巧来处理。我将在下一章中展示如何操作。

由于`{print $0}`是默认的操作，我可以直接省略这一部分，得到相同的结果，如下所示：

```
[donnie@fedora ~]$ awk '/donnie/' /etc/passwd
donnie:x:1000:1000:Donald A. Tevault:/home/donnie:/bin/bash
[donnie@fedora ~]$ 
```

接下来，我们只打印出这一行的第一个字段。这样做：

```
[donnie@fedora ~]$ awk '/donnie/ {print $1}' /etc/passwd
donnie:x:1000:1000:Donald
[donnie@fedora ~]$ 
```

`$1`表示我要打印第 1 个字段。但等等，这样还是不对，因为命令实际上打印了第 1 到第 5 个字段。那是因为`awk`的默认字段分隔符是空格。字段 5 包含我的全名，而“Donald”和“A.”之间有一个空格。所以，对`awk`来说，这是第二个字段的开始。为了修正这个问题，我会使用`-F:`选项，让`awk`将冒号识别为字段分隔符，像这样：

```
[donnie@fedora ~]$ awk -F: '/donnie/ {print $1}' /etc/passwd
donnie
[donnie@fedora ~]$ 
```

最后，我得到了我想要的输出。

如果你需要显示多个字段，请在动作中使用逗号分隔的字段标识符列表，像这样：

```
[donnie@fedora ~]$ awk -F: '/donnie/ {print $1, $7}' /etc/passwd
donnie /bin/bash
[donnie@fedora ~]$ 
```

所以，在这里，我显示的是字段 1 和字段 7。（请注意，字段编号之间的逗号是在输出中为字段之间添加空格的原因。如果省略逗号，输出将不会有这个空格。）

你也可以将另一个程序的输出通过管道传递给`awk`，像这样：

```
[donnie@fedora ~]$ cat /etc/passwd | awk -F: '/donnie/ {print $1}'
donnie
[donnie@fedora ~]$ 
```

是的，我知道。通常来说，将`cat`的输出管道传递给另一个工具不是一种好形式，尤其是在你可以直接使用其他工具的情况下。但现在这样做是为了演示这个概念。

到目前为止，我展示的所有示例也可以使用`cat`、`grep`、`cut`或它们的组合来完成。所以，你可能会疑惑，为什么要使用`awk`呢？好吧，稍等一下，你很快就会看到`awk`可以做一些其他工具做不到，或者做得不如它的精彩事情。我们将从更深入地了解`awk`如何处理来自普通文本文件的输入开始。

# 从文本文件中获取输入

正如你可能已经猜到的那样，`awk`的默认操作方式是逐行读取文件，搜索每行中指定的模式。当它找到包含指定模式的行时，会对该行执行指定的操作。我们将从前一节中展示的`passwd`文件示例开始，继续构建。

## 查找人类用户

`/etc/passwd`文件包含了系统上所有用户的列表。我一直觉得很奇怪的是，系统用户账户和普通人类用户账户都混杂在同一个文件中。但是，假设作为管理员职责的一部分，你需要维护每台机器上的普通人类用户列表。做到这一点的一个方法是使用`awk`在`passwd`文件中搜索对应人类用户的**用户 ID 号码**（**UIDs**）。要查找普通用户的 UID 号码，你可以查看大多数 Linux 系统上的`/etc/login.defs`文件。这个文件在不同的 Linux 系统上可能会有所不同，所以我只会展示在我的 Fedora 机器上的情况。在第 142 到 145 行，你可以看到以下内容：

```
# Min/max values for automatic uid selection in useradd(8)
#
UID_MIN                  1000
UID_MAX                 60000 
```

所以，为了查看我系统上的所有人类用户账户，我会搜索所有`passwd`文件中字段 3（即 UID 字段）包含大于或等于 1000 且小于或等于 60000 的行，如下所示：

```
donnie@fedora:~$ awk -F: '$3 >= 1000 && $3 <= 60000 {print $1, $7}' /etc/passwd
donnie /bin/bash
vicky /bin/bash
frank /bin/bash
goldie /bin/bash
cleopatra /bin/bash
donnie@fedora:~$ 
```

你还可以将输出重定向到一个文本文件中，只需在动作结构中放置输出重定向符号，如下所示：

```
donnie@fedora:~$ awk -F: '$3 >= 1000 && $3 <= 60000 {print $1, $7 > "users.txt"}' /etc/passwd
donnie@fedora:~$ cat users.txt
donnie /bin/bash
vicky /bin/bash
frank /bin/bash
goldie /bin/bash
cleopatra /bin/bash
donnie@fedora:~$ 
```

在这两个示例中，请注意我在命令的模式部分使用了`&&`作为`and`操作符。

提示

如果你计划将此命令用于自己的用途，确保验证你的操作系统正在使用的 UID 号码范围。大多数 Linux 发行版使用 1000 到 60000 的范围，这个范围定义在`/etc/login.defs`文件中。非 Linux 操作系统，如 OpenIndiana 和 FreeBSD，则不使用`login.defs`文件，我还没能找到关于它们使用的 UID 范围的明确答案。根据我所知道的，你可以查看`/etc/passwd`文件，看看第一个人类用户帐户分配的 UID 是什么，并查阅你所在组织的政策手册，看看组织希望为人类用户使用的 UID 号码范围。

当然，每次想使用这个命令时，你不必每次都键入整个长命令。相反，只需将其放入一个普通的 Shell 脚本中，我将其命名为`user_list.sh`。它将类似于以下内容：

```
#!/bin/bash
awk -F: '$3 >= 1000 && $3 <= 60000 {print $1, $7 > "users.txt"}' /etc/passwd 
```

这很好，但如果你需要一个可以轻松导入到电子表格中的逗号分隔值（`.csv`）文件呢？好吧，我已经为你准备好了。只需添加一个`BEGIN`部分，在其中定义输出的新字段分隔符以及输入字段分隔符，如下所示：

```
donnie@fedora:~$ awk 'BEGIN {OFS = ","; FS = ":"}; $3 >= 1000 && $3 <= 60000 {print $1,$7 > "users.csv"}' /etc/passwd
donnie@fedora:~$ cat users.csv
donnie,/bin/bash
vicky,/bin/bash
frank,/bin/bash
goldie,/bin/bash
cleopatra,/bin/bash
victor,/bin/bash
valerie,/bin/bash
victoria,/bin/bash
donnie@fedora:~$ 
```

这可能看起来有点奇怪，因为当你在`awk`动作中定义输入字段分隔符时，你使用了`-F`选项。但当你在`BEGIN`部分定义输入字段分隔符时，你使用了`FS`选项。同样，在`BEGIN`部分，你使用`OFS`来定义输出字段分隔符。（在`awk`术语中，字段分隔符实际上被称为**字段分隔符**，这也解释了为什么这些`BEGIN`选项被称为`FS`和`OFS`。）

`awk`命令或脚本的`BEGIN`部分是你可以添加任何初始化代码的地方，这些代码将在`awk`开始处理输入文件之前运行。你可以用它来定义字段分隔符，向输出添加标题，或初始化你稍后将使用的全局变量。

这很好，但我还想看到用户的 UID。所以，我只需将第 3 列添加进去，如下所示：

```
donnie@fedora:~$ awk 'BEGIN {OFS = ","; FS = ":"}; $3 >= 1000 && $3 <= 60000 {print $1,$3,$7 > "users.csv"}' /etc/passwd
donnie@fedora:~$ cat users.csv
donnie,1000,/bin/bash
vicky,1001,/bin/bash
frank,1003,/bin/bash
goldie,1004,/bin/bash
cleopatra,1005,/bin/bash
victor,1006,/bin/bash
valerie,1007,/bin/bash
victoria,1008,/bin/bash
donnie@fedora:~$ 
```

现在我已经将我的`awk`命令设置为我想要的样子，我将把它放入名为`user_list2.sh`的 Shell 脚本中，脚本如下所示：

```
#!/bin/bash
awk 'BEGIN {OFS = ","; FS = ":"}; $3 >= 1000 && $3 <= 60000 {print $1,$3,$7 > "users.csv"}' /etc/passwd 
```

你可以用其他编程语言编写一个程序来做同样的事情，比如 Python、C 或 Java。但是，程序代码会相当复杂，且更难以正确实现。使用`awk`，只需一个简单的一行命令就可以完成。

接下来，让我们看看`awk`如何帮助忙碌的 Web 服务器管理员。

## 解析 Web 服务器访问日志

Web 服务器访问日志包含大量信息，可以帮助网站所有者、web 服务器管理员或网络安全管理员。有很多先进的日志解析工具可以生成详细报告，告诉你发生了什么，如果你需要的话。但有时候，你可能想快速提取一些特定数据，而不花时间运行复杂的工具。有几种方法可以做到这一点，而 `awk` 是其中最好的之一。

要开始这个场景，你需要一台安装了活动 web 服务器的虚拟机。为了简化操作，我将在我的 Fedora Server 虚拟机上安装 Apache 服务器和 PHP 模块，方法如下：

```
donnie@fedora:~$ sudo dnf install httpd php 
```

接下来，启用并启动 Apache 服务，方法如下：

```
donnie@fedora:~$ sudo systemctl enable --now httpd 
```

要从不同的机器访问服务器，你需要在防火墙上打开端口 80，方法如下：

```
donnie@fedora:~$ sudo firewall-cmd --add-service=http --permanent
donnie@fedora:~$ sudo firewall-cmd --reload
donnie@fedora:~$ 
```

最后，在 `/var/www/html/` 目录下，创建 `test.php` 文件，并添加以下内容：

```
<?php echo "<strong><center>This is the awk Test Page</strong></center>";
?> 
```

**提醒**：为了使其正常工作，请确保将你的虚拟机设置为桥接模式网络，这样网络中的其他机器就能访问它。还要确保用于访问 web 服务器的虚拟机也设置为桥接模式，以便每台机器在访问日志文件中都有自己的 IP 地址。

现在，从尽可能多的其他机器上访问 Fedora Web 服务器测试页面和 `test.php` 页面。然后，尝试访问一个不存在的页面。你的 URL 应该像这样：

```
http://192.168.0.11
http://192.168.0.11/test.php
http://192.168.0.11/bogus.html 
```

在 web 服务器虚拟机上，使用 `less` 打开 `var/log/httpd/access_log` 文件，方法如下：

```
donnie@fedora:~$ sudo less /var/log/httpd/access_log 
```

注意文件的结构，以及它如何使用空格作为某些字段的分隔符，而使用双引号作为其他字段的分隔符。例如，以下是我自己访问文件中的一条记录：

```
192.168.0.17 - - [18/Dec/2023:16:40:03 -0500] "GET /test.php HTTP/1.1" 200 59 "-" "Mozilla/5.0 (X11; SunOS i86pc; rv:120.0) Gecko/20100101 Firefox/120.0" 
```

所以，你会看到这里有些字段被双引号包围，而有些则没有。现在，我们来看一下访问过这台机器的 IP 地址，方法如下：

```
donnie@fedora:~$ sudo awk '{print $1}' /var/log/httpd/access_log | sort -V | uniq
192.168.0.10
192.168.0.16
192.168.0.17
192.168.0.18
192.168.0.27
192.168.0.37
192.168.0.107
192.168.0.251
192.168.0.252
donnie@fedora:~$ 
```

如你所见，我将 `awk` 输出管道传输到 `sort -V`，然后再管道传输到 `uniq`。使用 `sort` 的 `-V` 选项会按照正确的数字顺序对 IP 地址进行排序。你不能使用 `-n` 选项，因为默认情况下，`sort` 会将 IP 地址中的点当作小数点处理。`-V` 选项会通过执行所谓的**自然排序**来覆盖这一行为。

这样做是不错的，但我仍然不知道每个 IP 地址访问 web 服务器的次数。我将使用带 `-c` 选项的 `uniq` 来查看，这将显示如下内容：

```
donnie@fedora:~$ sudo awk '{print $1}' /var/log/httpd/access_log | sort -V | uniq -c
      6  192.168.0.10
     20 192.168.0.16
     12 192.168.0.17
      2  192.168.0.18
      4  192.168.0.27
     15 192.168.0.37
      6  192.168.0.107
     14 192.168.0.251
      6  192.168.0.252
donnie@fedora:~$ 
```

这样不错，但我真的想创建一个`.csv`文件，就像我为我的用户列表做的那样。因为计数字段直到我将输出管道到`uniq -c`之后才会创建，所以这次我不能通过在`BEGIN`部分定义`OFS`参数来做到这一点。所以，我稍微“作弊”一下，安装`miller`包，它包含在大多数 Linux 发行版以及 FreeBSD 的仓库中。在 Mac 上，你应该可以通过`homebrew`安装它。无论如何，为了创建`.csv`文件，我将把来自这个`awk`命令的输出管道到`mlr`工具，它是`miller`包的一部分。看起来是这样的：

```
donnie@fedora:~$ sudo awk '{print $1}' /var/log/httpd/access_log | sort -V | uniq -c | mlr --p2c cat > ip_list.csv
donnie@fedora:~$ cat ip_list.csv
6,192.168.0.10
27,192.168.0.16
12,192.168.0.17
2,192.168.0.18
4,192.168.0.27
15,192.168.0.37
6,192.168.0.107
14,192.168.0.251
6,192.168.0.252
donnie@fedora:~$ 
```

`mlr`的`--p2c`选项用于将输出转换为`.csv`格式。接下来是`cat`，这是`mlr`的动词。你会看到这个`cat`会像它的`bash`版本一样，将输出直接显示到屏幕或文件中。

我正在向你展示如何创建`.csv`文件，因为出于历史原因，`.csv`是最流行的纯文本数据文件格式，而你的雇主或客户可能期望你使用这种格式。然而，正如我们稍后会看到的，这并不总是最佳格式。在某些情况下，你可能会发现最好的选择是忘记`.csv`文件，而是将数据保存为**制表符分隔值**（`.tsv`）文件，它们可能像这样：

```
donnie@fedora:~$ cat inventory.tsv
Kitchen spatula	$4.99	       Housewares
Raincoat	       $36.99	 	Clothing	On Sale!
Claw hammer	 	$7.99	       Tools
donnie@fedora:~$ 
```

你还可以将它们导入你最喜欢的电子表格程序，在某些情况下，它们更容易处理。

如果你需要的是`.json`文件而不是`.csv`文件，`mlr`也能帮你做到。你只需将`--p2c`选项替换为`--ojson`选项，如下所示：

```
donnie@fedora:~$ sudo awk '{print $1}' /var/log/httpd/access_log | sort -V | uniq -c | mlr --ojson cat > ip_list.json
donnie@fedora:~$ cat ip_list.json
{ "1": "      6 192.168.0.10" }
{ "1": "     27 192.168.0.16" }
{ "1": "     12 192.168.0.17" }
{ "1": "      2 192.168.0.18" }
{ "1": "      4 192.168.0.27" }
{ "1": "     15 192.168.0.37" }
{ "1": "      6 192.168.0.107" }
{ "1": "     14 192.168.0.251" }
{ "1": "      6 192.168.0.252" }
donnie@fedora:~$ 
```

你可以用`mlr`做的事情远远超过我在这里展示的内容。要了解更多，查阅`mlr`的手册页或 Miller 文档网页。 （你可以在*进一步阅读*部分找到指向 Miller 网页的链接。）不过，尽管`mlr`非常优秀，它确实有一个缺点。也就是，它不能处理包含空格或逗号的字段。所以，你不能在 Apache 访问日志中的每个字段上使用它。

现在，假设我们想查看哪些操作系统和浏览器正在访问这个服务器。包含这些信息的**用户代理**字符串位于第 6 个字段中，并且被一对双引号包围。所以，我们需要使用双引号作为字段分隔符，如下所示：

```
donnie@fedora:~$ sudo awk -F\" '{print $6}' /var/log/httpd/access_log | sort | uniq
Lynx/2.8.9rel.1 libwww-FM/2.14 SSL-MM/1.4.1 OpenSSL/1.1.1t-freebsd
Lynx/2.9.0dev.10 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.1
Lynx/2.9.0dev.12 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.8
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Safari/605.1.15
. . .
. . .
Mozilla/5.0 (X11; Linux x86_64; rv:120.0) Gecko/20100101 Firefox/120.0
Mozilla/5.0 (X11; Linux x86_64; rv:45.0) Gecko/20100101 Firefox/45.0
Mozilla/5.0 (X11; SunOS i86pc; rv:120.0) Gecko/20100101 Firefox/120.0
Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/113.0
donnie@fedora:~$ 
```

你可以看到，我必须用反斜杠转义双引号，这样 shell 就不会错误地解析它。

接下来，让我们查找特定的用户代理。假设我们想知道有多少用户使用的是 Mac。只需添加模式，像这样：

```
donnie@fedora:~$ sudo awk -F\" '/Mac OS X/ {print $6}' /var/log/httpd/access_log | sort | uniq -c
      8 Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Safari/605.1.15
      6 Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:109.0) Gecko/20100101 Firefox/115.0
      6 Mozilla/5.0 (Macintosh; PPC Mac OS X 10.4; FPR10; rv:45.0) Gecko/20100101 Firefox/45.0 TenFourFox/7450
      9 Mozilla/5.0 (Macintosh; U; PPC Mac OS X 10_4_11; en) AppleWebKit/533.19.4 (KHTML, like Gecko) Version/4.1.3 Safari/533.19.4
donnie@fedora:~$ 
```

英特尔 Mac 的条目来自我 2010 年的 Mac Pro，所以它们是合理的。但是，输出的第 3 和第 4 行中出现的 PPC 是怎么回事？这也很好解释。只是为了好玩，我启动了我的 21 年历史的 eMac，它配备了老式的摩托罗拉 PowerPC G4 处理器，用它访问了测试 Web 服务器。（我猜测你在自己的网络上永远不会看到这些设备。）

你也可以仅使用纯 `awk` 获得这些信息，无需将输出传递给其他工具。通过构建一个**关联数组**，可以实现这一点，格式如下：

```
donnie@fedora:~$ sudo awk '{count[$1]++}; END {for (ip_address in count) print 
ip_address, count[ip_address]}' /var/log/httpd/access_log
192.168.0.251 14
192.168.0.252 6
192.168.0.10 6
192.168.0.16 22
192.168.0.17 12
192.168.0.107 6
192.168.0.18 2
192.168.0.27 4
192.168.0.37 15
donnie@fedora:~$ 
```

添加一个 `BEGIN` 部分，并定义 `OFS`，你就可以创建一个 `.csv` 文件。它是这样的：

```
donnie@fedora:~$ sudo awk 'BEGIN {OFS = ","}; {count[$1]++}; END {for (ip_address in count) print ip_address, count[ip_address]}' /var/log/httpd/access_log > ip_addresses.csv
donnie@fedora:~$ cat ip_addresses.csv
192.168.0.251,14
192.168.0.252,6
192.168.0.10,6
192.168.0.16,27
192.168.0.17,12
192.168.0.107,6
192.168.0.18,2
192.168.0.27,4
192.168.0.37,15
donnie@fedora:~$ 
```

与你之前见过的普通 Shell 脚本数组不同，`awk` 的关联数组使用文本字符串，而不是数字，作为索引。此外，`awk` 中的数组是动态定义的，无需在使用前声明它们。所以在这里你会看到 `count` 数组，它的索引值对应字段 1 的各个值。`++` 运算符假定初始值为 0，表示每个索引字符串在文件中出现的次数。这个命令的第一部分，以分号结尾，按行处理文件，正如 `awk` 通常所做的那样。`END` 关键字标志着代码会在逐行处理完成后运行。在这种情况下，你会看到一个 `for` 循环，打印出唯一 IP 地址的摘要，并显示每个地址出现的次数。不幸的是，用纯 `awk` 对输出进行排序并没有简单的方法，但你仍然可以将输出传递给 `sort`，像这样：

```
donnie@fedora:~$ sudo awk 'BEGIN {OFS = ","}; {count[$1]++}; END {for (ip_address in count) print ip_address, count[i]}' /var/log/httpd/access_log | sort -t, -V -k1,1 > ip_addresses.csv
donnie@fedora:~$ cat ip_addresses.csv
192.168.0.10,6
192.168.0.16,27
192.168.0.17,12
192.168.0.18,2
192.168.0.27,4
192.168.0.37,15
192.168.0.107,6
192.168.0.251,14
192.168.0.252,6
donnie@fedora:~$ 
```

为了使其生效，我使用 `-t,` 选项对 `sort` 进行排序，将逗号定义为字段分隔符。（是的，我知道。如果所有 Linux 和 Unix 工具都使用相同的选项开关来定义字段分隔符，那就好了，但现实情况并非如此。）

使用纯 `awk` 的另一个区别是，每个 IP 地址的出现次数位于第二列，而不是你之前看到的第一列。这里没问题，但对于其他字段可能不太适用。例如，让我们将命令更改为查看字段 6 中的用户代理：

```
donnie@fedora:~$ sudo awk -F\" '{count[$6]++}; END { for (i in count) print i, count[i]}' /var/log/httpd/access_log | sort
Lynx/2.8.9rel.1 libwww-FM/2.14 SSL-MM/1.4.1 OpenSSL/1.1.1t-freebsd 2
Lynx/2.9.0dev.10 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.1 4
Lynx/2.9.0dev.12 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.8 3
Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Safari/605.1.15 8
. . .
. . .
Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/113.0 6
donnie@fedora:~$ 
```

是的，方法有效，但在第二列中显示出现次数使得输出的可读性稍差。

正如我之前提到的，`.csv` 格式并不总是最适合用作纯文本数据文件。事实上，将字段 6 的输出转化为任何电子表格程序能够正确显示的 `.csv` 文件几乎是不可能的。因为该字段包含空格和逗号，会导致电子表格程序误认为 `.csv` 文件中有比实际更多的字段。

所以，最简单的解决方案是用一对双引号将字段 6 中的文本括起来，并将输出保存为 `.tsv` 文件。然后，当你在电子表格中打开该文件时，将 `"` 定义为字段分隔符。无论如何，以下是创建 `.tsv` 文件的命令，未使用关联数组：

```
donnie@fedora:~$ sudo awk 'BEGIN {FS="\""} {print "\"" $6 "\""}' /var/log/
httpd/access_log | sort | uniq -c | sort -k 1,1 -nr > user_agent.tsv
donnie@fedora:~$ 
```

默认情况下，`awk` 会去掉原始日志文件中字段 6 周围的双引号。所以，为了使这个操作正常工作，我必须将双引号放回最终输出文件中。在操作部分，你会看到我在`$6`之前和之后都打印了一个双引号。通过省略你通常会在各个`print`元素之间放置的逗号，我确保双引号和文本之间没有空格。然后，我只是像之前所展示的那样，将输出通过管道传递给 `sort`、`uniq`，然后再次传递给 `sort`。下面是文件的样子：

```
donnie@fedora:~$ cat user_agent.tsv
     23 "Mozilla/5.0 (X11; Linux x86_64; rv:120.0) Gecko/20100101 Firefox/120.0"
      9 "Mozilla/5.0 (X11; SunOS i86pc; rv:120.0) Gecko/20100101 Firefox/120.0"
      9 "Mozilla/5.0 (Macintosh; U; PPC Mac OS X 10_4_11; en) AppleWebKit/533.19.4 (KHTML, like Gecko) Version/4.1.3 Safari/533.19.4"
      8 "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Safari/605.1.15"
      6 "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/113.0"
. . .
. . .
      3 "Lynx/2.9.0dev.12 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.8"
      2 "Lynx/2.8.9rel.1 libwww-FM/2.14 SSL-MM/1.4.1 OpenSSL/1.1.1t-freebsd"
donnie@fedora:~$ 
```

如果你更喜欢使用关联数组，可以像这样操作：

```
donnie@fedora:~$ sudo awk -F\" '{count[$6]++}; END { for (ip_address in count) printf "%s \"%s\"\n", count[ip_address], ip_address}' /var/log/httpd/access_log | sort -nr > user_agent.tsv
donnie@fedora:~$ cat user_agent.tsv
      23 "Mozilla/5.0 (X11; Linux x86_64; rv:120.0) Gecko/20100101 Firefox/120.0"
      9 "Mozilla/5.0 (X11; SunOS i86pc; rv:120.0) Gecko/20100101 Firefox/120.0"
      9 "Mozilla/5.0 (Macintosh; U; PPC Mac OS X 10_4_11; en) AppleWebKit/533.19.4 (KHTML, like Gecko) Version/4.1.3 Safari/533.19.4"
      8 "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.2 Safari/605.1.15"
      6 "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/113.0"
. . .
. . .
      3 "Lynx/2.9.0dev.12 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.8"
      2 "Lynx/2.8.9rel.1 libwww-FM/2.14 SSL-MM/1.4.1 OpenSSL/1.1.1t-freebsd"
donnie@fedora:~$ 
```

注意我如何需要将 `printf` 的参数用一对双引号括起来。第一个 `%s` 参数是用于计数字段，第二个 `%s` 参数是用于用户代理字段。为了在正确的位置添加双引号，我在第二个 `%s` 前后都加了一个 `\"`。使用关联数组方法或非关联数组方法都能得到相同的结果。现在，当我在电子表格程序中打开文件时，我只需将双引号定义为字段分隔符。下面是它在 LibreOffice Calc 中的样子：

![B21693_14_1](img/B21693_14_01.png)

图 14.1：在 LibreOffice Calc 中设置字段分隔符

我知道我还没有完全解释 `print` 和 `printf` 的区别，但我会在下一章中详细说明。

`user_agent.tsv` 文件现在应该能够正确显示，第一列是计数，第二列是用户代理字符串。

接下来，让我们统计一下每个 URL 在 Web 服务器上的访问次数。我喜欢将计数作为输出的第一个字段，所以我再次将 `awk` 的输出通过管道传递给 `uniq` 和 `sort`，如下所示：

```
donnie@fedora:~$ sudo awk '{print $7}' /var/log/httpd/access_log | sort | uniq -c | sort -k1,1 -n
      1 /something.html
      1 /test
      1 /test.html
      1 /test/php
      1 /test.php?module=..../
      2 /test.php?module=....//....//....//....//....//....//....//....//....//....//proc/self/environ%0000
      2 /test.php?page=../../../../../../../../../../../../../../../proc/self/environ%00
     11 /icons/poweredby.png
     11 /poweredby.png
     11 /report.html
     12 /favicon.ico
     17 /
     18 /test.php
donnie@fedora:~$ 
```

使用空格作为字段分隔符意味着 URL 字段是字段 7。最后，我再次将输出通过管道传递给 `sort`，以便按每个 URL 的点击次数输出列表。但实际上，我希望最受欢迎的 URL 显示在列表的顶部。所以，我只需加上 `-r` 选项来反向排序，如下所示：

```
donnie@fedora:~$ sudo awk '{print $7}' /var/log/httpd/access_log | sort | uniq -c | sort -k1,1 -nr
     18 /test.php
     17 /
     12 /favicon.ico
     11 /report.html
     11 /poweredby.png
     11 /icons/poweredby.png
      2 /test.php?page=../../../../../../../../../../../../../../../proc/self/environ%00
      2 /test.php?module=....//....//....//....//....//....//....//....//....//....//proc/self/environ%0000
      1 /test.php?module=..../
      1 /test/php
      1 /test.html
      1 /test
      1 /something.html
donnie@fedora:~$ 
```

我们看到的一件事是，有人试图对我进行目录遍历攻击，从以 `/test.php?page` 或 `/test.php?module` 开头的行中可以看出。我稍后会在 *第十八章：安全专家的 Shell 脚本* 中给你更多讲解。

我们最后要看的字段是字段 9，它是**HTTP 状态码**。同样，我们将使用空格作为字段分隔符，如下所示：

```
donnie@fedora:~$ sudo awk '{print $9}' /var/log/httpd/access_log | sort | uniq -c | sort -k1,1 -nr
     53 200
     17 404
     17 403
      2 304
donnie@fedora:~$ 
```

下面是这些代码含义的分解：

+   200：`200` 代码意味着用户可以正常访问网页，没有任何问题。

+   304：`304` 代码意味着当用户重新加载一个页面时，页面没有任何变化，因此不需要重新加载。

+   403：`403` 代码意味着有人尝试访问一个用户没有授权的页面。

+   404：`404` 代码意味着用户尝试访问一个不存在的页面。

好的，这一切都很好。所以现在，让我们把所有这些放到`access_log_parse.sh`脚本中，它将如下所示：

```
#!/bin/bash
#
timestamp=$(date +%F_%I-%M-%p)
awk 'BEGIN {FS="\""} {print "\"" $6 "\""}' /var/log/httpd/access_log | sort | uniq -c | sort -k 1,1 -nr > user_agent_$timestamp.tsv
awk '{print $1}' /var/log/httpd/access_log | sort -V | uniq -c | sort -k1,1 -nr >> source_IP_addreses_$timestamp.tsv
awk '{print $7}' /var/log/httpd/access_log | sort | uniq -c | sort -k1,1 -nr > URLs_Requested_$timestamp.tsv
awk '{print $9}' /var/log/httpd/access_log | sort | uniq -c | sort -k1,1 -nr > HTTP_Status_Codes_$timestamp.tsv 
```

我决定为每个功能创建单独的`.tsv`文件，并且我想在每个文件名中加上时间戳。`date`命令的`%F`选项以`YEAR-MONTH-DATE`格式打印日期，这很好。我本可以使用`%T`选项打印时间，但那样会在文件名中加入冒号，这将要求我每次从命令行访问这些文件时都要转义冒号。因此，我改用了`%I-%M-%p`组合，以将冒号替换为破折号。（要了解更多格式选项，请参阅`date`手册页。）脚本的其余部分包括您已经看到的命令，所以我不会重复任何解释。

当然，您可以使用正则表达式来优化任何这些脚本，以满足您自己的需求和需求。只需使用我在前几章中展示给您的技术，向输出添加标记语言标签，并将输出文件转换为`.html`或`.pdf`格式。对于我刚刚展示给您的多功能脚本，您可以添加`if..then`或`case`结构，以允许您选择要运行的特定功能。如果您需要回头查看以了解任何操作方式，也不必感到难过。相信我，我不会因此责怪您。

到目前为止，我们一直在整个文件中解析以找到我们想要查看的内容。不过有时候，您可能只想查看特定行或一系列行的信息。例如，假设您只想查看第 10 行。请按照以下方式操作：

```
donnie@fedora:~$ sudo awk 'NR == 10' /var/log/httpd/access_log
192.168.0.16 - - [17/Dec/2023:17:31:20 -0500] "GET /test.php HTTP/1.0" 200 59 "-" "Lynx/2.9.0dev.10 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.1"
donnie@fedora:~$ 
```

`NR`变量表示**记录数**。由于`access_log`文件中的每条记录仅占一行，这与定义您想要查看的行号相同。要查看一系列行，请执行以下操作：

```
donnie@fedora:~$ sudo awk 'NR == 10, NR == 15 {print $1}' /var/log/httpd/access_log
192.168.0.16
192.168.0.16
192.168.0.16
192.168.0.107
192.168.0.107
192.168.0.107
donnie@fedora:~$ 
```

在这里，我正在查看第 10 到 15 行的第 1 字段。当然，您也可以通过结合`tail`、`head`和`cut`工具来做同样的事情，但这种方式更简单。

在本节中，我们已经看过`FS`、`OFS`和`NR`。我还没告诉你的是，这三个结构都是内置到`awk`中的变量。还有许多其他内置变量，但我现在不想一下子就把它们都解释清楚，以免让你感到不知所措。如果您有兴趣了解所有这些内容，只需打开`awk`手册页并滚动到**内置变量**部分即可。

我在这里介绍的日志解析技术可以用于任何类型的日志文件。要设计您的`awk`命令，请查看要处理的日志文件，并注意每行的字段布局及其包含的信息。

在我们继续之前，让我们看看一些其他日志解析技术。

## 使用正则表达式

您可以像使用其他文本处理工具一样，通过正则表达式增强您的`awk`体验。

如果需要，可以回顾一下*第二章，文本流过滤器 – 第二部分*和*第九章，使用 grep、sed 和正则表达式过滤文本*，复习正则表达式和 POSIX 字符类的概念。

对于我们的第一个例子，假设你需要在 `/etc/passwd` 文件中查找所有用户名以小写字母 *v* 开头的用户。只需使用一个简单的正则表达式，像这样：

```
donnie@fedora:~$ awk '/^v/' /etc/passwd
vicky:x:1001:1001::/home/vicky:/bin/bash
victor:x:1006:1006::/home/victor:/bin/bash
valerie:x:1007:1007::/home/valerie:/bin/bash
victoria:x:1008:1008::/home/victoria:/bin/bash
donnie@fedora:~$ 
```

之所以这样有效，是因为用户名字段是每一行的第一个字段。所以，你不需要做任何复杂的操作，仅仅这样就可以了。但是，假设你需要在另一个字段中搜索以特定字符开头的内容。比如说，你需要在 Apache 访问日志文件中查找所有属于 400 范围的 HTTP 状态码。你只需要像这样做：

```
donnie@fedora:~$ sudo awk '$9 ~ /⁴/{print $9}' /var/log/httpd/access_log | sort -n | uniq -c
     17 403
     20 404
donnie@fedora:~$ 
```

模式中的 `$9` 表示你在第 9 个字段中查找某个特定模式。`~` 表示你希望该字段中的内容匹配后面正斜杠之间的内容。在这种情况下，它是在寻找第 9 个字段中以数字 4 开头的内容。在输出中，你会看到我找到了 403 和 404 状态码。如果需要，你还可以将输出保存为 `.csv` 文件，像这样：

```
donnie@fedora:~$ sudo awk '$9 ~ /⁴/{print $9}' /var/log/httpd/access_log | sort -n | uniq -c | mlr --p2c cat > status_code.csv
donnie@fedora:~$ cat status_code.csv
17,403
20,404
donnie@fedora:~$ 
```

如果你更喜欢的话，你可以通过省略 `mlr --p2c cat` 部分，将其保存到 `.tsv` 文件中，像这样：

```
donnie@fedora-server:~$ sudo awk '$9 ~ /⁴/{print $9}' /var/log/httpd/access_log | sort -n | uniq -c > status_code.tsv
donnie@fedora-server:~$ cat status_code.tsv
      6 403
      4 404
donnie@fedora-server:~$ 
```

要查看所有*除了*某个模式以外的内容，可以使用 `!~` 构造，像这样：

```
donnie@fedora:~$ sudo awk '$9 !~ /⁴/{print $9}' /var/log/httpd/access_log | sort -n | uniq -c
     53 200
      2 304
donnie@fedora:~$ 
```

这样，我们就可以查看所有除 400 状态码以外的内容。

接下来，假设我们想要查看所有使用 Safari 或 Lynx 浏览器的实例。你可以这样做：

```
donnie@fedora:~$ sudo awk '/Safari|Lynx/' /var/log/httpd/access_log 
```

但是，这样并不行，因为它会包含像这样的条目：

```
192.168.0.27 - - [17/Dec/2023:17:43:40 -0500] "GET / HTTP/1.1" 403 8474 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36 Edg/120.0.0.0" 
```

出于某种奇怪的原因，Apache 无法准确识别 Microsoft Edge 浏览器，所以它将其报告为多个不同的可能性，包括 Safari。为了缩小范围，我会排除所有使用 Windows 的用户。（Safari 曾经在 Windows 上可用，但 Windows 版本在 2012 年就停止了。）下面是它的表现方式：

```
donnie@fedora:~$ sudo awk -F\" '$6 ~/Safari|Lynx/ && $6 !~ /Windows/' /var/log/httpd/access_log 
```

你会发现这里有点奇怪。当你在正则表达式中使用 `or` 或 `and` 操作符时，你使用的是单个 `|` 或单个 `&`。而当你在正则表达式外部使用 `or` 或 `and` 操作符时，你需要使用 `||` 或 `&&`。我知道，这有点让人困惑，但就是这样。不管怎样，如果你现在查看输出，你会看到没有来自 Windows 用户的行。

我知道，这只是 `awk` 和正则表达式的冰山一角。如果你请求得特别恳切，我可能会在下一个部分给你展示一些更多的例子，内容是如何使用 `awk` 处理来自其他命令的信息。

# 从命令获取输入

让我们从一个简单的例子开始，获取一些基本的进程信息。在运行 Apache 的 Fedora Server 虚拟机上，搜索所有包含 `httpd` 模式的 `ps aux` 输出行，像这样：

```
donnie@fedora:~$ ps aux | awk '/httpd/ {print $0}'
root        1072  0.0  0.2  19108 10796 ?        Ss   14:36   0:01 /usr/sbin/httpd -DFOREGROUND
apache      1111  0.0  0.1  19204  6788 ?        S    14:36   0:00 /usr/sbin/httpd -DFOREGROUND
apache      1112  0.0  0.2 2158280 8448 ?        Sl   14:36   0:02 /usr/sbin/httpd -DFOREGROUND
apache      1113  0.0  0.2 2420488 8592 ?        Sl   14:36   0:03 /usr/sbin/httpd -DFOREGROUND
apache      1114  0.0  0.2 2158280 8448 ?        Sl   14:36   0:02 /usr/sbin/httpd -DFOREGROUND
donnie      1908  0.0  0.0   9196  3768 pts/0    S+   17:13   0:00 awk /httpd/ {print $0}
donnie@fedora:~$ 
```

接下来，假设我们想查看所有由 root 用户拥有的进程。也很简单，直接执行以下命令：

```
donnie@fedora:~$ ps aux | awk '$1 == "root" {print $0}'
root           1  0.0  0.6  74552 27044 ?        Ss   14:26   0:05 /usr/lib/systemd/systemd --switched-root --system --deserialize=36 rhgb
root           2  0.0  0.0      0     0 ?        S    14:26   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        S    14:26   0:00 [pool_workqueue_release]
. . .
. . .
root        1688  0.0  0.1  16332  6400 ?        S    16:53   0:00 systemd-userwork: waiting...
donnie@fedora:~$ 
```

这里没有什么新内容，因为这些 `awk` 命令和你在上一节中看到的一样。事实上，几乎所有解析日志文件的技巧在这里也同样适用。所以，接下来就不再重复这些内容了。

解析 `ps` 工具的信息的第一步是查看各个字段的含义。我个人发现 `aux` 选项组合最为实用，因为它显示了我最需要查看的特定信息。要查看显示字段名的 `ps` 头部，我会将 `ps aux` 的输出通过管道传递给 `head -1`，像这样：

```
[donnie@fedora ~]$ ps aux | head -1
USER      PID   %CPU   %MEM    VSZ   RSS   TTY    STAT   START   TIME   COMMAND
[donnie@fedora ~]$ 
```

这是所有这些字段的含义说明：

+   `USER`：拥有每个进程的用户。

+   `PID`：每个进程的**进程 ID**。

+   `%CPU`：每个进程消耗的 CPU 资源百分比。

+   `RSS`：**常驻内存集大小**，即每个进程使用的实际物理内存（未交换的内存）。(虽然这里的顺序可能有点乱，但很快你会明白原因。)

+   `%MEM`：进程的 RSS（常驻内存集）与计算机中安装的物理内存总量的比率。

+   `VSZ`：**虚拟内存集大小**，即每个进程使用的虚拟内存量。（以 1024 字节单位表示。）

+   `TTY`：控制每个进程的终端。

+   `STAT`：此列显示每个进程的状态代码。

+   `START`：每个进程的启动时间或日期。

+   `TIME`：每个进程的累计 CPU 时间，格式为“`[DD-]HH:MM:SS`”。

+   `COMMAND`：启动每个进程的命令。

请记住，`ps` 有许多选项开关，每个都显示不同类型的进程数据。你还可以通过组合这些选项来获取所有你需要的信息，并以你喜欢的格式显示。不过，现在我会坚持使用 `ps aux`，主要是因为这是最适合我的选项组合。有关详细信息，请参阅 `ps` 的手册页。

默认情况下，`ps` 命令总是显示此头部和你想查看的信息。因此，如果你将 `ps` 的输出通过管道传递给 `awk` 命令而不对其进行过滤，你也会看到这个头部。例如，看看那些不是 root 用户拥有的进程，像这样：

```
donnie@fedora:~$ ps aux | awk '$1 != "root" {print $0}'
USER        PID  %CPU %MEM   VSZ    RSS   TTY   STAT START  TIME COMMAND
systemd+    800  0.1      0.1           16240  7328   ?        Ss       14:27      0:13   /usr/lib/systemd/systemd-oomd
systemd+    801  0.0      0.4           27656  16236 ?        Ss        14:27     0:00   /usr/lib/systemd/systemd-resolved
. . .
. . .
donnie       1745  200    0.1            9888   4520   pts/0    R+      17:10     0:00   ps aux
donnie        746  0.0     0.0            9196   3804   pts/0    S+       17:10     0:00   awk $1 != "root" {print $0}
donnie@fedora:~$ 
```

这样通常是没问题的。但是，如果你需要将信息保存到格式化的文本文件中，头部可能会给你带来麻烦。为了解决这个问题，我们可以使用我在前一节中展示过的 **记录数**（`NR`）变量。这样操作：

```
donnie@fedora:~$ ps aux | awk 'NR > 1 && $1 != "root" {print $0}'
systemd+     800  0.1  0.1  16240  7328 ?        Ss   14:27   0:15 /usr/lib/systemd/systemd-oomd
systemd+     801  0.0  0.4  27656 16236 ?        Ss   14:27   0:00 /usr/lib/systemd/systemd-resolved
. . .
. . .
donnie      1765  250  0.1   9888  4620 pts/0    R+   17:26   0:00 ps aux
donnie      1766  0.0  0.0   9196  3780 pts/0    S+   17:26   0:00 awk NR > 1 && $1 != "root" {print $0}
donnie@fedora:~$ 
```

`NR > 1` 条件意味着我们只想查看第 1 记录之后的记录。换句话说，我们不想看到输出的第一行，在这种情况下那就是 `ps` 头部。

现在我们知道了每个字段的含义，我们可以创建一些有用的一行命令来提取信息。首先，让我们看看有多少进程处于 **运行中** 或 **僵尸** 状态。（僵尸进程是已经死掉的进程，但其父进程尚未完全销毁它们。你可以查看 `ps` 的手册页来了解更多。）我们知道 `STAT` 列是第 8 个字段，因此命令将如下所示：

```
[donnie@fedora ~]$ ps aux | awk '$8 ~ /^[RZ]/ {print}'
donnie      2803 30.5  1.6 4963152 1058324 ?     Rl   14:00  70:12 /usr/lib64/firefox/firefox
donnie      4383  1.9  0.4 3157216 307460 ?      Rl   14:03   4:18 /usr/lib64/firefox/firefox -contentproc -childID 33 -isForBrowser -prefsLen 31190 -prefMapSize 237466 -jsInitLen 229864 -parentBuildID 20231219113315 -greomni /usr/lib64/firefox/omni.ja -appomni /usr/lib64/firefox/browser/omni.ja -appDir /usr/lib64/firefox/browser {d458c911-d065-4a3c-bdd5-9d06fc1d030d} 2803 true tab
donnie     46283  0.0  0.0      0     0 ?        R    17:50   0:00 [Chroot Helper]
donnie     46315  0.0  0.0 224672  3072 pts/0    R+   17:50   0:00 ps aux
[donnie@fedora ~]$ 
```

它看起来有些混乱，因为第二个进程的 `COMMAND` 字段包含了一个很长的字符串，但这没关系。你仍然可以看到每一行的第 8 字段都以字母 `R` 开头。（你在系统中很少会看到僵尸进程，所以不用担心这里没有看到它们。而且，不，僵尸进程不会四处寻找大脑来偷。）

`w` 命令会显示所有已登录系统的用户以及他们正在做什么。它看起来是这样的：

```
donnie@fedora:~$ w
 17:37:44 up  3:11,  4 users,  load average: 0.03, 0.06, 0.02
USER        TTY         LOGIN@  IDLE     JCPU     PCPU      WHAT
donnie      tty1        17:31   5:51     0.06s    0.06s     -bash
donnie      pts/0       15:29   0.00s    0.35s    0.06s     w
vicky       pts/1       17:32   4:50     0.05s    0.05s     -bash
frank       pts/2       17:33   3:18     0.05s    0.05s     -bash
donnie@fedora:~$ 
```

你可以看到，我是通过 `tty1` 登录的，这是本地终端。我也通过一个 `pts` 终端远程登录，并且 Vicky 和 Frank 也在其中。（记住，`tty` 终端表示本地登录，而 `pts` 终端表示远程登录。）现在，让我们使用 `ps` 查看远程用户正在运行的进程的更多信息，如下所示：

```
donnie@fedora:~$ ps aux | awk '$7 ~ "pts"'
donnie    1492  0.0  0.1  8480  5204 pts/0  Ss   15:29   0:00 -bash
vicky     1845  0.0  0.1  8348  4960 pts/1  Ss+  17:32   0:00 -bash
frank     1892  0.0  0.1  8348  5000 pts/2  Ss+  17:33   0:00 -bash
donnie    1936  200  0.1  9888  4640 pts/0  R+   17:37   0:00 ps aux
donnie    1937  0.0  0.0  9064  3908 pts/0  S+   17:37   0:00 awk $7 ~ "pts"
donnie@fedora:~$ 
```

刚才，我向你展示了如果你的 `awk` 命令没有自动去除头部行时，如何去除它。这次，`awk` 命令已经去除了它，但我决定还是要看到它。为了解决这个问题，我会这样做：

```
donnie@fedora:~$ ps aux | awk '$7 ~ "pts" || $1 == "USER"'
USER   PID   %CPU  %MEM  VSZ  RSS  TTY   STAT START  TIME COMMAND
donnie 1492  0.0   0.1   8480 5204 pts/0 Ss   15:29  0:00   -bash
vicky  1845  0.0   0.1   8348 4960 pts/1 Ss+  17:32  0:00   -bash
frank  1892  0.0   0.1   8348 5000 pts/2 Ss+  17:33  0:00   -bash
donnie 2029  400   0.1   9888 4416 pts/0 R+   17:48  0:00   ps aux
donnie 2030  0.0   0.0   9064 3928 pts/0 S+   17:48  0:00   awk $7 ~ "pts" || $1 == "USER"
donnie@fedora:~$ 
```

请注意，我正在使用两种不同的方法来查找模式。`$7 ~ "pts"` 部分会查找第 7 字段中包含 `pts` 文本字符串的所有行。因此，我们看到 `pts/0`、`pts/1` 和 `pts/2` 都符合这个搜索条件。`$1 == "USER"` 部分则是在查找一个精确的、完整的单词匹配。为了演示这一点，请查看这个 `user.txt` 文件：

```
[donnie@fedora ~]$ cat user.txt
USER		donnie
USER1	vicky
USER2	cleopatra
USER3	sylvester
[donnie@fedora ~] 
```

使用 `~` 在第 1 字段中查找所有包含 `USER` 的行，我们得到如下结果：

```
[donnie@fedora ~]$ awk '$1 ~ "USER" {print $1}' user.txt
USER
USER1
USER2
USER3
[donnie@fedora ~]$ 
```

将 `~` 替换为 `==` 后，我们得到如下结果：

```
[donnie@fedora ~]$ awk '$1 == "USER" {print $1}' user.txt
USER
[donnie@fedora ~]$ 
```

所以你可以看到两者之间有很大的区别。另外，请注意，由于我在搜索一个字面文本字符串，因此在使用 `~` 时，我可以将搜索词（`USER`）放在一对双引号或一对斜杠内。但在使用 `==` 时，你需要使用双引号，因为使用双斜杠什么也不会显示。

话题稍微有些跑题，回到我们的正题。

我现在决定查看所有虚拟内存大小（`VSZ`）超过 500,000 字节的进程。`VSZ` 信息在第 5 字段，因此我的 `awk` 命令看起来是这样的：

```
donnie@fedora:~$ ps aux | awk '$5 > 500000 {print $0}'
USER   PID  %CPU %MEM  VSZ     RSS  TTY  STAT START   TIME COMMAND
apache 1098 0.0  0.2   2420488 8628  ?   Sl   14:28   0:04 /usr/sbin/httpd -DFOREGROUND
apache 1099 0.0  0.2   2223816 8248  ?   Sl   14:28   0:03 /usr/sbin/httpd -DFOREGROUND
apache 1100  0.0 0.2   2158280 8232  ?   Sl   14:28   0:04 /usr/sbin/httpd -DFOREGROUND
donnie@fedora:~$ 
```

但这些信息对我来说其实有些多余。我只是想查看相关的字段，并决定将每个字段单独放在一行上，并加上标签。以下是我的做法：

```
donnie@fedora:~$ ps aux | awk 'NR > 1 && $5 > 500000 {print "USER:" "\t\t" $1 "\n" "PID:" "\t\t" $2 "\n" "VSZ:" "\t\t" $5 "\n" "COMMAND:" "\t" $11 "\n\n"}'
USER:		apache
PID:		 	1147
VSZ:			2354952
COMMAND:		/usr/sbin/httpd
USER:		apache
PID:			1148
VSZ:			2158280
COMMAND:		/usr/sbin/httpd
USER:		apache
PID:			1149
VSZ:		2158280
COMMAND:		/usr/sbin/httpd
donnie@fedora:~$ 
```

如前所述，`NR > 1` 防止 `ps` 打印默认的标题。在动作部分，`print` 命令在 `USER`、`PID`、`VSZ` 及其相应值之间放置一对制表符（`\t\t`）。由于 `COMMAND` 标签较长，因此它和其值之间只需要一个制表符（`\t`）。这确保了第二列的所有值整齐对齐。在 `USER`、`PID` 和 `VSZ` 的值后面的换行符（`\n`）使得下一个字段在新的一行上打印。`COMMAND` 的值后面，我放置了两个换行符（`\n\n`），以确保每个记录之间有一个空白行。

接下来，让我们将这个命令转换成一个可以添加到我在*第十章—理解函数*中展示的 `sysinfo.lib` 函数库的函数。我们新的 `VSZ_info()` 函数如下所示：

```
VSZ_info() {
           echo "<h2>Processes with more than 500000 bytes VSZ
           size.</h2>"
           echo "<pre>"
           ps aux | awk 'NR > 1 && $5 > 500000 {print "USER:" "\t\t" $1 "\n" "PID:" "\t\t" $2 "\n" "VSZ:" "\t\t" $5 "\n" "COMMAND:" "\t" $11 "\n\n"}'
           echo "</pre>"
} 
```

现在，在我在*第十章*展示给你看的 `system_info.sh` 脚本中，你需要将 `$(VSZ_info)` 添加到脚本调用的函数列表的末尾。现在，当你运行脚本时，你会看到输出文件末尾的 `VSZ` 信息。（两个文件太长，无法在这里展示，但你可以从 GitHub 下载它们。）

好的，关于在 shell 脚本中使用`awk`的基础知识就讲到这里。让我们总结一下并继续。

# 总结

在本章中，我向你介绍了使用 `awk` 的神秘技巧。我们从研究 `awk` 的多个实现方式开始，并了解了基本 `awk` 命令的构建方式。接着，我们看到了如何使用 `awk` 处理来自文本文件或其他程序的信息。然后，我们学习了如何在普通的 shell 脚本中运行 `awk` 命令。最后，我们将一个 `awk` 命令转化为函数，并将其添加到我们的函数库文件中。

在下一章中，我们将学习如何创建 `awk` 程序脚本。到时候见。

# 问题

1.  一个 `awk` 命令有哪两个部分？（选择两个。）

    1.  action

    1.  expression

    1.  pattern

    1.  command

    1.  E. 顺序

1.  `{print $0}` 在 `awk` 命令中做什么？

    1.  它打印你正在运行的脚本的名称，因为`$0`是保存脚本名称的 bash 位置参数。

    1.  它打印分配给 `0` 变量的值。

    1.  它打印记录的所有字段。

    1.  这是一个无效的命令，不会执行任何操作。

1.  如果你想对所有包含某个特定文本字符串的行执行精确的全字匹配，以下哪种 `awk` 操作符适合使用？

    1.  `==`

    1.  `=`

    1.  `eq`

    1.  `~`

1.  在哪里定义 `FS` 和 `OFS` 的值最合适？

    1.  在 `awk` 命令的 END 部分。

    1.  不能。它们已经有了预定义的值。

    1.  在 `awk` 命令的 `BEGIN` 部分。

    1.  在 `awk` 命令的动作部分。

1.  如何在 `awk` 中使用正则表达式？

    1.  将正则表达式用一对斜杠括起来。

    1.  将正则表达式用一对单引号括起来。

    1.  将正则表达式用一对双引号括起来。

    1.  不要在正则表达式周围加上任何东西。

# 深入阅读

+   GNU awk 用户指南：[`www.gnu.org/software/gawk/manual/gawk.html`](https://www.gnu.org/software/gawk/manual/gawk.html)

+   《Awk 实例》：[`developer.ibm.com/tutorials/l-awk1/`](https://developer.ibm.com/tutorials/l-awk1/)

+   awklang.org——关于 awk 语言的相关内容网站：[`www.awklang.org/`](http://www.awklang.org/)

+   awk: One True awk：[`github.com/onetrueawk/awk`](https://github.com/onetrueawk/awk)

+   Awk Scripts YouTube 频道：`www.youtube.com/@awkscripts`

+   Miller 文档：[`miller.readthedocs.io/en/latest/`](https://miller.readthedocs.io/en/latest/)

+   《GAWK 手册 - 有用的“一行代码”》：[`web.mit.edu/gnu/doc/html/gawk_7.html`](http://web.mit.edu/gnu/doc/html/gawk_7.html)

+   如何在 Bash 脚本中使用 awk：[`www.cyberciti.biz/faq/bash-scripting-using-awk/`](https://www.cyberciti.biz/faq/bash-scripting-using-awk/)

+   《高级 Bash Shell 脚本指南 - Shell 包装器》：[`www.linuxtopia.org/online_books/advanced_bash_scripting_guide/wrapper.html`](https://www.linuxtopia.org/online_books/advanced_bash_scripting_guide/wrapper.html)

+   Awk：一个 40 年历史语言的力量与前景：[`www.fosslife.org/awk-power-and-promise-40-year-old-language`](https://www.fosslife.org/awk-power-and-promise-40-year-old-language)

# 回答

1.  a 和 c

1.  c

1.  a

1.  c

1.  a

# 加入我们的 Discord 社区！

与其他用户、Linux 专家及作者本人一起阅读本书。

提问、为其他读者提供解决方案、通过“问我任何问题”环节与作者互动，更多精彩内容尽在其中。扫描二维码或访问链接加入社区。

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code10596186092701843.png)

# 留下您的评论！

感谢您从 Packt 出版社购买此书——希望您喜欢它！您的反馈对我们非常宝贵，帮助我们改进和成长。完成阅读后，请花一点时间在亚马逊上留下评价；这仅需一分钟，但对像您这样的读者来说意义重大。

扫描下面的二维码，领取您选择的免费电子书。

[`packt.link/NzOWQ`](https://packt.link/NzOWQ)

![](img/review.png)
