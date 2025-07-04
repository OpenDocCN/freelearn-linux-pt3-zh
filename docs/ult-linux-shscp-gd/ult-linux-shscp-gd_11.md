# 11

# 执行数学运算

各种操作系统的 shell 都可以从命令行或 shell 脚本中执行数学运算。在本章中，我们将讨论如何使用整数和浮点数数学进行运算。

本章包括以下内容：

+   使用表达式进行整数运算

+   使用整数变量进行整数运算

+   使用`bc`进行浮点数学运算

如果你准备好了，我们开始吧。

# 技术要求

你可以使用任何 Linux 虚拟机来进行此操作。而且，和往常一样，你可以通过以下方式下载脚本：

```
git clone https://github.com/PacktPublishing/The-Ultimate-Linux-Shell-Scripting-Guide.git 
```

# 使用表达式进行整数运算

你可以直接在`bash`中进行整数运算，这有时非常方便。但是，`bash`没有进行浮点运算的能力。对于浮点运算，你需要使用一个单独的工具，稍后我们会讨论。

如果你尝试使用`echo`在命令行上执行数学运算，你会发现它无法正常工作。你得到的结果可能像这样：

```
[donnie@fedora ~]$ echo 1+1
1+1
[donnie@fedora ~]$ echo 1 + 1
1 + 1
[donnie@fedora ~]$ 
```

这是因为`echo`将你的数学问题视为普通文本字符串。所以，你需要其他方法来解决数学问题。幸运的是，有几种不同的方法可以做到这一点。

## 使用`expr`命令

`expr`命令用于计算表达式。这些表达式可以是普通文本字符串、正则表达式或数学表达式。现在，我将仅讨论如何使用它来计算数学表达式。以下是其基本用法示例：

```
[donnie@fedora ~]$ expr 1 + 1
2
[donnie@fedora ~]$ 
```

注意，你需要在运算符和每个操作数之间留有空格，否则你会得到如下结果：

```
[donnie@fedora ~]$ expr 1+1
1+1
[donnie@fedora ~]$ 
```

如果没有空格，`expr`只是将你输入的内容原样回显。

你可以使用`expr`与`+`、`-`、`/`、`*`和`%`运算符来执行加法、减法、除法、乘法或取余操作。（取余操作会显示除法操作后的余数。）使用`*`运算符时需要特别注意，因为在 shell 中它会被解释为通配符。因此，在执行乘法时，你需要用`\`对`*`进行转义，像这样：

```
[donnie@fedora ~]$ expr 3 \* 3
9
[donnie@fedora ~]$ 
```

如果在`*`前面没有反斜杠，你将会收到一个错误，错误信息如下：

```
donnie@fedora:~$ expr 3 * 3
expr: syntax error: unexpected argument '15827_zip.zip'
donnie@fedora:~$ 
```

对于更复杂的问题，正常的数学规则适用，如你所见：

```
[donnie@fedora ~]$ expr 1 + 2 \* 3
7
[donnie@fedora ~]$ 
```

数学法则规定，除法和乘法总是优先于减法和加法。所以，你可以看到`2 * 3`的运算先于加`1`。但是，就像在常规数学中一样，你可以通过在你想先执行的操作周围放置一对括号来改变运算顺序。它看起来是这样的：

```
[donnie@fedora ~]$ expr \( 1 + 2 \) \* 3
9
[donnie@fedora ~]$ 
```

注意我如何使用反斜杠转义每个括号符号，并且在`(`和`1`之间、`2`和`\`之间留有空格。

你也可以在`expr`中使用变量。为了演示这一点，创建一个名为`math1.sh`的脚本，像这样：

```
#!/bin/bash
val1=$1
val2=$2
echo "val1 + val2 = " ; expr $val1 + $val2
echo
echo "val1 - val2 = " ; expr $val1 - $val2
echo
echo "val1 / val2 = " ; expr $val1 / $val2
echo
echo "val1 * val2 = " ; expr $val1 \* $val2
echo
echo "val1 % val2 = " ; expr $val1 % $val2 
```

使用 88 和 23 作为输入值运行脚本，看起来像这样：

```
[donnie@fedora ~]$ ./math1.sh 88 23
val1 + val2 =
111
val1 - val2 =
65
val1 / val2 =
3
val1 * val2 =
2024
val1 % val2 =
19
[donnie@fedora ~]$ 
```

请记住，`expr`只能处理整数。所以，任何涉及小数的结果都会四舍五入到最接近的整数。

当然，你也可以像在这个`math2.sh`脚本中一样，使用`expr`和命令替换。

```
#/bin/bash
result1=$(expr 20 \* 8)
result2=$(expr \( 20 + 4 \) / 8)
echo $result1
echo $result2 
```

这就是`expr`的全部内容。接下来，让我们再看看`echo`。

## 使用`echo`与数学表达式

我知道，我刚刚告诉你不能使用`echo`来执行数学运算。嗯，实际上你是可以的，但有一种特殊的方式。你只需要将数学问题放入`$(( ))`构造或`$[ ]`构造中，像这样：

```
[donnie@fedora ~]$ echo $((2+2))
4
[donnie@fedora ~]$ echo $[2+2]
4
[donnie@fedora ~]$ 
```

两种构造给出的结果是一样的，所以——至少在`bash`中——你使用哪种只是个人喜好的问题。

这两个构造的一个非常酷的地方是，你不需要使用`$`来回调它们内部任何变量的值。同样，你也不需要用反斜杠来转义`*`字符。以下是`math3.sh`脚本，展示给你看：

```
#!/bin/bash
val1=$1
val2=$2
val3=$3
echo "Without grouping: " $((val1+val2*val3))
echo "With grouping: "  $(((val1+val2)*val3)) 
```

你还可以看到，在第二个`echo`命令中，我用一对括号将`val1+val2`括起来，以便让加法运算优先于乘法运算。无论如何，以下是我运行脚本时发生的情况：

```
[donnie@fedora ~]$ ./math3.sh 4 8 3
Without grouping:  28
With grouping:  36
[donnie@fedora ~]$ 
```

如果你觉得使用嵌套括号太混乱，你可能想改用方括号构造，像这样：

```
#!/bin/bash
val1=$1
val2=$2
val3=$3
echo "Without grouping: " $[val1+val2*val3]
echo "With grouping: "  $[(val1+val2)*val3] 
```

无论如何，你会得到相同的结果。

不过，这里有一个陷阱。方括号构造在某些其他 shell 中不起作用，比如 FreeBSD 和 OpenIndiana 中的`/bin/sh`，以及 Debian 及其衍生版中的`/bin/dash`。所以，为了让你的脚本更具可移植性，你需要使用`((..))`构造来处理数学问题，尽管它可能会让人有点困惑。

这里是一个更实用的例子，展示了如何使用括号来改变运算的优先级。在这个`new_year.sh`脚本中，我在计算距离新年还有多少周。我首先使用`date +%j`命令来计算当年的天数。以下是该命令的输出：

```
[donnie@fedora ~]$ date +%j
304
[donnie@fedora ~]$ 
```

我是在 10 月 31 日写的这个，这一天是 2023 年的第 304^(天)。然后，我将这个结果从一年中的天数中减去，并将其除以 7，得到最终的答案。以下是脚本的样子：

```
#!/bin/bash
echo "There are $[(365-$(date +%j)) / 7 ] weeks left until the New Year." 
```

你可以看到，我使用了方括号构造，以避免有太多嵌套括号带来的混乱。但正如我之前所说，这在某些非`bash`的 shell 中是无法使用的。

这是我运行脚本时得到的结果：

```
[donnie@fedora ~]$ ./new_year.sh
There are 8 weeks left until the New Year.
[donnie@fedora ~]$ 
```

当然，你可能正在 2024 年、2028 年或甚至 2032 年阅读这篇文章，那些年份都是闰年，有 366 天，但这没关系。那一天的额外天数对这个特定的数学问题不会产生影响。

好吧，再来一个带有数学表达式的脚本示例？创建一个`math4.sh`脚本，像这样：

```
#!/bin/bash
start=0
limit=10
while [ "$start" -le "$limit" ]; do
  echo "$start... "
  start=$["$start"+1]
done 
```

这个脚本以 0 作为`start`的初始值开始。然后它打印出`start`的值，将其增加 1，再打印下一个值。循环会一直进行，直到达到`limit`的值。以下是它的运行结果：

```
[donnie@fedora ~]$ ./math4.sh
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
[donnie@fedora ~]$ 
```

这基本上涵盖了使用数学表达式的内容。现在让我们来看一下使用新类型变量的方法。

# 使用整数变量进行整数数学运算

你可以使用**整数变量**来代替数学表达式。

你已经看到了不适用的情况，它看起来是这样的：

```
[donnie@fedora ~]$ a=1
[donnie@fedora ~]$ b=2
[donnie@fedora ~]$ echo $a + $b
1 + 2
[donnie@fedora ~]$ 
```

这是因为默认情况下，变量的值是文本字符串，而不是数字。为了使其正常工作，可以使用`declare -i`命令来创建整数变量，像这样：

```
[donnie@fedora ~]$ declare -i a=1
[donnie@fedora ~]$ declare -i b=2
[donnie@fedora ~]$ declare -i result=a+b
[donnie@fedora ~]$ echo $result
3
[donnie@fedora ~]$ 
```

这是`math5.sh`脚本中的实现方式：

```
#!/bin/bash
declare -i val1=$1
declare -i val2=$2
declare -i result1=val1+val2
declare -i result2=val1/val2
declare -i result3=val1*val2
declare -i result4=val1-val2
declare -i result5=val1%val2
echo "Addition: $result1"
echo "Division: $result2"
echo "Multiplication: $result3"
echo "Subtraction: $result4"
echo "Modulus: $result5" 
```

在`declare -i`命令中，你不需要在变量名前加上`$`来调用它们的值。你也不需要使用命令替换来将数学运算的结果赋值给变量。总之，以下是我运行它时的样子：

```
[donnie@fedora ~]$ ./math5.sh 38 3
Addition: 41
Division: 12
Multiplication: 114
Subtraction: 35
Modulus: 2
[donnie@fedora ~]$ 
```

由于这是整数运算，任何包含小数的结果都会被四舍五入到最近的整数。

有时，整数运算已经足够了。但如果你需要更多呢？那就在下一部分，我们敬请期待。

# 使用 bc 进行浮点数学运算

你刚才看到的几种从 shell 执行数学运算的方法都有两个限制。首先，这些方法只能处理整数。其次，当使用这些方法时，你只能进行基本的数学运算。幸运的是，`bc`工具解决了这两个问题。事实上，要充分利用`bc`的特性，你需要成为一名数学专家。（我不属于这个类别，但我还是可以向你展示使用`bc`的基本操作。）

你应该会发现`bc`已经安装在你的 Linux 或 Unix 系统上，因此你可能不需要再去安装它。

使用`bc`有三种方式，分别是：

+   交互模式：你只需打开`bc`，然后在其命令行中输入数学命令。

+   程序文件：在`bc`语言中创建程序，并使用`bc`执行它们。

+   将数学问题通过管道传递给 bc：你可以从 shell 命令行或 shell 脚本中执行此操作。

让我们先看看交互模式。

## 在交互模式下使用 bc

你可以通过在命令行输入`bc`来启动`bc`的交互模式，像这样：

```
[donnie@fedora ~]$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
```

现在，你只需在`bc`命令提示符下输入一个数学问题。让我们先从 3 除以 4 开始，如下所示：

```
3/4
0 
```

你会看到结果是 0。但等等，`bc`不应该支持浮点运算吗？当然支持，但你必须使用`-l`选项启动它，以加载可选的数学库。所以，我们先输入`quit`退出并重新启动，如下所示：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
3/4
.75000000000000000000 
```

如果你不想看到这么多小数位，可以使用`scale`命令。假设你只想看到两位小数，可以这样设置：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
scale=2
3/4
.75 
```

现在，让我们做一些更实际的事情，来解决一些几何问题。正如你可能知道的，三角形有三个角，三个角度的总和总是等于 180 度，正如你在这里看到的：

![](img/B21693_11_01.png)

图 11.1：一个三角形，三个 60 度的角

例如，你可以有一个三角形，角度分别为 40 度、50 度和 90 度。然而，有时候你可能只知道两个角度的度数，并且需要找出第三个角度的度数。你可以通过将已知的两个角度的度数相加，并从 180 中减去它们的和来得到第三个角度。下面是在 `bc` 交互模式下的计算方式：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
a=40
b=50
180-(a+b)
90 
```

这也演示了如何在 `bc` 中使用变量。这非常方便，因为如果我想在另一组数值上重新运行计算，我只需要输入新的变量赋值。然后，我只需使用键盘上的上箭头键回到公式。

接下来，我们来看一个直径为 25，半径为 12.5 的圆。（单位不重要，可以是英寸、厘米、英里或公里，实际上都没关系。）要计算圆的周长，我们需要将圆的直径乘以 `pi`（Π）的值，如下所示：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
pi=3.14159265358979323846
diameter=25
circumference=pi*diameter
print circumference
78.53981633974483096150 
```

要计算圆的面积，使用 Π 乘以半径的平方，如下所示：

```
radius=12.5
area=pi*(radius²)
print area
490.87385212340519350937 
```

你在这里看到，`^` 用来表示指数运算，在这个例子中是 2。你还会看到，在 `bc` 语言中没有 `echo` 命令。相反，你需要使用 `print`。

除了处理浮点数学的功能外，你还会在可选库中找到许多有用的函数。例如，你可以使用 `ibase` 和 `obase` 函数将数字从一种数字系统转换为另一种。在这里，你看到我将一个十进制数转换为十六进制数：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
obase=16
10
A 
```

`obase=16` 行告诉 `bc` 我希望所有的数字以十六进制格式输出。我不需要使用 `ibase` 行来指定输入的数字系统，因为它默认是十进制。当我输入 10 作为要转换的数字时，得到的结果是 A，这就是十进制数字 10 的十六进制等价。我还可以将十六进制转换回十进制，像这样：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
ibase=16
A
10 
```

同样，由于十进制是默认值，我不需要为 `obase` 指定它。（如果我在之前的例子中设置了 `obase` 为 16 后没有关闭并重新打开 `bc`，那我就得指定了。）

这是同时设置 `ibase` 和 `obase` 的一个例子：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
ibase=2
obase=16
1011101
10110 
```

在这个例子中，我选择将一个二进制数转换为十六进制。

你还可以将 `ibase` 和 `obase` 设置为相同的值，以便在不同的数字系统中进行数学运算。以下是如何进行二进制运算的示例：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
ibase=2
obase=2
101-1
100
101+101
1010
101/101
1.000000000000000000000000000000000000000000000000000000000000000000\
0 
```

是的，我希望在上世纪八十年代学习二进制数学时能有这个，那时候会方便很多。但是要注意，在除法命令中，有太多的尾随零，所以使用`\`来将它们继续到下一行。

你可以使用`scale=`命令来进行更改，但是在二进制模式下使用它时，可能会得到一些令人惊讶的结果。这里是我的意思：

```
scale=2
101/101
1.0000000
scale=1
101/101
1.0000 
```

我不知道为什么是这样，但没关系。

提示

请注意，如果你决定将一个十六进制数转换为另一种格式，那么字母 A 到 F 必须以大写形式输入。

`bc` 库中的其他函数包括：

+   `s (x)`: x 的正弦，单位是弧度。

+   `c (x)`: x 的余弦，单位是弧度。

+   `a (x)`: x 的反正切，单位是弧度。

+   `l (x)`: x 的自然对数。

+   `e (x)`: 对数 e 的 x 次幂。

+   `j (n,x)`: x 的整数阶贝塞尔函数。

例如，假设你需要找到数字 80 的自然对数。只需像这样做：

```
[donnie@fedora ~]$ bc -l
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
l(80)
4.38202663467388161226 
```

好的，我想这应该涵盖了交互模式。现在让我们看看一些`bc`程序。

## 使用 bc 程序文件

使用交互模式的主要问题是，一旦关闭`bc`，所有的工作都会消失。使你的工作永久化的一种方法是创建一个程序文件。让我们开始创建`geometry1.txt`文件，它看起来像这样：

```
print "\nGeometry!\n"
print "Let's say that you want to calculate the area of a circle.\n"
print "Enter the radius of the circle: "; radius = read()
pi=3.14159265358979323846
area=pi*(radius²)
print "The area of this circle is: \n"
print area
print "\n"
quit 
```

你已经看到如何进行数学计算了，所以我不会再重复了。但是，我想让你注意到`print`命令不会自动在行末插入换行符。因此，在你的`print`命令末尾加上`\n`序列来手动换行。另外，请注意第 3 行如何使用`read()`函数接收用户输入并将其赋值给`radius`变量。最后一个命令必须是`quit`，否则程序将无法退出。要运行此程序，只需输入`bc -l`，然后输入程序文件的名称，就像这样：

```
[donnie@fedora ~]$ bc -l geometry1.txt
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
Geometry!
Let's say that you want to calculate the area of a circle.
Enter the radius of the circle: 20
The area of this circle is:
1256.63706143591729538400
[donnie@fedora ~]$ 
```

**提示**

我在这里有点作弊，从一个已经计算好的网站上复制了 П 的值。如果你愿意自己计算 П 的值，可以用这个公式：

```
pi=4*a(1) 
```

下一个示例是我从`bc`文档页面借来的支票簿平衡程序。它的样子是这样的：

```
scale=2
print "\nCheck book program\n!"
print "  Remember, deposits are negative transactions.\n"
print "  Exit by a 0 transaction.\n\n"
print "Initial balance? "; bal = read()
bal /= 1
print "\n"
while (1) {
  "current balance = "; bal
  "transaction? "; trans = read()
  if (trans == 0) break;
  bal -= trans
  bal /= 1
}
quit 
```

在这里，你可以看到`bc`语言具有与普通 Shell 脚本中已经看到的相同的编程结构。（在这种情况下，你看到了一个`while`循环。）`bc`语言对它们实现有些不同，但没关系。接下来要注意的是`scale=2`和`bal /= 1`这两行。这两个命令确保程序的输出始终保留小数点后两位，即使你只输入一个没有小数的整数。为了说明我的意思，打开交互模式中的`bc`，并输入这些命令：

```
[donnie@fedora ~]$ bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
scale=2
bal=1000
print bal
1000
bal /= 1
print bal
1000.00 
```

你看到在我调用`bal /= 1`命令之前，`print bal`只显示 1000，没有小数点。所以，为什么这样做有效呢？其实就是`bal /= 1`命令是表达除以 1 的简写方式。换句话说，它做的和`bal=(bal/1)`命令完全一样，只不过少了些输入。在这种情况下，我们将 1000 除以 1，结果还是 1000。但因为我们将 scale 设置为 2，所以任何数学操作的结果打印出来时都会显示两位小数。

程序文件中的下一个需要注意的地方是`bal -= trans`这一行。`-=`运算符使得余额根据代表财务交易的`trans`数值减少。我真不明白程序作者为什么这么做，因为这意味着用户必须输入一个正数来减少余额，输入负数来增加余额。

将这一行改为`bal +=` trans 会更有意义。这样，负数代表扣款，正数代表存款，一切都会顺利进行。无论如何，我有点跑题了。

运行程序的效果如下：

```
[donnie@fedora ~]$ bc checkbook_bc
bc 1.07.1
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006, 2008, 2012-2017 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'.
Check book program
!  Remember, deposits are negative transactions.
  Exit by a 0 transaction.
Initial balance? 1000
current balance = 1000.00
transaction? 
```

现在，只需输入你的交易记录，然后输入 0 退出。

你可以用`bc`程序文件做更多的事情，但我想你已经掌握了大致的思路。现在，让我们看看如何在普通的 Shell 脚本中使用`bc`。

## 在 Shell 脚本中使用 bc

使用`bc`的第三种也是最后一种方式是从普通的 Shell 环境中运行`bc`命令。所以，你可以从 Shell 命令行运行`bc`命令，或者将它们放入普通的 Shell 脚本中。这里有一个示例：

```
[donnie@fedora ~]$ pi=$(echo "scale=10; 4*a(1)" | bc -l)
[donnie@fedora ~]$ echo $pi
3.1415926532
[donnie@fedora ~]$ 
```

在这里，我使用命令替换将一个值赋给`pi`。在命令替换构造中，我将一个`bc`风格的数学公式传递给`bc`。你首先看到的是我将 scale 设置为 10。`4*a(1)`意味着我正在计算 1 的反正切，并将结果乘以 4，这是你可以用来近似`pi`（π）值的许多公式之一。（记住，π是一个无理数，这意味着你永远无法得到它的准确值。）

现在，让我们将这些内容放入`pi_bc.sh`脚本中，它看起来是这样的：

```
#!/bin/bash
if [ -z $1 ]; then
        echo "Usage:"
        echo "./pi_bc.sh scale_value"
else
        pi=$(echo "scale=$1; 4*a(1)" | bc -l)
        echo $pi
fi 
```

我稍微优化了一下，允许你在调用脚本时指定自己的缩放值。你可以看到，如果不输入缩放值，它会返回一条提示信息，告诉你需要输入。运行脚本的效果如下：

```
[donnie@fedora ~]$ ./pi_bc.sh 20
3.14159265358979323844
[donnie@fedora ~]$ 
```

当然，你可以根据需要输入更大的缩放值。

你也可以使用`bc`来创建你自己的函数库。例如，查看我在`/usr/local/lib/`目录下创建的这个`baseconv.lib`库文件：

```
 h2d() {
           h2dnum=$(echo "ibase=16; $input_num" | bc)
           }
       d2h() {
           d2hnum=$(echo "obase=16; $input_num" | bc)
           }
       o2h() {
           o2hnum=$(echo "obase=16; ibase=8; $input_num" | bc)
           }
       h2o() {
           h2onum=$(echo "obase=8; ibase=16; $input_num" | bc)
           } 
```

使用这个库的`bc-func_demo.sh`脚本太长，无法在此完整展示。但是，你可以从 GitHub 仓库下载它。目前，我只会展示一些片段并提供解释。

脚本的顶部部分看起来是这样的：

```
#!/bin/bash
. /usr/local/lib/baseconv.lib
until [ "$choice" = "q" ]; do
        echo "Choose your desired function from the following list: "
        echo "For hex to decimal, press \"1\"."
        echo "For decimal to hex, press \"2\"."
        echo "For octal to hex, press \"3\"."
        echo "For hex to octal, press \"4\"."
        echo "To quit, press \"q\"." 
```

你首先看到的是我通过`. /usr/local/lib/baseconv.lib`命令引入库文件。接下来，你会看到一个我之前没展示过的结构。`until. .do`结构会一直显示菜单，直到你按下*q*键。也就是说，如果你做出其他选择，系统会提示你输入一个想要转换的数字。转换完成后，菜单会立即重新出现，直到你按下*q*。接下来是一个`case. .esac`结构，它根据你从菜单中选择的任务执行相应的操作。这里是其中的第一部分：

```
read choice
        case $choice in
                1) echo "Enter the hex number that you would like to convert. "
                        read input_num
                        h2d
                        echo $h2dnum
                        echo
                        echo
                        ;; 
```

运行脚本时会是这样：

```
[donnie@fedora ~]$ ./bc-func_demo.sh
Choose your desired function from the following list:
For hex to decimal, press "1".
For decimal to hex, press "2".
For octal to hex, press "3".
For hex to octal, press "4".
To quit, press "q".
2
Enter the decimal number that you would like to convert.
10
A 
```

在此之后，超出了我在此处能够展示的范围，菜单会重新出现，等待你的下一个输入。

现在，这种方式确实有效，但它使用全局变量将函数中的值传递回主脚本。我已经告诉过你，这并不是最安全的做法，最好是结合使用局部变量和命令替换。修改后的库文件内容过长，无法在这里完全展示，但这里有一段代码示例：

```
 h2d() {
           local h2dnum=$(echo "ibase=16; $input_num" | bc)
           echo $h2dnum
           }
       d2h() {
           local d2hnum=$(echo "obase=16; $input_num" | bc)
           echo $d2hnum
           }
       o2h() {
           local o2hnum=$(echo "obase=16; ibase=8; $input_num" | bc)
           echo $o2hnum
           } 
```

你可以从 Github 下载整个`baseconv_local.lib`文件，以及使用它的`bc-func_local_demo.sh`脚本。该脚本与之前的版本大致相同，除了在`case. .esac`结构中的代码，它调用了不同的函数。这里有一段代码示例：

```
read choice
        case $choice in
                1) echo "Enter the hex number that you would like to convert. "
                        read input_num
                        result=$(h2d)
                        echo $result
                        echo
                        echo
                        ;; 
```

我已经在*第十章——理解函数*中解释过这个结构，所以在这里不再赘述。

我将向你展示的最后一个例子，使用了你曾以为永远不会用到的许多文本流过滤器之一。这涉及到使用`paste`命令来帮助计算各种版本 Windows 操作系统的总市场份额。为了让你理解我的意思，看看这个你可以从 Github 下载的`os_combined-ww-monthly-202209-202309-bar.csv`文件：

```
"OS","Market Share Perc. (Sept 2022 - Oct 2023)"
"Windows 11",17.21
"Windows 10",45.62
"Windows 7",3.5
"Windows 8.1",1.5
"Windows 8",1.3
"Windows XP",0.6
"OS X",17.66
"Unknown",6.55
"Chrome OS",3.15
"Linux",2.89
"Other",0.01 
```

这些市场份额统计数据来自于*Statcounter Global Stats*网站，针对的是各种桌面操作系统。

好吧，有点像是这样。因为很多年前，当我首次为我教的 Shell 脚本课程创建这个演示时，Statcounter 的人们把 Windows 市场份额按不同版本进行了划分，如你所见。但现在，他们只在这份综合报告中列出 Windows 的总体市场份额，并在单独的 Windows-only 报告中细分 Windows 各版本的市场份额。所以，我不得不稍微修改一下这个文件，以重现以前的报告方式，使得演示能继续进行。（不过，管它怎么做，只要有效就好，对吧？）这个演示的脚本是`report_os.sh`，你也可以从 Github 下载。

脚本中的第一个`echo`命令会将所有版本的 Windows 市场份额加总，以计算 Windows 的总市场份额。以下是显示方式：

```
echo "The market share for Windows is $(grep 'Win' $file | cut -d, -f2 | paste -s -d+ | bc)%." > report_for_$(date +%F).txt 
```

所以，在我从所有 Windows 行中提取出第二字段之后，我使用 `paste` 以串行模式，并使用 `+` 作为 `paste` 字段分隔符。我将所有内容通过管道传输给 `bc`，然后将输出重定向到一个以今天日期命名的文本文件中。

其余操作系统的 `echo` 命令更加简单，因为它们不需要进行数学计算。以下是 macOS 的命令：

```
echo "The market share for macOS is $(grep 'OS X' $file | cut -d, -f2)%." >> report_for_$(date +%F).txt 
```

是的，我知道苹果将他们的操作系统更名为 macOS。但是，Statcounter 的人们仍然将其列为 OS X，所以我需要在脚本中使用这个作为搜索词。无论如何，运行脚本的命令是这样的：

```
[donnie@fedora ~]$ ./report_os.sh os_combined-ww-monthly-202209-202309-bar.csv
[donnie@fedora ~]$ 
```

结果报告文件如下所示：

```
The market share for Windows is 69.73%.
The market share for macOS is 17.66%.
The market share for Linux is 2.89%.
The market share for Chrome is 3.15%.
The market share for Unknown is 6.55%.
The market share for Others is 0.01%. 
```

当我修改输入文件以列出各种版本的 Windows 时，我确保 Windows 的总市场份额仍然符合预期。所以，是的，当我在 2023 年 10 月写这篇文章时，Windows 的市场份额确实是 69.73%。

顺便提一下，如果你有兴趣查看更多关于操作系统使用的统计信息，请访问 Statcounter Global Stats 网站：[`gs.statcounter.com/`](https://gs.statcounter.com/)

我认为这已经涵盖了 shell 脚本数学运算的内容。让我们总结一下并继续前进。

# 总结

在这一章中，我向你展示了几种在 `bash` 中进行数学运算的方法，并提供了一些技巧，帮助你确保你的数学脚本可以在非 `bash` 的 shell 中运行。我从执行整数运算的各种方法开始，然后展示了使用 `bc` 进行浮点数运算的多种方法。正如我之前所说，你需要成为数学专家，才能充分利用 `bc` 的所有功能。但即使你不是数学专家，依然能做很多事情。而且，网上有很多数学教程可以帮助你。只需使用你喜欢的搜索引擎找到它们。

在下一章中，我将向你展示如何使用 here 文档。到时见。

# 问题

1.  以下哪种方法可以在命令行中执行整数运算？

    1.  `echo 1+1`

    1.  `echo 1 + 1`

    1.  `echo $(bc 1+1)`

    1.  `expr 1+1`

    1.  `expr 1 + 1`

1.  你希望确保你的 shell 脚本不仅能在 `bash` 上运行，还能在非 `bash` 的 shell 上运行。以下哪些命令可以在你的脚本中使用？

    1.  `echo $((1+2+3+4))`

    1.  `echo $[1+2+3+4]`

    1.  `echo $[[1+2+3+4]]`

    1.  `echo $(1+2+3+4)`

1.  你想进行浮点数运算。以下哪些命令可以使用？

    1.  `bc`

    1.  `bc -f`

    1.  `bc -l`

    1.  `bc --float`

1.  你需要找到 8 的自然对数。你该如何做？

    1.  `expr log(8)`

    1.  `echo [log(8)]`

    1.  使用 `l(8)` 和 `bc`

    1.  使用 `log(8)` 和 `bc`

1.  以下哪个命令可以用来找到 П 的近似值？

    1.  `pi=$("scale=10; 4*a(1)" | bc -l)`

    1.  `pi=$(echo "scale=10; 4*a(1)" | bc -l)`

    1.  `pi=$(bc -l "scale=10; 4*a(1)" )`

    1.  `pi=$(echo "scale=10; 4*arc(1)" | bc -l)`

# 进一步阅读

+   Bash 数学运算（Bash Arithmetic）解析：[`phoenixnap.com/kb/bash-math`](https://phoenixnap.com/kb/bash-math)

+   bc 命令手册：[`www.gnu.org/software/bc/manual/html_mono/bc.html`](https://www.gnu.org/software/bc/manual/html_mono/bc.html)

+   在 Linux 上使用什么是好的命令行计算器：[`www.xmodulo.com/command-line-calculator-linux.html`](https://www.xmodulo.com/command-line-calculator-linux.html)

+   几何速查表：[`s3-us-west-1.amazonaws.com/math-salamanders/Geometry/Geometry-Information-Pages/Geometry-Cheat-Sheets/geometry-cheat-sheet-4-2d-shapes-formulas.pdf`](https://s3-us-west-1.amazonaws.com/math-salamanders/Geometry/Geometry-Information-Pages/Geometry-Cheat-Sheets/geometry-cheat-sheet-4-2d-shapes-formulas.pdf)

+   数学 LibreTexts：[`math.libretexts.org/`](https://math.libretexts.org/)

+   Statcounter 全球统计：[`gs.statcounter.com/`](https://gs.statcounter.com/)

# 答案

1.  e

1.  a

1.  c

1.  c

1.  b

# 加入我们的 Discord 社区！

与其他用户、Linux 专家以及作者本人一同阅读本书。

提问、为其他读者提供解决方案、通过“问我任何问题”环节与作者聊天，等等。扫描二维码或访问链接加入社区。

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code10596186092701843.png)
