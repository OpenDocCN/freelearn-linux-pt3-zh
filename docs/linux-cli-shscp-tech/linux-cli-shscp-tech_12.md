# 第十二章：使用参数和函数

每当我们尝试用任何编程语言编写任何类型的应用程序或脚本时，我们应该始终尽量使我们的代码模块化，并且容易维护。在创建脚本的这一方面，帮助我们很多的概念是**函数**。

本章将涵盖以下食谱：

+   在 shell 脚本中使用自定义函数

+   将参数传递给函数

+   局部和全局变量

+   处理函数的返回值

+   将外部函数加载到 shell 脚本中

+   通过函数实现常用过程

# 技术要求

对于这些食谱，我们将使用一台 Linux 机器。我们可以使用任何`cli1`虚拟机，因为它是最方便使用的，因为它仅是**命令行界面**（**CLI**）机器。所以，总的来说，我们需要以下内容：

+   安装了 Linux 的虚拟机——任何发行版（在我们的案例中，将使用**Ubuntu 20.02**）。

+   花点时间消化使用 VI(m)编辑器的复杂性。Nano 更简单，因此它会更容易学习。

所以，启动你的虚拟机，开始吧！

# 在 shell 脚本中使用自定义函数

到目前为止，我们所做的只是创建非常简单的脚本，最多只有几个命令。这将是你大多数脚本的样子，因为通过脚本解决的许多问题都是简单地消除重复的任务。在本章中，我们将通过函数来创建脚本中的代码模块。它们的主要目的是避免脚本中重复的代码块，进一步简化脚本本身。

## 准备工作

说到函数，Bash 有点奇怪。你可能从其他语言中了解的函数，在`bash`中看起来相似，但又完全不同。我们将从如何定义函数开始。为了让事情从一开始就变得复杂，`bash`使用了两种非常相似的表示方法，一种看起来更像是你在其他语言中会看到的，另一种则更符合`bash`的语法规则。

在我们提到它们之前，请记住，函数的定义在功能或其他方面没有任何区别——我们可以使用它们中的任何一个，结果是完全相同的。

第一个定义的语法看起来像是你在任何编程语言中都会看到的。没有关键字——我们只是指定函数的名称，后跟两个普通括号，然后在大括号中定义构成函数的命令块。

然而，`bash`和几乎所有其他编程语言之间有一个很大的区别。通常，在任何语言中，括号用于将参数传递给函数。而在`bash`中，它们始终是空的——它们唯一的作用是定义一个函数。参数是通过完全不同的方式传递的：

```
function_name () {
<commands>
}
```

定义函数的另一种方式更符合`bash`的常规方式。这里有一个保留字`function`；因此，为了定义一个函数，我们只需这样做：

```
function function_name {
<commands>
}
```

这个版本更可能提醒你，参数是以不同的方式提供的，但这可能是两者之间唯一的区别。

函数必须在使用之前定义。这是完全合乎逻辑的，因为 Shell 是逐行执行每条命令的，要理解一条命令，必须先定义它是内部命令、外部命令还是函数。与其他一些编程语言不同，参数和返回值不会预先定义——或者更准确地说，根本不定义。

## 如何做到这一点……

和往常一样，我们将从一个`hello world`脚本开始，但会做一些小小的改变。我们将在一个函数内使用`echo`命令，并且脚本的主要部分将运行这个函数。我们还将创建一个函数的替代版本，旨在展示两种定义函数的方式是等效的。

在这个脚本中有几点需要注意——当我们定义函数时，并没有*唯一正确*的方式；两种方式都可以使用，但它们的工作方式不同。我们倾向于使用明确包含`function`关键字的格式，因为这样可以立即引起注意，表明这是一种函数定义，但这只是我们的个人偏好——你可以使用任何你喜欢的格式：

```
#!/bin/bash
# Hello World done by a function
function HelloWorld {
    echo Hello World!
}
HelloWorld_alternate () {
    echo Hello World!
}
#now we call the functions
HelloWorld
HelloWorld_alternate
```

当我们运行脚本时，我们可以看到两个函数的表现完全相同：

```
demo@cli1:~/scripting$ bash functions.sh 
Hello World!
Hello World!
```

现在，我们将创建一个更有意义的示例。许多脚本都需要你将内容输出到屏幕或文件中。输出的某些部分会反复出现——这是函数特别适合的任务：

```
#!/bin/bash
function PrintHeader {
    echo -----------------------
    echo Header of some sort
    echo -----------------------
}
echo In order to show how this looks like
echo we are going to print a header
PrintHeader
echo And once again
PrintHeader
echo That was it.
demo@cli1:~/scripting$ bash function.sh 
In order to show how this looks like
we are going to print a header
-----------------------
Header of some sort
-----------------------
And once again
-----------------------
Header of some sort
-----------------------
That was it.
```

我们的函数所做的工作是为输出创建一个头部。当我们学习如何向函数传递参数时，我们将经常使用这个技巧，特别是当我们需要将格式化的文本输出到日志中，或者当我们有一大块文本，并且需要填入几个变量时。

## 它是如何工作的……

函数是`bash`中重复执行的代码块，每当我们在脚本中引用它们时，它们就会被重新执行。它们的主要目的是使脚本更易读、更易调试。使用函数的另一个原因是：避免代码错误。如果我们需要在脚本的不同部分重用某些代码，我们可以直接复制和粘贴，但这可能会引入错误，导致脚本出现 bug。

## 另见

+   [`www.shell-tips.com/bash/functions/`](https://www.shell-tips.com/bash/functions/)

+   [`tldp.org/LDP/abs/html/complexfunct.html`](https://tldp.org/LDP/abs/html/complexfunct.html)

# 向函数传递参数

我们通过展示一个简单的脚本来开始演示函数的样子，这是我们能创建的最简单的脚本。我们仍然没有定义如何*与*函数进行交互，也不知道如何给函数传递参数或参数并获得返回结果。在这个配方中，我们将解决这个问题。

## 准备工作

既然我们提到了参数，我们需要稍微谈一下它们。`bash`在函数中的参数处理与在脚本本身中处理参数相同——参数在函数块内部变成局部变量。为了返回一个值，我们几乎是采用与处理整个脚本时相同的方式——我们仅仅从函数块中返回一个值，然后在主脚本体内读取它。

记得我们说过你可以引用最初调用脚本时传递给脚本的参数吗？我们使用了名为`$1`、`$2`、`$3`等的变量来获取命令行中的第一个、第二个、第三个等参数吗？在函数中也是如此。此时，我们使用与引用传递给函数的参数时相同的变量名。

## 如何实现...

为了向一个简单的函数发送两个参数并显示它们，我们可以使用类似下面的方式：

```
#!/bin/bash
#passing arguments to a function
function output {
     echo Parameters you passed are $1 and $2
}
output First Second
```

当我们尝试运行这个脚本时，参数会按我们预期的顺序逐个传递，然后我们的函数会输出它们：

```
demo@cli1:~/scripting$ bash functionarg.sh 
Parameters you passed are First and Second
```

你可能会想知道我们的脚本如何处理传递给脚本的参数，与我们传递给函数的参数相比有何不同。简短的回答是，名为`$1`等的变量具有函数局部的值，并且由我们传递给函数的参数定义。在函数代码块外部，这些变量的值则是传递给脚本的参数。详细的答案将在下一个配方中说明，这涉及局部变量和全局变量的概念。使用参数其实就是声明局部变量的一种特殊情况；我们传递的参数会在函数内变成局部变量：

```
#!/bin/bash
#passing arguments to a function
function output {
    echo Parameters you passed are $1 and $2
}
#we are going to take input arguments of the script itself and #reverse them
output $2 $1
```

我们更改参数顺序的原因是为了展示参数传递给函数的顺序，并确保我们不会在函数中使用传递给脚本的参数，因为它们的名字相同。这个脚本将从命令行获取两个参数，交换它们的顺序，然后将它们作为参数传递给我们的函数。函数将简单地输出它们：

```
demo@cli1:~/scripting$ bash functionarg2.sh First Second
Parameters you passed are Second and First
```

这里发生的事情也正如我们所预期的那样。接下来，我们将检查一个可能让一些人感到困惑的事情。函数是否知道一些参数被传递给了脚本，还是这些参数严格是局部的？为了检查这一点，我们将忽略脚本命令行中的内容，并向函数传递一对硬编码的字符串。如果`bash`像我们预期的那样工作，我们的脚本将输出这些硬编码的值。如果命名为`$1`和`$2`的变量被设置为命令行中的值，并且这些值在函数内仍然存在，我们应该能在`echo`语句中看到这些值。我们将创建一个`functionarg3.sh`文件，包含以下代码：

```
#!/bin/bash
#passing arguments to a function
function output {
    echo Parameters you passed are $1 and $2
}
#we are going to ignore input parameters
output Hardcoded Variables
```

现在，我们将运行它并检查发生了什么：

```
demo@cli1:~/scripting$ bash functionarg3sh First Second
Parameters you passed are Hardcoded and Variables
```

我们可以看到我们的假设是正确的，传递给函数的参数总是优先于其他内容。

我们接下来要做的是展示如何使用函数处理简单的操作。关于可以对变量执行的操作，我们在本书的其他部分已有涉及，但在这里，我们将使用一个之前没有用过的例子。我们将简单地将命令行中的两个参数相加。

为了做到这一点，我们将从命令行将参数传递给函数，然后使用`echo`输出计算结果。用于获取结果的函数部分也非常有趣，因为它提醒我们，必须显式使用一个函数来加两个数字。否则，如果我们尝试将变量相加，最终会得到一个字符串——像这样：

```
demo@cli1:~/scripting$ a=1
demo@cli1:~/scripting$ b=2
demo@cli1:~/scripting$ echo $a+$b
1+2
demo@cli1:~/scripting$ echo $(($a+$b))
3
```

这是最终版本，已被纳入我们的脚本中：

```
#!/bin/bash
#Doing some maths
function simplemath {
add=$(($1+$2))
echo $add is the result of addition
}
#we are going to take input arguments and pass them all the way
simplemath $1 $2
```

请注意，在这个例子中，我们在函数内部使用一个新变量来加数字，然后将该变量的值作为结果输出。这比直接在输出中进行操作要好得多——使用这些临时变量的代码总是比试图查找和理解嵌入到输出字符串中的变量更容易阅读和理解。

## 它是如何工作的……

接下来我们想展示的是一个很有趣的小功能，这个功能在大多数编程语言中并不常见。由于`bash`在函数中处理参数的方式与处理脚本中的参数相同，并且使用相同的逻辑将这些参数转化为函数内部的变量，因此我们实际上可以在不预先定义参数数量的情况下向函数传递多个参数。当然，我们的函数需要能够理解类似这样的东西。

## 参见

+   [`linuxize.com/post/bash-functions/`](https://linuxize.com/post/bash-functions/)

+   [`linuxhint.com/create-bash-functions-arguments/`](https://linuxhint.com/create-bash-functions-arguments/)

# 本地变量和全局变量

当在脚本中声明任何变量时——或者更广泛地说，任何地方——对于该变量，一个至关重要的属性就是它的作用域。作用域指的是*变量值被声明的地方*。作用域非常重要，因为如果我们不理解它是如何工作的，就可能在某些情况下得到意外的结果。

## 准备开始

定义变量的全局作用域是`bash`的默认行为，不需要我们与之交互。所有定义的变量都是全局变量；它们的值在整个脚本中都是相同的。如果我们通过重新赋值来改变变量的值（记住，对值的操作不会改变值本身），那么这个值会全局变化，旧值将被丢失。

在声明变量时，我们还可以做另一件事，那就是将其声明为局部变量。简单来说，这意味着我们明确告诉`bash`，我们将在代码的某个有限部分使用这个变量，并且它需要只在这里保存值，而不是在整个脚本中作为全局变量存在。

为什么要声明一个局部变量？有几个原因，其中最重要的是确保我们不会更改任何全局变量的值。如果一个变量与全局变量同名并在局部作用域中声明，`bash`将创建该变量的另一个实例，并会分别跟踪全局值和局部值。

全局变量和局部变量以及它们是如何工作的，最好的解释方法就是使用一个示例。

## 如何实现……

我们将使用的脚本展示如何工作的例子，几乎可以在互联网上的每一个示例中找到，或者在任何涉及该主题的书籍中都会出现。这个示例的思路是创建一个全局变量，然后在函数中创建一个与全局变量同名的局部变量。全局变量的值应该与局部变量的值不同，当我们显示这个值时，应该能够看到根据我们引用的是全局变量还是局部变量，值会有所不同：

```
#!/bin/bash
# First we define global variable
# Value of this variable should be visible in the entire script
VAR1="Global variable"
Function func {
# Now we define local variable with the same name
# as the global one. 
local VAR1="Local variable"
#we then output the value inside the function
echo Inside the function variable has the value of: $VAR1 \
}
echo In the main script before function is executed variable \
has the value of: $VAR1
echo Now calling the function
func
# Value of the global variable shouldn't change
echo returned from function
echo In the main script after function is executed value is: \
$VAR1
```

如果我们执行这个脚本，我们将看到变量是如何交互的：

```
demo@cli1:~/scripting$ bash funcglobal.sh
In the main script before function is executed variable has the value of: Global variable
Now calling the function
Inside the function variable has the value of: Local variable
returned from function
In the main script after function is executed value is: Global variable
```

这是完全预期的——如果存在同名的全局变量和局部变量，局部变量将只在其定义的块中有自己的值；否则，将使用全局值。

我们说过像这样的脚本是常见的示例，但如果我们只定义局部变量会发生什么呢？`bash`与大多数其他语言不同，因为默认情况下，如果我们错误地引用了未定义的变量，它不会显示错误信息。在调试脚本时，这可能是一个大问题，因为未定义的变量和没有值的已定义变量在我们尝试引用它们时，乍一看它们是完全一样的。

为了展示这一点，我们将对脚本做一个小修改，只需删除第一个变量定义。这将导致我们的全局值变为未定义——只有局部值才有实际的值：

```
#!/bin/bash
# We are not defining the value for our variable in the global #block
function func {
# Now we define local variable that is not defined globally
# as the global one. 
local VAR1="Local variable"
#we then output the value inside the function
echo Inside the function variable has the value of: $VAR1
}
echo In the main script before function is executed undefined \
variable has the value of: $VAR1
echo Now calling the function
func
# Value of the global variable shouldn't change
echo returned from function
echo In the main script after function is executed undefined \
value is actually: $VAR1 
```

在任何严格的编程语言中，类似的做法都会产生错误。但在`bash`中，情况有所不同：

```
demo@cli1:~/scripting$ bash funcglobal1.sh 
In the main script before function is executed undefined variable has the value of:
Now calling the function
Inside the function variable has the value of: Local variable
returned from function
In the main script after function is executed undefined value is actually:
```

我们可以看到，脚本并没有报错，而是忽略了变量的值，并用“空值”替换它。正如我们所提到的，虽然我们预期会出现这种行为，但要注意，这可能会导致一些意想不到的后果。另一个在脚本中重要的点是局部值。我们可以看到，局部变量*只存在*于其定义的代码块中；定义它并不会创建一个全局变量，而且一旦函数或代码块执行完毕，局部值将会丢失。

## 它是如何工作的……

在脚本中使用全局变量还有一个好处——在函数之间传递值。变量的这一特性是非常有用的，但同时也取决于你个人的编程风格。以这种方式使用全局变量很简单——你只需要在脚本开始时声明一个变量，然后在需要时更改其值。通常，你会在执行特定函数之前赋值，然后在函数执行完毕后读取同一个变量。这样，你的函数只需改变变量，就能给你期望的值。

然而，在这种看似合理的使用全局变量方式中，存在一个大问题。由于你无法确定函数是否按预期执行，并且是否已经到达需要改变变量值的阶段，你根本无法知道值本身是否符合预期。如果函数因为某些原因失败，变量将保持你传给它的值，导致可能出现错误的情况。

我们想说的是，以这种方式使用全局变量应该避免，尽管你可以这样做——正确的方式是通过使用参数并通过一个我们将在下一个示例中讨论的机制来返回值，来处理函数和传递值。

## 参见

+   [`www.thegeekstuff.com/2010/05/bash-variables/`](https://www.thegeekstuff.com/2010/05/bash-variables/)

+   [`tldp.org/LDP/abs/html/localvar.html`](https://tldp.org/LDP/abs/html/localvar.html)

# 处理函数返回值

我们提到过，可以使用全局变量将值传递给脚本中的函数，并返回结果。这是最糟糕的做法。如果我们需要将某个值传递给函数，使用参数才是正确的方式。我们仍然面临的问题是，如何在函数执行完后获取结果。我们将在本章中解决这个问题。

## 准备工作

如果没有别的，`bash`在其使用的语法上是逻辑一致的。之所以提到这一点，是因为当函数返回一个值时，它们使用的机制和脚本返回变量时使用的机制完全相同——即`return`命令。通过使用这个命令，函数在被调用时可以返回一个值，但这个值的范围只能在`0`到`255`之间。也有可能设置一个全局变量来返回函数值——例如，如果我们需要返回一个字符串——但尽量避免这样做，因为这会产生难以调试的代码。当你在互联网上浏览函数`return`语句时，你也可能会遇到一种第三种解决方案，这种方案使用了一种叫做*引用传递*或`nameref`的技术。这是一个更复杂的解决方案，你需要了解它，但我们故意在这个例子中避免它，因为它只在`bash`的最新版本（`4.3`及以上）中有效，这会破坏我们脚本的兼容性和可用性。

## 如何做到这一点…

我们将向你展示两种返回函数值的方法，从我们认为错误的方法开始。之所以展示一个错误的解决方案，是因为你在不同的脚本中（尤其是从互联网上下载的脚本）经常会遇到这种情况，如果你不了解这种方法，可能一开始会有点困惑，因为变量通常是在函数内部首次定义的，在函数第一次调用之前并不存在：

```
#!/bin/bash
#Doing some string adding inside a function and returning #values
#function takes two strings and returns them concatenated 
function concatenate {
RESULT=$1$2
}
# calling the function with hardcoded strings
concatenate "First " "and second"
echo $RESULT
```

我们所做的就是将两个字符串传递给一个函数，函数返回它们连接后的结果。当然，这样做很傻——我们完全可以仅仅通过函数中使用的表达式来完成这个任务。这个例子如此基础，甚至没有使用任何运算符。

我们返回值的方式很重要。通过仅仅赋予一个新值，并因此创建了一个名为`RESULT`的全局变量，我们得到了我们的字符串，并能够使用`echo`将其写入屏幕。为什么这会是个问题？

我们已经解释过这个问题了。我们在这里所做的事情是危险的，因为我们无法知道函数是否完成了它必须做的事情。我们唯一拥有的就是一个名为`RESULT`的变量，它可能包含我们期望的值。在这个简单的例子中，我们可以检查结果，但那样会违背有一个专用函数的目的。为了稍微减少不确定性，我们可以做一个小技巧。

请考虑对脚本做出这样的修改：

```
#!/bin/bash
#Doing some string adding inside a function and returning \
values
#function takes two strings and returns them concatenated 
function concatenate {
RESULT=$1$2
}
concatenate "First " "and second"
[ $? -eq 0 ] && echo $RESULT || echo Function did not finish!
```

我们所做的是创建一个条件输出。条件本身的格式你现在应该已经熟悉——我们使用逻辑函数来打印函数的结果，或者打印出函数没有正确执行的消息。提醒一下，当我们介绍逻辑运算符时，我们脚本在最后一行做的事情是检查一个名为`$?`的变量的值。如果变量值等于`0`，我们打印函数的结果。如果值不是零，我们输出错误信息，因为我们知道函数的命令块内部某处出了问题。

我们之所以能做到这一点很简单——我们已经提到过，函数与脚本之间的通信方式与脚本与操作系统其他部分的通信方式相同。这包括传递参数以及能够使用`return`语句返回值，还意味着`bash`在函数结束时会设置一个名为`?`的变量。当我们用它来理解脚本发生了什么（我们已经解释过），如果我们检查这个变量并且它的值为`0`，这意味着函数正确执行完毕，或者至少最后一条命令正确执行完毕。

这是一个简单的解决方案，针对我们本不应该一开始就创建的问题；只要可能，我们应该使用`return`来获取我们的值。以下是一个示例：

```
#!/bin/bash
#simple adding of two numbers
#function takes two numbers and returns result of addition
function simpleadd {
    local RESULT=$(($1+$2))
    return $RESULT
}
#we are going to hardcode two numbers
simpleadd 4 5 
echo $?
```

如果我们确保数字在`0`到`255`的范围内，这是一种更好的方式。我们输出函数的结果，操作就像引用正确的变量一样简单。我们还可以检查函数执行后的变量值是否为`0`，这意味着函数正常工作，然后再输出结果。

另一个你应该知道的事情是，函数可以使用`exit`命令。通过使用它，你是在告诉`bash`立即停止函数正在执行的操作并退出函数命令块。在这种情况下，将返回的值是执行`exit`命令之前最后一条命令的错误级别。

这是一个示例：

```
#!/bin/bash
#exiting from a function before function finishes
function never {
echo This function has two statements, one will never be \
printed. 
exit
echo This is the message that will never print
}
#here we run the function
never
```

将要打印的是输出的第一行；由于我们使用了`exit`语句，第二部分输出将永远不会执行：

```
demo@cli1:~/scripting$ bash funcreturn3.sh 
This function has two statements, one will never be printed.
```

## 它是如何工作的…

所有这一切存在的主要原因是为了让你更紧密地控制函数以及更一般来说，脚本中命令执行的顺序。`bash`在处理这个话题时非常基础，这也正是它的多功能性所在。为了使用函数，你只需要了解脚本中参数的工作方式——所有的变量名和背后的逻辑在应用于函数时是相同的。

## 另请参阅

+   [`www.assertnotmagic.com/2020/06/19/bash-return-multiple/`](https://www.assertnotmagic.com/2020/06/19/bash-return-multiple/)

+   [`www.linuxjournal.com/content/return-values-bash-functions`](https://www.linuxjournal.com/content/return-values-bash-functions)

# 将外部函数加载到 shell 脚本中

当你需要创建更复杂的 shell 脚本时，常见的问题之一就是如何将其他代码包含到脚本中。一旦你开始编写脚本，你通常会创建一些常用的函数——比如打开与服务器的连接，执行一些操作，或者其他类似的操作。

有时候，为了避免每次脚本被调用时都需要输入变量，你的脚本必须使用许多由用户预先定义的变量，这些变量需要在脚本运行前就设置好。

当然，解决这两个问题的方法可以是将相关代码直接复制并粘贴到脚本中，并让用户在运行脚本之前编辑它。我们不应该这样做的原因是，每次复制和粘贴内容时，我们都会创建代码的新版本。如果我们在代码中发现错误，就需要在所有重复使用这段代码的脚本中进行修复。幸运的是，解决这个问题有一个更好的方法，那就是将脚本拆分成不同的文件，然后在需要时引用它们。

## 准备工作

这个方案将在两种不一定互斥的场景中非常有用。我们已经简要提到过这两种场景。

第一个方案是使用外部函数。通常，在创建脚本时，所有内容都会放在一个文件中。所有函数、定义、变量和命令都会集中在一个地方。如果我们只是在创建专门完成特定任务的脚本，这种做法通常完全没问题。

更常见的是，我们需要解决一些之前在其他解决方案中已经处理过的问题。在这种情况下，我们通常已经有一些可以被认为是解决方案一部分的现成函数。

在复杂的脚本解决方案中，你可能会使用一些通用的功能，比如菜单、界面、页眉、页脚、日志等，这些内容在你创建的每个脚本中都是完全相同的。

另一个非常常见的问题是需要用户进行设置的配置。大型脚本可能会包含服务器名称、端口、文件名、用户以及脚本运行所需的其他许多信息。你可以将这些信息作为命令行参数传递，但这种做法看起来不太好，而且会使脚本容易出错，因为用户每次运行脚本时都必须手动输入大量内容。

在这种情况下，一种常见做法是将所有内容作为变量放在一个文件中，然后让用户在脚本的安装过程中编辑此文件。当然，你也可以将所有内容与脚本本身放在一起，但这几乎肯定会导致某些用户更改他们不应该更改的内容。

一如既往，解决方案是存在的。

## 如何操作……

`bash`具有内置的功能，可以将不同的文件包含到脚本中。这个思想很简单——有一个*主脚本文件*，它作为脚本本身执行。在该文件中，有一些命令告诉`bash`包含不同的文件和脚本。

和其他事情一样，尽管这是一件非常简单的事情，但你需要了解一些事情。我们首先要使用的命令是`source`。在我们解释所有内容之前，我们将创建两个脚本。第一个是用户将要运行的脚本，它看起来是这样的。将文件命名为`main.sh`：

```
#!/bin/bash
#first we are going to output some environment variables and #define a few of our own
echo Shell level before we include $SHLVL
echo PWD value before include $PWD
TESTVAR='main'
echo Shell level after include $SHLVL
echo PWD value after include $PWD
echo Variable value after include $TESTVAR
```

我们将运行它，只是为了看看脚本的行为：

```
demo@cli1:~/includes$ bash main.sh 
Shell level before include 2
PWD value before include /home/demo/includes
Shell level after include 2
PWD value after include /home/demo/includes
Variable value after include main
```

结果正如我们预期的那样——我们当前的目录与我们运行脚本时所在的目录相同，并且`$SHLVL`为`2`，因为我们在与命令行(`lvl1`)不同的独立 shell (`lvl2`) 中运行了脚本。我们的变量定义为`main`，并且它没有发生变化。

现在，我们将创建第二个脚本并命名为`auxscript.sh`：

```
echo Inside included file Shell level is $SHLVL
echo Inside included PWD is $PWD
echo Before we changed it variable had a value of: $TESTVAR
TESTVAR='AUX'
echo After we changed it variable has a value of: $TESTVAR
```

这里最大的问题是我们没有在脚本开始时使用通常的`#!/bin/bash`标记。这是故意的，因为这个文件是为了被包含在其他脚本中，而不是独立运行的。

之后，我们做的事情大致与主脚本中一样，输出一些文本和数值，并且操作变量。

我们更改变量的原因是为了展示在文件的包含部分实际发生了什么，以及它是如何与主脚本主体交互的。

现在，我们将更改`main.sh`脚本并只添加一行：

```
#!/bin/bash
#first we are going to output some environment variables and #define a few of our own
echo Shell level before we include $SHLVL
echo PWD value before include $PWD
TESTVAR='main'
source auxscript.sh
echo Shell level after include $SHLVL
echo PWD value after include $PWD
echo Variable value after include $TESTVAR
```

现在最主要的事情是再次运行`main.sh`脚本：

```
demo@cli1:~/includes$ bash main.sh 
Shell level before include 2
PWD value before include /home/demo/includes
Inside included file Shell level is 2
Inside included PWD is /home/demo/includes
Before we changed it variable had a value of: main
After we changed it variable has a value of: AUX
Shell level after include 2
PWD value after include /home/demo/includes
Variable value after include AUX
```

这里发生了一些有趣的事情。我们可以看到，环境变量没有发生变化，但测试变量却发生了变化。

我们将解释这一点，但在此之前，我们要做一件事——我们将使用另一个命令，而不是`source`。很多刚接触脚本的人往往会将我们刚刚展示的`source`命令与执行脚本混淆。毕竟，我们是在脚本中包含一个脚本，所以这些东西看起来很相似。我们将尝试在我们的例子中实现这一点。

我们将在主脚本中更改一行，但我们的`aux`脚本保持不变。我们可以用多种方式做到这一点，但我们故意选择了运行`bash`并显式地运行第二个脚本。原因很简单——其他方法要求我们的脚本设置执行位（这是我们没有做的），或者依赖于类似于直接运行`exec`命令的较不易理解的版本：

```
#!/bin/bash
#first we are going to output some environment variables and #define a few of our own
echo Shell level before include $SHLVL
echo PWD value before include $PWD
TESTVAR='main'
bash auxscript.sh
echo Shell level after include $SHLVL
echo PWD value after include $PWD
echo Variable value after include $TESTVAR
```

我们唯一改变的就是，我们没有包含脚本——我们在执行它：

```
demo@cli1:~/includes$ bash mainexec.sh 
Shell level before include 2
PWD value before include /home/demo/includes
Inside included file Shell level is 3
Inside included PWD is /home/demo/includes
Before we changed it variable had a value of:
After we changed it variable has a value of: AUX
Shell level after include 2
PWD value after include /home/demo/includes
Variable value after include main
```

然而，我们可以看到，这个小小的变化在脚本的工作方式上产生了巨大的不同。

## 它是如何工作的…

我们做的最后一个例子需要很多解释，我们需要从`bash`的工作方式开始。

使用`source`命令告诉`bash`去查找一个文件并在我们引用该文件的地方使用它的内容。`bash`做的事情很简单——它只是用我们指向的整个文件替换这一行。所有的行都会被插入，然后像我们将整个文件复制粘贴到原始脚本中一样执行。

这就是为什么在我们的第一个示例中什么都没有改变的原因。我们的脚本从主文件开始运行，继续从辅助文件运行命令，然后返回主文件执行接下来的命令。

当我们将`source`替换为`bash`时，创建了一个完全不同的场景。通过在脚本中使用`bash`命令，我们告诉 shell 启动另一个实例并执行我们引用的脚本。这意味着会创建整个环境，除非我们明确指定在新环境中需要一些变量，否则这些变量不会被导出。

这也是我们的`$SHLVL`变量增加的原因——因为我们在脚本中调用了另一个 shell，shell 级别必须提高。

我们的测试变量消失了，因为我们没有导出它，因此它在被设置之前没有值，而且由于我们的环境仅仅是为了运行这几行代码而创建的，因此当我们调用的脚本结束时，同样的变量也消失了。

记住，执行脚本和引用脚本是完全不同的事情，当你不确定时，思考你到底想做什么。如果你想像常规命令一样执行脚本中的某个内容，使用`bash`或`exec`。如果你需要从另一个脚本复制粘贴代码，使用`source`。

在完成本教程之前，我们还需要提到函数。包含函数的方式与包含任何其他脚本部分完全相同，有一个重要的区别。为了让代码正常工作，你*必须*在脚本的开头或在尝试使用这些函数之前包含函数。如果没有这样做，结果会是和根本没有定义函数一样，导致错误。

## 另见

+   [`bash.cyberciti.biz/guide/Source_command`](https://bash.cyberciti.biz/guide/Source_command)

+   [`tldp.org/LDP/Bash-Beginners-Guide/html/sect_02_01.html`](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_02_01.html)

# 通过函数实现常用程序

到目前为止，我们已经创建了很多不同且非常简单的脚本，这些脚本或多或少使用了`echo`和一些命令，仅仅是为了展示`bash`中某个特定功能的工作原理。在本篇教程中，我们将给你一些关于如何使用我们迄今为止学到的内容的想法。

## 准备工作

我们将创建一个小脚本，展示如何轻松自动化任何系统中的日常任务。这里的重点不是展示每个可能的任务，而是教你如何解决最常见的问题。

## 如何做……

在我们开始编写脚本之前，我们需要回到之前的教程，回顾如何开始编写脚本。我们讨论的是当我们创建和运行这个脚本时，我们所做的前提条件和假设。

每个脚本都会有其特定的前提条件。这些通常是脚本运行所需的东西清单——可能是需要不同的包，或者是需要满足某些条件才能使脚本正常工作，例如一个正常工作的数据库或正常工作的 Web 服务器。

对于这个脚本，我们假设你已经安装了一个叫做`curl`的包，并且你已经连接到了互联网。

现在，对于我们所依赖的假设，这个脚本包含了一些会影响系统用户和组的命令。这意味着，为了使脚本的这一部分正常工作，我们绝对需要脚本由`root`用户或拥有管理员权限的其他用户来运行。

该脚本还假设了很多关于用户的信息，并且只检查我们是否有足够的参数，而不是检查提供的参数质量。这意味着用户可以给脚本提供一个数字而不是字符串，脚本会把它当作有效的参数。我们将在开始解析脚本时解释如何处理这个问题。

作为负责编写脚本的人，你的工作之一是要清楚这些前提条件，并确保处理好这些条件。有两种方法可以做到这一点——第一种是以某种形式在文档中说明你的脚本期望的条件，这个文档会跟随脚本一起发布。

你还可以做的另一件事（我们强烈推荐这样做）是检查你能想到的每一种可能的条件，如果出现问题，要么打印错误信息并停止脚本，要么，如果你知道问题所在，尝试修复它。

你可以在脚本中解决的一些问题示例是管理员权限——你的脚本可以测试是否能够运行，如果权限不足，它会要求用户提升权限。你还可以测试系统中是否存在特定的包，如果你看到某个非标准命令失败，可以进行检查。

最终，如何解决脚本中的问题将取决于你和你的技能水平，但在你做任何事之前，记住，编写脚本时你需要测试一切。

现在，这是实际的脚本：

```
#!/bin/bash
#shell script that automates common tasks
function rsyn {
rsync -avzh $1 $2 
}
function usage {
echo In order to use this script you can:
echo "$0 copy <source> <destination> to copy files from source \
to destination"
echo "$0 newuser <name> to createuser with the username \
<username>"
echo "$0 group <username> <group> to add user to a group"
echo "$0 weather to check local weather"
echo "$0 weather <city> to check weather in some city on earth"
echo "$0 help for this help"
}
if [ "$1" != "" ] 
            Then
    case $1 in
         help)
            Usage
            Exit
            ;;
        copy)
                 if [ "$2" != "" && "$3" != "" ]
                 then 
            rsyn $2 $3
          fi
            ;;

              group)
            if [ "$2" != "" && "$3" != "" ]
                  then 
                       usermod -a -G $3 $2
            fi
                                        ;;
              newuser) 
                  if [ "$2" != "" ]
                  then
                               useradd $2
                          fi
                          ;;
               weather)
                  if [ "$2" != "" ]
                          then 
                                curl wttr.in/$2
                          else 
                                curl wttr.in
                  fi
                  ;;
               *)
            echo "ERROR: unknown parameter $1\""
            usage
            exit 1
            ;;
    esac
                 else
            Usage
fi
```

## 它是如何工作的……

这个脚本需要一些解释，我们故意没有对其进行注释，原因有两个。一个是注释会使得脚本变得过长，导致需要打印太多页，另一个是为了能够在这次解释中逐块讲解，而不会因为短小的注释打断你的思路。话虽如此，记住，脚本中一定要加上注释！

所以，我们的脚本从一个函数开始。考虑到这个函数只有一行，你可能会惊讶于我们决定把它拆成一个函数，但我们是有目的的。

一些命令，如`rsync`或`tar`，例如，有一长串常用的开关。在创建脚本时，有时将其中一些命令放入函数中，这样就可以在不记住每个开关的情况下调用该函数。对于需要许多预设参数的命令，这种方法也适用。将所有这些参数放入一个函数中，然后只用最基本的参数调用该函数。

我们将*usage*（用法）放入函数中，它是一个帮助用户运行脚本的文本块，提供足够的信息，让他们无需其他帮助。

如果可能，请为您的脚本编写更详细的帮助页面。你甚至可以创建一个`阅读帮助页面以获取更多信息`。

在这个函数中，我们使用了`$0`位置参数来输出脚本的名称。当你在提供脚本使用示例时，使用这种方式来为用户提供帮助。避免硬编码脚本名称，因为你不知道用户是否更改了脚本的文件名，而硬编码的名称可能会让他们完全困惑。

此外，如果你在文本中使用任何特殊字符，请使用引号；否则，可能会遇到错误，或者更糟糕的是，出现完全无法解释的错误。

脚本的下一部分处理每个单独的命令。在创建像这样的命令行工具时，事先决定你是要创建一个使用*命令*（如这个命令）、*开关*（如-`h`或`—something`）还是某种简单文本界面的工具。这些方法各有优缺点，但从本质上讲，我们选择的格式通常用于可以一次执行多个任务的脚本。开关允许你为任务引入多个参数，而**用户界面**（**UIs**）则面向没有经验的用户。此外，请记住，您的脚本可能会在其他脚本中使用，因此要避免使用会阻碍这一过程的界面。

在`case`语句中，我们检查了几个事项。首先，我们测试第一个参数是否是有效命令。然后，我们检查给定命令是否有足够的参数，以确保可以无误运行。即便如此，我们仍然没有进行足够的参数有效性测试。在阅读时，尝试添加一些其他的合理性检查，比如*参数是否有效*、*用户是否输入了包含空格的有效参数并将其分成了多个字符串*，等等。

我们不会详细讨论单个命令；我们只会提到那个看起来完全不合适的命令。我们当然在说的是`weather`命令，它为你提供所在城市的天气报告：

![图 12.1 – wttr.in 是许多有趣的在线服务之一](img/Figure_12.1_B16269.jpg)

图 12.1 – wttr.in 是许多有趣的在线服务之一

互联网充满了有用的服务，而[wttr.in](http://wttr.in)绝对是其中之一。如果你访问[wttr.in](http://wttr.in)或者运行`curl wttr.in`，系统会给你一个关于你所在城市的天气报告。这里面有一些深奥的技术——系统会根据你的**互联网协议**（**IP**）地址来尝试猜测你的位置，甚至在进行这个猜测的过程中，几乎会立即提供一个相当准确的天气预报。

我们故意选择这个例子来展示——如果你在[wttr.in](http://wttr.in)链接后添加一个城市名，系统会显示该城市的天气，甚至会尝试猜测准确的城市名称。像这样的在线服务有不少，可以通过命令行访问，使用其中一些可以让你以最不寻常的方式扩展你的脚本。

在这个过程的最后，注意我们正在检查脚本以三种不同方式调用时可能出现的不同错误。始终尝试预测这类错误。

## 另见

如果你在命令行中做任何操作，下面的网页是必看的：

+   [`stackify.com/top-command-line-tools/`](https://stackify.com/top-command-line-tools/)
