

# 第十章：理解函数

函数是几乎所有现代编程语言中的重要组成部分。它们只是执行特定任务的代码块。它们的酷点在于，允许程序员重用代码。也就是说，一个代码块可以在程序的不同地方被调用，或者可以被不同的程序从函数库中调用。在本章中，我将向你展示如何在 shell 脚本中使用函数的基础知识。

本章内容包括：

+   函数简介

+   定义函数

+   在 shell 脚本中使用函数

+   创建函数库

+   查看一些现实世界的例子

如果你准备好了，我们就开始吧。

# 技术要求

使用任何你的 Linux 虚拟机或你的 Linux 主机。

此外，和往常一样，你可以从 Github 上抓取脚本和文本文件，像这样：

```
git clone https://github.com/PacktPublishing/The-Ultimate-Linux-Shell-Scripting-Guide.git 
```

# 函数简介

正如我上面提到的，函数是执行特定任务的编程代码块。函数之所以这么酷，是因为它们大大减少了你需要输入的代码量。你不必每次执行一个任务时都输入相同的代码，而是可以直接调用函数。这减少了脚本的大小，也使得输入脚本时更不容易出错。

此外，在函数第一次被调用之后，它会驻留在系统内存中，这可以提高脚本的性能。

你可以创建接受主程序传递参数的函数，返回值给主程序的函数，或者执行一些独立任务的函数，这些任务既不涉及传递参数也不涉及返回值。学习创建函数很简单，因为大多数时候你只需使用我已经向你展示过的相同编码技巧。我的意思是，是的，你会需要学习一些新东西，但实际上没有什么难度。

如果你在工作站上运行 Linux，你已经在不自觉中使用了很多功能。要查看系统正在运行的功能，只需运行一个 `declare -F` 命令，像这样：

```
[donnie@fedora ~]$ declare -F
declare -f __expand_tilde_by_ref
declare -f __fzf_comprun
declare -f __fzf_defc
. . .
. . .
declare -f gawkpath_prepend
declare -f quote
declare -f quote_readline
[donnie@fedora ~]$ 
```

这些功能每次你登录到计算机时都会由 shell 加载到系统内存中，并被构成 `bash` 生态系统的各种实用工具使用。

运行 `declare` 命令并加上小写 `-f` 参数，还可以显示每个函数的代码。你也可以仅列出函数的名称，来只查看该函数的代码，像这样：

```
[donnie@fedora ~]$ declare -f __expand_tilde_by_ref
__expand_tilde_by_ref ()
{
    if [[ ${!1-} == \~* ]]; then
        eval $1="$(printf ~%q "${!1#\~}")";
    fi
}
[donnie@fedora ~]$ 
```

要查看这些函数是在哪里定义的，你需要进入调试模式，然后使用 `declare -F` 命令，像这样：

```
[donnie@fedora ~]$ bash --debugger
[donnie@fedora ~]$ declare -F __expand_tilde_by_ref
__expand_tilde_by_ref 1069 /usr/share/bash-completion/bash_completion
[donnie@fedora ~]$ 
```

所以，你会看到这个函数是在 `/usr/share/bash-completion/` 目录下的 `bash_completion` 函数库文件的第 1069 行定义的。去看一下那个文件，你会看到它里面定义了很多函数。当你完成调试模式后，只需输入 `exit`。

你还可以使用 `type` 命令获取关于函数的信息，像这样：

```
[donnie@fedora ~]$ type quote
quote is a function
quote ()
{
    local quoted=${1//\'/\'\\\'\'};
    printf "'%s'" "$quoted"
}
[donnie@fedora ~]$ 
```

现在你知道了什么是函数，接下来我们来看看如何定义一些函数。

# 定义一个函数

你可以在以下地方定义一个函数：

+   在命令行中

+   在一个 shell 脚本中

+   在一个函数库中

+   在一个 shell 配置文件中

从命令行定义函数很有用，特别是当你需要快速测试或尝试新代码时。下面是它的样子：

```
[donnie@fedora ~]$ howdy() { echo "Howdy, world!" ; echo "How's it going?"; }
[donnie@fedora ~]$ howdy
Howdy, world!
How's it going?
[donnie@fedora ~]$ 
```

这个函数的名字是`howdy`，`()`告诉 shell 这是一个函数。函数的代码包含在一对大括号内。这个函数由两个`echo`命令组成，它们通过分号隔开。请注意，你需要在最后一个命令的末尾放置一个分号。你还需要确保在第一个大括号和第一个命令之间有一个空格，以及在最后一个分号和闭合的大括号之间有一个空格。

一些教程可能会告诉你，可以使用`function`关键字开始函数，像你在这里看到的那样：

```
[donnie@fedora ~]$ function howdy() { echo "Howdy, world!" ; echo "How's it going?"; }
[donnie@fedora ~]$ howdy
Howdy, world!
How's it going?
[donnie@fedora ~]$ 
```

不过，创建没有`function`关键字的函数也可以正常工作，所以实际上没有必要使用它。

请注意，使用`function`关键字是一个非常糟糕的主意，因为它不具备可移植性。我的意思是，它在`bash`、`zsh`和`ash`中有效，但在`ksh`、`dash`或 Bourne Shell 中不起作用。即使可以使用`function`关键字，它也没有实际用途。因此，你最好的做法是永远不要使用它。

你在命令行中定义的任何函数都是临时的，一旦关闭终端就会消失。所以你需要将它们保存到文件中，使其持久化。而且，你在命令行中定义的任何函数在`bash --debug`会话中是不可见的，因为它们没有被导出到子 shell。

在 shell 脚本、函数库文件和 shell 配置文件中创建函数的方式没有区别。但是，你在结构化函数时有几种不同的方法。

第一个方法是将整个函数放在一行中，可以选择是否使用前面的`function`关键字。这和你在命令行中创建函数的方式完全一样。既然你已经见过这个方法了，我就不再多说了。

另一种方法是将函数结构化为多行格式，就像在其他编程语言（如 C 或 Java）中一样。这实际上是最好的选择，因为它使代码更加易读，减少了出错的可能性。一个快速的方法是通过`declare -f`查看我刚才在命令行上创建的单行`howdy`函数，像这样：

```
[donnie@fedora ~]$ declare -f howdy
howdy ()
{
    echo "Howdy, world!";
    echo "How's it going?"
}
[donnie@fedora ~]$ 
```

`declare -f`命令会自动将我在命令行中输入的函数转换为多行格式。如果我愿意，我可以将它复制粘贴到 shell 脚本文件中。事实上，我决定将它复制到一个新的`howdy_func.sh`脚本中。它将是这样的：

```
#!/bin/bash
howdy ()
{
    echo "Howdy, world!";
    echo "How's it going?"
} 
```

如前所述，前面的`function`关键字是可选的。我选择省略它，因为它没有实际作用，而且不兼容某些非`bash` shell。

好的，这一切看起来都不错，但当我运行脚本时看看会发生什么：

```
[donnie@fedora ~]$ ./howdy_func.sh
[donnie@fedora ~]$ 
```

什么都没有发生，这可不好。我会在下一部分展示为什么。

# 在 shell 脚本中使用函数

你会惊讶于函数如何让你的 shell 脚本变得更加*有功能*。（是的，我知道，那个真的是个糟糕的双关笑话。）继续阅读，看看如何实现这一点。

## 创建并调用函数

当你将一个函数放入一个 shell 脚本中时，函数代码不会被执行，直到你调用该函数。让我们修改一下`howdy_func.sh`脚本来实现这一点，像这样：

```
#!/bin/bash
howdy ()
{
    echo "Howdy, world!";
    echo "How's it going?"
}
howdy 
```

你看到我所做的只是将函数名称放在函数代码块之外。另外，请注意，函数定义必须出现在函数调用之前。否则，shell 将无法找到该函数。无论如何，当我现在运行脚本时，它可以正常工作：

```
[donnie@fedora ~]$ ./howdy_func.sh
Howdy, world!
How's it going?
[donnie@fedora ~]$ 
```

我还应该指出，第一个大括号可以像上面那样单独放在一行，也可以与函数名称放在同一行，像这样：

```
#!/bin/bash
howdy() {
        echo "Howdy, world!"
        echo "How's it going?"
}
howdy 
```

我个人更喜欢这种方式，有一个简单的理由。就是我偏爱的文本编辑器是`vim`，并且将第一个大括号放在与函数名称同一行会导致`vim`激活其自动缩进功能。

重用代码的能力是函数最酷的功能之一。你不必在每个需要执行某项任务的地方都输入相同的代码，只需输入一次，然后在需要的地方调用它。让我们对`howdy_func.sh`脚本再做一点修改，像这样：

```
#!/bin/bash
howdy ()
{
    echo "Howdy, world!";
    echo "How's it going?"
}
howdy
echo
echo "Let's do some more stuff and then"
echo "we'll call the function again."
echo
howdy 
```

运行修改过的脚本时是这样的：

```
[donnie@fedora ~]$ ./howdy_func.sh
Howdy, world!
How's it going?
Let's do some more stuff and then
we'll call the function again.
Howdy, world!
How's it going?
[donnie@fedora ~]$ 
```

所以，我调用了这个函数两次，它执行了两次。多酷啊？不过，它还会变得更好。

## 将位置参数传递给函数

你还可以像将位置参数传递给 shell 脚本一样，将位置参数传递给函数。为此，像你在这个`greet_func.sh`脚本中看到的那样调用函数并传递你希望作为位置参数的值：

```
#!/bin/bash
greetings() {
        echo "Greetings, $1"
}
greetings Donnie 
```

你看到函数中的`$1`位置参数。当我调用该函数时，我传递了`Donnie`这个值给函数。下面是我运行脚本时发生的情况：

```
[donnie@fedora ~]$ ./greet_func.sh
Greetings, Donnie
[donnie@fedora ~]$ 
```

你也可以像平常一样将位置参数传递给脚本，然后将它传递给函数，像你在这里看到的那样：

```
#!/bin/bash
greetings() {
        echo "Greetings, $1"
}
greetings $1 
```

这是我运行这个脚本时发生的情况：

```
[donnie@fedora ~]$ ./greet_func2.sh Frank
Greetings, Frank
[donnie@fedora ~]$ 
```

这很好，但它只适用于一个参数。你传入的任何额外参数都不会出现在输出中。你可以通过定义更多的参数来解决这个问题，像这样：

```
#!/bin/bash
greetings() {
        echo "Greetings, $1 $2"
}
greetings $1 $2 
```

如果你知道总是想要传递指定数量的参数，这样也可以。如果你不想限制自己，可以使用`$@`作为位置参数。这样，每次运行脚本时都可以传递任意数量的参数。看起来是这样的：

```
#!/bin/bash
greetings() {
        echo "Greetings, $@"
}
greetings $@ 
```

让我们看看当我输入四个名称时会发生什么情况：

```
[donnie@fedora ~]$ ./greet_func4.sh Vicky Cleopatra Frank Charlie
Greetings, Vicky Cleopatra Frank Charlie
[donnie@fedora ~]$ 
```

看起来不错。但我还可以稍微修饰一下，就像这样：

```
[donnie@fedora ~]$ ./greet_func4.sh Vicky, Cleopatra, Frank, "and Charlie"!
Greetings, Vicky, Cleopatra, Frank, and Charlie!
[donnie@fedora ~]$ 
```

相当灵巧，是吧？更加灵巧的是，让函数将值传递回主程序。

## 从函数传递值

好的，这里我需要稍作停顿，并澄清一些其他编程语言的老手可能会遇到的事情。与其他语言不同，Shell 脚本函数不能*直接*将值返回给它的调用者，通常是程序的主体部分。

好的，在函数中有一个`return`命令，但它与其他语言中的`return`命令不同。在 Shell 脚本中，你可以通过`return`仅传递命令的退出状态。

你可以创建一个返回值的函数，但你必须使用全局变量或局部变量和命令替换的组合来实现。另一个区别是，在其他语言中，变量默认是局部的，你必须指定哪些变量是全局的。在 Shell 脚本中，情况正好相反。Shell 脚本变量默认是全局的，你必须指定哪些变量是局部的。

现在，对于那些不是其他编程语言老手的人，请允许我解释一下局部和全局变量之间的区别。

全局变量可以从脚本的任何地方访问，即使你在函数内定义它。例如，看看这个`value1.sh`脚本中的`valuepass`函数：

```
#/bin/bash
valuepass() {
        textstring='Donnie is the great BeginLinux Guru'
}
valuepass
echo $textstring 
```

看看我运行脚本时会发生什么：

```
donnie@debian12:~$ ./value1.sh
Donnie is the great BeginLinux Guru
donnie@debian12:~$ 
```

在这个脚本中，你可以看到函数除了定义全局变量`textstring`并设置其值外什么也没做。`textstring`的值在函数外部是可用的，这意味着调用函数后，我可以回显`textstring`的值。

现在，让我们像这样在`value2.sh`脚本中将`textstring`定义为局部变量：

```
#/bin/bash
valuepass() {
        local textstring='Donnie is the great BeginLinux Guru'
}
valuepass
echo $textstring 
```

看看这次我运行脚本时会发生什么：

```
donnie@debian12:~$ ./value2.sh
donnie@debian12:~$ 
```

这次没有输出，因为局部变量在函数外部是不可访问的。或者更准确地说，局部变量在函数外部是*直接*不可访问的。有方法可以使函数将局部变量的值传递给外部调用者，我马上就会给你展示。

不过首先，我想回答一个你一定很想问的问题。也就是说，如果我们可以使用全局变量将值传递到函数外部，那为什么还需要本地变量呢？其实，如果你在函数中只使用全局变量，你可能会把脚本弄成一个调试噩梦。因为如果你不小心在函数外创建了一个与函数内全局变量同名的变量，你就会覆盖函数内部变量的值。

通过在函数内部仅使用本地变量，你可以避免这个问题。如果你有一个与函数内的本地变量同名的外部变量，外部变量不会覆盖本地变量的值。为了演示这个，我们来创建一个 `value3.sh` 脚本，像这样：

```
#/bin/bash
valuepass() {
        textstring='Donnie is the great BeginLinux Guru'
}
valuepass
echo $textstring
textstring='Who is the great BeginLinux Guru?'
echo $textstring 
```

你会看到，我在函数内将 `textstring` 变量重新设置为全局变量，并且在函数外再次定义了这个变量。现在，当我运行脚本时，会发生以下情况：

```
donnie@debian12:~$ ./value3.sh
Donnie is the great BeginLinux Guru
Who is the great BeginLinux Guru?
donnie@debian12:~$ 
```

第二个 `echo $textstring` 命令使用了 `textstring` 的新值，覆盖了之前的值。如果这是你希望发生的情况，那是没问题的，但通常这并不是你想要的结果。

让一个函数通过本地变量传递值的一种方法是通过命令替换结构来调用该函数。为了演示这一点，创建一个 `value4.sh` 脚本，像这样：

```
#!/bin/bash
valuepass() {
        local textstring='Donnie is the great BeginLinux Guru'
        echo "$textstring"
}
result=$(valuepass)
echo $result 
```

你会看到，我创建了外部的 `result` 变量，并将通过 `$(valuepass)` 命令替换获得的值赋给了它。

获取函数值的另一种方法是使用 **结果变量**。这涉及到在调用函数时将一个 *变量名* 传递到函数中，然后在函数内部给该变量赋值。让我们看一下 `value5.sh` 脚本，看看它是如何工作的：

```
#/bin/bash
valuepass() {
        local __internalvar=$1
        local myresult='Shell scripting is cool!'
        eval $__internalvar="'$myresult'"
}
valuepass result
echo $result 
```

`valuepass result` 命令调用函数，并通过 `$1` 位置参数将 `result` 变量的名称传递给函数。不过需要理解的是，`result` 仍然没有被赋值，所以你传递给函数的只是变量的名称，而没有传递其他任何东西。

到目前为止，你只看到过位置参数用于在调用脚本时传递值。这里，你看到了位置参数也可以用于将值从脚本的主体传递到函数中。

在函数内部，这个变量名由 `$1` 位置参数表示。这个变量名被赋值给本地变量 `__internalvar`。下一行将 `Shell scripting is cool!` 赋值给本地变量 `myresult`。在最后一行，`eval` 命令将 `$myresult` 的值设置为 `$__internalvar` 变量的值。

请记住，`__internalvar`变量的值是`result`，这是我们传入函数的变量名。同时，请注意，由于这里没有指定`local`关键字，这个变量现在是全局变量。这意味着它的值可以返回到脚本的主体部分。

所以现在，`$__internalvar`实际上就是`result`变量，且它的值是`Shell scripting is cool!`。

不过要理解的是，你传入函数的变量名不能与函数内部的局部变量名相同。否则，`eval`命令将无法正确设置变量的值。这就是为什么我在局部变量名前加了两个下划线，以确保避免在函数外部有相同名称的变量。（当然，单个下划线也能达到目的，但约定俗成的做法是使用两个下划线。）

接下来，我们来创建一些库。

# 创建函数库

为了方便地使函数可重用，只需将它们放入函数库中。然后，在你的脚本中，使用`source`命令或点命令调用所需的库。如果只有你一个人会使用这些库，可以将它们放在你的主目录中。

如果你希望所有人都能使用它们，可以将它们放在一个中心位置，比如`/usr/local/lib/`目录下。

为了简单演示，我在`/usr/local/lib/`目录中创建了`donnie_library`文件，里面包含了两个简单的函数。它看起来是这样的：

```
howdy_world() {
echo "Howdy, world!"
echo "How's it going?"
}
howdy_georgia() {
echo "Howdy, Georgia!"
echo "How's it going?"
} 
```

（我住在乔治亚州，这也解释了第二个函数的存在。）

这个文件不需要设置可执行权限，因为它只包含将在可执行脚本中被调用的代码。

接下来，我创建了`library_demo1.sh`脚本，它看起来是这样的：

```
#!/bin/bash
source /usr/local/lib/donnie_library
howdy_world 
```

你看到我使用`source`命令引用了`donnie_library`文件。然后，我像通常那样调用了`howdy_world`函数。运行脚本看起来是这样的：

```
[donnie@fedora ~]$ ./library_demo1.sh
Howdy, world!
How's it going?
[donnie@fedora ~]$ 
```

我还创建了`library_demo2.sh`文件，它看起来是这样的：

```
#!/bin/bash
. /usr/local/lib/donnie_library
howdy_georgia 
```

这次我用点命令替换了`source`命令，向你展示它们实际上执行的是相同的操作。

请注意，使用`source`关键字适用于`ash`、`ksh`、`zsh`和`bash`，但在某些其他 shell（如`dash`或 Bourne）上不起作用。如果你希望脚本能够在尽可能多的 shell 上运行，你需要使用点命令（dot），而避免使用`source`。

然后，我在库文件中调用了第二个函数。运行这个脚本看起来是这样的：

```
[donnie@fedora ~]$ ./library_demo2.sh
Howdy, Georgia!
How's it going?
[donnie@fedora ~]$ 
```

为了做一个稍微更实用的例子，我们加入一个网络检查函数，使用`case..esac`结构。不幸的是，`donnie_library`文件现在太大，无法在书中展示，但你可以从 Github 仓库下载它。不过，我还是想指出关于这个`case..esac`结构的一点：

```
network() {
site="$1"
case "$site" in
"")
        site="google.com"
;;
*)
        site="$1"
;;
esac
. . .
. . .
} 
```

请注意，在第一个条件中，我使用了`"")`构造。成对的双引号，中间没有空格，意味着位置参数没有传入任何值。这也意味着`google.com`会作为默认的网站名称。

现在，使用这个新函数的`library_demo3.sh`脚本如下所示：

```
#!/bin/bash
. /usr/local/lib/donnie_library
network $1 
```

如果没有提供参数，运行脚本的结果是这样的：

```
[donnie@fedora ~]$ ./libary_demo3.sh
2023-10-27 . . . Success! google.com is reachable.
[donnie@fedora ~]$ 
```

这是当我提供有效网站名称作为参数时的结果：

```
[donnie@fedora ~]$ ./libary_demo3.sh www.civicsandpolitics.com
2023-10-27 . . . Success! www.civicsandpolitics.com is reachable.
[donnie@fedora ~]$ 
```

最后，这是当我提供一个无效的网站名称时得到的结果：

```
[donnie@fedora ~]$ ./libary_demo3.sh www.donnie.com
ping: www.donnie.com: Name or service not known
2023-10-27 . . . Network Failure for site www.donnie.com!
[donnie@fedora ~]$ 
```

我知道，真可惜没有一个网站是以我命名的。（也许某天我得解决这个问题。）

你猜怎么着？这就是我对函数库要说的全部内容。接下来让我们进入现实世界。

# 看一些真实世界的例子

这里有几个函数实际应用的酷例子，快来看看吧！

## 检查网络连接

这个函数本质上与我在前面一节中展示的函数相同。唯一的真正不同之处在于，我使用了`if..then`结构，而不是`case..esac`结构。

此外，我将这个函数放入了实际可执行的脚本中。不过，如果你愿意，也可以轻松将它移到库文件中。（由于书籍排版的限制，我不能在这里展示完整脚本，但你可以从 Github 仓库下载。它是`network_func.sh`文件。）总之，下面是我真正希望你看到的重点部分：

```
#!/bin/bash
network() {
if [[ $# -eq 0 ]]; then
        site="google.com"
else
        site="$1"
fi
. . .
. . .
}
network
network www.civicsandpolitics.com
network donnie.com 
```

你看，在我定义了`network()`函数之后，我调用了它三次。我第一次调用时没有传入位置参数，这样它就会 ping `google.com`。接着我又分别用另一个真实网站和一个假的网站作为位置参数调用了它。下面是我运行这个新脚本时得到的结果：

```
[donnie@fedora ~]$ ./network_func.sh
2023-10-27 . . . Success for google.com!
2023-10-27 . . . Success for www.civicsandpolitics.com!
ping: donnie.com: Name or service not known
2023-10-27 . . . Network Failure for donnie.com!
[donnie@fedora ~]$ 
```

那部分没问题。现在让我们查看系统日志文件，看看函数中的`logger`工具是否正确记录了日志条目：

```
[donnie@fedora ~]$ sudo tail /var/log/messages
. . .
. . .
Oct 27 16:14:21 fedora donnie[38134]: google.com is reachable.
Oct 27 16:14:23 fedora donnie[38137]: www.civicsandpolitics.com is reachable.
Oct 27 16:14:23 fedora donnie[38140]: Could not reach donnie.com.
[donnie@fedora ~]$ 
```

是的，那部分也正常工作。所以再次证明，我们已经实现了酷炫功能。（当然，你也可以修改脚本，使你能够将自己的位置参数作为参数传入。我就不再赘述这部分内容了。）

请注意，我是在一台 Fedora 机器上测试这个脚本的，它仍然使用旧版的`rsyslog`日志文件。如果你在 Debian 上尝试此操作，你需要自己安装`rsyslog`。默认情况下，Debian 12 现在只使用`journald`日志系统，并且没有安装`rsyslog`。

接下来，这是我为自己编写的一些小东西。

## 使用 CoinGecko API

CoinGecko 是一个很酷的网站，你可以在这里查看几乎所有加密货币的价格和统计信息。如果你只需要查看一个币种，它使用起来相当快速，但如果你需要查看多个币种，就可能会比较耗时。

你可以创建一个自定义的硬币投资组合，这在你总是希望看到相同的硬币列表时很有用。但如果你总是想查看不同的硬币列表，这样做就不太有用了。所以，你可能想简化这个过程。为此，你可以创建一个使用 CoinGecko **应用程序编程接口**（**API**）的 shell 脚本。

要充分利用 CoinGecko API 的所有功能，你需要成为付费客户，这个费用相当高。幸运的是，做这个演示时你不需要这样做。你只需使用 CoinGecko 的公共 API 密钥，它是免费的。为了演示这一点，我们来做一个动手实验。

### 动手实验 – 创建 coingecko.sh 脚本

首先，查看 CoinGecko 的文档页面（https://www.coingecko.com/api/documentation），并按照以下步骤进行操作：

1.  向下滚动页面，你会看到可以使用公共 API 执行的各种功能列表。你可以使用下拉列表中的表单创建你将复制到 shell 脚本中的代码。我已经创建了`coingecko.sh`脚本，它执行其中三个功能，你可以从 Github 仓库下载。在使用之前，你需要确保你的系统上已安装`curl`和`jq`软件包。无论如何，下面是脚本的工作方式。

1.  在`gecko_api`函数的顶部，我将`coin`和`currency`变量定义为位置参数`$1`和`$2`。`if [ -z $coin ]; then`这一行检查`coin`的值是否为零长度。如果是，说明在调用脚本时我没有输入任何参数。因此，脚本会显示一系列关于如何调用脚本的说明。

1.  接下来，你会看到这个`elif`结构：

    ```
    elif [ $coin == list ]; then
            curl -X 'GET' \ 
    "https://api.coingecko.com/api/v3/coins/list?include_platform=true" \
      -H 'accept: application/json' | jq | tee coinlist.txt 
    ```

1.  我从 CoinGecko 的文档页面复制了这个`curl`命令，并将其粘贴到脚本中。

1.  然后，我将输出管道传递给`jq`工具，将普通的**json**输出转换为**美化后的 json**格式。

1.  然后，我将输出管道传递给`tee`命令，以便在屏幕上查看输出并将其保存到文本文件中。使用`list`参数调用脚本将下载 API 支持的所有硬币列表。下一行的`elif`结构与这个`elif`结构相同，只是它将显示可以用于定价硬币的*对比货币*列表。

1.  函数最后一行的`else`结构要求你输入一个硬币和一个对比货币作为参数。文档页面上的原始代码包含了我在文档页面表单中输入的硬币和对比货币的实际名称。我将它们分别更改为`$coin`和`$currency`变量。接着，我需要将围绕 URL 的一对单引号改为一对双引号，以便变量名中的`$`能够正确解析。

1.  假设你想查看比特币的美元价格。首先，打开你在运行`./coingecko list`命令时创建的`coinlist.txt`文件。搜索比特币，你会找到一段类似于这样的内容：

    ```
    {
        "id": "bitcoin",
        "symbol": "btc",
        "name": "Bitcoin",
        "platforms": {}
      }, 
    ```

1.  接下来，打开你在运行`./coingecko currencies`命令时创建的`currency_list.txt`文件。向下滚动，直到找到你想用来为特定币种定价的基准货币。现在，要查看你想要的币种在所选货币中的价格，输入 `coinlist.txt` 文件中该币种的 `"id:"` 值作为第一个参数，并将期望的基准货币作为第二个参数，像这样：

    ```
    [donnie@fedora ~]$ ./coingecko.sh bitcoin usd
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   157    0   157    0     0    876      0 --:--:-- --:--:-- --:--:--   872
    {
      "bitcoin": {
        "usd": 34664,
        "usd_market_cap": 675835016546.567,
        "usd_24h_vol": 27952066301.047348,
        "usd_24h_change": 2.6519150329293883,
        "last_updated_at": 1698262594
      }
    }
    [donnie@fedora ~]$ 
    ```

1.  所以，看起来当前比特币的价格是 $34664 美元。

你可以通过输入用逗号分隔的列表一次查看多个币种，像这样：

```
[donnie@fedora ~]$ ./coingecko.sh bitcoin,ethereum,dero usd
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   471    0   471    0     0   3218      0 --:--:-- --:--:-- --:--:--  3226
{
  "bitcoin": {
    "usd": 34662,
    "usd_market_cap": 675835016546.567,
    "usd_24h_vol": 27417000940.815952,
    "usd_24h_change": 2.647099155664393,
    "last_updated_at": 1698262727
. . .
. . .
[donnie@fedora ~]$ 
```

1.  如果你是一个交易者，你可能想要查看某个山寨币相对于其他货币的价格，比如比特币的 satoshi。这里，我正在查看 Dero 的比特币 satoshi 价格：

    ```
    [donnie@fedora ~]$ ./coingecko.sh dero sats
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   161    0   161    0     0    746      0 --:--:-- --:--:-- --:--:--   748
    {
      "dero": {
        "sats": 8544.47,
        "sats_market_cap": 106868665749.11504,
        "sats_24h_vol": 50996466.4091239,
        "sats_24h_change": -3.2533636291339687,
        "last_updated_at": 1698263552
      }
    }
    [donnie@fedora ~] 
    ```

1.  所以，Dero 的当前 satoshi 价格是 8544.47 sats。（当然，当你查看时，这些价格会有所不同。）

1.  输出的前三行是脚本用来下载有关币种信息的 curl 命令。由于某种奇怪的原因，bash 把这个 curl 消息当做错误消息，尽管它实际上并不是错误消息。不过，这其实是好事，因为你可以通过在命令末尾使用标准错误重定向器来避免看到这个消息，就像这样：

    ```
    donnie@fedora:~$ ./coingecko.sh dero sats 2>/dev/null
    {
      "dero": {
        "sats": 3389.23,
        "sats_market_cap": 42965410638.71136,
        "sats_24h_vol": 112468964.13986385,
        "sats_24h_change": 2.81262920882357,
        "last_updated_at": 1718482347
      }
    }
    donnie@fedora:~$ 
    ```

1.  哦，我差点忘了脚本中最重要的部分，那就是调用函数的命令：

    ```
    gecko_api $1 $2 
    ```

它要求输入两个位置参数，但如果你只输入 `list` 或 `currencies` 作为单个参数，它也能正常工作。

1.  好的，这很好。但如果你有一个非常长的币种列表，并且每次调用这个脚本时都不想输入这么长的列表怎么办？这个很简单。我将创建一个 mycoins.sh 脚本，它将调用 coingecko.sh 脚本。它的样子如下：

    ```
    #!/bin/bash
    ./coingecko.sh bitcoin,dero,ethereum,bitcoin-cash,bitcoin-cash-sv,bitcoin-gold,radiant,monero usd 2>/dev/null 
    ```

当然，你可以编辑这个脚本，加入你自己喜欢的币种列表。

1.  作为此主题的变体，也可以下载 `coingecko-case.sh` 脚本。它与之前的脚本相同，但我使用了 `case..esac` 结构，而不是 `if..then` 结构。

这就是关于函数的内容，至少目前是这样。不过，如果你在本书的其余部分看到更多内容，也不要太惊讶。

现在，让我们总结一下并继续前进。

# 总结

函数是几乎所有现代编程语言中的重要组成部分。在本章中，我向你展示了如何构建函数、如何调用函数，以及如何创建函数库，你可以为自己使用或与其他用户共享。

在下一章，我将向你展示如何使用 shell 脚本执行数学运算。到时见。

# 问题

1.  以下哪两种方法可以用来创建函数？（选择两项。）

    1.  `function function_name()`

    1.  `function_name()`

    1.  `declare -f function_name`

    1.  `declare -F function_name`

1.  你想创建一个函数，将变量的值返回给主程序。以下哪项应当使用来实现这一功能？

    1.  一个全局变量

    1.  一个局部变量

    1.  一个 `return` 语句

    1.  一个 continue 语句

1.  你想创建一个任何用户都可以在他或她的脚本中使用的函数。最好的方法是什么？

    1.  在一个中心位置创建一个函数库文件，并使其可执行。

    1.  将新函数添加到 `/etc/bashrc` 文件中。

    1.  在一个中心位置创建一个函数库文件，但不要使其可执行。

    1.  D. 在你自己的主目录中创建一个函数库文件。

1.  下面哪两个命令可以用来在你的 shell 脚本中引用一个函数库？（选择两个）

    1.  `load`

    1.  `reference`

    1.  `source`

    1.  `.`

1.  你想查看加载到内存中的函数列表，以及它们的源代码。你应该使用哪个命令？

    1.  `declare -f`

    1.  `declare -F`

    1.  `debugger -f`

    1.  `debugger -F`

# 进一步阅读

+   Bash 函数: [`linuxize.com/post/bash-functions/`](https://linuxize.com/post/bash-functions/)

+   从 Bash 函数返回值: [`www.linuxjournal.com/content/return-values-bash-functions`](https://www.linuxjournal.com/content/return-values-bash-functions)

+   Bash 函数及其使用方法 {变量、参数、返回值}: [`phoenixnap.com/kb/bash-function`](https://phoenixnap.com/kb/bash-function)

+   编写 shell 脚本 - 第 15 课：错误、信号和陷阱: [`linuxcommand.org/lc3_wss0150.php`](http://linuxcommand.org/lc3_wss0150.php)

+   （请注意，本教程的作者使用全大写字母来表示脚本变量名称，我个人不推荐这样做。除此之外，这是一个很好的教程。）

+   Bash 脚本高级指南: [`tldp.org/LDP/abs/html/abs-guide.html#FUNCTIONS`](https://tldp.org/LDP/abs/html/abs-guide.html#FUNCTIONS)

# 答案

1.  a 和 b

1.  b

1.  c

1.  c 和 d

1.  a

# 加入我们的 Discord 社区！

与其他用户、Linux 专家和作者本人一起阅读这本书。

提问、为其他读者提供解决方案、通过 "问我任何问题"（Ask Me Anything）环节与作者聊天，等等。扫描二维码或访问链接加入社区。

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code10596186092701843.png)
