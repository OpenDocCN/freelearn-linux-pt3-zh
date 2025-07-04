# 第十章：使用循环

在上一章中，我们开始处理脚本编写，并且学习了很多关于脚本如何工作以及它们是如何结构化的内容。然而，我们忽略了脚本中的一个重要主题——影响脚本执行时命令执行顺序。这里有一些内容需要我们讲解，因为我们有多种方式可以影响脚本中接下来要执行的命令。

我们将从一个概念开始，这个概念叫做 **迭代器** 或更常见的 **循环**。日常任务中有很多事情需要重复执行，通常每次迭代只改变其中的一个小部分。这就是循环发挥作用的地方。

在本章中，我们将介绍以下配方：

+   `for` 循环

+   `break` 和 `continue`

+   `while` 循环

+   `test-if` 循环

+   `case` 循环

+   使用 `and`、`or` 和 `not` 进行逻辑循环

# for 循环

当我们谈论循环时，我们通常会根据变量值变化的执行位置来区分。`for` 循环在这方面属于那一类，在每次迭代之前设置变量，并保持其值直到下一次迭代运行。我们将通过使用 `for` 循环来执行的最常见任务是使用循环遍历一组事物，通常是数字或名称。

## 准备工作

在开始介绍不同的 `for` 循环使用方式之前，我们需要先讲解它的抽象形式：

```
for item in [LIST]
Do
  [COMMANDS]
done
```

我们这里有什么？首先需要注意的是，我们有一些保留关键字，这些关键字使得 Bash 明白我们要使用 `for` 循环。在这个特定的例子中，`item` 实际上是变量的名称，它将持有列表中每次循环迭代的一个值。`in` 这个词是一个关键字，进一步帮助我们理解我们将使用一组值，这组值目前我们称之为列表，尽管它也可以是其他东西。

在列表之后，有一个块，定义了我们每次执行循环时打算运行的命令。目前，我们将把这个块当作一个整体来处理，里面包含的命令会一个接一个地执行，直到没有中断。稍后在本章中，我们将介绍一些条件分支，这将使我们能够覆盖更多可能的工作流解决方案，但目前为止，块是不可中断的。

可能会让你感到惊讶的是，`for` 循环通常是直接从命令行中使用的，甚至比在脚本中使用的次数还要多。原因很简单——有很多任务我们可以通过使用简单的 `for` 循环来完成，反而通过创建脚本来让它们变得复杂是应该避免的。以这种形式写的 `for` 循环看起来与我们第一次示例中展示的有些不同，主要区别在于当我们用一行代码编写循环时，关键字之间使用了分号分隔。

## 如何操作…

让我们从一个简单的例子开始。我们将遍历一个服务器列表，并在每次循环迭代中输出其中一个。注意，shell 会从提示符获取我们的命令，并在执行前将其重复一遍：

```
root@cli1:~# for name in srv1 srv2 srv3 ;do echo $name; \
done; 
Srv1
Srv2
srv3
```

在测试循环时，使用`echo`作为占位符命令是很常见的。我们将在示例中多次使用这种调试风格。`echo`作为命令在这种情况下可能是最有用的，因为它不会改变任何东西，同时使我们能够看到实际的输出结果。

在创建对象列表时，我们不需要使用任何特殊字符来分隔各个条目；bash 会将空格作为分隔符，只要我们用空格分隔值，`bash`就能理解我们的意图。稍后，我们会展示如何更改列表中分隔值的字符，但空格在几乎所有情况下都能正常工作。

我们在迭代中使用的列表可以明确定义，但更多时候，我们需要在运行循环时创建它，无论是在命令行还是脚本中。

这方面的典型例子是对目录中的一组文件运行循环。实现这个的方法是使用 shell 扩展。这意味着让 shell 运行一个命令，然后将其输出作为`for`循环的列表。我们可以通过反引号（`` ` ``）指定命令，或者使用`bash`的`$`（命令）语法。两者的结果相同——命令运行后输出会被传递给列表。

我们的示例将是一个循环，它遍历当前目录，并对每个文件运行`file`命令，向我们提供关于该文件实际内容的信息。我们仍然处于命令行环境：

```
root@cli1:~# for name in `ls`; do file $name; done;
donebackups.lst: ASCII text
snap: directory
testfile: empty
```

现在，让我们处理一些更有趣的内容。通常，我们需要在循环中使用数字，无论是为了计数还是创建其他对象。几乎所有编程语言都有某种类型的循环来实现这一点。Bash 在这方面有些例外，因为它能通过几种不同的方法来实现这一功能。其中一种方法是使用`echo`命令并加一点 shell 扩展来完成此任务。

如果你不熟悉这个，当给`echo`传递一个由大括号格式化的数字作为参数时，它将输出你指定区间中的所有数字：

```
demo@cli1:~/scripting$ echo {0..9}
0 1 2 3 4 5 6 7 8 9
```

要在循环中使用它，我们只需要做与前一个示例中相同的技巧：

```
root@cli1:~# for number in `echo {0..9}`; do echo $number; \
done; 
for number in `echo {0..9}`; do echo $number; done; 
0
1
2
3
4
5
6
7
8
9
```

我们并不局限于使用固定的步长；如果我们仅提到一个区间后面跟着一个数字，那么这个数字将被视为步长值。步长值本质上是指在每次循环迭代中，变量将增加的数值。

我们将尝试一个使用`20`倍数的简单循环：

```
root@cli1:~# for number in `echo {0..100..20}`; do echo \
$number; done; 
for number in `echo {0..100..20}`; do echo $number; done; 
0
20
40
60
80
100
```

我们可以像在命令行中那样结合使用 shell 扩展，为我们的循环创建不同的值。例如，为了为三组服务器创建服务器名称，每组包含六个服务器，我们可以使用一个简单的单行循环：

```
root@cli1:~# for name  in srv{l,w,m}-{1..6}; do echo $name; \
done;
srvl-1
srvl-2
srvl-3
srvl-4
srvl-5
srvl-6
srvw-1
srvw-2
srvw-3
srvw-4
srvw-5
srvw-6
srvm-1
srvm-2
srvm-3
srvm-4
srvm-5
srvm-6
```

当然，循环可以嵌套，只需将内层循环放入外层循环的 `do-done` 块中。在这个特定的示例中，我们使用 shell 扩展在两个循环中遍历一个值列表：

```
root@cli1:~# for name  in {user1,user2,user3,user4}; do \
for server in {srv1,srv2,srv3,srv4}; do echo "Trying to ssh \
$name@$server"; done;done; 
Trying to ssh user1@srv1
Trying to ssh user1@srv2
Trying to ssh user1@srv3
Trying to ssh user1@srv4
Trying to ssh user2@srv1
Trying to ssh user2@srv2
Trying to ssh user2@srv3
Trying to ssh user2@srv4
Trying to ssh user3@srv1
Trying to ssh user3@srv2
Trying to ssh user3@srv3
Trying to ssh user3@srv4
Trying to ssh user4@srv1
Trying to ssh user4@srv2
Trying to ssh user4@srv3
Trying to ssh user4@srv4
```

## 它是如何工作的……

现在是时候慢慢地从命令行过渡到如何在脚本中使用这些循环了。这里最大的区别是，`for` 循环在脚本中格式化后要比命令行更容易阅读。

对于我们的第一个例子，我们将提到另一种在循环中创建数字集合的方法，所谓的 *C 风格循环*。正如名字所示，这种循环的语法来自 C 语言。每个循环有三个单独的值。其中前两个是强制性的；第三个不是。第一个值称为 *初始化值* 或 *起始值*。它给我们提供了变量在第一次循环迭代中的值。这里需要注意的一点是，我们需要显式地分配初始值，这与 *常规* `for` 循环中通常使用的风格有显著不同。

这个循环变体中的第二个值是 *测试条件*，有时也叫 *边界条件*。它表示在我们完成循环之前，循环迭代器所能达到的最后一个有效值，或者更简单地说，就是当我们按递增顺序计数时的最大值。

第三个值可以省略；它将默认为 `1`。如果我们使用它，这将是循环将使用的默认步长或增量。

从理论上讲，这个 C 风格的 `for` 循环将是这样的：

```
for ((INITIALIZATION; TEST; STEP))
Do
  [COMMANDS]
done
```

实际上，它有更复杂的语法，但对于所有有 C 编程经验的人来说，它看起来非常熟悉，正如名字所示：

```
for ((i = 0 ; i <= 100 ; i=i+20)); do
  echo "Counter: $i"
done
```

在我们继续之前，让我们看一个我们已经使用过的循环示例，并以脚本中的格式呈现：

```
#!/usr/bin/bash
# for loop test script 1 
for name  in {user1,user2,user3,user4}; do
        for server in {srv1,srv2,srv3,srv4}; do
                echo "Trying to ssh $name@$server"
        Done
done
```

正如我们所看到的，唯一的实际区别是格式化和省略分号，这是因为我们不需要在一行中解析整个脚本。

## 另请参见

为了理解循环，你可能需要一些例子。从这些链接开始：

+   [`linuxhint.com/30_bash_loop_examples`](https://linuxhint.com/30_bash_loop_examples)/

+   [`tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-7.html`](https://tldp.org/HOWTO/Bash-Prog-Intro-HOWTO-7.html)

# break 和 continue

到目前为止，我们在脚本中实际上并没有做任何条件分支。我们做的所有事情都是线性的，即使是循环也是如此。我们的脚本能够逐行执行命令，从第一行开始，如果有循环，它会一直运行，直到满足我们在循环开始时定义的条件。这意味着我们的循环有一个固定的、预定的迭代次数。有时候，或者更准确地说，通常我们需要做一些事情来打破这种想法。

## 准备就绪

假设有这样一个例子——你有一个循环，它必须迭代若干次，*除非*满足某个条件。我们说过，循环的迭代次数是在循环开始时就固定的，所以显然我们需要一种方法来提前结束循环。

这就是为什么我们有一个叫做 `break` 的命令。顾名思义，这个命令通过跳出它所在的命令块来打破循环，并结束循环，无论循环的定义中使用了什么条件。之所以这么重要，是因为它能帮助我们控制循环，处理任何可能要求我们在循环中不完成任务的状态。还需要注意的是，`break` 命令不仅限于 `for` 循环；它可以在任何其他代码块中使用，等到我们学习如何将脚本结构化为块时，这会变得更加有用。

## 如何实现…

开始时使用一个例子总是比较容易，但在这个特定的案例中，我们将从这个命令如何工作的整体视角开始。我们将使用抽象命令代替实际命令，以帮助你理解这个循环的结构。之后，我们会创建一些实际的例子：

```
for I in 1 2 3 4 5
do
#main part of the loop, will execute each time loop is started
  command1      
  command2
#condition to meet if we need to break the loop
  if (break-condition)
  then
#Leave the loop
      break          
  fi
#This command will execute if the condition is not met
  statements3              
done
command4
```

这里发生了什么？`for` 循环本身是一个正常的循环，它使用 `1`、`2`、`3`、`4` 和 `5` 作为值来执行。`command1` 和 `command2` 会按照预期至少执行一次，因为它们是循环开始后的第一个命令。

`if` 语句是事情变得有趣的地方。我们将会更多地讨论 `if` 语句，但我们需要在这里提到它们的最基本形式。在这里，我们有一个叫做终止条件的东西。它可以是任何可以解析为逻辑值的东西。这意味着我们的条件结果必须是 `true` 或 `false`。如果结果为 `false`，则条件未满足，循环将继续执行 `command3`，并回到循环的开始，给变量赋予下一个值。

我们更感兴趣的是，如果`break`条件为真时会发生什么。这意味着我们已经满足了条件，并需要运行后续的代码块。这里有一个简单的`break`语句，它没有参数。接下来发生的事情是，脚本会立即退出循环并执行`command4`以及其后的所有内容。重要的是，`command3`在这种情况下不会被执行，循环也不会重复，无论循环变量的值是什么。

还有一个叫做`continue`的语句也很有用，虽然它不像`break`那样使用频繁。`Continue`也以某种方式跳出循环，但不是永久性的。一旦在循环中使用`continue`，程序的流程将立即跳转到循环块的开始，而不执行剩余的语句。

## 它是如何工作的…

讲完抽象结构后，现在是时候创建一个例子了。

假设我们在使用`for`循环计数，但我们希望一旦变量的值为`4`时就跳出循环。当然，我们也可以通过简单地将`5`指定为我们计数的上限来实现这一点，但我们需要展示循环的工作原理，因此我们将使用`break`语句跳出循环：

```
#!/usr/bin/bash
# testing the break command
for number  in 1 2 3 4 5
do 
echo running command1, number is $number
echo running command2, number is $number
if [ $number -eq 4 ]
        Then
                echo breaking out of loop, number is $number
                Break
fi
echo running command3, number is $number
done
```

是时候拆解我们的脚本了，但我们在拆解之前先运行一次：

```
demo@cli1:~/scripting$ bash forbreak.sh 
running command1, number is 1
running command2, number is 1
running command3, number is 1
running command1, number is 2
running command2, number is 2
running command3, number is 2
running command1, number is 3
running command2, number is 3
running command3, number is 3
running command1, number is 4
running command2, number is 4
breaking out of loop, number is 4
```

我们的示例脚本看起来非常像我们的*抽象*示例，但我们用了实际的`echo`命令来模拟应该发生的事情。我们需要讨论的最重要部分是`if`命令；其他部分和我们在食谱第一部分中所说的相同。

我们提到过，为了让`break`语句有意义，我们需要一个条件。在这个特定的例子中，我们使用了带有`test`条件的`if`语句；基本上，我们在告诉`bash`去比较两个值，看看它们是否相等。在`bash`中，有两种方式可以做到这一点——一种是使用我们习惯的`=`操作符，另一种是使用`-eq`或`equals`操作符。这两者的区别在于，`=`用于比较`字符串`，而`-eq`用于比较整数。我们将在后续的食谱中详细讲解它们，因为它们在脚本中非常重要。

现在，让我们看看`continue`命令是如何工作的。我们将稍微修改一下脚本，使其在变量值为`3`时跳过第三个命令：

```
#!/usr/bin/bash
# testing the continue command
for number  in 1 2 3 4 5
do
echo running command1, number is $number
echo running command2, number is $number
if [ $number -eq 3 ]
        Then
                echo skipping over a statement, number is \
$number
                Continue
fi
echo running command3, number is $number
done
```

我们所做的只是简单地修改了`if`语句；我们更改了条件，使其检查变量值是否等于`3`，然后创建了一个命令块，当条件满足时跳过循环的其余部分。运行它很简单：

```
demo@cli1:~/scripting$ bash forcontinue.sh
running command1, number is 1
running command2, number is 1
running command3, number is 1
running command1, number is 2
running command2, number is 2
running command3, number is 2
running command1, number is 3
running command2, number is 3
skipping over a statement, number is 3
running command1, number is 4
running command2, number is 4
running command3, number is 4
running command1, number is 5
running command2, number is 5
running command3, number is 5
```

这里唯一需要注意的是我们完成了所有迭代；唯一跳过的事情是脚本中第三个命令的执行。还需要注意的是，循环中的 `continue` 命令会跳过当前循环到达其末尾并返回到循环的开始，而 `break` 语句则会跳过整个循环并不再重复执行。

## 另见

中断命令流一开始可能是个问题。更多信息请参考以下链接：

+   [`tldp.org/LDP/abs/html/loopcontrol.html`](https://tldp.org/LDP/abs/html/loopcontrol.html)

+   [`linuxize.com/post/bash-break-continue/`](https://linuxize.com/post/bash-break-continue/)

# `while` 循环

到目前为止，我们处理的循环都是固定次数的迭代。原因很简单——如果你使用 `for` 循环，你需要指定循环要运行的值，或者指定变量在循环中将拥有的值。

这种循环方法的问题在于，有时你无法提前知道需要多少次迭代才能完成某个操作。这时，`while` 循环就派上用场了。

## 准备开始

你需要了解的最重要的事情是，`while` 循环在循环开始时进行测试。这意味着我们需要构建我们的脚本，使其在某些条件为真时运行 *while*。这也意味着我们可以创建一个永远不会执行的循环；如果我们创建一个 `while` 循环，其条件未满足，`bash` 将完全不执行它。这具有很多优点，因为它使我们可以灵活地根据需要多次使用循环，而不必考虑边界，并且我们仍然可以在条件未满足时使用 `break` 退出循环。

## 如何实现……

`while` 循环看起来比标准的 `for` 循环更简单；我们有一个必须满足的条件和一个将要执行的命令块。如果条件不满足，命令将不会执行，`bash` 会跳过该块，继续执行 `done` 语句后面的内容，该语句用于终止该块：

```
while [ condition ]; do commands; done
```

条件，在这个例子中，与之前提到的逻辑条件相同。还有一种使用 `while` 循环的方法，通过拥有一个称为 `control-command` 的命令，它会执行并直接提供信息以启动循环。我们将经常使用这种方法，因为它使我们能够，例如，逐行读取文件，而无需预先指定文件的行数：

```
while control-command; do COMMANDS; done
```

## 它是如何工作的……

和往常一样，我们将提供一些示例。首先，我们将重复使用 `for` 循环已经完成的任务。我们的目标是循环直到值达到 `4`，然后结束循环。请注意，值可以是字符串，而不一定是数字：

```
#!/bin/bash
x=0
while [ $x -le 4 ]
do
  echo number is $number
  x=$(( $x + 1 ))
done
```

有几个小点需要强调。第一个是我们使用的条件。在我们的`for`循环中，我们比较了值是否为`4`，然后使用`break`跳出循环。在这种情况下，我们不能这么做；如果检查`x`变量的值是否为`4`，循环将永远不会运行，因为初始值是`1`。

在`while`循环中，我们需要检查相反的条件——我们希望循环一直运行，直到值变为`4`，因此条件必须在所有情况下都为真，*除了*当我们的变量恰好是`4`时。

幸运的是，正是这个`while`关键字帮助我们创建条件。

我们提到过，除了条件之外，我们还可以使用命令。一个你将经常使用的典型例子是读取文件。我们可以使用`for`循环来实现这一点，但这会不必要地复杂。`for`循环需要在开始之前知道迭代的次数。为了使用`for`循环解决这个问题，我们需要在开始循环之前先统计文件中的行数，而这既复杂又慢，因为它需要我们打开文件两次——第一次是统计行数，第二次是在循环中读取。

一个更简单的方法是使用`while`循环。我们只需在命令有输出时运行循环——在本例中，就是在从文件中读取内容时。只要命令失败，循环就结束：

```
#!/bin/bash
FILE=testfile.txt
# read testfile and display it one line at a time
while read line
do
     # just write out the line prefixed by >
     echo "> $line"
done < $FILE
```

你会注意到，在这些脚本中有一些我们还没有看到的内容。第一个是变量的使用。当我们处理`for`语句时，某种程度上我们已经做过这件事了，但在这里你可以看到变量是如何声明的，以及它是如何被使用的。稍后我们会详细讨论这个问题。另一个问题是我们是如何实际读取文件的。`read`命令没有参数，它是用来处理标准输入的。既然我们知道如何重定向输入输出，我们就可以将文件中的内容重定向为`read`命令的输入。这就是为什么我们在脚本的最后一行使用了重定向。它看起来可能有些不自然，但这是做这件事的正确方式。

有时，我们需要使用一个永不结束的循环，即所谓的无限循环。它看起来与直觉相悖，但这种循环在脚本中非常常见，当我们需要不断运行脚本，却不知道需要多少次迭代时。我们有时甚至希望脚本不断运行，并在发生某些事件时使用`break`语句来停止它。无限的`while`循环很简单；只需将`:`作为条件：

```
#!/bin/bash
while :
do
     echo "infinite loops [ hit CTRL+C to stop]"
done
```

## 另见

+   [`linuxize.com/post/bash-while-loop/`](https://linuxize.com/post/bash-while-loop/)

+   [`tldp.org/LDP/Bash-Beginners-Guide/html/sect_09_02.html`](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_09_02.html)

+   [`www.redhat.com/sysadmin/bash-scripting-while-loops`](https://www.redhat.com/sysadmin/bash-scripting-while-loops)

# 测试-如果循环

严格来说，当谈到循环时，我们通常将它们分为`for`循环和`while`循环。还有一些其他结构，我们有时也称之为循环，尽管它们更像是命令块的结构。这些结构有时被称为`decision`循环或`decision`块，但出于传统原因，它们通常被称为`test-if`循环、`case`循环或`logical`循环。

其背后的主要思想是，任何决策性部分的代码实际上都会将代码分支到包含命令块的不同路径中。由于分支和决策是你在脚本中最常做的事情之一，我们将向你展示一些最常用的结构，这些结构或多或少会出现在你编写的任何脚本中。

## 准备工作

对于这个教程，最重要的是理解，对于任何条件分支，或者说任何你在代码中放入的条件，你将使用逻辑表达式。简单来说，逻辑表达式就是可以为真或为假的语句。

例如，考虑以下语句：

+   `something.txt` 文件存在。

+   数字`2`大于数字`0`。

+   `somedir` 目录存在且 Joe 用户可以读取。

+   `unreadable.txt` 文件任何用户都无法读取。

这里的每个语句都是可以为真或为假的。最重要的是，这里没有其他逻辑状态可以定义在任何语句上。另一个需要注意的点是，这里每个语句都指向一个特定的对象，比如文件、目录或数字，并且给我们该对象的某些属性或状态。

牢记这一点，我们将介绍 Shell 测试的概念，然后使用它来帮助我们编写脚本。

## 如何实现...

我们已经引入了使用`condition`的`if`语句，以便将代码分支到其中一个*已评估*的代码块中。这个条件必须得到满足，这意味着它需要被解析为`true`或`false`语句。然后，`if`命令会决定哪一部分代码将被执行。

这种评估也叫做*测试*，并且在 shell 中有两种方法可以执行测试。`bash` shell 有一个叫做`test`的命令，有时会在脚本中使用。这个命令接受一个*表达式*并对其进行评估，以查看结果是 `true` 还是 `false`。命令的结果不会在输出中打印出来，而是将其*退出状态*分配给相应的值。

退出状态是每个命令在完成后会设置的一个值，我们可以从命令行内部或从脚本中检查它。这个状态通常用来查看是否执行特定命令时发生了错误，或者传递一些信息，比如测试表达式的逻辑值。

为了测试退出状态，我们可以使用一个简单的`echo`命令。让我们通过一些示例，使用简单的表达式和`test`命令来演示。

第一个例子使用`echo`命令输出`test`命令的退出状态。在所有例子中，`0`表示`true`，`1`表示`false`：

```
demo@cli1:~/$ test "1"="0" ; echo $?
0
```

那么，为什么我们得到了`1=0`是对的结果呢？我们故意犯了一个语法错误，目的是向你展示脚本中最常见的错误。所有命令通常都会使用非常严格的语法，而*test*命令也不例外。这个命令的问题在于它不会显示错误；相反，它会将我们的表达式当作*一个单一的参数*来处理，然后认为它是`true`。

我们可以通过使用一个完全无意义的参数来检查这一点，比如一个单词：

```
demo@cli1:~/$ test whatever ; echo $?
0
```

如你所见，结果在逻辑上是对的，即使它在实际中没有任何意义。实际上，`test`命令需要空格来理解表达式的各个部分是运算符还是操作数。我们之前例子的正确写法应该是这样的：

```
demo@cli1:~/$ test "1" = "0" ; echo $?
1
```

这是我们预期的结果。为了检查，我们将尝试评估另一个表达式：

```
demo@cli1:~/$ test "0" = "0" ; echo $?
0
```

所以，这个是对的。完全符合我们的预期。我们使用引号的原因是我们实际上不是在评估数字，而是在比较*字符串*。如果我们去掉引号会怎么样？

```
demo@cli1:~/$ test 0 = 0 ; echo $?
0
```

这也没问题；只是为了检查，我们将重新尝试一个应该是假的表达式：

```
demo@cli1:~/$ test 0 = 1 ; echo $?
1
```

结果也完全是我们预期的样子。现在让我们尝试别的东西。我们曾说过，比较数字和字符串之间是有区别的。一个数字的值是固定的，无论它前面有多少个零：

```
demo@cli1:~/$ test 01 = 1 ; echo $?
1
```

我们的命令现在表示这两个不相等。为什么？因为*字符串*不相等。`Bash`使用不同的运算符来比较字符串和数字，而由于我们为字符串使用了`1`，所以这两个值不相同。即使使用引号也是如此，下面是如何处理引号的演示：

```
demo@cli1:~/$ test "01" = "1" ; echo $?
1 
```

我们应该使用的整数比较运算符是`-eq`；它会理解我们在比较数字，并根据此进行比较：

```
demo@cli1:~/$ test "01" -eq "1" ; echo $?
0
```

无论我们是否使用引号，结果应该是一样的：

```
demo@cli1:~/$ test 01 -eq 1 ; echo $?
0
```

对于最后一个例子，我们将看看当我们把运算符反过来使用并尝试使用整数比较来比较字符串时会发生什么：

```
demo@cli1:~/scripting$ test 0a -eq  0a ; echo $?
bash: test: 0a: integer expression expected
2
```

这个结果意味着什么？首先，我们的测试尝试评估条件，并意识到比较有误，因为它不能比较字符串和整数，或者更准确地说，一个整数不能包含字母。我们在输出中得到了错误，因此命令以`2`状态退出，这表示出现错误。结果在逻辑上没有意义，所以结果既不是`0`也不是`1`。

下一步我们需要做的是将所学内容应用到实际脚本中，但在此之前，我们需要再解决一个问题。创建测试的方式有两种。一种是显式使用`test`命令，另一种是使用方括号（`[ ]`）。虽然在需要根据某些条件在命令行运行某些命令时我们会经常使用`test`，但在使用`if`语句时，我们大多数时候会使用方括号，因为它们更易于书写，并且在浏览脚本时看起来更整洁。为了确保理解，下面是我们使用的一个表达式，以不同的方式写出。请注意方括号内的空格；方括号和我们使用的表达式之间需要有一个空格：

```
demo@cli1:~/$ [ 01 -eq 1 ] ; echo $?
0
```

## 它是如何工作的……

我们将编写一个小脚本来测试文件是否存在于脚本运行的目录中。为此，我们需要讨论一些我们可以使用的其他运算符。

如果你查看`man`页面中的`test`命令或`bash`手册，你会看到有许多不同的测试，我们可以根据想要检查的内容来选择；我们最常用的测试可能如下（直接取自`test(1)`的手册页）：

+   `-d`文件：文件存在并且是一个目录。

+   `-e`文件：文件存在。

+   `-f`文件：文件存在并且是常规文件。

+   `-r`文件：文件存在且具有读取权限。

+   `-s`文件：文件存在并且大小大于零。

+   `-w`文件：文件存在并且具有写入权限。

+   `-x`文件：文件存在并且具有执行（或搜索）权限。

让我们使用这个来创建一个脚本：

```
#!/usr/bin/bash
# testing if a file exists
if [ -f testfile.txt ]
      then
           echo testfile.txt exists in the current directory
      else 
           echo File does not exist in the current directory! 
fi
```

这里最重要的内容可能是学习`else`语句的结构和用法。在`if`语句中，我们定义了两个代码块或*部分*——一个叫做`then`，另一个叫做`else`。它们的作用如其名；如果我们在语句中使用的条件为真，那么`then`代码块将会被执行。如果条件不成立，则会执行`else`代码块。这两个代码块是互斥的；它们中只会有一个被执行。

现在，我们要处理一个有时会让你感到困惑的话题。我们已经提到，脚本有一个它运行的上下文。除了其他事项之外，你每次运行脚本时需要知道两件事——它是从哪里运行的，以及是哪个用户运行了脚本。

这两条信息至关重要，因为它们定义了我们如何引用需要的文件，以及从脚本中可以获得哪些权限。

我们接下来的任务是创建一个脚本，展示如何处理所有这些问题。我们将测试脚本是否可以读取和写入`root`目录，以及该目录是否存在。我们对这个目录的引用将是相对的，因此我们假设脚本是从`/`目录运行的，尽管通常情况下并非如此。然后，我们将尝试在不同的目录和不同的用户下运行脚本，并比较结果：

```
#!/usr/bin/bash
# testing permissions and paths 
if [ -d root ]
     then
           echo root directory exists!
     else 
           echo root directory does NOT exist! 
fi
if [ -r root ]
        then
                echo Script can read from the directory!
        else
                echo Script can NOT read from the directory!    
fi
if [ -w root ]
        then
                echo Script can write to the directory!
        else
                echo Script can not write to the directory!    
fi
```

如你所见，我们基本上在测试三种不同的条件。首先，我们试图检查目录是否存在，其次是检查脚本是否具有读写权限。

首先，我们将在脚本创建的目录中作为当前用户尝试运行它。然后，我们将转到`/`目录并从那里运行它：

```
demo@cli1:~/scripting$ bash testif2.sh 
root directory does NOT exists!
Script can NOT read from the directory!
Script can not write to the directory!
demo@cli1:~/scripting$ cd /
demo@cli1:/$ bash home/demo/scripting/testif2.sh 
root directory exists!
Script can NOT read from the directory!
Script can not write to the directory!
```

这些信息告诉我们什么？第一次运行时，由于我们在脚本中使用了相对路径，所以无法找到该目录。这使得脚本运行时的目录变得非常重要。

我们学到的另一个东西是如何进行检查。我们可以独立检查文件或目录是否存在，以及当前用户对特定文件拥有的不同权限。我们将通过在`root`用户下使用`sudo`命令来运行脚本来演示这一点：

```
demo@cli1:~/scripting$ cd /
demo@cli1:/$ sudo bash home/demo/scripting/testif2.sh 
[sudo] password for demo: 
root directory exists!
Script can read from the directory!
Script can write to the directory!
```

一旦我们改变了上下文，就可以看到同一个脚本不仅能够看到该目录存在，而且还拥有完全的使用权限。

现在，我们将完全修改我们的脚本，演示如何将检查嵌套在一起。我们的脚本将再次测试`root`目录是否在当前目录中，但这次，只有在目录存在时，脚本才会检查是否具有读写权限。毕竟，检查一个不存在的目录是否可以读取是没有意义的：

```
#!/usr/bin/bash
# testing permissions and paths 
if [ -d root ]
        then
                echo root directory exists!
                if [ -r root ]
                      then
                        echo Script can read from the \
directory!
                      else
                        echo Script can NOT read from the \
directory!
                fi
                if [ -w root ]
                      then
                        echo Script can write to the directory!
                      else
                        echo Script can not write to the \
directory!
                fi
        else
                echo root directory does NOT exists!
fi
```

现在，我们将在两个目录中运行它，以查看我们的脚本是否有效；主要的区别应该在于输出。同时，当你有这样的嵌套结构时，始终保持缩进的一致性非常重要。这意味着你应该始终确保同一块中的命令缩进一致，这样可以立刻清楚地知道每个命令属于哪个部分：

```
demo@cli1:~/scripting$ bash testif3.sh
root directory does NOT exists!
demo@cli1:~/scripting$ cd /
demo@cli1:/$ bash home/demo/scripting/testif3.sh 
root directory exists!
Script can NOT read from the directory!
Script can not write to the directory!
```

我们现在已经看到如何在`bash`中使用不同的测试和条件。接下来的话题与此类似——`case`语句或`case`循环。

## 另见

+   [`www.thegeekdiary.com/bash-if-loop-examples-if-then-fi-if-then-elif-fi-if-then-else-fi/`](https://www.thegeekdiary.com/bash-if-loop-examples-if-then-fi-if-then-elif-fi-if-then-else-fi/)

+   [`tldp.org/LDP/Bash-Beginners-Guide/html/sect_07_01.html`](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_07_01.html)

+   [`ryanstutorials.net/bash-scripting-tutorial/bash-if-statements.php`](https://ryanstutorials.net/bash-scripting-tutorial/bash-if-statements.php)

# case 循环

到目前为止，我们已经处理了一些基本命令，这些命令允许我们在编写脚本时完成必要的操作，比如循环、分支、跳出和继续程序流程。`case` 循环，本食谱的主题，并不是严格必要的，因为其背后的逻辑可以通过多层嵌套的 `if` 命令来实现。我们之所以提到这个，纯粹是因为 `case` 是我们在脚本中会频繁使用的东西，而使用 `if` 语句的替代方案不仅难以编写和阅读，而且调试起来也很复杂。

## 准备工作

可以简单地说，`case` 循环或 `case` 语句只是另一种编写多个 `if then else` 测试的方式。`case` 并不能代替普通的 `if` 语句，但有一个常见的情况，在这种情况下，`case` 语句能让我们的生活变得更加简单，脚本也更容易调试和理解。但在我们深入讨论之前，我们需要先了解一些关于变量和分支的知识。一旦我们开始使用 `if` 语句，就会迅速意识到它们大致可以用两种不同的方式。第一种是大家在想到 `if` 语句时最常考虑的方式——我们有一个变量，并将它与另一个变量或一个值进行比较。这是很常见的，也是脚本中经常使用的。稍微不那么常见的是，当我们需要将一个变量与一组值进行比较时。这通常出现在我们需要将事物分类或根据用户输入执行某个代码块时。

用户输入可能是使用 `case` 语句最常见的原因。在脚本中，当我们开始使用参数时，通常会用到它。我们的脚本需要根据用户在运行脚本时选择的参数重新配置内容。稍后当我们开始处理传递参数到脚本时，我们将专门使用 `case` 语句来执行相应的命令。

用户菜单是另一个通过使用 `case` 语句解决的问题；广义来说，每次用户需要对一个问题作出多项选择时，都可以通过 `case` 语句来处理。

## 如何实现…

解释 `case` 语句的最佳方法是通过一个例子。假设一个用户启动了一个脚本，他们有四个选项可以选择脚本执行的操作。现在，我们还没有准备好处理用户输入的方式，所以我们暂且假设有一个变量 `$1`，它包含了以下四个值中的一个——`copy`、`delete`、`move` 和 `help`。我们的脚本需要根据用户的输入执行相应的代码部分。事实上，这就是如何处理参数的方法，不过我们稍后会讨论这一点。

我们的第一个版本将使用 `if – then – elif` 循环：

```
#!/usr/bin/bash
# $1 contains either copy, delete, move or help
if [ $1 = "copy" ]
        then
                echo you chose to copy!
        elif  [ $1 = "delete" ]
                then
                        echo you chose to delete!
        elif  [ $1 = "move" ]
                then
                        echo you chose to move!  
        elif  [ $1 = "help" ]
                then
                        echo you chose help!  
else    
                echo please make a choice!
fi
```

这个方法有效，但有两个问题。一个是如果没有提供参数，它会抛出错误，因为这意味着我们在比较一个值和一个没有值的变量。另一个问题是，即使我们特别注意使用正确的缩进，这段代码也很难阅读。我们将使用`case`语句重新编写：

```
#!/usr/bin/bash
# $1 contains either copy, delete, move or help
case $1 in 
      copy)      echo you chose to copy! ;;
      delete) echo you chose to delete! ;;
      move)   echo you chose to move! ;; 
      help)   echo you chose help!  ;;
      *)    echo please make a choice!
esac
```

你首先会注意到的是，这看起来非常简单和清晰。除了更容易编写之外，如果需要的话，代码也更容易阅读和调试。只需注意两件简单的事——语句块的结束由`esac`定义，这是`case`反过来拼写的，类似于`if`语句通过`fi`来结束。另一个是你必须使用`;;`来终止一行，因为这是在`case`循环中用于分隔选项的符号。

在匹配值时，你还可以使用有限的正则表达式；这也是为什么使用`*` `glob`来表示*零个或多个字符*。

## 它是如何工作的……

现在我们已经了解了更多关于脚本编写的内容，我们将编写一个简单的脚本，在一个目录中搜索一个字符串并告诉我们发生了什么。我们不关心文本在哪里；我们只想知道我们在运行脚本的目录中是否有使用过该文本。

在开始之前，我们需要了解以下内容：

+   `$1` 将保存一个字符串值，这个值是我们要搜索的文本。

+   `$?` 保存了刚刚在脚本中完成的命令的`exit`值。

+   `grep`作为命令返回`0`（如果找到内容），`1`（如果没有找到），或`2`（如果发生错误）。

+   有一个特殊的设备`/dev/null`，如果我们需要消除一些输出，可以使用它。

多亏了`case`语句，这成了一项简单的任务：

```
#!/usr/bin/bash
# $1 contains string we are searching for
grep $1 * &> /dev/null
case $? In
      0)    echo Something was found! ;;
      1)       echo Nothing was found! ;;
      2)        echo grep reported an error! ;;
esac
```

对于最后一个脚本，我们将使用`case`来结合本章中另一个测试目录的脚本，并将其放入一个更大的脚本中。我们将创建一个接受命令和文件名作为参数的脚本。命令将是`check`、`copy`、`delete`或`help`。如果我们指定了`copy`或`delete`，脚本将检查是否有权限执行该任务，然后执行它通常会调用的`echo`命令。

如果我们指定`check`，脚本将检查给定文件的权限：

```
#!/usr/bin/bash
# $1 contains either check, copy, delete or help
#script expects two arguments: a command and a file name
case $1 in 
      copy) 
       echo you chose to copy! 
       if [ -r $2 ]
      then
      echo Script can read the file use cp $2 ~ to copy to \
your home Directory!
      else
      echo Script can NOT read the file!    
      fi
          ;;
      delete) 
       echo you chose to delete! 
           if [ -w $2 ]
             then
             echo Script can write the file, use rm $2 to \
remove it!
             else
             echo Script can NOT read the file!    
           fi
           ;;
      check)
       if [ -f $2 ]
              then
                 echo File $2 exists!
                  if [ -r $2 ]
                        then
                                echo Script can read $2!
                           else
                                echo Script can NOT read $2!
                      fi
                      if [ -w $2 ]
                          then
                                 echo Script can write to $2!
                          else
                                 echo Script can not write to $2!
                      fi
            else
                      echo File $2 does NOT exist!
           fi  ;;
      help)         
      echo you chose help, please choose from check, copy or \
delete!  ;;
      *)   echo please make a choice, available are copy \
check delete and help!
esac
```

我们在这里所做的就是将到目前为止做过的所有事情结合成一个实际执行的脚本。我们之前没有提到的唯一一件事是脚本中的第二个参数`$2`。在这种情况下，我们使用它来获取需要运行命令的文件名。以下是从命令行运行时的效果：

```
demo@cli1:~/scripting$ bash testcas4.sh check testfile.txt
File testfile.txt exists!
Script can read testfile.txt!
Script can write to testfile.txt!
demo@cli1:~/scripting$ bash testcas4.sh check testfile.tx
File testfile.tx does NOT exist!
```

## 另见

当你在脚本中使用`case`时，你会很快发现很多示例是从不同网站复制粘贴的。以下链接是两个很好的资源：

+   https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_07_03.html

+   https://www.shellhacks.com/case-statement-bash-example/

# 使用 `AND`、`OR` 和 `NOT` 进行逻辑循环

在计算机中，逻辑是无法逃避的。我们已经处理了一些可以用来评估条件的操作，但在 `bash` 中还有很多其他操作可以进行。

在这个教程中，我们将介绍一些不同的逻辑运算符，它们有助于我们在脚本编写中解决问题。首先，我们将讨论在命令行上可以完成的操作，然后再将其应用到脚本中。

## 准备工作

首先，让我们快速讨论一下逻辑运算符。到目前为止，我们提到了值为 true 或 false 的表达式。我们还提到了一些 `bash` 中内置的各种不同表达式，因为它们提供了在命令行上日常工作中至关重要的功能。现在是时候谈谈逻辑运算符，它们帮助我们将表达式组合起来，创建复杂的解决方案。我们将从常见的运算符开始：

+   `&&`（逻辑 `AND`）

+   `||`（逻辑 `OR`）

这些运算符的有趣之处在于它们可以直接在命令行上使用。在 `bash` 中，命令行有四种执行命令的方式。其中一种是逐行运行命令。这是我们在交互模式下工作的常见方式。

## 如何实现……

有时，我们需要（或希望）在一行上运行多个命令。这通常通过使用 `;` 来分隔命令，例如：

```
demo@cli1:~/scripting$ pwd ; ls
/home/demo/scripting
backupexample.sh  errorfile  forbreak.sh  forcontinue.sh  forloop1.sh  helloworld.sh  helloworldv1.sh  outputfile  readfile.sh  testcas1.sh
```

如我们所见，这与单独运行每个命令完全相同，但 shell 会按顺序执行它们。当我们使用 `test` 命令测试不同的表达式时，我们已经用到了这一点。我们需要检查该命令的退出状态，因此在运行测试后我们总是直接使用 `echo`。

然而，有时我们可以使用一些逻辑来创建快捷方式。这时，逻辑运算符发挥了作用。剩下的两种运行多个命令的方法，使用它们不仅运行命令，还可以根据条件运行命令。

假设我们想在进行某些测试后执行一个命令——例如，我们想要打开一个文件，但仅当该文件确实存在时。我们可以在这里写一个 `if` 语句，但将事情复杂化完全没有意义。这时我们可以使用逻辑 `AND`：

```
demo@cli1:~/scripting$ [ -f outputfile ] && cat outputfile 
Hello World!
demo@cli1:~/scripting$ [ -f idonotexist ] && cat outputfile
demo@cli1:~/scripting$
```

通常，在命令之间使用 `&&` 告诉 `bash` 只有在左侧命令成功时，才运行右侧的命令。在我们的示例中，这意味着我们在目录中有一个名为 `output` 的文件。在左侧，我们快速测试文件是否存在。一旦测试成功，我们运行 `cat` 来输出文件内容。

在第二个示例中，我们故意使用了错误的文件名，而 `cat` 命令没有被执行，因为文件不存在。

另一个我们可以使用的逻辑运算符是逻辑 `OR`。使用的运算符是 `||`，与之前一样。这个运算符指示 `bash` 只有在左侧命令失败时，才运行右侧的命令：

```
demo@cli1:~/scripting$ [ -f idonotexist ] || cat outputfile 
Hello World!
demo@cli1:~/scripting$ [ -f outputfile ] || cat outputfile
demo@cli1:~/scripting$
```

这是与前一个示例完全相反的情况。我们的`cat`命令只有在测试失败时才会执行。像这样的结构有时会在脚本中使用，以创建故障保护机制或快速运行诸如更新之类的任务。

好的一点是，这使我们能够根据测试的结果立即执行某些操作：

```
demo@cli1:~/$[ -f outputfile ] && echo exists || echo not \
exists
Exists
demo@cli1:~/$[ -f idonotexist ] && echo exists || echo exists \
not
exists not
```

这些运算符也存在于测试表达式中，允许我们创建不同的条件，否则需要多个`if`语句。

## 它是如何工作的…

测试条件现在应该对你来说完全熟悉了。我们将尝试结合几个条件来解释不同运算符的作用。例如，如果我们想快速检查一个文件是否存在并且可读，我们可以通过测试它是否可读，或者明确地将这两者结合成一个语句来实现：

```
demo@cli1:~/$ [ -f outputfile ] &&  [ -r outputfile ] ; echo \
$?
0
```

这些测试在处理字符串和数字时最为有用。例如，我们可以尝试在脚本中检查一个数字是否在一个区间内，如下所示：

```
#!/usr/bin/bash
# testing if a number is in an interval
if [ $1 -gt 1 ]
      then
          if [ $1 -lt 10 ] 
                 then 
                  echo Number is between 1 and 10
          else 
                echo Number is not between 1 and 10
                fi
     else 
            echo number is not between 1 and 10! 
fi
```

我们将要运行这个脚本，但在我们开始之前，可以看到它看起来很复杂，比应有的复杂。问题不仅仅在于我们必须使用两个`if`语句来确保处理外部区间的两个部分；即使这个脚本只有几行，它也需要大量的解释。它能正常工作吗？是的，正如我们在这里看到的：

```
demo@cli1:~/scripting$ bash testmultiple.sh 42
Number is not between 1 and 10
demo@cli1:~/scripting$ bash testmultiple.sh 2
Number is between 1 and 10
demo@cli1:~/scripting$ bash testmultiple.sh -1
number is not between 1 and 10!
```

现在，我们将使用逻辑运算符来优化我们的脚本：

```
#!/usr/bin/bash
# testing if a number is in an interval
if [[ $1 -gt 1  &&  $1 -lt 10 ]]
        then
                        echo Number is between 1 and 10
        else
                        echo Number is not between 1 and 10
fi
```

我们在这里使用双括号是因为我们必须这么做。有多种方式可以实现相同的目标，还有一些旧版本的语法，但处理多个表达式时，最好使用双括号。

## 另见

处理逻辑运算符部分之所以复杂，是因为它们种类繁多。你可以在这里找到更多信息：

+   [`linuxhint.com/bash_operator_examples/#o23`](https://linuxhint.com/bash_operator_examples/#o23)

+   [`opensource.com/article/19/10/programming-bash-logical-operators-shell-expansions`](https://opensource.com/article/19/10/programming-bash-logical-operators-shell-expansions)
