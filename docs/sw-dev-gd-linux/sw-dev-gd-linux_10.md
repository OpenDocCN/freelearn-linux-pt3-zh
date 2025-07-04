# 10

# 配置软件

迟早，每个人都需要在 Linux 系统上配置软件。虽然有许多方法可以做到这一点，但幸运的是，你可以遵循一个通用模式来获得你想要的结果。在 Linux 上，这尤其常见；因为 Linux 中的许多标准工具遵循“小而尖锐的工具”哲学，趋向于大量的小而强大的程序，通过支持广泛的配置提供灵活性。

在本章中，你将了解设计良好的程序通常使用的配置层级结构。无论你是需要查看手册页面找到某个命令的命令行参数，还是想设置一个适用于你在 shell 中运行的所有命令的环境变量，你将看到如何操作。

接下来，我们将向你展示几乎所有 Unix 软件使用的通用配置层级结构，这样你就能始终知道该检查哪里，如果程序的行为与你根据配置期望的不一致。

最后，你将看到这种配置如何转化为通过 `systemd` 管理的程序，`systemd` 是 Linux 上最流行的服务管理工具。

总结来说，本章将涵盖以下主题：

+   配置层级结构

+   命令行参数

+   环境变量

+   配置文件

+   Docker 中的配置

# 配置层级结构

在 Linux 上运行程序时，你首先要做的事情之一就是根据你的具体需求调整它们。事实上，你已经做到了这一点：通过向 `ls`、`grep` 等命令传递参数，你已经改变了这些程序的行为。

你可能对这一过程有直观的理解，因为你一生都在使用软件。例如，你可能会觉得传递命令行参数来覆盖程序默认值是很自然的：`ls -l` 给出的输出不同于 `ls` 的默认输出。

现在让我们更严格地探讨一下这个直觉，并看看能否为在 Unix 环境中配置*一般*工作方式绘制一些启发式的规则。大多数标准 Unix 命令行程序遵循的规范之一是特定的配置层级结构，其中较早的值会被后面的值覆盖。如果你编写过需要用户配置的软件，可能已经创建过类似这样的优先级层级：

1.  将可配置的值设置为内置默认值。

1.  检查通过配置文件传递的值，覆盖这些默认值。

1.  检查环境变量（人们可能会称这些为 `env vars`），覆盖配置文件和早期的值。

1.  检查命令行界面参数（你可能会听到这些被称为 CLI 参数），并根据需要更新值，覆盖早期的值。

每个后续级别都更接近运行软件的用户，因此每个后续级别都会优先于之前的级别。

例如，如果你的软件在配置文件和程序启动时传递的 CLI 参数中检测到冲突的值，它应该优先选择命令行参数中的值。换句话说，**离程序调用更近的值**会“**覆盖**”（在遮蔽或替换的意义上）**离执行更远的值**。CLI 参数值会替代配置文件中的值，因为配置文件离调用点比传递给程序的参数更远。这应该是直观的：如果软件忽略了你的命令行标志，偏向使用程序默认设置，那你是不能依赖这个软件的。`ls -l` 不应该和 `ls` 给你相同的输出。

大多数 Linux 上的软件都遵循这个层次结构，当有多种方式来配置该软件时。请记住，并非所有软件都会使用我们在这里展示的所有配置路径，也并非所有软件都会严格遵循这个配置顺序。

让我们再看一下这个层次结构，这一次将它与具体的**nginx** Web 服务器程序的实际例子联系起来。你可能会在职业生涯中某个时刻使用 nginx，因为它是世界上最流行的 Web 服务器之一，广泛用于动态 Web 应用程序的前端。让我们看看我们刚才讨论的每个优先级层次是如何映射到实际的 nginx 配置的：

1.  **内置默认值**：启动后，`nginx` 执行的用户的硬编码默认值是 `nobody`。

1.  **全局配置文件** 可以为所有 nginx 进程更改这个设置，因此通常会在 `/etc/nginx/nginx.conf` 找到一个全局 nginx 配置文件，其中包含 “`user www;`” 的值，指示 nginx 以 `www` 用户身份运行。

1.  **用户级别配置文件** 通常是“点文件”（以 `.` 开头的文件，这使得它们不会出现在常规 `ls` 列表中），位于用户的 `/home` 目录中。例如，`/home/dave/.bashrc` 是存放用户特定 bash 配置的地方。`nginx` 是一个长期运行的进程，通常不会以常规 Linux 用户身份运行，但它确实有类似的东西：各个站点通常会在它们自己的独立配置文件中配置，通常位于 `/etc/nginx/conf.d/yourwebsite.conf`。这些文件通常会继承来自上一层的全局配置值。

1.  **环境变量**。`nginx` 从名为 `TZ` 的环境变量获取时区信息。

1.  **命令行参数**是在软件运行时指定的，无论是手动运行还是自动运行（例如通过 cron 或单元文件）。在调试问题时，务必查看这些可能的外部命令行参数来源——它们往往是导致程序行为和配置文件不一致的常见原因。`nginx` 接受各种命令行参数来修改其行为：从覆盖它将使用的配置文件，到完全阻止其作为 Web 服务器运行，转而通知已运行的 nginx 进程停止或重新加载。

现在你已经看到了这一配置层级如何与 Linux 上你运行的所有程序及你可能为其编写的所有程序进行交互的理论和实践方面，让我们一步步地深入了解这个配置层级，仔细看看每一层。我们将从最直接、最强大的配置形式开始，它会覆盖所有其他方式：在调用程序的时刻传递命令行参数。

# 命令行参数

你已经熟悉了配置程序的最常见方式：使用命令行参数。这些参数在程序被作为 Shell 命令调用时配置该程序。

要查找程序的有效命令行参数，可以从你命令的 man（手册）页面开始。除了最简化的系统外，Unix 软件通常会随附手册页面，记录大多数程序，解释可用的标志，并且—通常在末尾—列出其他配置方法，比如配置文件。

让我们来看看 `find` 命令的手册页面开头内容：

```
man find
FIND(1)                                                               General Commands Manual                                                              FIND(1)

NAME
     find – walk a file hierarchy

SYNOPSIS
     find [-H | -L | -P] [-EXdsx] [-f path] path ... [expression]
     find [-H | -L | -P] [-EXdsx] -f path [path ...] [expression]

DESCRIPTION
     The find utility recursively descends the directory tree for each path listed, evaluating an expression (composed of the "primaries" and "operands" listed
     below) in terms of each file in the tree.

     The options are as follows:

     -E      Interpret regular expressions followed by -regex and -iregex primaries as extended (modern) regular expressions rather than basic regular expressions
             (BRE's).  The re_format(7) manual page fully describes both formats.

     -H      Cause the file information and file type (see stat(2)) returned for each symbolic link specified on the command line to be those of the file
             referenced by the link, not the link itself.  If the referenced file does not exist, the file information and type will be for the link itself.  File
             information of all symbolic links not on the command line is that of the link itself. 
```

你可以看到，这份手册页面大部分内容都记录了运行`find`时可以使用的各种命令行参数。自从*第一章*以来，你已经使用了大量命令行参数，因此这些内容应该都很熟悉。

让我们来看下一种稍微更远一点的配置方式：环境变量。

# 环境变量

虽然命令行参数非常强大，但它只适用于包含它的单次程序调用。当你输入`ls -l`时，只有这一次`ls`命令会显示长格式输出。但如果你希望某个配置值在多次命令调用中保持有效呢？举个例子，如果你在编写一个脚本，它将在不同的地方安装包，你希望设置一个配置选项*一次*，而不是每次运行安装包命令时都反复添加它作为命令行参数。这时，环境变量就派上用场了。

作为一名开发人员，您可能已经知道环境变量：类似于其他编程语言中的变量，环境变量是 shell 中的值。它们与命令行参数不同，因为它们作用于更高的层级。环境变量为您提供了更多的灵活性：一旦在 shell 中设置了一个配置变量，它就适用于该 shell 会话中的所有程序调用。设置一次后，任何查找该环境变量的程序每次运行时都会遵守它，直到变量改变或您结束 shell 会话。

**注意**

我们将在*第十二章*《使用 Shell 脚本自动化任务》中深入讨论环境变量，但本节涵盖了基本内容。

大多数标准的 Unix 环境使用环境变量作为指定许多不同程序相关常见配置的一种方式，而不仅仅是一个。例如，环境变量可以跟踪用户的 `/home` 目录在哪里（`$HOME`），当前工作目录在哪里（`$PWD`），默认使用哪个 shell（`$SHELL`），在哪里查找与通过命令行接口（CLI）接收到的命令对应的可执行文件（`$PATH`）等等。

现在可以随时检查它们；您可以通过 `echo` 命令打印出来查看特定环境变量的值：

```
$ echo $SHELL
/bin/zsh 
```

或者，您可以使用 `env` 命令查看当前设置的所有环境变量：

```
$ env
...
# many lines of output, one for each of your environment variables ... 
```

要在当前 shell 中设置环境变量，只需使用 `=` 进行赋值（确保等号两侧没有空格）：

```
MYVAR=fruitloops 
```

您已为当前 shell 设置了它：

```
$ echo $MYVAR
fruitloops 
```

要使此变量在您启动的任何子 shell 中持久存在（例如，在运行脚本时），请使用 `export` 内建命令：

```
export MYVAR=fruitloops 
```

您将在*第十二章*《使用 Shell 脚本自动化任务》中学到更多，但上述命令是您需要将环境变量配置传递给大多数与之交互的程序的全部内容。

回到 `find` 示例：如果您向下滚动到 `find` 手册页中的足够远的地方（我们在上一节中查看了该手册页），您会看到一个标题为 `ENVIRONMENT` 的部分：

```
ENVIRONMENT
     The LANG, LC_ALL, LC_COLLATE, LC_CTYPE, LC_MESSAGES and LC_TIME environment variables affect the execution of the find utility as described in environ(7). 
```

这是另一层配置——这些配置指令不是在运行时作为命令参数传递，而是可以从 shell 环境变量中读取。

为什么程序应该将环境变量与命令行参数区别对待？让我们想一想：命令行参数 `–H` 非常具体，因为它是在命令调用层面定义的。因此，它仅适用于当前正在运行的命令。

另一方面，环境变量则不那么具体。它们在 shell 层面上定义，因此可以被从该 shell 运行的所有命令访问。

让我们继续向上走配置层级：如果某个值在运行时没有通过命令行参数或在启动程序的 shell 会话中作为环境变量设置，那么配置来自哪里呢？

# 配置文件

程序寻找配置文件的下一个地方是在其配置文件中。*程序*寻找配置的地方可能会有所不同，但有一些标准的位置是可以查找的。

## 系统级配置在`/etc/`

首先，`/etc/`目录是一个不错的起点。你在*第五章*，*介绍文件*中见过这个目录。`/etc/programname`——其中`programname`是你想配置的程序的名字——是软件存放系统范围配置的常见目录。对于许多程序来说，这已经足够了。例如，nginx Web 服务器是一个系统级程序：不同的用户通常不会在同一台机器上运行自己的 Web 服务器实例，所以只需要一个系统范围的配置。

话虽如此，大型或复杂程序的配置仍然可以在`/etc/programname`目录中拆分。Nginx 就是一个很好的例子；它的主配置文件位于`/etc/nginx.conf`，额外的配置文件则从`/etc/nginx/conf.d/`目录中的附加文件中加载。

## 用户级配置在`~/.config`

对于有显著每用户配置的程序——比如文本编辑器、开发工具、游戏等——`~/.config`目录是用来存放这些配置的。回想一下*第一章*，*命令行的工作原理*，`~`是“当前用户的主目录”的简写，而以点字符（“`.`”）开头的目录，除非传递`-a`标志，否则在`ls`输出中会被省略。`~/.config`目录是 XDG 基础目录标准的一部分，你可以在这里获取概述：[`wiki.archlinux.org/title/XDG_Base_Directory`](https://wiki.archlinux.org/title/XDG_Base_Directory)。

作为一个例子，我的`neovim`配置与其他开发者的配置有显著不同，但系统上的单个 neovim 二进制文件可以支持数百名开发者在同一台机器上同时工作，因为每个开发者启动 neovim 时都会使用保存在`~/.config/nvim/`中的用户特定配置文件。这很好！

你可以想象，如果只有一个系统范围的地方来配置这个程序在`/etc/`目录下会引发怎样的混乱——每个开发者都必须在运行 neovim 编辑器之前设置无数的环境变量，或者通过无数的命令行标志来调用编辑器命令。

现在你已经了解了 Unix 程序的经典配置来源，让我们来看一下一个 Linux 特有的复杂问题：如何管理通过环境文件和 CLI 参数配置的程序，这些程序通过`systemd`进行控制。

# systemd 单元

在大多数 Linux 发行版中——除了 Docker 容器——`systemd`负责管理所有服务。我们已经在本书中讨论了`systemd`的基础知识（参见*第三章*《使用 systemd 管理服务》），在这一节中，我们将快速了解`systemd`如何管理程序的配置。

首先，快速回顾一下，以防*第三章*看起来有些遥远：在一个由 systemd 管理的 Linux 环境中，服务被打包进`systemd`单元文件中，这些文件封装并控制实际的可执行二进制文件、其参数、启动、重启和停止单元的命令等。

如我们已经讨论过，systemd 单元类型有很多种，但我们在这里关注的是`service`单元类型。

我们已经讨论过，单元文件可以存在于多个不同的目录中，具体取决于它们的用途，但你自己的自定义 systemd 单元通常会存放在`/etc/systemd/system`中。

为了理解 systemd 单元如何影响我们在本章中讨论的配置层次结构，让我们通过编写一个名为`yourprogram`的虚拟程序的 systemd 单元，来创建一个由 systemd 管理的服务。

## 创建你自己的服务

作为开发者，你可能需要将你编写的程序包装成一个服务，这样比手动（交互式）调用程序更容易管理。这本身是非常有用的，但在本章中，我们深入探讨了 systemd 单元为你提供的额外控制，如何以及在哪里配置你的程序。让我们通过将二进制文件与`systemd`单元文件结合，来走一遍创建服务的过程。

首先，确保你已经将可执行文件复制到一个默认的 `$PATH` 目录中：`/usr/local/bin/yourprogram`。为了最大化效果，可以使用一个手动编译的程序，如前面*第九章*《管理已安装软件》中创建的`htop`二进制文件，并将假设的`yourprogram`替换为`htop`。

现在，在`/etc/systemd/system/yourprogram.service`路径下创建以下`systemd`单元文件：

```
[Unit]
Description=Your program description.
After=network-online.target
[Service]
Type=exec
ExecStart=/usr/local/bin/yourprogram -clioption=1 –clioption2
EnvironmentFile=-/etc/yourprogram/prod_defaults
Restart=on-failure
[Install]
WantedBy=multi-user.target 
```

你能在这个单元文件中找到与配置相关的两行吗？

你可以看到，`ExecStart`行指定了当有人启动这个 systemd 服务时，程序是如何被调用的。我们使用 systemd 单元文件将命令行参数传递给程序，确保每次有人启动服务时，程序都会以我们希望的选项运行。每当有人运行`systemctl start yourprogram`时，我们已确保`yourprogram`会被调用，带有`-clioption=1`和`-clioption2`。

其次，`EnvironmentFile`行指定了`systemd`检查的文件路径，在该文件中，它可以找到与此程序相关的环境变量。这些变量将由`systemd`用来运行二进制文件的 shell 解析，它应包含类似于以下的 shell 变量赋值：

```
# yourprogram environment variables
ENV=production
DB_HOST=localhost
DB_PORT=5432 
```

让 systemd 重新读取它的配置文件，确保它能看到你定义的新的服务`Unit`：

```
$ sudo systemd daemon-reload 
```

现在，你可以像管理任何其他`systemd`服务一样管理这个服务：

`systemctl start yourprogram`

`systemctl status yourprogram`

`systemctl stop yourprogram`

`systemctl enable yourprogram`

`systemctl disable yourprogram`

您知道每次启动该服务时，位于`/etc/yourprogram/prod_defaults`的环境文件将用于源环境变量，而`ExecStart`行将传递您指定的命令行选项。

我们在这里展示了一个非常简单的服务，旨在帮助您理解如何使用 systemd 来控制程序配置，但这里有*许多*其他配置指令可以传递。如果您手头有更复杂的服务，花些时间阅读 `systemd` 单元文档（[`www.freedesktop.org/software/systemd/man/latest/systemd.unit.html#%5BUnit%5D%20Section%20Options`](https://www.freedesktop.org/software/systemd/man/latest/systemd.unit.html#%5BUnit%5D%20Section%20Options)）。

# 快速提示：Docker 中的配置

本章早些时候提到过，Docker 在配置方面通常是一个例外。因为 Docker 容器是一个更为简化的环境，它们没有传统 Unix 系统中那种大量的额外二进制文件、服务和配置文件。但由于现在许多软件开发者创建的软件运行在容器中，而不是传统的完整操作系统环境中，我们希望在这里介绍一些基础知识，确保您对 Docker 容器中的配置有基本的直觉。我们将在*第十五章*《使用 Docker 容器化应用程序》中深入探讨 Docker 容器。

在容器环境中——无论是 Docker 还是其他容器运行时——您所面临的环境要小得多。安装的程序和工具非常少，`init`被极大简化为代替`systemd`，文件系统也被大大缩减，没有我们在这里提到的许多目录。

配置层级的原则依然适用。大多数容器化应用程序期望从以下位置获取配置：

+   容器文件系统上的某个配置文件，通常是容器调度器在容器启动之前动态创建的

+   环境变量，由容器调度器或启动它的操作员传递

+   命令行参数

即使这是一个简化版的配置层级结构，您会注意到它基本上与我们在传统非容器 Linux 系统中探索的结构完全相同。

我们将在*第十五章*《使用 Docker 容器化应用程序》中更深入地探讨容器。

# 结论

本章为您概述了 Linux 配置层级结构以及它如何应用于您每天使用（和编写）的程序。您了解了命令行参数、环境变量，以及所有其他在更大层级结构中为程序提供配置的内容。

如果你跟着做，你甚至创建了一个 systemd 服务来包装一个程序，并允许你以更统一的方式管理其配置。

# 在 Discord 上了解更多

要加入本书的 Discord 社区——在这里你可以分享反馈，向作者提问，并了解新版本——请扫描下方的二维码：

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code1768422420210094187.png)
