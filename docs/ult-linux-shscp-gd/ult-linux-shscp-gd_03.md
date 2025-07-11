

# 第三章：理解变量和管道

在前一章中，您看到了 shell 如何解释用户的命令，并看到了如何编写您自己的命令的各种示例。在本章中，我将告诉您有关变量和管道的内容。

创建变量并为其分配值的能力是任何编程环境的重要组成部分。正如您期望的那样，`bash`和`zsh`都具备这种能力。在本章的第一部分，我们将介绍有关环境变量和编程变量的基础知识。

在本章的第二部分，我们将介绍如何使用管道。管道非常简单，您可能已经在某个时候使用过它们。因此，我保证会简短而言之地介绍这个内容。（实际上，关于这两个主题，现在还没有太多需要讲的，这就是为什么我将它们合并到一个章节中。）

本章涵盖的主题包括：

+   理解环境变量

+   理解编程变量

+   理解管道

如果您准备好了，让我们开始吧。

# 理解环境变量

**环境变量** 控制操作系统 shell 的配置和功能。当您安装 Linux 或类 Unix 操作系统（如 FreeBSD 或 OpenIndiana）时，您会发现在全局和用户级别已经定义了一组默认的环境变量。

要查看环境变量及其设置的列表，请使用`env`命令，就像这样：

```
[donnie@fedora ~]$ env
SHELL=/bin/bash
IMSETTINGS_INTEGRATE_DESKTOP=yes
COLORTERM=truecolor
XDG_CONFIG_DIRS=/etc/xdg/lxsession:/etc/xdg
HISTCONTROL=ignoredups
. . .
. . .
MAIL=/var/spool/mail/donnie
OLDPWD=/etc/profile.d
_=/bin/env
[donnie@fedora ~]$ 
```

环境变量的完整列表非常广泛。幸运的是，您不需要记住每个项的作用。您所需了解的大多数变量都是不言自明的。

而不是查看整个列表，您还可以查看特定项的值。只需使用`echo`命令，并在变量名之前加上`$`符号，就像这样：

```
[donnie@fedora ~]$ echo $USER
donnie
[donnie@fedora ~]$ echo $PATH
/home/donnie/.local/bin:/home/donnie/bin:/home/donnie/.cargo/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin
[donnie@fedora ~]$ echo $EDITOR
/usr/bin/nano
[donnie@fedora ~]$ 
```

在这里，我们可以看到我（`donnie`）是当前登录用户，我的路径设置是什么，以及我的默认编辑器是什么。您可以以同样的方式查看任何其他环境变量的值。

一个重要的注意事项是，所有环境变量的名称始终由全部大写字母组成。操作系统或 shell 中没有任何东西阻止使用小写字母，但不使用它们有一个非常好的理由。变量名称是区分大小写的。最佳实践是仅使用大写字母命名环境变量，对于编程变量名称则可以使用全部小写字母或大小写混合。这将防止您意外地覆盖环境变量的值。（我将在下一节中展示更多相关内容。）

如我之前提到的，环境变量可以在全局和用户级别进行配置。全局级别的变量设置会影响所有 `bash` 和 `zsh` 用户。对于 `bash`，你会在 `/etc/profile` 文件、`/etc/bashrc` 文件以及 `/etc/profile.d/` 目录中的各种文件中找到这些全局设置。对于 `zsh`，你会在 `/etc/zprofile`、`/etc/zshrc` 和 `/etc/zshenv` 文件中找到这些设置。（请注意，`zsh` 也会引用与 `bash` 相同的 `/etc/profile` 文件。）如果你此时打开其中一个文件，你可能并不理解它们的内容。没关系，因为现在这并不重要。但你会很容易找到环境变量的设置位置，因为变量名都是大写字母。

现在，假设你不喜欢某个特定的设置。举个例子，假设你想自定义命令行提示符以符合自己的喜好。在我的 Fedora 工作站上，我的 `bash` 提示符是这样的：

```
[donnie@fedora ~]$ 
```

提示符的格式由 `PS1` 环境变量决定。我们可以像这样查看 `PS1` 的设置：

```
[donnie@fedora ~]$ echo $PS1
[\u@\h \W]\$
[donnie@fedora ~]$ 
```

这是你刚刚看到的内容的解析：

+   `[`: 这是一个字面字符，它是我们在提示符中看到的第一个字符。

+   `\u`：这会使当前用户的用户名显示出来。

+   `@`：这是另一个字面字符。

+   `\h`：这会使机器的主机名的第一个组件显示出来。

+   `\W`：这会显示当前工作目录的名称。请注意，大写的 W 不会显示整个路径名。

+   `]`：这是另一个字面字符。

+   `\$`：这会使所有普通用户看到 `$`，而根用户看到 `#`。

不久前，我说过我们可以使用 `\` 强制 shell 将元字符解释为字面字符。但在这里，我们看到了 `\` 的另一种用法。当配置 `PS1` 参数时，`\` 表示我们将要使用一个**宏**命令。（可以将宏看作是当你执行某个简单操作时，例如按下特定的键或点击特定的按钮时会运行的命令。）

现在，假设我们想让当前工作目录的整个路径显示出来，同时显示当前的日期和时间。为此，我们将 `\W` 替换为 `\w`，并添加 `\d` 和 `\t` 宏，像这样：

```
[donnie@fedora ~]$ export PS1="[\d \t \u@\h \w]\$"
[Wed Aug 09 18:14:26 donnie@fedora ~]$ 
```

请注意，我必须将新参数用一对引号括起来，以便 shell 能正确解释元字符。同时，注意当我执行`cd`进入下一级目录时会发生什么：

```
[Wed Aug 09 18:14:26 donnie@fedora ~]$cd /etc/profile.d/
[Wed Aug 09 18:29:15 donnie@fedora /etc/profile.d]$ 
```

将`/w`替换为`/W`会显示当前工作目录的整个路径。

当你从命令行配置`PS1`参数时，新的设置将在你注销机器或关闭终端窗口后消失。为了使设置永久生效，只需编辑位于主目录中的`.bashrc`文件。在文件末尾添加`export PS1="[\d \t \u@\h \w]$ "`这一行，下一次你登录机器或打开一个新终端窗口时，就会看到新的提示符。

还有很多其他方式可以自定义命令提示符，我没有一一展示。有关更完整的列表，请查看我在*进一步阅读*部分提供的参考资料。还要注意，我这里只展示了如何使用`bash`进行设置，因为`zsh`使用不同的命令提示符参数。（在*第二十二章，使用 Z Shell*中我会详细介绍这部分内容。）

我能读懂你的心思，看到你在想环境变量与 Shell 脚本有什么关系。其实，有时候你需要让脚本执行一个依赖于特定环境变量值的特定操作。例如，假设你只希望脚本在 root 用户下运行，而不是在任何没有特权的用户下运行。由于我们知道 root 用户的用户标识号是`0`，我们可以编写代码，允许脚本在`UID`变量设置为`0`时运行，而当`UID`设置为非`0`时则阻止脚本运行。

顺便说一句，如果我能读懂你的心思而让你觉得有些不自在，我为此道歉。

这就是我们对环境变量的介绍。现在让我们快速看看编程变量。

# 理解编程变量

有时候，定义变量在脚本中是必要的。你可以根据需要定义、查看和取消设置这些变量。请注意，尽管系统允许你使用全大写字母来创建编程变量名，但这被认为是一种不太好的做法。最佳实践是始终使用小写字母来命名编程变量，这样你就不会冒险用相同名字覆盖环境变量的值。（当然，覆盖环境变量不会造成长期损害。但是，为什么要冒险覆盖一个你以后可能需要在脚本中使用的环境变量呢？）

为了展示这些内容是如何工作的，让我们从命令行创建一些编程变量，并查看它们的值。首先，我们将创建`car`变量，并将其值设置为`Ford`，如下面所示：

```
[donnie@fedora ~]$ car=Ford
[donnie@fedora ~]$ echo $car
Ford
[donnie@fedora ~]$ 
```

要查看变量的值，使用`echo`，并在变量名之前加上`$`，就像我们对环境变量所做的那样。现在，让我们使用`bash`命令打开一个子 shell，看看我们是否还能查看到这个`car`变量的值，然后退出回到父 shell，如下所示：

```
[donnie@fedora ~]$ bash
[donnie@fedora ~]$ echo $car
[donnie@fedora ~]$ exit
exit
[donnie@fedora ~]$ 
```

这次我们看不到`car`的值，因为我们没有**导出**这个变量。导出变量将允许子 Shell 访问该变量。正如你可能猜到的，我们将使用`export`命令来完成这个操作，像这样：

```
[donnie@fedora ~]$ export car=Ford
[donnie@fedora ~]$ echo $car
Ford
[donnie@fedora ~]$ bash
[donnie@fedora ~]$ echo $car
Ford
[donnie@fedora ~]$ exit
exit
[donnie@fedora ~]$ 
```

这次，我们看到`car`的值现在在子 Shell 中显示了出来。

我们将在本书的余下部分中使用变量，所以你将学习更多关于如何使用它们的内容。不过目前，快速介绍就足够了。

接下来，让我们做一些管道操作。

# 理解管道

**管道**会将一个命令的输出作为另一个命令的输入。它由`|`符号表示，这个符号与反斜杠在同一个键上。你通常会从命令行调用一个简单的**管道**进行各种目的。但你也可以为 Shell 脚本创建非常复杂的多阶段管道。

![](img/B21693_03_01.png)

图 3.1：创建管道。请注意，stdout 是标准输出的简称，而 stdin 是标准输入的简称。

为了看看这如何有用，假设你想查看某个目录中所有文件的列表。但是文件太多，输出会滚动出屏幕，你根本看不到它。你可以通过将`ls`命令的输出作为`less`命令的输入来解决这个问题。它看起来像这样：

```
[donnie@fedora ~]$ ls -l | less 
```

`ls -l`的列表将会在`less`分页器中打开，你可以滚动输出或搜索特定的文本字符串。

现在，假设我只想看到文件名中包含文本字符串`alma`的文件。这很简单。我只需将`ls -l`的输出管道到`grep`中，像这样：

```
[donnie@fedora ~]$ ls -l | grep alma
-rw-r--r--.  1 donnie donnie        1438 Nov  2  2022 alma9_default.txt
-rw-r--r--.  1 donnie donnie        1297 Nov  2  2022 alma9_future.txt
-rw-r--r--.  1 donnie donnie          81 Jan 11  2023 alma_link.txt
[donnie@fedora ~]$ 
```

现在，假设我不想看到文件名，但我确实想知道有多少个文件。我只需再添加一个管道阶段，像这样：

```
[donnie@fedora ~]$ ls -l | grep alma | wc -l
3
[donnie@fedora ~]$ 
```

`wc -l`命令计算输出中的行数，在这个例子中，它告诉我们在文件名中包含文本字符串`alma`的文件数量。

如果你不熟悉`grep`，目前只需理解它是一个可以搜索特定文本字符串或文本模式的工具。它可以在不打开文件的情况下搜索文本文件，或者它可以搜索从另一个命令的输出管道到它中的文本字符串或模式。

好吧，我已经向你展示了一些如何从命令行创建管道的简单例子。现在，我想向你展示一些我希望你*永远*不要做的事情。这涉及到使用`cat`工具。

使用`cat`你可以将文本文件的内容输出到屏幕上。这主要用于查看小文件，正如你在这里看到的：

```
[donnie@fedora ~]$ cat somefile.txt
This is just some file that I created to demonstrate cat.
[donnie@fedora ~]$ 
```

如果你使用`cat`查看一个大文件，输出将滚动出屏幕，你可能无法查看它。我见过有人仍然使用`cat`来转储文件，然后将输出管道到`less`，像这样：

```
[donnie@fedora ~]$ cat files.txt | less 
```

我也看到过有人将 `cat` 的输出通过管道传递给 `grep` 来查找文本字符串，像这样：

```
[donnie@fedora ~]$ cat files.txt | grep darkfi
drwxr-xr-x.  1 donnie donnie         414 Apr  3 12:41 darkfi
[donnie@fedora ~]$ 
```

这两个例子都可以工作，但直接使用 `less` 或 `grep` 而不通过 `cat`，会少输入一些内容，像这样：

```
[donnie@fedora ~]$ less files.txt
. . .
. . .
[donnie@fedora ~]$ grep darkfi files.txt
drwxr-xr-x.  1 donnie donnie         414 Apr  3 12:41 darkfi
[donnie@fedora ~]$ 
```

换句话说，尽管将 `cat` 的输出通过管道传递给其他工具是可行的，但它比直接使用其他工具效率低，可能会让你的脚本变得不那么高效，也更难阅读。而且，作为一个爱猫的人，每次看到有人用 `cat` 进行滥用时，我总是感到很烦。

我展示的只是管道操作的冰山一角。Linux 和 Unix 管理员需要创建 shell 脚本的一个主要原因是自动化从文本文件或某些程序输出中提取和格式化信息。这通常需要非常长、复杂的、多阶段的管道，每个阶段都有不同的提取或格式化工具。在你能开始做这些之前，你需要学习如何使用这些工具，你将在 *第六章，文本流过滤器 - 第一部分* 中开始学习。而在我们进入那部分之前，还需要再介绍一些 `bash` 和 `zsh` 的基础功能。

# 总结

尽管这一章比较简短，但我们已经覆盖了许多重要的信息。我们首先查看了环境变量，展示了如何修改其中一个变量，并解释了在 shell 脚本中为何需要使用环境变量。接着，我们讲解了编程变量，说明了如何创建和导出它们。最后，我们做了一些管道操作。

在下一章，我们将介绍输入/输出重定向的概念。我们下次见。

# 问题

1.  以下哪些元字符允许你查看变量的赋值？

    1.  `*`

    1.  `%`

    1.  `$`

    1.  `^`

1.  以下哪项陈述是正确的？

    1.  使用全大写或全小写字母来创建编程变量名都是完全正确的。

    1.  变量名不区分大小写。

    1.  创建编程变量名时，应始终使用大写字母。

    1.  创建编程变量名时，应始终使用小写字母。

1.  如何使子 shell 识别你在父 shell 中创建的变量？

    1.  在子 shell 中，从父 shell 导入变量。

    1.  在父 shell 中，创建变量时使用 `export` 关键字。

    1.  你不能这样做，因为父 shell 和子 shell 是相互隔离的。

    1.  子 shell 和父 shell 默认会共享变量。

1.  你会使用以下哪些元字符将一个命令的输出传递给另一个命令作为输入？

    1.  `$`

    1.  `|`

    1.  `&`

    1.  `%`

# 深入阅读

+   如何在 Linux 中更改/设置 bash 自定义提示符（PS1）：[`www.cyberciti.biz/tips/howto-linux-unix-bash-shell-setup-prompt.html`](https://www.cyberciti.biz/tips/howto-linux-unix-bash-shell-setup-prompt.html)

+   如何计算 Linux 中目录下的文件数？：[`www.linuxjournal.com/content/how-count-files-directory-linux`](https://www.linuxjournal.com/content/how-count-files-directory-linux)

# 答案

1.  c

1.  d

1.  b

1.  b

# 加入我们的 Discord 社区！

与其他用户、Linux 专家以及作者本人一同阅读本书。

提问、为其他读者提供解决方案、通过“问我任何问题”环节与作者聊天，还有更多内容等你参与。扫描二维码或访问链接加入社区。

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code10596186092701843.png)

# 留下评论！

感谢您从 Packt 出版社购买本书—我们希望您喜欢它！您的反馈对我们至关重要，能帮助我们改进和成长。阅读完本书后，请花一点时间在 Amazon 上留下评论；这只需一分钟，但对像您一样的读者帮助巨大。

扫描下面的二维码，获得你选择的免费电子书。

[`packt.link/NzOWQ`](https://packt.link/NzOWQ)

![](img/review.png)
