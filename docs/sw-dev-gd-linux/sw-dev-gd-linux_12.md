

# 第十二章：使用 Shell 脚本自动化任务

有时候，你会发现自己一遍又一遍地重复相同的几条命令，可能只是稍微有些变化。你会感到沮丧，然后说：“算了，我来写个脚本吧。”作为一个 CLI 大师，你做了如下操作：

1.  运行 `tail -n 20 ~/.bash_history > myscript.sh` 来创建一个文件，包含你最后运行的 20 条 Bash 命令。

1.  然后运行 `bash myscript.sh` 来执行脚本。

虽然这不是推荐的做法（我们将在本章中讨论这个问题），但它是创建和运行 Bash 脚本的一种完全有效的方法。

本章是一个 Bash 脚本速成课程。像任何编程速成课程一样，除非你实际跟着做，自己敲代码并在自己的 Linux 环境中运行，否则它完全没有用。除了向你展示被认为是现代且最佳实践的 Bash 语法子集外，我们还会分享多年来积累的经验，指出常见的陷阱和尖锐的边缘。

虽然 Bash 不是我们最喜欢的语言，但有时它正是解决你面对的问题的最佳工具。我们也会尽量帮助你理解这一点。

在本章中，我们将涵盖以下内容：

+   Bash 脚本基础

+   Bash 与其他 Shell 的对比

+   Shebang 和可执行文本文件

+   测试

+   条件语句

# 为什么你需要掌握 Bash 脚本基础

Shell 脚本是任何开发者不可或缺的工具；即使你不是每周都编写脚本，你也会阅读脚本。在本章中，我们将介绍你需要了解的基础知识，这样你就能在例如以下情况时感到得心应手：

+   你会面对别人几年前写的 Shell 脚本，例如“你能检查一下我们能否重用 Steve 在离开谷歌之前写的自动化脚本吗？”

+   当你看到有机会编写自己的 Shell 脚本，处理已经有现成 Shell 程序可以解决的工作（如过滤、搜索、排序输出，或者将一个程序的输出传递给另一个程序）时，你就会有这样的想法。

+   你希望在构建镜像时精确控制每个 Docker 层中包含的内容。

+   你需要在 Linux 服务器的操作系统环境中协调其他软件：启动顺序、错误检查、程序间的提前中止等等。

有无数的使用案例，Shell 脚本*恰好*是解决你问题空间的最佳工具。完成本章后，你将具备编写自定义脚本所需的技能。

# 基础

Bash 可以像学习任何其他编程语言一样学习。它有一个环境（Unix 或 Linux）、一种标准库（系统上安装的任何 CLI 驱动程序）、变量、控制流（循环、测试和迭代）、插值、一些内置的数据结构（数组、字符串和布尔值——算是吧）等等。

本书假设你是一个软件开发人员，因此知道如何编程，因此我们不会向你教授这些标准的编程语言特性，而是简单地展示它们在 Bash 中的表现，并提供一些习惯用法（或常见误用）的建议。

## 变量

像其他编程语言一样，Bash 有可以为空或设置为某个值的变量。未设置的变量就是“空的”，除非你通过`set -u`设置了`-u`选项（在未设置变量时产生错误），否则 Bash 会愉快地使用它们而不会慌张。

### 设置

设置变量时，使用等号。

`FOOBAR=nice` 将把 `FOOBAR` 变量的值设置为 `nice`。

Bash 没有类型——它几乎是最不具类型的编程语言。

变量符号本身可以包含字母、数字和下划线，但不能以数字开头。

通常的做法是，环境变量使用大写字母命名，而 Bash 脚本中的变量使用小写字母命名。这些变量名通常使用下划线来分隔单词。当在变量名中使用数字时，避免以数字开头。Bash 不允许这样做。和其他语言一样，命名时最好让变量名能表示它的用途，并指示它是否是常量，或者对于数组使用复数名称：

+   不合法：`%foo&bar=bad`

+   不合法：`2foo_bar=bad`

+   合法但不推荐：`foo_BAR123=still_very_bad`

+   正确的环境变量：`PORT=443`

+   正确示例：`local_var=512`

+   正确的环境变量：`FOO_BAR123=good`

+   正确的本地数组变量：`words=(foo bar baz)`

## 获取

要使用变量，可以通过`$`符号引用它：

```
$ echo $FOOBAR
nice 
```

# Bash 与其他 shell 的比较

在类 Unix 环境中存在着各种各样的 shell 程序；你可以认为，Unix 的流行原因之一就是它一直是一个几乎没有障碍的脚本和自动化环境。

本章将教你如何用 Bash 编写自己的脚本。你在这里学到的很多内容也可以在其他 shell（例如 `ksh` 和你在 `/bin/sh` 找到的其他常见最小化 shell）上使用，但我们这里关注的是 Bash。

如果你在编写 shell 脚本，Bash 在广泛可用性和足够大的语言功能集之间取得了完美的平衡，足以让你编写小型程序时感到舒适。

# Shebang 和可执行文本文件，也就是脚本

在类 Unix 系统中，“脚本”只是一个可执行的纯文本文件。操作系统（在 Linux 中通常被称为“内核”）查看第一行，以确定将文件的内容传递给哪个解释器。

第一行是所谓的“shebang”（或称 hashbang），由一个井号和一个感叹号（`#!`）字符组成，后面跟着用于执行文件代码的解释器的路径。以下是一个示例的 shebang 行：

```
#!/usr/bin/env bash 
```

当 Unix 类系统的内核运行一个具有可执行位的文件时，它们会查看文件的前几个字节。这可能包含一个魔术数字。这个数字可以是二进制文件的一部分，或者是像 shebang 中那样的人类可读字符。内核使用这些信息来判断是否有适当的方式执行它。例如，这可以防止内核尝试执行一个图像文件并崩溃。根据系统的不同，内核或 shell 会确保随后的命令被执行。`env`程序将运行该命令，并考虑`PATH`环境变量来查找并执行`bash`。

虽然井号在大多数脚本语言中表示注释，因此会被解释器忽略，但文件开头的这个特殊注释告诉操作系统应该运行哪个命令来解释文件的其余部分。这里是你会看到的一些常见示例：

+   `#!/bin/sh`：在这个特定的文件系统位置使用这个特定的 shell 程序

+   `#!/usr/bin/python3`：使用这个特定的 Python 二进制文件

+   `#!/usr/bin/env python`：使用`env`程序来确定在该环境中使用哪个 Python 二进制文件（不同的系统可能会在不同的路径下安装同一个程序的不同版本）

虽然你会看到所有这些变体，但最佳选择始终是使用`/usr/bin/env`以确保可移植性。`/bin/sh`是一个例外，因为每个符合 POSIX 的系统都必须在该位置提供一个符合 POSIX 的 shell。

## 常见的 Bash 设置（选项/参数）

由于 shebang 行作为命令执行，因此也可以传递参数。虽然保持简单通常是一个好主意，但一个常见的做法是向 shell 传递额外的参数，特别是对于 Bash 来说，它常用于大型脚本，因为与通常位于`/bin/sh`的较小 shell 相比，Bash 提供了更多的特性。

在 Bash 脚本中，你会经常看到传递的`-e`、`-u`、`-x`、`-o pipefail`等参数。你可能会在 shebang 行本身看到这些参数：

```
#!/usr/bin/env bash -euxo pipefail 
```

或者，使用 Bash 中的`set`命令设置参数或选项，正如下一个语句所示：

```
#!/usr/bin/env bash
set -eu -o pipefail 
```

设置这些选项使得 Bash 的行为更像你习惯的编程语言，具体包括：

+   如果命令管道的任何组成部分失败，立即退出，并且

+   将未设置的变量视为致命错误。

这是这些选项的文档拆解，并附带一个有用的调试选项（`-x`）作为额外内容：

+   `-o pipefail`：在使用管道时，这将确保管道中发生的错误会被传递。如果发生多个错误，将使用最右侧的错误。

+   `-e`：如果出现错误或命令失败，这将确保 shell 脚本立即退出。

+   `-u`：如果使用任何未设置的变量，将抛出错误。

对于调试，`-x`非常有用：

+   `-x`：这将启用跟踪。这意味着每个命令将在执行之前写入标准错误。

除了 `–o pipefail` 外，所有参数都可以在大多数 Unix shell 中找到。有关 Bash 选项的更多信息，请参阅其手册页：[`manpages.org/bash`](https://manpages.org/bash)。

## /usr/bin/env

记住一件事：`/bin/sh` 是一个 POSIX 标准路径，指向任何 POSIX 兼容的 shell。你可以放心它在任何 Linux 或 Unix 系统上都会存在。通常这不是 Bash，而是一个更简约的 shell，提供足够的功能以满足 POSIX 标准，使你能够编写非常便携的 shell 脚本。对于脚本可能需要的所有其他 shell 和解释器，最佳实践是使用 `#!/usr/bin/env` 前缀，这样可以确保使用 `PATH` 中的正确路径，并防止当二进制文件没有在 `/usr/bin/` 中时出现“命令未找到”错误。

有多种情况可能导致 `/usr/bin/bash` 或 `/bin/bash` 不是正确的路径。例如：

+   包管理器或公司特定的配置脚本通常会将解释器安装在与开发系统不同的地方。

+   手动安装软件的人，例如，为了绕过或重现一个 bug，通常会将生成的二进制文件放在 `/usr/local`。

+   各种脚本语言的虚拟环境将会把二进制文件放到每个源代码 `project/repository` 的子目录中。

+   没有 root 权限的用户安装解释器，例如，在他们的主目录中。

+   使用解释器版本管理器的人（例如，rvm、nvm 等）。

+   各种类 Unix 操作系统和一些 Linux 发行版不会将第三方软件包安装到 `/usr/` 目录。

尽管很多人无法想象他们的脚本最终会出现在如此不标准的位置，但你最终很可能会遇到这种情况。与其冒着软件因环境的轻微变化而崩溃的风险，不如养成在脚本中写入 `/usr/bin/env bash`（或你的代码所使用的解释器）的习惯。这样可以防止其他人——或者在凌晨 3 点被 pager 叫醒的你——不得不注意、排除故障、查找或在源文件中做出这样的更改。

## 特殊字符与转义

你应该经常使用的一个特殊字符是哈希符号（`#`），它会使该行符号后面的所有内容成为注释，解释器会忽略它们。

其他字符在 Bash 中有特殊含义，当你将它们作为变量的值的一部分使用时，需要用正斜杠（`\`）转义。以下是一些例子：

+   引号（`"` 和 `'`）

+   括号和圆括号（`{`、`}`、`[`、`]` 和 `(`、`)`）

+   插入符号（`<` 和 `>`）

+   波浪线：`~`

+   星号（Bash 中的“通配符”字符）：`*`

+   和号：`&`

+   问号：`?`

+   常见操作符：`!`、`=`、`|` 等

像在大多数其他编程语言中一样转义它们：

```
$ FOO="jaa\$\'" 
```

## 命令替换

Shell 脚本的一个优势和主要用例是任何命令都很容易访问。一个非常常见的例子是命令替换。当你想要使用一个或多个命令的输出时，这非常有用。你可以通过命令替换来做到这一点：

```
echo "Right now, it's $(date)" 
```

这将执行这些命令 —— 在这个例子中只是 `date`，但它也可以是你通过管道连接起来的复杂表达式。另一种做法是使用反引号。以下示例将产生相同的输出：

```
echo "Right now, it's `date`" 
```

# 测试

这里显示的测试命令通常与 `if/else` 控制流语句一起使用。字符串测试函数（`[[`）和算术测试函数（`((`）如果测试结果为 `true`，则返回 0；如果结果为 `false`，则返回 1。这是因为命令的退出代码 0 表示成功，这与你可能熟悉的其他编程语言不同，后者通常将零值评估为 false。Bash 中没有原生的 `boolean` 数据类型；整数 0 和 1 在布尔上下文中被使用。在脚本中，有时会初始化并使用变量 `true` 和 `false`。

## 测试运算符

这里是一些你可以在 Bash 中用来构造语句的基本布尔运算符 —— 本质上就是你在其他编程语言中已经习惯的运算符：

+   `!` – 非（否定）

+   `&&` – 且

+   `||` – 或

这些运算符可以与字符串和算术测试类型一起使用：

+   `==` – 等于

+   != 表示不等于

## [[ 文件和字符串测试 ]]

`[[` 复合命令允许你执行（并结合）“字符串”比较。如前所述，Bash 没有你在其他编程语言中习惯的严格数据类型，因此我们将其称为“字符串”或“类似字符串”的比较，因为这对软件开发人员来说是一个熟悉的概念。

如果用户的主目录不存在，创建它：

```
if [[ ! -d $HOME ]]; then
    echo "Creating home directory: ${HOME}..."
    mkdir -p $HOME
    echo "done"
fi 
```

那个 `!` 字符是 Bash 的否定运算符，因此你可以将这个示例的第一行理解为 `if NOT (test) is-a-directory $HOME, then…`。

这是一个稍微复杂一点的示例。如果用户的主目录不存在，`或者` 如果 `ALWAYSCREATE` 变量设置为 `yes`，则创建主目录：

```
ALWAYSCREATE=yes
if ! [[ -d $HOME ]] || [[ $ALWAYSCREATE == yes ]]; then
    echo "Creating home directory: ${HOME}..."
    mkdir -p $HOME
    echo "done"
fi 
```

### 有用的字符串测试运算符

+   `-z` 是未设置（用于变量）

+   `-n` 是非零的（`set` – 用于变量）

+   `=~` 是一个左操作数，用于匹配正则表达式（右操作数），例如 `[[ foobar =~ f*bar ]]`

### 文件测试的有用运算符

+   `-d`: 目录

+   `-e`: 存在

+   `-f`: 一个常规文件

+   `-S`: 一个套接字文件

+   `-w`: 可写，从这个 Bash 进程的角度来看

## (( 算术测试 ))

在 `((` 测试中进行的算术评估如果表达式的结果为 0，将把测试的退出值设置为 1；否则，它将返回退出状态 0。使用你在几乎所有其他编程语言中都知道的运算符，这使得测试变得非常直观：

+   `>` 和 `>=` – 大于和大于或等于

+   `<` 和 `<=` – 小于和小于或等于

+   `==` – 测试相等性

`(( $SOME_NUMBER == 24 ))` 是一个相当直接的算术测试。让我们看看它的行为。

对于数字 24：

```
→ SOME_NUMBER=24
→ (( $SOME_NUMBER == 24 ))
→ echo $?
0 
```

“`echo $?`” 命令输出上一个命令的退出状态，这让我们能够看到算术测试实际评估的结果。对于其他值，包括非数字值：

```
→ SOME_NUMBER=foobar
→ (( $SOME_NUMBER == 24 ))
→ echo $?
1 
```

如果 `$SOME_NUMBER` 未设置（例如，`[[ -z $SOME_NUMBER ]]`）：

```
→ unset SOME_NUMBER
→ (( $SOME_NUMBER == 24 ))
zsh: bad math expression: operand expected at `== 24 ' 
```

所以，总结一下：

+   `(( $SOME_NUMBER == 24 ))` 如果 `SOME_NUMBER` 变量设置为 24，将评估为 0。

+   如果 `$SOME_NUMBER` 设置为一个 `不是 24` 的值（包括非数字值），它将评估为 1。

+   如果 `$SOME_NUMBER` 被 *未设置*，你将会得到一个错误，因为你的算术测试没有左操作数可以用于比较。

# 条件语句：if/then/else

一个 Bash `if` 语句通常以这种形式出现：

`if [[ $TEST ]]; then $STATEMENTS else $OTHER_STATEMENTS fi`

在我们查看示例之前，记住这个形式的一些事项：

+   `if` 和 `fi` 分别用于开始和结束 `if` 块。

+   `;` 用于分隔 Bash 中的语句；在测试后加一个分号。

+   `[[` 和 `]]` 用于分隔你的测试表达式。

+   `else` 子句是可选的。

这就是 Bash 中 `if` 语句的样子：

```
if [[ -e "example.txt" ]]; then
    echo "The file exists!" 
```

### ifelse

如果你想在这个结构中加上 `else` 子句，当然可以！

```
if [[ -e "example.txt" ]]; then
    echo "The file exists!"
else
    echo "The file does not exist!"
fi 
```

# 循环

Bash 循环的一般格式是 `for / do / done`。它们还支持 `break` 和 `continue` 语句，分别用来跳出循环和跳到下一个迭代。

## C 风格的循环

Bash 支持 C 风格的循环，包括初始化表达式、条件表达式和计数表达式：

```
for (( i=0; i<=9; i++ ))
do  
  echo "Loop var i is currently $i"
done 
```

## for…in

让我们来谈谈使用 `for...in` 循环的迭代。试着在你的 Shell 中运行以下命令：

```
for i in 1 2 3 4 5
do
  echo $i
done 
```

这是一个包含一些控制流的循环：

```
for os in FreeBSD Linux NetBSD "macOS" DragonflyBSD
do
  echo "Checking out ${os}..."
  if [[ "$os" == 'NetBSD' ]]; then
    echo "(I'm pretty sure this would run on my toaster, actually)"
  fi
  sleep 1
done 
```

## While

另一个你可能在其他编程语言中熟悉的常见控制结构是 `while` 循环。在 Bash 中，这个语法非常相似。要跳出循环，可以使用 `break` 语句。

以下脚本将逐行读取 `lines.txt` 文件，直到遇到 `STOP` 行。最后一行还展示了如何将文件通过管道传递给循环。`read` 命令将逐行处理文件：

```
file="lines.txt"
while read line; do
    if [[ $line == "STOP" ]]; then
        echo "Encountered STOP. Exiting loop."
        break
    fi
    echo "Processing: $line"
    # Additional commands to process $line can be added here.
done < "$file" 
```

## 变量导出

通过在变量前加上 `export` 来导出一个变量，确保从你脚本的进程派生出来的任何子 Shell 也能访问该变量的值。这是一种确保变量能够传递到当前 Shell 变量作用域或命名空间的任何未来子作用域（或子命名空间）中的方法。

在你的 Shell 中设置一个变量：

```
MYDIR=$HOME 
```

创建并运行这个脚本（警告：这将失败！）：

```
#!/usr/bin/env bash
LISTING=$(ls "${MYDIR}/Documents")
echo $LISTING 
```

你会看到一个错误，`ls: /Documents: No such file or directory`，因为运行这个脚本启动了一个子 Shell，而该子 Shell 无法访问父 Shell 中未导出的变量（换句话说，就是你交互式 Shell 的父 Shell）。让子 Shell 访问你的变量必须通过 `export` 关键字明确地进行：

```
export MYDIR=$HOME 
```

导出变量后重新运行示例脚本，你会发现它现在可以访问`MYDIR`变量。

# 函数

我们通常建议，当你发现自己需要使用 Bash 函数时，应该已经找到一种其他语言来编写你日益增长的程序。然而，有时 Bash 仍然是解决问题的正确语言，我们希望向你展示最基本的内容，尤其是我们推荐的使用方式。

使用`function`关键字来定义一个函数：

```
function my_great_function {
  $EXPRESSIONS
} 
```

通过简单地调用函数的名称来调用函数：

```
my_great_function 
```

## 偏好使用本地变量

Bash 工作在更或少是全局作用域——更准确地说，是每个（子）Shell 的作用域。许多现代编程语言提供了独立的函数作用域，这样函数状态在函数退出后不会污染全局状态。

在函数中使用`local`变量可以防止这种情况发生，我们建议你使用它们：

```
#!/usr/bin/env bash
important_var=somevalue
function local_var_example() {
    local important_var="changed this locally, don't worry"
    echo "local_var_example: ${important_var}"
}
function bad_example() {
    important_var="this is mutating the global var because I'm bad, and I should feel bad."
    echo "bad_example: ${important_var}"
}
echo "before functions: ${important_var}"
local_var_example
echo
echo "after local_var_example: ${important_var}"
echo
bad_example
echo "after bad_example: ${important_var}"
exit 0 
```

运行这段代码，自己看看使用本地`vars`带来的不同。

# 输入和输出重定向

当你运行脚本时，你通常会希望将它们的输出重定向：

+   重定向到另一个程序（通过管道`|`——更多细节见*第十一章*，*管道和重定向*）

+   重定向到常规文件（如日志文件）

+   重定向到一个特殊位置，如`/dev/null`，它可以作为一个“黑洞”，用于存放你不需要的数据

除了管道外，以下是你在实际中最常见的输入/输出重定向技巧：

## `<`：输入重定向

这通常用于从文件中获取输入，而不是通过 Shell 启动进程：

```
grep foobar < stuff.txt 
```

## > 和`>>`：输出重定向

`>`符号会将输出流重定向到你指定的地方，如果它是一个常规文件，将覆盖已有的内容：

```
ps aux | grep foo > /var/log/foo_overwrite.log 
```

每次运行这个命令时，`ps aux | grep foo`的输出会被写入`/var/log/foo_overwrite.log`，并覆盖任何现有的文件内容。

使用`>>`代替会*附加*到输出文件，保留现有内容不变。这通常是你处理日志文件时需要的：

```
echo $(date && cat /proc/stat) >> /var/log/kernelstate.log 
```

## 使用`2>&1`来重定向`STDERR`和`STDOUT`

有时，你可能希望将标准输出和标准错误都重定向到一个文件：

```
consul agent -dev >> /var/log/consul.log 2>&1 & 
```

这个命令以`dev-mode`模式运行 Hashicorp 的 Consul，并将进程放到后台（结尾的`&`符号），将标准输出重定向到日志文件。`2>&1`告诉 Bash“将文件描述符 2（`STDERR`）重定向到和 1（`STDOUT`）相同的位置”——在这种情况下，就是`/var/log/consul.log`。

你已经了解了文件描述符——`STDIN`、`STDOUT`和`STDERR`。如果你只想将标准错误重定向到一个*不同*的文件，而不是标准输出呢？

# 变量插值语法——`${}`

为了在大多数编程语言中实现“字符串插值”——用变量的值替换字符串的一部分——你需要使用 Bash 的变量插值，`${}`。

自己尝试一下：

```
MYNAME=dave
echo "I can't do that, ${MYNAME}." 
```

在 Bash 中还有其他方法可以插入变量，但这是我们最喜欢的方式，因为它在处理形状不确定的输入（空格、特殊字符等）时，最不容易让程序崩溃。

如果你打算将一个变量用作类似字符串的值，请使用这种语法——即使该变量本身并不需要插入到其他字符串中：

```
NAME="${MYNAME}" 
```

这将防止出现许多奇怪的 Bug 和 Bash 中的异常行为，所以这是一个值得养成的好习惯。

**注意**

在处理变量插值时，你几乎总是会希望以`-u`选项运行 Bash（可以通过`-u`调用它，或者在脚本开头使用`set -euo pipefail`，如我们建议的那样）。这将防止你在使用变量之前需要检查其是否为零值。

# Shell 脚本的局限性

Bash 有无数功能，许多我们在这里没有涵盖。如果你需要深入了解 Bash 语言和环境，有许多书籍和大量免费的网络资源可供参考。Bash 手册页（`man bash`）也是一个很好的起点，既然你已经有了基本的了解。

我们预期你在职业生涯中会遇到许多 Bash 脚本。然而，很可能你花更多时间阅读和解读现有脚本，而不是编写大型新的 Bash 程序。Bash 非常适合解决小问题和系统任务，尤其是那些可以通过现有软件解决的问题，只需要将它们拼接成一个解决方案。对于超出标准 Linux 和 Unix 程序组合的大问题，它通常是一个*糟糕*的选择。

使用 Bash 时，我们发现：

+   小巧优于庞大

+   清晰优于巧妙

+   安全总比后悔好

用不同编程语言（通常是 Python）编写的工具替代 Bash 脚本并不少见，尤其是当脚本变得庞大时。这并不是说 Bash 不好！它在其所占据的领域内非常完美，这也是它能长期广泛使用的原因。如果你偶尔停下来问自己，“Bash 脚本仍然是解决这个问题的正确方案吗？”你会没问题的。

# 结论

本章是一个不留情面的、从消防栓中喝水般的 Bash 脚本速成课程。内容密集，但它涵盖了成为高效 Bash 脚本编写者所需的所有基础知识。如果有必要，多看几遍。除了语法外，我们还介绍了我们认为的最佳实践，使得编写可读、可维护的实际脚本变得更容易（或者至少是可能的）。

多加练习，多加练习，多加练习——最好是在你遇到的实际问题上练习，而不仅仅是玩具示例。没有比这更快的提升方式了。

# 引用

+   *Bash 测试和比较函数*。用于[[和((选项表。访问日期：2022 年 9 月 25 日 [`developer.ibm.com/tutorials/l-bash-test/`](https://developer.ibm.com/tutorials/l-bash-test/))。

# 在 Discord 上了解更多

要加入本书的 Discord 社区——在这里你可以分享反馈、向作者提问，并了解新版本的发布——请扫描下面的二维码：

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code1768422420210094187.png)
