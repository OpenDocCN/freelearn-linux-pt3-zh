

# 使用 Shell 脚本选项与`getopts`

通常，管理员需要同时向 shell 脚本传递参数和选项。通过向脚本传递位置参数，这一点在前几章中已经讲解过。 但是，如果你需要使用正常的 Linux/Unix 风格的选项开关，并且需要为某些选项使用参数，那么你就需要一个辅助程序。在本章中，我将向你展示如何使用`getopts`将选项、参数以及带参数的选项传递给脚本。

本章的主题包括：

+   理解使用`getopts`的必要性

+   理解`getopt`与`getopts`的区别

+   使用`getopts`

+   查看实际的例子

所以，如果你准备好了，我们就开始吧。

# 技术要求

你可以使用你的 Fedora 或 Debian 虚拟机来完成本章内容。而且，像往常一样，你可以通过运行以下命令来获取脚本：

```
git clone https://github.com/PacktPublishing/The-Ultimate-Linux-Shell-Scripting-Guide.git 
```

# 理解`getopts`的必要性

在我们做任何事情之前，先复习一下**选项**和**参数**之间的区别。

+   选项修改程序或脚本的行为。

+   参数是程序或脚本将作用于的对象。

最简单的方式是使用简单的`ls`命令。如果你想查看某个特定文件是否存在，只需像这样使用`ls`和文件名作为参数：

```
donnie@fedora:~$ ls coingecko.sh
coingecko.sh
donnie@fedora:~$ 
```

如果你想查看文件的详细信息，你需要添加一个选项，像这样：

```
donnie@fedora:~$ ls -l coingecko.sh
-rwxr--r--. 1 donnie donnie 842 Jan 12 13:11 coingecko.sh
donnie@fedora:~$ 
```

你可以看到`-l`选项如何修改`ls`命令的行为。

你已经见过如何使用普通的位置参数将选项和参数传递给脚本。很多时候，这种方式足够好了，你不需要其他更多的东西。然而，如果没有某种辅助工具，创建能够正确解析由单个短横线组合的单字母选项，或具有带参数选项的脚本将变得非常笨重。这就是`getopts`的作用。你可以在脚本中使用它来简化这一过程。不过，在我们讨论这个之前，我想快速介绍一些可能会让你感到困惑的内容。

# 理解`getopt`与`getopts`的区别

`getopt`工具已经存在很久了，从 Unix 的早期就有了。它的一个大优点是可以处理长选项。换句话说，除了可以向它传递单字母选项，例如`-a`或`-b`，你还可以向它传递完整的单词选项，例如`--alpha`或`--beta`。就这些，这就是它的唯一优点。

它也有一些缺点。`getopt`的原始实现无法处理包含空格的参数文本。例如，如果你需要处理一个文件名中包含空格的文件，你无法将该文件名作为参数传递给使用原始`getopt`的 shell 脚本。另外，`getopt`的语法比`getopts`稍微复杂，这使得`getopts`的使用相对更简单。

在某个时间点，我不确定是何时，一些随机的 Linux 开发者决定创建一个新的`getopt`实现，以修复其中一些缺陷。（这被称为**util-linux**实现。）这听起来很棒，但它仍然*只有*在 Linux 上。所有的 Unix 和类 Unix 操作系统仍然拥有原始的、未增强的`getopt`。因此，如果您在 Linux 机器上创建一个使用增强`getopt`的脚本，您将无法在任何 Unix 或类 Unix 操作系统上运行它，比如 FreeBSD 或 OpenIndiana。

较新的`getopts`命令是一个内置的 Shell 命令，没有自己的可执行文件，可以处理包含空格的参数。但是，它只能处理单字母选项，不能处理长选项。最大的优势是它在 Linux、Unix 和类 Unix 操作系统上具有可移植性，无论使用哪种 Shell。因此，如果可移植性是您的目标之一，最好拒绝`getopt`，选择`getopts`。

因此，我在本章节只讨论`getopts`，不再提及老旧的`getopt`。

说到这里，让我们深入探讨问题的核心。

# 使用 getopts

我们将从`getopts-demo1.sh`脚本开始，这是世界上最简单的`getopts`脚本。它的样子如下：

```
#!/bin/bash
while getopts ab options; do
        case $options in
                a) echo "You chose Option A";;
                b) echo "You chose Option B";;
        esac
done 
```

我们正在使用`while`循环将所选选项或选项传递给`case`语句。在`getopts`命令后面的`ab`意味着我们正在定义`a`和`b`作为可选项。 （这个可选项列表称为**optstring**。）您可以将任何字母数字字符用作选项。也可以使用某些其他字符，但不建议这样做。由于稍后将变得清晰，绝对*不能*将`?`或`:`用作选项。在`ab`之后，您会看到`options`，这只是我们将在`case`结构中使用的变量名称。 （您可以随意命名变量。）无论如何，运行脚本看起来像这样：

```
donnie@fedora:~$ ./getopts-demo1.sh -a
You chose Option A
donnie@fedora:~$ ./getopts-demo1.sh -b
You chose Option B
donnie@fedora:~$ 
```

好吧，*大不了*，你可能会说。我们可以用普通的位置参数做这种事情。啊，但这是普通位置参数做不到的一件事：

```
donnie@fedora:~$ ./getopts-demo1.sh -ab
You chose Option A
You chose Option B
donnie@fedora:~$ 
```

因此，您看到`getopts`如何允许您将选项与单破折号组合，就像您习惯于在普通 Linux 和 Unix 实用程序中做的那样。另一件普通位置参数做不到的事情，至少不容易做到的事情，是有需要自己参数的选项。这里是展示如何做到这一点的`getopts-demo2.sh`脚本：

```
#!/bin/bash
while getopts a:b: options; do
        case $options in
                a) var1=$OPTARG;;
                b) var2=$OPTARG;;
        esac
done
echo "Option A is $var1 and Option B is $var2" 
```

当你在 optstring 中为允许的选项后加上冒号时，运行脚本时需要为该选项提供一个参数。每个选项的参数存储在 `OPTARG` 中，这是一个内建变量，和 `getopts` 一起使用。（请注意，该变量名只能由大写字母组成。）`while` 循环会针对每个使用的选项执行一次，这意味着同一个 `OPTARG` 变量可以用于多个选项参数。为了好玩，我们试着将我的名字作为 `-a` 选项的参数，将我的姓氏作为 `-b` 选项的参数。以下是结果：

```
donnie@fedora:~$ ./getopts-demo2.sh -a Donnie -b Tevault
Option A is Donnie and Option B is Tevault
donnie@fedora:~$ 
```

目前，如果你在运行脚本时使用了无效的选项，shell 会返回自己的错误信息，类似这样：

```
donnie@fedora:~$ ./getopts-demo2.sh -x
./getopts-demo2.sh: illegal option -- x
Option A is  and Option B is
donnie@fedora:~$ 
```

在这种情况下，`./getopts-demo2.sh: illegal option -- x` 这一行是由 `bash` 生成的错误信息。你可以通过在 optstring 开头再加一个冒号来抑制该错误信息。可选地，你可以在 `case` 结构中添加一个 `\?` 选项，用来自定义显示无效选择的错误信息，并使用 `:` 选项来提醒你如果尝试在没有提供所需选项参数的情况下运行脚本。另一个问题是，如果我没有提供所需的选项参数，最终的 `echo` 命令仍然会执行，而它实际上不应该执行。无论如何，以下是所有修复后的 `getopts-demo3.sh` 脚本：

```
#!/bin/bash
while getopts :a:b: options; do
        case $options in
                a) var1=$OPTARG;;
                b) var2=$OPTARG;;
                \?) echo "I don't know the $OPTARG option";;
                :) echo "Both options require an argument.";;
        esac
done
[[ -z $var1 ]] || echo "Option A is $var1"
[[ -z $var2 ]] || echo "Option B is $var2" 
```

当我使用所有必要的参数运行脚本时，结果是这样的：

```
donnie@fedora:~$ ./getopts-demo3.sh -a Donnie -b Tevault
Option A is Donnie
Option B is Tevault
donnie@fedora:~$ 
```

如果我没有使用所有选项，或者在使用选项时没有提供参数，情况是这样的：

```
donnie@fedora:~$ ./getopts-demo3.sh -a Donnie
Option A is Donnie
donnie@fedora:~$ ./getopts-demo3.sh -a
Both options require an argument.
donnie@fedora:~$ ./getopts-demo3.sh -a Donnie -b
Both options require an argument.
Option A is Donnie
donnie@fedora:~$ 
```

到目前为止，我们做的工作对需要参数的选项来说非常完美。在 `getopts-demo4.sh` 脚本中，我将介绍两个新概念。你将看到如何为某些选项要求参数而不为其他选项要求参数，以及如何使用非选项参数。以下是示例：

```
#!/bin/bash
while getopts :a:b:c options; do
        case $options in
                a) var1=$OPTARG;;
                b) var2=$OPTARG;;
                c) echo "This option doesn't require an argument.";;
                \?) echo "I don't know the $OPTARG option";;
                :) echo "Both options require an argument.";;
        esac
done
[[ -z $var1 ]] || echo "Option A is $var1"
[[ -z $var2 ]] || echo "Option B is $var2"
shift $((OPTIND-1))
[[ -z $1 ]] || ls -l "$1" 
```

`:a:b:c` 的 optstring，在 `c` 后没有冒号，表示 `-a` 和 `-b` 选项需要参数，但 `-c` 选项不需要。如果提供了作为非选项参数的文件名，底部的最后一行将对这些文件执行 `ls -l` 命令。（请注意，`[[ -z $@ ]]` 结构如果没有非选项参数时将返回退出码 0。如果有非选项参数，则返回退出码 1，这将触发 `ls -l` 命令。）不过，诀窍在于：

当你运行这个脚本时，首先需要指定选项和选项参数。然后，在命令的最后指定将作为非选项参数的文件名。或者，如果你愿意，也可以只指定一个非选项参数，而不使用任何选项。`shift $((OPTIND-1))` 这一行是必须的，以便脚本能识别非选项参数。

`OPTIND`是另一个内置的变量，用来存储已处理的`getopts`选项的数量。理解`shift $((OPTIND-1))`命令的最简单方式是把它看作一个清理机制。换句话说，它会清除已经使用的所有`getopts`选项，为非选项参数腾出空间。另一种理解方式是它会将位置参数计数重置回`$1`。这样，你就可以像平常一样访问非选项参数，如最后一行所示。此外，注意最后一行中的`ls -l "$1"`命令。这是一个必须*用双引号*括住位置参数（在此情况下是`$1`）的场景。否则，如果你试图访问文件名中有空格的文件时，会收到错误消息。最后，注意你可以列出多个位置参数，这样你就可以有多个参数，就像你以前做的那样。或者，你也可以用`"$@"`替代`"$1"`，这样你就可以使用无限数量的非选项参数。

所以，在没有选项，只有一个非选项参数的情况下运行这个脚本，效果如下：

```
donnie@fedora:~$ ./getopts-demo4.sh "my new file.txt"
-rw-r--r--. 1 donnie donnie 0 Feb 25 13:00 'my new file.txt'
donnie@fedora:~$ 
```

如果我去掉`ls -l`命令中位置参数周围的双引号，当我尝试访问文件名中包含空格的文件时，就会收到错误。下面是这样做的效果：

```
donnie@fedora:~$ ./getopts-demo4.sh "my new file.txt"
ls: cannot access 'my': No such file or directory
ls: cannot access 'new': No such file or directory
ls: cannot access 'file.txt': No such file or directory
donnie@fedora:~$ 
```

所以，记得使用双引号。

当我告诉你使用`getopt`的缺点时，我提到过`getopt`的语法比`getopts`更复杂。如果我使用`getopt`编写这些脚本，我需要在`case`结构中的每个项后面添加一个或多个`shift`命令，以便脚本能够正确识别选项。而使用`getopts`时，这个移动操作是自动完成的。我唯一需要使用`shift`的地方就是在最后的`shift $((OPTIND-1))`命令中。所以你可以看到，`getopts`要简单得多。

此外，如果你想了解更多关于如何在`getopts`中使用`OPTIND`的内容，我已经在*进一步阅读*部分中放置了一些参考资料。

事实上，这就是`getopts`的基本用法。所以，让我们继续往下看。

# 查看真实世界的例子

现在我已经展示了如何使用`getopts`的理论，是时候通过一些真实世界的例子开始实际操作了。享受吧！

## 修改版 Coingecko 脚本

在*第十章，理解函数*中，我展示了一对我为自己编写的很酷的脚本。回顾一下，这些脚本使用 Coingecko API 来自动获取有关加密货币的信息。

`coingecko.sh`脚本使用了一个`if. .elif. .else`结构，允许我选择要执行的功能。在`coingecko-case.sh`脚本中，我使用了`case. .esac`结构来实现相同的功能。现在，我展示了这个脚本的第三个版本，它使用了`getopts`。

`coingecko-getopts.sh`脚本太大，无法在此处完整展示，所以你需要从 GitHub 获取它。我确实需要指出其中的一些要点，所以在这里我只展示相关的代码片段。

在脚本的开头，我添加了`gecko_usage()`函数，它看起来是这样的：

```
gecko_usage() {
        echo "Usage:"
        echo "To create a list of coins, do:"
        echo "./coingecko.sh -l"
        echo
        echo "To create a list of reference currencies, do:"
        echo "./coingecko.sh -c"
        echo
        echo "To get current coin prices, do:"
        echo "./coingecko.sh coin_name(s) currency"
} 
```

接下来，我删除了`gecko_api()`函数内的`coin=$1`和`currency=$2`变量定义，因为我们不再需要它们。

然后，我在现有的`case`结构中添加了一个错误检查选项，修改了**Usage**选项以调用`gecko_usage()`函数，然后用一个`while`循环将其包裹起来，像这样：

```
while getopts :hlc options; do
        case $options in
                h)
                        gecko_usage
l)
        curl -X 'GET' \
        "https://api.coingecko.com/api/v3/coins/list?include_platform=true" \
         -H 'accept: application/json' | jq | tee coinlist.txt
         ;;
. . .
. . .
 \?) echo "I don't know this $OPTARG option"
                        ;;
                esac
                done 
```

记得安装`jq`包，就像我在*第十章*中展示的那样。

起初，我也有一个`p:`选项来获取当前的币种价格。但它没有工作，因为这个命令需要两个参数，即以逗号分隔的币种列表和参考货币。然而，`getopts`只允许每个选项使用一个参数。为了解决这个问题，我将币种价格命令移出了`gecko_api()`函数，并将其放在文件的末尾，如下所示：

```
gecko_api $1
shift $((OPTIND-1))
[[ -n $1 ]] && [[ -n $2 ]] && curl -X 'GET' \
  "https://api.coingecko.com/api/v3/simple/price?ids=$1&vs_currencies=$2&include_market_cap=true&include_24hr_vol=true&include_24hr_change=true&include_last_updated_at=true&precision=true" \
  -H 'accept: application/json' | jq 
```

调用`gecko_api()`函数现在只需要一个位置参数，该参数将是所选的`-h`、`-l`或`-c`选项，或者是组合的`-hlc`选项集。如果你想分别使用每个选项，而不是将它们合并，你需要将`gecko_api $1`改为`gecko_api $1 $2 $3`或`gecko_api $@`。

所以，改编原始的 Coingecko 脚本以使用`getopts`其实非常简单。但，这样做的优势是什么呢？嗯，在这个情况下，实际上只有一个优势。也就是说，通过使用`getopts`选项切换，你现在可以通过仅运行一次脚本来执行多个任务。这是原始脚本做不到的。无论如何，你可以将这个作为模板，适用于你自己在使用`getopts`时可能更具优势的脚本。

现在，让我们来看一个很酷的系统监控脚本。

## Tecmint 监控脚本

我从 Tecmint 网站借用了这个实用的仅限 Linux 的`tecmint_monitor.sh`脚本，并稍微调整了一下，使其在较新的 Linux 系统上运行得更好。作者将其发布在 Apache 2.0 自由软件许可下，所以我能够将修改版上传到 GitHub，方便你使用。（我还在*进一步阅读*部分提供了原始 Tecmint 文章的链接。）

我不会在这里解释整个脚本，因为作者已经通过在各处插入注释做得非常出色。但是，我会指出其中的一些要点。

首先需要注意的是，作者在使用`case`结构时做了一些不同的处理。与将他想要执行的所有代码放入`case`选项中不同，他只是使用`case`选项来设置`iopt`或`vopt`变量，具体取决于选择了`-i`还是`-v`选项。下面是它的实现方式：

```
while getopts iv name
do
        case $name in
          i)iopt=1;;
          v)vopt=1;;
          *)echo "Invalid arg";;
        esac
done 
```

可执行代码被放置在一对`if...then`结构中。下面是第一个结构：

```
if [[ ! -z $iopt ]]
then
{
wd=$(pwd)
basename "$(test -L "$0" && readlink "$0" || echo "$0")" > /tmp/scriptname
scriptname=$(echo -e -n $wd/ && cat /tmp/scriptname)
su -c "cp $scriptname /usr/bin/monitor" root && echo "Congratulations! Script Installed, now run monitor Command" || echo "Installation failed"
}
fi 
```

所以，如果`iopt`变量的值不是零长度，意味着选择了`-i`选项，那么此脚本安装例程将会执行。它并不是一个非常复杂的命令。它所做的只是提示你输入根用户的密码，将`techmint-monitor.sh`脚本复制到`/usr/bin/`目录，并将其重命名为`monitor`。

现在，这里有一个我还没有解决的小问题。因为自从作者在 2016 年创建这个脚本以来，Linux 用户不为根用户账户设置密码已经变得越来越常见。事实上，这是 Ubuntu 发行版的默认行为，很多其他发行版也是可选的。（事实上，我在这台 Fedora 工作站上从未为根用户设置过密码。）因此，`su -c`命令在这里不起作用，因为它会在提示时要求你输入根用户的密码。所以，如果你没有为根账户设置密码，在没有使用`sudo`的情况下运行脚本时，你将总是遇到身份验证失败的问题。但是，当我使用`sudo`运行脚本时，它工作得很好，因此我没有急于更改这一点。

接下来，我们有针对`vopt`变量的`if...then`结构，它看起来是这样的：

```
if [[ ! -z $vopt ]]
then
{
#echo -e "tecmint_monitor version 0.1\nDesigned by Tecmint.com\nReleased Under Apache 2.0 License"
echo -e "tecmint_monitor version 0.1\nDesigned by Tecmint.com\nTweaked by Donnie for more modern systems\nReleased Under Apache 2.0 License"
}
fi 
```

这个选项的作用仅仅是显示关于脚本的版本信息。在底部，我添加了`Tweaked by Donnie`这一行，以显示我对原脚本进行了修改。（你可以说我在这里做了一些调整，以表明我做了一些修改。）

接下来，我们看到一个长的`if...then`结构，它执行实际的工作。它从`if [[ $# -eq 0 ]]`开始，这意味着如果没有选择任何选项，那么`if...then`结构中的代码将会执行。

在这个长长的`if...then`块中包含了获取各种系统信息的命令。我不得不调整检查 DNS 服务器信息的命令，因为原作者让它去寻找`resolv.conf`文件中某一特定行的 DNS 服务器 IP 地址。我修改了它，让它去查找任何以`nameserver`开头的行，效果如下：

```
# Check DNS
#nameservers=$(cat /etc/resolv.conf | sed '1 d' | awk '{print $2}')
nameservers=$(cat /etc/resolv.conf | grep -w ^nameserver | awk '{print $2}')
echo -e '\E[32m'"Name Servers :" $tecreset $nameservers 
```

我还对检查磁盘使用情况的命令进行了调整，以使其更加便携。以下是调整后的效果：

```
# Check Disk Usages
#df -h| grep 'Filesystem\|/dev/sda*' > /tmp/diskusage
df -Ph| grep 'Filesystem\|/dev/sda*' > /tmp/diskusage
echo -e '\E[32m'"Disk Usages :" $tecreset
cat /tmp/diskusage 
```

我在这里所做的只是为`df`命令添加了`-P`选项，这样`df`的输出将始终符合 POSIX 规范。当然，如果需要，你也可以添加更多的驱动器。

最后，在最底部，我们有`shift $(($OPTIND -1))`命令。如果你注释掉这一行，脚本实际上也能正常运行，因为我们不需要处理任何非选项参数。不过，许多人认为，为了确保所有选项信息在脚本退出时被清除，最好在使用`getopts`的脚本末尾始终加上这一行。

在所有这些示例中，你看到我使用`while`循环来实现`getopts`，因为这是我个人的偏好。然而，如果你更喜欢使用`for`循环，完全可以。选择权在你。

好的，我想我们已经差不多完成了关于`getopts`的讨论。让我们总结一下并继续。

# 总结

在本章中，我们介绍了`getopts`，它为我们提供了一种非常酷且简单的方法，来创建识别 Linux 和 Unix 风格选项开关的脚本，这些开关可能会有自己的参数。我们首先回顾了命令选项和参数的概念，然后澄清了关于`getopt`和`getopts`的一个可能的混淆点。在介绍了如何使用`getopts`之后，我们总结了一些实际使用它的脚本示例。

在下一章中，我们将探讨 Shell 脚本如何帮助安全专业人员。我在那里等你。

# 问题

1.  以下哪个陈述是正确的？

    1.  你应该始终使用`getopt`，因为它比`getopts`更容易使用。

    1.  `getopt`可以处理包含空格的参数，而`getopts`则不能。

    1.  `getopt`要求你在每个选项后使用一个或多个 shift 命令，但`getopts`则不需要。

    1.  `getopts`可以处理长选项，而`getopt`则不能。

1.  在这行`while getopts :a:b:c options; do`中，第一个冒号有什么作用？

    1.  它抑制了来自 Shell 的错误信息。

    1.  它使得`-a`选项需要一个参数。

    1.  它什么也不做。

    1.  冒号使得选项之间被分开。

1.  你需要创建一个接受选项、带有参数的选项和非选项参数的脚本。你必须在脚本中插入以下哪个命令，才能使非选项参数生效？

    1.  `shift $(($OPTARG -1))`

    1.  `shift $(($OPTIND +1))`

    1.  `shift $(($OPTIND -1))`

    1.  `shift $(($OPTARG +1))`

# 进一步阅读

+   小型 getopts 教程：[`web.archive.org/web/20190509023321/https://wiki.bash-hackers.org/howto/getopts_tutorial`](http://web.archive.org/web/20190509023321/https://wiki.bash-hackers.org/howto/getopts_tutorial)

+   如何轻松处理脚本中的命令行选项和参数？：[`mywiki.wooledge.org/BashFAQ/035`](http://mywiki.wooledge.org/BashFAQ/035)

+   如何在 Bash 中使用 OPTARG：[`linuxsimply.com/bash-scripting-tutorial/functions/script-argument/bash-optarg/`](https://linuxsimply.com/bash-scripting-tutorial/functions/script-argument/bash-optarg/)

+   一个用于监控 Linux 网络、磁盘使用、系统运行时间、负载平均值和内存使用情况的 Shell 脚本：[`www.tecmint.com/linux-server-health-monitoring-script/`](https://www.tecmint.com/linux-server-health-monitoring-script/)

注意：以下链接包含有关`OPTIND`的额外信息。

+   如何使用 getopts 解析 Linux Shell 脚本选项：[`www.howtogeek.com/778410/how-to-use-getopts-to-parse-linux-shell-script-options/`](https://www.howtogeek.com/778410/how-to-use-getopts-to-parse-linux-shell-script-options/)

+   在 shell 内建命令 getopts 中，OPTIND 变量如何工作: [`stackoverflow.com/questions/14249931/how-does-the-optind-variable-work-in-the-shell-builtin-getopts`](https://stackoverflow.com/questions/14249931/how-does-the-optind-variable-work-in-the-shell-builtin-getopts)

+   如何在 Bash 中使用 “getopts”：[`linuxsimply.com/bash-scripting-tutorial/functions/script-argument/bash-getopts/`](https://linuxsimply.com/bash-scripting-tutorial/functions/script-argument/bash-getopts/)

# 答案

1.  c

1.  a.

1.  c

# 加入我们在 Discord 上的社区！

与其他用户、Linux 专家以及作者本人一起阅读这本书。

提问、为其他读者提供解决方案、通过“问我任何问题”环节与作者互动等等。扫描二维码或访问链接加入社区。

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code10596186092701843.png)
