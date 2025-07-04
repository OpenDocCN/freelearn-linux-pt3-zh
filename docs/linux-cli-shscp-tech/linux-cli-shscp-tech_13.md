# *第十三章*：使用数组

在前一章的某个操作中，我们提到数组是`bash`支持的复合数据类型之一。我们说过，`bash`只有两种数据类型（字符串和数字），但如果需要的话，还有其他方式来使用更多的数据类型。数组就是其中之一——我们需要能够使用它，因为我们需要比单一值变量更复杂的东西来解决一些问题。

在本章中，我们将涵盖与数组相关的两个基本操作：

+   基础数组操作

+   高级数组操作

你已经可以看到，我们在这里故意做得比较宽泛；数组就是这样——表面上简单，但如果需要使用它们的话，有一些小技巧。

# 技术要求

和所有涉及脚本编写的章节一样，我们使用的是任何可以工作的机器，像数组这样基础的内容将在所有运行`bash`的机器上工作。因此，你将需要以下内容：

一个**虚拟机**（**VM**），安装任何版本的 Linux（在我们的案例中，将使用**Ubuntu 20.10**）

所以，启动你的虚拟机（VM）吧！

# 基础数组操作

`bash`和`bash`中的变量看起来很简单，但实际上它们非常具有迷惑性。没有正式的类型声明，或者说基本上没有任何形式的声明。类型由 Shell 本身来确定，我们可以隐式地做很多事情。这一点对于*常规*变量尤为明显。数组则稍微复杂一些，在使用时会有一些语法上的特殊之处，但它们是一个非常有用的工具。你可能会想，既然它们不过是同一个变量名下存储一个值，为什么我们还要在任何上下文中提到它们呢？嗯，主要原因是我们经常需要这样的结构。很多时候，我们必须存储属于某一数据集合的多个值。通常，这些值可能是一个无序的值列表，适用于我们不在意值的顺序的情况，或者是一个有序的值列表，适用于我们关心顺序的情况。

## 准备工作

通常，数组被定义为*一维索引数组*，因为它们在数组的定义中嵌入了明确的顺序。实际上，当我们有任何多值变量时，它可以是*有序*的也可以是*无序*的。区别在于我们是否可以定义值声明的顺序。如果我们能存储值，但无法获取存储的顺序，那就叫做无序集合。在`bash`中，我们只有有序列表，称之为数组。这意味着变量中的每个值不仅有它本身的值，还有一个定义的索引或位置。我们不仅可以添加或移除数组中的值，还可以直接读取其中任何一个值，如果需要的话，还能重新排序。

我们有两种类型的数组可供使用。两者都是有序的，但一种是*索引的*，这意味着我们有一个数值来定义数组的特定元素。我们还有一种名为关联数组的东西，有时也称为*哈希表*。这种类型的数组很有用，因为它不使用数值，而是使用字符串键来定义特定的数组元素。我们将详细讨论这两种。

## 如何做到这一点...

我们需要做的第一件事是声明一个变量：

```
demo@ubuntu:~/includes$ TEST=(first second third fourth fifth)
demo@ubuntu:~/includes$ echo $TEST
first
```

我们的开始并不顺利。显然，我们正确声明了变量，因为没有错误，但是一旦尝试打印它，就会遇到问题。以前我们尝试打印数组时也遇到过这种情况。解决方法是使用特殊字符来表示索引，并告诉 bash 我们想要打印的不仅是第一个值，而是数组中的所有值：

```
demo@ubuntu:~/includes$ echo ${TEST[@]}
first second third fourth fifth
```

这样做是有效的，因为`bash`理解需要快速循环并遍历数组中的每个索引，打印所有值。与此相对的替代方法是使用以下方式：

```
demo@ubuntu:~/includes$ echo ${TEST[*]}
first second third fourth fifth
```

这与上一条命令完全相同的输出。

这应该让你思考：*还有没有其他使用索引的方式？* 当然有——我们可以直接访问数组中的单个值：

```
demo@ubuntu:~/includes$ echo ${TEST[2]}
third
demo@ubuntu:~/includes$ echo ${TEST[4]}
fifth
```

请注意，这个例子中我们*偏移了一个*，因为数组的第一个元素的索引是`0`。关于数组还有一个很少提到的非常有趣的属性。在展示之前，我们需要解释另一种声明数组的方式。

使用简单括号声明变量是最常见的方法，但我们也可以通过直接指定数组中的元素来完成。有趣的是，我们可以按任意顺序进行操作：

```
demo@ubuntu:~/includes$ ORDER[2]=second
demo@ubuntu:~/includes$ ORDER[3]=third
```

如果我们现在尝试打印我们的数组，结果会比惊讶更危险：

```
demo@ubuntu:~/includes$ echo ${ORDER[*]}
second third
```

我们的数组在索引下存储了两个值。我们故意在索引中犯了一个错误——`second`的值存储在索引号`2`下，使其成为第三个数组元素。但我们没有声明数组的第一个元素。我们说前面命令的结果危险是因为从其输出中，您无法看到特定元素的索引，因此您不知道特定值的实际索引是多少。这使得混淆值变得很容易——或者更明显地说，像这样的东西不会返回一个值，尽管我们可能认为它应该返回：

```
demo@ubuntu:~/includes$ echo ${ORDER[0]}
demo@ubuntu:~/includes$ echo ${ORDER[1]}
```

这些东西通常是我们需要排除故障的常见错误源，如果使用直接指定值和其索引的语法，尤其复杂。

我们要说的是，除非有特殊原因需要这种方式声明数组，否则不要使用这种方式。否则，以后可能会变得混乱。

还有第三种声明数组的方式，这是最不常用的方法。使用`declare`语句和`-a`开关，你可以显式声明某个变量将保存一个值的数组。我们在代码中很少看到这种方式的原因是，当我们使用前面提到的隐式声明时，我们的变量会自动成为适当类型，因此除非你想提高代码的可读性，否则没有必要做两次声明。

这些只是创建和打印普通索引数组的不同方式。我们提到过，还有一种叫做关联数组的数组类型，也称为哈希表、字典或键值对数组。这种类型在`bash` 4.0 中引入，并且在某些平台上仍然不可用；最著名的是，某些版本的 OS X 需要你升级`bash`才能使用这种类型。

现实生活中有许多可以被视为值对的事物。像用户名/密码、姓名/地址、姓名/电话号码等，都是自然形成的数据对，通常会在脚本中使用。显然，我们可以使用普通数组来存储这些数据，但为了能够理解哪些值是某一对数据的一部分，我们不仅需要两个单独的数组，还需要花点心思去声明它们的索引，以便我们可以使用相同的索引来获取第一项和第二项值：

```
#!/bin/bash
#we are declaring two arrays, one for the names, one for the #phone numbers
NAMES=(John Luke "Ivan from work" Ida "That guy")
NUMBERS=(12345 12344 113312 11111 122222)
#now we need to pair them up:
for i in {0..4}
do
              echo Name:${NAMES[i]} number:${NUMBERS[i]}
done
```

这将给我们一个类似于以下的输出：

```
demo@ubuntu:~/variable$ bash pairs.sh 
Name:John number:12345
Name:Luke number:12344
Name:Ivan from work number:113312
Name:Ida number:11111
Name:That guy number:122222
```

如果我们需要打印出所有的数据，这看起来和运行良好。现在想象一下，我们有一些需要搜索的信息——假设我们在寻找某个人的电话号码。如果我们需要使用普通数组来实现，可能会像这样做：

```
#!/bin/bash
#we are declaring two arrays, one for the names, one for the #phone numbers
NAMES=(John Luke "Ivan from work" Ida "That guy")
NUMBERS=(12345 12344 113312 11111 122222)
#now we need to pair them up:
for i in {0..4}
do
            if [ "${NAMES[i]}" == "$1" ]
                          then
                                       echo Name:${NAMES[i]} number:${NUMBERS[i]}
            fi
done
```

为了检查所有的值，我们需要遍历一个数组中的每个元素，然后如果找到匹配的项，就打印出对应的对。首先，我们将测试这个：

```
demo@ubuntu:~/variable$ bash search.sh John
Name:John number:12345
demo@ubuntu:~/variable$ bash search.sh 
demo@ubuntu:~/variable$
```

虽然这样是可行的，但这并不是正确的方法。这个方法的一些缺点显而易见：

+   数组是有索引的，因此它可以在不同的索引位置存储相同的内容。

+   为了找到某个东西，我们需要遍历所有的值。

+   如果我们弄错了任何一个数组，可能会创建无效的数据。

+   这个脚本对于一个简单任务来说非常复杂。

我们提到的另一个*关联*数组类型就是解决这个问题和更多问题的方案。在普通数组中，我们使用数字来索引我们所使用的值。而在这种特定的数组类型中，我们可以使用任何值作为*键*来引用数组中的特定值。

这样做需要显式声明数组的类型，必须使用`declare`语句和`-A`选项来完成。要特别注意这个选项，因为它使用了大写字母*A*，而*普通*数组是使用小写字母来声明的。虽然你可以通过多种方式隐式声明索引数组，但关联数组只能通过这种方法显式声明。为了完全演示这一点，我们需要先展示如何声明这种类型的数组。除了必须使用`declare`语句外，我们还需要声明索引，因为它们可以是任何字符串，而不仅仅是数字。输出看起来像这样：

```
demo@ubuntu:~/variable$ declare -A NAMES
demo@ubuntu:~/variable$ NAMES["John"]=12345
demo@ubuntu:~/variable$ NAMES["Luke"]=12344
demo@ubuntu:~/variable$ NAMES["Ivan from work"]=113312
```

还有一种方法可以在一行中声明这些数组，通过指定键值对：

```
demo@ubuntu:~/variable$ NAMES=([Ida]=11111 ["That guy"]=122222)
```

现在我们已经以两种方式定义了这个数组，让我们做一个小的后续操作，看看我们可以进行哪些其他操作。我们应该能够输出数组的值。首先，我们将尝试使用我们之前在常规数组中使用的方法：

```
demo@ubuntu:~/variable$ echo ${NAMES[*]}
11111 122222
```

这当然是一个问题，但我们早已预料到。这个语法是为具有数字索引的数组设计的，并且会被转化成人类可以理解的内容，这意味着：*通过使用所有可能的索引，打印出名为 NAMES 的数组的所有值*。

这里的关键是我们并不是打印某个值的索引，而只是打印该值本身。由于我们在数组中同时使用了键和值，我们需要能够看到某个特定值的键是什么。这可以通过使用`for`循环来完成，但在此之前，我们有一点需要说明——按照设计，数组有多个值，因此我们不仅可以重新定义整个数组，还可以从中添加或删除元素。我们已经展示过通过创建一个新索引下的新值来添加元素的例子。

所有这些不仅适用于关联数组，也适用于数组的一般情况；然而，我们将仅使用关联数组，以便让你更熟悉这种类型。我们将重复之前的例子，但会做一些改变：

```
demo@ubuntu:~/variable$ declare -A NAMES
demo@ubuntu:~/variable$ NAMES["John"]=12345
demo@ubuntu:~/variable$ NAMES["Luke"]=12344
demo@ubuntu:~/variable$ NAMES["Ivan from work"]=113312
demo@ubuntu:~/variable$ echo ${NAMES[*]}
113312 12344 12345
demo@ubuntu:~/variable$ NAMES=([Ida]=11111 ["That guy"]=122222)
demo@ubuntu:~/variable$ echo ${NAMES[*]}
11111 122222
```

发生了什么？首先，我们定义了一个由三个值对组成的数组。我们通过单独声明每个值对来实现这一点。之后，我们使用了另一种数组声明方式，但由于我们基本上是重新声明了该数组，因此我们使用的值完全替换了数组之前的值。我们本该做的是*添加*这些值。实现这一点有两种方法——一种是重新定义每一对值：

```
demo@ubuntu:~/variable$ NAMES["John"]=12345
demo@ubuntu:~/variable$ NAMES["Ivan from work"]=113312
demo@ubuntu:~/variable$ NAMES["Luke"]=12344
demo@ubuntu:~/variable$ echo ${NAMES[*]}
113312 11111 12344 12345 122222
```

现在我们可以看到数组中有了预期的值数量，尽管我们仍然不知道如何打印它们。

另一种方法是通过使用加法。我们将在例子中只更改一个字符来实现：

```
demo@ubuntu:~/variable$ declare -A NAMES
demo@ubuntu:~/variable$ NAMES["John"]=12345
demo@ubuntu:~/variable$ NAMES["Luke"]=12344
demo@ubuntu:~/variable$ NAMES["Ivan from work"]=113312
demo@ubuntu:~/variable$ NAMES+=([Ida]=11111 ["That guy"]=122222)
demo@ubuntu:~/variable$ echo ${NAMES[*]}
113312 11111 12344 12345 122222
```

你能注意到变化吗？我们所做的就是在等号前面使用加号操作符，告诉`bash`我们希望将这些键值对添加到数组中。这种表示法与我们使用`NAMES=NAMES+([Ida]=11111 ["That guy"]=122222)`完全相同——只是稍微简短一些。

我们需要知道的最后一件事是如何从数组中删除一个值。

解决方案很简单——有一个叫做`unset`的命令，它可以简单地删除与特定索引或键相关联的值。通常，这个命令用于键值对，因为在这种情况下更为合理，但你也可以在常规数组上使用它：

```
demo@ubuntu:~/variable$ echo ${NAMES[*]}
113312 11111 12344 12345 122222
demo@ubuntu:~/variable$ unset NAMES["John"]
demo@ubuntu:~/variable$ echo ${NAMES[*]}
113312 11111 12344 122222
```

现在，我们将解决一个大问题，并重新编写之前的脚本，使用我们唯一的关联数组。

这种做法的原理基于`bash`如何使用对象集合。我们将从所有的键创建一个这样的集合，然后打印出键和值。要直接访问键，我们可以使用感叹号：

```
#!/bin/bash
#we are declaring one associative array for pairs of values: 
declare -A PAIRS
PAIRS=(["John"]=12345 ["Luke"]=12344 ["Ivan from work"] =113312 \
["Ida"]=11111 ["That guy"]=122222)
#now we need to get them printed
for name in "${!PAIRS[@]}"
do
              echo Name:"$name" number:${PAIRS["$name"]} 
done
```

在这个脚本中，你需要注意一些小细节。例如，引用符号非常重要，因为我们的键包含空格。这里的一般规则是，只要你使用任何字符串作为某个值，它应该用引号括起来，以便正确解析空格。

`for`循环将逐一遍历键，键将作为整个键值对使用，包括空格。某些手册会指出，键必须是一个单词，但官方来说，它可以是任何东西。然而，这种类型变量的使用场景是，在大多数情况下我们将使用一个或两个单词：

```
demo@ubuntu:~/variable$ bash associative.sh 
Name:Ivan from work number:113312
Name:Ida number:11111
Name:Luke number:12344
Name:John number:12345
Name:That guy number:122222
```

## 它是如何工作的……

我们已经展示了数组在实践中的工作方式，如何创建它们，如何从中读取数据，以及如何删除数组中的单个元素。数组与常规变量的一个不同之处在于，变量只持有一个值，因此没有操作可以修改该值。如果我们需要改变它，我们只需重新声明整个值。而在处理数组时，我们处理的是一个数组中的多个值，这仍然意味着我们必须重新声明需要更改的任何值，但我们只是改变多个值中的一个，而不是整个数组。这也是我们有添加和删除元素操作的主要原因。

一些已经熟悉不同编程语言的你们可能会对某些定义方式感到有些困惑，特别是当我们谈论常规数组以及它们如何访问和打印单个值时。最令人困惑的部分可能是你如何完全跳过一段索引区间，却依然能得到一个有效的数组。我们将在下一个食谱中详细讲解这个问题，但`bash`在某些方面的模糊性是非常不一致的，而这就是其中之一。

关联数组是你不时需要用到的，特别是当你需要处理具有某些属性的对象时。每个键无法存储多个属性，但即便如此，这种情况也创造了一个不错的环境，因为这个单一的值可以是一个索引值，指向不同的数组，存储关于某个特定对象的其他所有信息。

## 另请参阅

数组很复杂，同时它们的语法也很简单。查看这些链接，你将看到更多的示例：

+   https://www.shell-tips.com/bash/arrays/

+   [`opensource.com/article/18/5/you-dont-know-bash-intro-bash-arrays`](https://opensource.com/article/18/5/you-dont-know-bash-intro-bash-arrays)

# 高级数组操作

现在我们已经完成了基本内容的处理，我们需要向你传授更多关于数组的知识。首先，我们需要做的是给你一些关于如何让你在脚本中创建的数组更加持久化的思路，因此我们需要处理数组的存储和恢复。

这很重要，因为数组可能相当大，具体取决于你创建的脚本。转储和重用脚本中的变量很容易，因为我们可以使用*source*来声明我们存储在文件中的变量。数组则更加复杂，因为它们可以包含多个变量，有时我们甚至需要从其他数据源创建它们。

## 准备工作

脚本的问题是它们有时需要处理大量数据。在许多情况下，我们可以使用文件来存储和加载数据。然而，也有一些情况，特别是关联数组，在处理大量数据时是必需的，这时我们需要知道如何将这些数据保存到磁盘并重新使用它。我们将向你展示如何解决这些问题，并在何时使用它提供建议。

## 如何实现…

在谈论这些内容时，我们总是需要一个示例，来说明某个特性为何有意义。我们处理过的某些`bash`特性过于通用，因此我们的示例也必须是通用的。而关联数组不同，尽管它们可以用于多种情况，但有些场景如此常见，以至于你会在考虑如何以及为什么使用它之前，就自动开始声明一个数组。

最常见的场景是保存用户设置。

任何处理任务的大型脚本都必须是可配置的。我们提到过，当我们说可以包含文件时，最常见的文件是包含变量的文件，用于存储不同设置的值。

大多数时候，脚本中的所有设置看起来像这样：

```
USER=demo
CMD=testing
HOSTNAME=demounit
```

这些设置只是一个示例，但大多数脚本都会在单独的文件中或脚本的开始部分有一个这样的块。

请注意，它们都有`KEY=VALUE`的格式，看起来像是为联想数组而创建的。话虽如此，我们需要强调的是，使用任何编程语言的特性时，要做最有利于性能和清晰度的选择，而不是仅仅因为你*知道*在某个特定的情况下使用了这个特性就去使用它。

这就是你需要经验的地方。如果一个脚本有一组设置，在我们最初加载并保存它们时它们就不再改变，那么使用数组毫无意义。如果一个脚本在启动时只定义了少量变量——在这种情况下也没有必要使用数组。

但如果一个脚本在不同的执行过程中有很多变化，并且如果你使用它们来存储一些重要的运行时操作，这些操作不仅在脚本开始时需要，而且在运行过程中也需要，那么数组可能是一个解决方案。

我们的示例将是一个小脚本，它需要记住一些设置，我们将使用数组从磁盘加载它们，使用它们，并稍后将它们存回磁盘。在这个过程中，我们还将对给定的数组执行一些任务，以展示如何在脚本中操作键值对。

但在我们做这些之前，我们需要通过一些高级示例来向你展示一些我们之前略过的内容是如何工作的。

首先，我们将处理使用`*`和`@`运算符读取数组中的索引和键的区别。我们之前说过，对于一个给定的数组，这两种运算符是不同的遍历所有索引的方式。这里有一个示例来说明这一点：

```
demo@ubuntu:~/variable$ SOMEARRAY=(0 1 2 3 4 5)
demo@ubuntu:~/variable$ echo ${!SOMEARRAY[*]}
0 1 2 3 4 5
demo@ubuntu:~/variable$ echo ${!SOMEARRAY[@]}
0 1 2 3 4 5
```

联想数组也一样：

```
demo@ubuntu:~/variable$ declare -A PAIRS
demo@ubuntu:~/variable$ PAIRS=(["John"]=12345 ["Luke"]=12344 ["Ivan from work"]=113312 ["Ida"]=11111 ["That guy"]=122222)
demo@ubuntu:~/variable$ echo ${!PAIRS[@]}
Ivan from work Ida Luke John That guy
demo@ubuntu:~/variable$ echo ${!PAIRS[*]}
Ivan from work Ida Luke John That guy
```

到目前为止，结果乍一看是相同的。这就是`bash`中的一个问题，它有时会让你感到绝望，因为当我们在循环中使用它们时，结果会有很大差异，我们将通过创建一个小的示例脚本来展示这一点：

```
#!/bin/bash
declare -A PAIRS
PAIRS=(["John"]=12345 ["Luke"]=12344 ["Ivan from work"] =113312 \
["Ida"]=11111 ["That guy"]=122222)
#now we are going to print keys, once using @ and then using \
#*
echo "for first example we are using @ sign"
for name in "${!PAIRS[@]}"
do
             echo Number: ${PAIRS["$name"]}
             echo Name: "$name"
             echo
done
echo --------------------------
echo "then we are using * sign"
for name in "${!PAIRS[*]}"
do
        echo Number: ${PAIRS["$name"]}
             echo Name: "$name"
             echo
done
```

如果我们运行这个脚本，且这两个表达式返回相同的值集，那么输出应该是相同的。但如果我们真的运行它，得到的结果却是这样的：

```
demo@ubuntu:~/variable$ bash arrayops.sh 
for first example we are using @ sign
Number: 113312
Name: Ivan from work
Number: 11111
Name: Ida
Number: 12344
Name: Luke
Number: 12345
Name: John
Number: 122222
Name: That guy
--------------------------
then we are using * sign
Number:
Name: Ivan from work Ida Luke John That guy
```

我们看到，当我们使用`@`时，得到了预期的结果，但一旦我们用`*`替换它，我们可以看到键（或索引），但没有返回值。

其背后的原因，像往常一样，是`bash`的工作方式。使用`@`符号告诉`bash`我们是尝试分别获取每个索引或键。另一方面，使用`*`符号会让`bash`将所有的索引或键作为一个由空格分隔的单一字符串返回。所以，在一种情况下，我们的脚本会逐个读取每个元素，而另一个循环只运行一次。由于我们试图查找的值与数组中的任何单个键值都不同，这次运行没有返回结果。最终，这两个表达式并不相同，但简单的输出是相同的——不要被它迷惑了。

现在，我们将创建一个用于操作数组的主脚本，并添加一些元素。

显然，起点是创建一个包含我们数组的文件。有多种方法可以做到这一点，有些方法比其他方法复杂，但几乎所有方法都依赖于使用某种循环来遍历数组，并将其读入或写入文件。

我们将以经典的 Linux 方式来做，使用`declare`语句。如果我们使用`-p`选项，我们就是在告诉它打印特定变量的定义及其存储的值：

```
demo@ubuntu:~/variable$ declare -p PAIRS
declare -A PAIRS=(["Ivan from work"]="113312" [Ida]="11111" [Luke]="12344" [John]="12345" ["That guy"]="122222" )
```

显然，这是非常棒的，因为这是我们需要记住的唯一事项，以确保变量中存储的所有内容都被保存。为了保存它，我们只需将其重定向到磁盘上的文件：

```
demo@ubuntu:~/variable$ declare -p PAIRS > PAIRS.save
```

为了证明这个方法有效，我们将取消设置变量，并验证它已经被删除，以确保值只存在于文件中：

```
demo@ubuntu:~/variable$ unset PAIRS
demo@ubuntu:~/variable$ echo ${!PAIRS[@]}
```

现在，让我们看看如何从磁盘重新加载脚本。注意，`declare`语句的输出实际上是一个`declare`语句，用于定义变量。如果我们从磁盘加载并执行它，一切应该都没问题：

```
demo@ubuntu:~/variable$ source PAIRS.save 
demo@ubuntu:~/variable$ echo ${!PAIRS[@]}
Ivan from work Ida Luke John That guy
```

互联网充满了更复杂的解决方案，但对于一个合理大小的数组，这个方法应该非常有效。与此同时，它也很容易在脚本中读取，且包含数据的文件是通用格式，可以被任何安装了`bash`的系统上的其他版本读取（前提是`bash`版本为 4.0，因为这是这些类型的数组首次被引入的版本）。

我们现在知道如何将数组读写到磁盘，但还能对它做什么呢？在我们的示例中，我们将切换到常规数组，向你展示一些可以做的事情：

```
demo@ubuntu:~/variable$ REGULAR=( zero one two three five four \
)
demo@ubuntu:~/variable$ echo ${REGULAR[@]}
zero one two three five four
```

所以，我们创建了一个数组，并且犯了一个错误。由于数组已经是错乱的顺序，我们将再次对其进行重新洗牌（`shuf`命令会随机打乱数组）：

```
demo@ubuntu:~/variable$ shuf -e "${REGULAR[@]}"
three
one
zero
five
two
four
```

虽然你可以为值的随机化创建自己的解决方案，但使用外部命令是最简单的解决方案。

洗牌很简单，但它不是永久性的。命令的作用是将我们的数组作为输入，打乱其中的值，然后打印结果，而原始数组保持不变。

我们将数组重新赋值为命令的结果。我们还创建了另一个变量并打印它，主要是为了展示洗牌是实时发生的，而且每次启动`shuf`命令时结果都会不同。

排序数组会更麻烦，因为它需要某种排序机制。要么你自己创建一个，要么你可以小心地使用`bash`中已包含的`sort`命令。

接下来，我们将展示如何使用索引范围。我们将重置数组，然后展示一些示例。当我们声明范围时，实际上，我们是在使用`bash`内置的一个机制，它允许我们定义一个数字范围。我们已经用它来创建循环和迭代器，所以这对你来说应该不算惊讶：

```
demo@ubuntu:~/variable$ REGULAR=( zero one two three four five \
)
demo@ubuntu:~/variable$ echo ${REGULAR[*]} 
zero one two three four five
demo@ubuntu:~/variable$ echo ${REGULAR[*]:2:3}
two three four
demo@ubuntu:~/variable$ echo ${REGULAR[*]:0:3}
zero one two
demo@ubuntu:~/variable$ echo ${REGULAR[*]:0:}
demo@ubuntu:~/variable$ echo ${REGULAR[*]:0:2}
zero one
demo@ubuntu:~/variable$ echo ${REGULAR[*]:3}
three four five
demo@ubuntu:~/variable$ echo ${REGULAR[*]:2}
two three four five
demo@ubuntu:~/variable$ echo ${REGULAR[*]:2:-2}
bash: -2: substring expression < 0
```

我们在`bash`中使用的是标准语法。变量后的第一个数字是我们想要打印的起始索引，后面的可选数字是我们需要的值的数量。请注意，这里负数不起作用，和其他一些地方不同；我们不能通过这种方式从数组的末尾往回查找。

接下来我们可以做的是拼接两个数组。

根据你想做的事情，这个操作的结果可能并不会得到你预期的结果：

```
demo@ubuntu:~/variable$ ANOTHER=(sixth seventh eighth ninth)
demo@ubuntu:~/variable$ NEW="${REGULAR[*]} ${ANOTHER[*]}"
demo@ubuntu:~/variable$ echo ${NEW[*]}
zero one two three four five sixth seventh eighth ninth
demo@ubuntu:~/variable$ echo ${NEW[@]}
zero one two three four five sixth seventh eighth ninth
```

另一个可以执行的操作是计算数组中的值的数量。我们之前也做过这件事，所以让我们检查一下我们的数组是否已正确合并：

```
demo@ubuntu:~/variable$ echo ${#REGULAR[@]}
6
demo@ubuntu:~/variable$ echo ${#ANOTHER[@]}
4
demo@ubuntu:~/variable$ echo ${#NEW[@]}
1
```

这里发生了什么？

在我们的拼接操作中，我们犯了一个大错。我们想做的是创建一个新数组，用于保存两个数组的所有值。但我们做的是创建了一个`string`变量，它包含了所有数组值组成的一个大字符串。我们需要通过使用括号来修复这个问题：

```
demo@ubuntu:~/variable$ NEW=(${REGULAR[*]} ${ANOTHER[*]})
demo@ubuntu:~/variable$ echo ${#NEW[@]}
10
demo@ubuntu:~/variable$ NEW=(${REGULAR[@]} ${ANOTHER[@]})
demo@ubuntu:~/variable$ echo ${#NEW[@]}
10
demo@ubuntu:~/variable$ declare -p NEW
declare -a NEW=([0]="zero" [1]="one" [2]="two" [3]="three" [4]="four" [5]="five" [6]="sixth" [7]="seventh" [8]="eighth" [9]="ninth")
```

所以，我们创建了数组并检查了它包含多少个值。由于我们想确保完全准确，因此使用了`declare`语句来展示所有的索引/值对。

在继续之前，我们需要通过使用一对引号制造一个小错误：

```
demo@ubuntu:~/variable$ NEW=("${REGULAR[@]} ${ANOTHER[@]}")
demo@ubuntu:~/variable$ echo ${#NEW[@]}
9
demo@ubuntu:~/variable$ declare -p NEW
declare -a NEW=([0]="zero" [1]="one" [2]="two" [3]="three" [4]="four" [5]="five sixth" [6]="seventh" [7]="eighth" [8]="ninth")
```

我们做的事情是创建了一个与我们第一个例子非常相似的东西，但同时完全错误。一个值是两个字符串的组合，而不是两个单独的值。尝试所有不同的引号组合，看看它们如何工作，以及使用`@`或`*`是否会对结果数组产生影响。

## 它是如何工作的……

我们已经详细处理了对数组可以执行的各种操作。剩下的就是看看关于数组还有什么需要了解的内容，以及如何检查数组是否包含某个值。数组的长度——或者更准确地说，值的数量是我们刚刚看过的。如果你需要知道数组是否已经存在，可以检查它的长度是否大于`0`。这将告诉你，要么你测试的数组没有定义，要么它不包含任何元素。如果你明确想检查一个变量是否已定义，可以使用`declare`语句并统计结果。在我们的示例中，我们有一个名为`TEST1`的变量和一个未定义的名字`TEST2`：

```
demo@ubuntu:~/variable$ TEST1=()
demo@ubuntu:~/variable$ declare -p TEST1
declare -a TEST1=()
demo@ubuntu:~/variable$ declare -p TEST2
bash: declare: TEST2: not found
demo@ubuntu:~/variable$ echo ${#TEST1[@]}
0
demo@ubuntu:~/variable$ echo ${#TEST2[@]}
0
```

在大多数情况下，只检查值的数量就足够了，但有时你需要知道一个变量是否已经定义。

另一个常见的操作是尝试找出数组中是否包含特定的值。你可以通过创建自己的循环来检查值，或者使用`bash`已经提供的内置测试。例如，你可以像这样做：

```
demo@ubuntu:~/variable$ [[ ${REGULAR[*]} =~ "one" ]] && echo \
yes || echo no
yes
demo@ubuntu:~/variable$ [[ ${REGULAR[*]} =~ "something" ]] && \
echo yes || echo no
no
```

我们再次使用了一行逻辑表达式来快速查看结果。

现在，这里有一个小脚本，它将展示我们在这个教程中学到的一些内容：

```
        #!/bin/bash
        #check if settings exist
                   function checkfile {
               if [ -f setting.list ] 
                          then
                                         return 0
                          else.
                                      return 1
                                       fi
}
function assign_settings {
             echo assigning settings
             SETTINGS=(["USER"]=John ["LOCALDIR"]=$PWD \
["HOSTNAME"]=hostname)

}
declare -A SETTINGS
SETTINGS=()
if checkfile
then 
             source setting.list
else 
             assign_settings
fi
echo Settings are:
for name in "${!SETTINGS[@]}"
do
             echo "$name"=${SETTINGS["$name"]}
done
declare -p SETTINGS > setting.list
```

我们已经知道了大部分内容，但我们还是会通过这个脚本进行回顾。

脚本中的第一个函数用来测试文件是否存在。我们本可以在代码后面使用`if`语句来做同样的事情，但我们想提醒你如何使用函数和逻辑检查。如果文件存在，函数返回`0`，如果文件不存在，则返回`1`。

接下来是我们称之为`assign_settings`的函数，它在文件未找到时使用。它所做的就是简单地创建一个包含一些数据的新关联数组。

然后，我们进入脚本的主体部分，首先声明我们的数组，因为它们不能隐式声明。接着，我们决定是否已经保存了文件，并判断是否应该从文件中加载数组，或者是否需要重新分配默认值。

之后，我们只是输出值，然后将它们保存到磁盘。

在普通脚本中，这通常是导入和保存重要设置的部分。脚本的其他部分将在保存变量的那一行之前。

我们将连续两次启动脚本。结果应该是，它会检测到我们没有配置，并为我们创建配置：

```
demo@ubuntu:~/variable$ bash settings.sh 
assigning settings
Settings are:
USER=John
HOSTNAME=hostname
LOCALDIR=/home/demo/variable
demo@ubuntu:~/variable$ bash settings.sh 
Settings are:
USER=John
HOSTNAME=hostname
LOCALDIR=/home/demo/variable
```

当你在做类似的事情时，我们还必须提醒你，局部变量和全局变量可能会带来大问题。如果你在函数中声明任何应该是全局的变量，或者更糟的是，如果你从函数中引用它，务必小心，因为作用域会限制变量在脚本中的传播。

在这里，我们将暂时离开数组，转向更有趣的内容——开始创建一些接口。

## 另见

+   *如何在 Shell 脚本中使用 Bash 数组*：[`linuxconfig.org/how-to-use-arrays-in-bash-script`](https://linuxconfig.org/how-to-use-arrays-in-bash-script)

+   *终极 Bash 数组教程，含 15 个示例*：[`www.thegeekstuff.com/2010/06/bash-array-tutorial/`](https://www.thegeekstuff.com/2010/06/bash-array-tutorial/)
