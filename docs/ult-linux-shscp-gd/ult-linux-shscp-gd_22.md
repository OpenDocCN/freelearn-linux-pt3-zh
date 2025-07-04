

# 第二十二章：介绍 Z Shell 脚本编写

到目前为止，我们主要讨论了 `bash`，因为它是 Linux 的主要登录 shell 和脚本环境。近年来，一个相对较新的登录 shell **Z Shell** (`zsh`) 作为默认登录 shell 越来越受欢迎。但即使在预装 `zsh` 作为登录 shell 的操作系统上，`bash` 仍然存在，并且大多数人仍然将其用于脚本编写。然而，需要注意的是，在 `zsh` 中进行脚本编写确实有一些优势，比如更好的数学能力、增强的变量展开功能，以及能够使用本地 `zsh` 命令而不是像 `find`、`grep` 或 `sed` 这样的实用程序。这些特性可以帮助您创建比等效的 `bash` 脚本更简单、更轻量的脚本。

大多数关于 `zsh` 的文章和 YouTube 视频都集中在用户界面增强上，而很少关注 `zsh` 脚本编写。不幸的是，少数关注 `zsh` 脚本编写的书籍和教程写得不太好，您不会从中获得太多收获。希望我能提供一些澄清。

与其提供详细的教程，我更想简要概述 `zsh` 脚本编写，让您自己决定是否适合您。

本章的主题包括：

+   介绍 `zsh`

+   安装 `zsh`

+   理解 `zsh` 脚本编写的独特功能

+   使用 `zsh` 模块

如果您准备好了，让我们开始吧。

# 技术要求

您可以在任何 Linux 虚拟机上使用本章内容。与往常一样，您可以通过执行以下操作获取演示脚本：

```
git clone https://github.com/PacktPublishing/The-Ultimate-Linux-Shell-Scripting-Guide.git 
```

# 介绍 zsh

Z Shell 由 Paul Falstad 于 1990 年创建。它是 Bourne Shell 的扩展，同时还包括来自 `bash`、Korn Shell (`ksh`) 和 C Shell (`tcsh`) 的功能。

**C Shell** 曾在 Unix 和类 Unix 发行版中颇受欢迎，但它与你之前接触的任何东西大不相同。编写 C Shell 脚本更类似于编写 C 语言程序，而不是你所熟悉的方式。所以如果你是 C 语言程序员，你可能会喜欢它。如果不是，那你可能不会那么喜欢。

Z Shell 是 macOS 和 Kali Linux 的默认登录 shell。对于几乎所有其他系统，您都需要自行安装。至于 `zsh` 脚本编写，大多数在 `bash` 中学到的内容也适用于 `zsh`。因此，在本章中，我将仅介绍 `zsh` 的独特功能，并展示一些简单的脚本示例。

# 安装 zsh

在我尝试过的所有 Linux 和 BSD 类型发行版以及 OpenIndiana 上，都可以使用 `zsh` 包。在所有情况下，包名称都是 `zsh`，因此您可以使用正常的包管理器安装它。

在 BSD 系统中，你会遇到与`bash`相同的路径问题。也就是说，在 BSD 系统中，`zsh`的可执行文件位于`/usr/local/bin/`目录下，而不是`/bin/`目录下。不过没关系，你可以像在`bash`中那样，在`/bin/`目录下创建一个符号链接。在我的 FreeBSD 机器上，命令如下：

```
donnie@freebsd-zfs:~ $ which zsh
/usr/local/bin/zsh
donnie@freebsd-zfs:~ $ sudo ln -s /usr/local/bin/zsh /bin/zsh
donnie@freebsd-zfs:~ $ which zsh
/bin/zsh
donnie@freebsd-zfs:~ $ 
```

现在，如果你需要在 Linux、OpenIndiana 和 BSD 机器上运行你的`zsh`脚本，可以使用相同的 shebang 行，格式如下：

```
#!/bin/zsh 
```

如果你想将`zsh`作为临时登录 shell 使用，可以直接在当前 shell 的命令提示符下输入`zsh`。如果你想将`zsh`设置为永久登录 shell，则可以在 Linux 和 BSD 系统上使用`chsh`命令，在 OpenIndiana 上使用`passwd`命令。Linux 和 BSD 系统上可以这样设置：

```
donnie@ubuntu2404:~$ chsh -s /bin/zsh
Password:
donnie@ubuntu2404:~$ 
```

在 OpenIndiana 上的表现如下：

```
donnie@openindiana:~$ passwd -e
Enter existing login password:
Old shell: /bin/bash
New shell: /bin/zsh
passwd: password information changed for donnie
donnie@openindiana:~$ 
```

请注意，当你为自己的用户账户更改 shell 时，系统会提示你输入自己的用户密码。然而，这与`sudo`无关，因为即使是非特权用户，也可以在 Linux/BSD 系统上使用`chsh`，在 OpenIndiana 上使用`passwd -e`来更改自己的默认 shell。

第一次使用`zsh`登录时，你会看到一个设置菜单，内容如下所示：

```
This is the Z Shell configuration function for new users,
zsh-newuser-install.
You are seeing this message because you have no zsh startup files
(the files .zshenv, .zprofile, .zshrc, .zlogin in the directory
~).  This function can help you with a few settings that should
make your use of the shell easier.
You can:
(q)  Quit and do nothing.  The function will be run again next time.
(0)  Exit, creating the file ~/.zshrc containing just a comment.
     That will prevent this function being run again.
(1)  Continue to the main menu.
--- Type one of the keys in parentheses --- 
```

按下**1**键返回主菜单，并选择你首选的设置选项。配置将保存到你家目录下的`.zshrc`文件中。

接下来，让我们来看一下`zsh`脚本的一些独特功能。

# 理解`zsh`脚本的独特功能

在`zsh`中编写脚本时，你可以利用一些`bash`没有的增强功能。这里简要回顾一下其中的一些增强功能。

## 变量扩展的区别

在*第八章—基本 Shell 脚本构建*中，我解释了**变量扩展**的概念，这个概念有时也叫做**参数扩展**。这使得你可以编写一些很酷的脚本来完成很酷的事情，比如一次性修改一批文件的文件扩展名。`bash`中大部分的变量扩展构造在`zsh`中也能使用，但`zsh`还提供了一些额外的扩展功能，增加了更多的能力。让我们来看几个例子。

### 替换值

在`zsh`中替换值的方式与`bash`相同，唯一的例外是，如果你在包含感叹号的文本字符串中替换值，在`zsh`脚本中需要对感叹号进行转义。例如，在`bash`中，效果如下：

```
donnie@fedora:~$ unset name
donnie@fedora:~$ echo "Hello, ${name:-Guest}!"
Hello, Guest!
donnie@fedora:~$ name=Horatio
donnie@fedora:~$ echo "Hello, ${name:-Guest}!"
Hello, Horatio!
donnie@fedora:~$ 
```

正如我在*第八章*中所解释的，`variable_name:-default_value`结构为任何尚未赋值的变量提供了默认值。这个结构本应在`zsh`中也能工作，但看我尝试它时发生了什么：

```
ubuntu2404% unset name
ubuntu2404% echo "Hello, ${name:-Guest}!"
dquote> "
Hello, Guest
ubuntu2404% 
```

当我在输入完`echo`命令后按下**Enter**键，`zsh`会将我带到`dquote>`提示符。如果我第二次按下`"`键，就能得到我想要的输出。为了第一次就正确输出，我只需在`!`之前加上`\`，像这样：

```
ubuntu2404% unset name               
ubuntu2404% echo "Hello, ${name:-Guest}\!"
Hello, Guest!
ubuntu2404% name=Horatio
ubuntu2404% echo "Hello, ${name:-Guest}\!"
Hello, Horatio!
Ubuntu2404% 
```

所以，在感叹号前加上`\`，它就能正常工作。我不知道为什么`zsh`中感叹号需要转义，不过，反正能用就行，对吧？

### 子字符串替换

子字符串替换在`bash`中部分有效，但不是完全有效。我的意思是这样。

我将首先创建`name`变量，赋值为`lionel`。然后，在`bash`和`zsh`中，我会尝试将第一个小写字母*l*替换成大写字母*L*。首先在`bash`中，像这样：

```
donnie@fedora:~$ echo $SHELL
/bin/bash
donnie@fedora:~$ name="lionel"
donnie@fedora:~$ echo "${name/l/L}"
Lionel
donnie@fedora:~$ 
```

现在，在`zsh`中：

```
ubuntu2404% echo $SHELL
/bin/zsh
ubuntu2404% name="lionel"
ubuntu2404% echo "${name/l/L}"
Lionel
ubuntu2404% 
```

在`${name/l/L}`构造中，你可以看到我首先列出了`name`变量，并用正斜杠后跟要替换的字母，然后是另一个正斜杠和要替换成的字母。如你所见，这在`bash`和`zsh`中都能正常工作。我还可以通过在`name`变量后使用两个正斜杠来替换所有的小写*l*，像这样：

在`bash`中：

```
donnie@fedora:~$ echo "${name//l/L}"
LioneL
donnie@fedora:~$ 
```

在`zsh`中：

```
ubuntu2404% echo "${name//l/L}"               
LioneL
ubuntu2404% 
```

这个方法在`bash`和`zsh`中都能使用。那么，如果它在这两种 shell 中都能工作，为什么我还要向你展示呢？其实，原因是：在`bash`中，这个方法仅在字符串不包含空格的情况下有效。

这里是我想表达的意思：

在`bash`中：

```
donnie@fedora:~$ names="Donnie and Vicky and Cleopatra"
donnie@fedora:~$ echo "${names//and/&}"
Donnie and Vicky and Cleopatra
donnie@fedora:~$ 
```

这一次，我想把所有的*and*替换成&符号。但在`bash`中这行不通，因为字符串包含空格。我们来试试在`zsh`中做这个操作。

在`zsh`中：

```
ubuntu2404% names="Donnie and Vicky and Cleopatra"
ubuntu2404% echo "${names//and/&}"
Donnie & Vicky & Cleopatra
ubuntu2404% 
```

在`zsh`中，字符串中包含空格并不影响操作。

### 大小写转换

这里是完全相反情况的示例。这一次，我将展示一个在`bash`中有效，但在`zsh`中无效的例子。然后，我会展示在`zsh`中使用的替代方法。

首先，假设我们想要将分配给`name`变量的值的首字母大写。以下是在`bash`中的方法：

```
donnie@fedora:~$ name=horatio
donnie@fedora:~$ echo "${name^}"
Horatio
donnie@fedora:~$ 
```

我这里做的只是将一个`^`放在`name`变量的`echo`语句后面。现在，假设我想让所有字母大写，我可以使用两个`^`字符，像这样：

```
donnie@fedora:~$ echo "${name^^}"
HORATIO
donnie@fedora:~$ 
```

如果你有一个全大写的字符串，可以使用一个或两个逗号，将第一个字母或所有字母转换为小写，就像这样：

```
donnie@fedora:~$ name=HORATIO
donnie@fedora:~$ echo "${name,}"
hORATIO
donnie@fedora:~$ echo "${name,,}"
horatio
donnie@fedora:~$ 
```

在`zsh`中，完全不管用，正如你看到的：

```
ubuntu2404% name=horatio
ubuntu2404% echo "${name^}"
zsh: bad substitution
ubuntu2404% echo "${name^^}"
zsh: bad substitution
ubuntu2404% name=HORATIO 
ubuntu2404% echo "${name,}"
zsh: bad substitution
ubuntu2404% echo "${name,,}"
zsh: bad substitution
ubuntu2404% 
```

我之所以向你展示这个，是因为一些`zsh`脚本书说这种方法在`zsh`中有效。但如你所见，显然无效。

那么，在`zsh`中的替代方法是什么呢？其实，你只需使用一个内建的`zsh`函数。我们首先使用`U`函数将整个字符串转换为大写：

```
ubuntu2404% name=horatio
ubuntu2404% echo "${(U)name}"
HORATIO
ubuntu2404% 
```

我没有在变量名后面加一对`^`字符，而是直接在`name`变量前加了一个`(U)`。

据我所知，`zsh`中并没有直接转换第一个字母的函数。所以，我们只能像这样转换第一个字母：

```
ubuntu2404% name=horatio
ubuntu2404% capitalized_name=${name/h/H}
ubuntu2404% echo $capitalized_name
Horatio
ubuntu2404% 
```

这和我在上一部分所展示的方法是一样的。

接下来，我们来做一些通配符操作。

### 扩展文件通配符

“**文件模式匹配**”，你问什么？嗯，其实你在本书的整个过程中都在使用它。只是我从未告诉过你它叫什么。它的意思是，你可以使用通配符字符，比如`*`和`?`，同时处理多个文件。例如，你可以这样做：

```
donnie@fedora:~$ ls -l *.zip
-rw-r--r--. 1 root   root      32864546 Jul 27  2023 15827_zip.zip
-rw-r--r--. 1 root   root      49115374 Jul 27  2023 21261.zip
-rw-r--r--. 1 root   root      36996449 Jul 27  2023 46523.zip
. . .
. . .
-rw-r--r--. 1 root   root      21798397 Jul 27  2023 tmvx.zip
-rw-r--r--. 1 root   root         60822 Jul 27  2023 U_CAN_Ubuntu_20-04_LTS_V1R4_STIG_SCAP_1-2_Benchmark.zip
-rw-r--r--. 1 root   root        425884 Jul 27  2023 zoneinfo.zip
donnie@fedora:~$ 
```

我在这里做的只是使用`*`通配符来查看我目录中的所有`.zip`文件。

你可以在`zsh`中做所有这些操作，甚至更多。唯一的条件是你需要开启`extendedglob`功能。要验证它是否开启，检查你家目录下的`.zshrc`文件。你应该能看到像这样的行：

```
setopt autocd beep extendedglob nomatch notify 
```

如果`extendedglob`选项开启，那么你就可以使用它了。如果没有开启，只需添加它。那么，现在我们来看看几个扩展文件模式匹配的例子。

为什么`zsh`中的扩展文件模式匹配有用？嗯，实际上，在许多情况下，你可以使用`zsh`的扩展模式匹配功能，而不是使用`find`工具。你可能会觉得这比使用`find`更简单。或者，如果你习惯使用`find`，你可能更喜欢使用它。我会让你自己做决定。

此外，为了演示的目的，我主要用`ls`工具来展示这个功能。但你也可以使用文件模式匹配与其他工具一起使用，比如`cp`、`mv`、`rm`、`chmod`或`chown`。

#### 过滤`ls`输出

启用这个酷炫的功能后，你可以做一些很酷的事情，比如排除你不想看到的文件。例如，假设你想查看目录中的所有文件，但不包括`.sh`文件。只需这样做：

```
ubuntu2404% ls -d ^*.sh          
bashlib-0.4		    My_Files
demo_files		    supersecret
deshc-deb		   	    supersecret_trace.txt
Documents		           sysinfo.html
encrypted_password	    sysinfo_posix.lib
encrypted_password.sh.x.c  test.sh.x
expand_1.txt		    user_activity_for_donnie_2024-05-12_09-30.txt
expand_2.txt		    user_activity_for_donnie_2024-07-22_10-40.txt
expand.txt		    user_activity_for_horatio_2024-07-06_07-12.txt
FallOfSudo		    while_demo
file2.txt		           while_demo.sh.x.c
file3.jpg
ubuntu2404% 
```

在`*.sh`前面的`^`是一个否定符号。这会阻止`ls`命令显示`.sh`文件。

注意，我必须为`ls`添加`-d`选项。否则，你还会看到任何可能存在的子目录中的文件。由于某种原因，`^`可以过滤掉我顶层目录中不需要的文件，但它不会过滤掉子目录中的不需要的文件。所以，如果没有`-d`，你还会看到子目录中的所有文件，包括那些你想过滤掉的文件。

你也可以使用`*~`操作符来实现相同的效果。看起来是这样的：

```
ubuntu2404% ls -d *~*.sh
bashlib-0.4		   My_Files
demo_files		   supersecret
deshc-deb		          supersecret_trace.txt
Documents		           sysinfo.html
encrypted_password	    sysinfo_posix.lib
encrypted_password.sh.x.c  test.sh.x
expand_1.txt		    user_activity_for_donnie_2024-05-12_09-30.txt
expand_2.txt		    user_activity_for_donnie_2024-07-22_10-40.txt
expand.txt		    user_activity_for_horatio_2024-07-06_07-12.txt
FallOfSudo		    while_demo
file2.txt		           while_demo.sh.x.c
file3.jpg
ubuntu2404% 
```

如你所见，输出与`ls -d ^.sh`的输出完全相同。要排除多种类型的文件，只需使用*或*操作符(`|`)，像这样：

```
ubuntu2404% ls -d *~(*.sh|*.jpg|*.txt|*.c)
bashlib-0.4  Documents		 My_Files      sysinfo_posix.lib
demo_files   encrypted_password  supersecret   test.sh.x
deshc-deb    FallOfSudo		 sysinfo.html  while_demo
ubuntu2404% 
```

所以在这个例子中，我过滤掉了`.sh`、`.jpg`、`.txt`和`.c`文件。现在，让我们看看下一个技巧。

#### 分组`ls`搜索

你在`bash`中做不到的另一件事是将`ls`搜索结果进行分组，像这样：

```
ubuntu2404% ls -ld (ex|test)*
-rw-r--r-- 1 donnie donnie    52 Apr 29 20:02 expand_1.txt
-rw-r--r-- 1 donnie donnie   111 Apr 29 20:02 expand_2.txt
-rw-r--r-- 1 donnie donnie    51 Apr 29 20:02 expand.txt
-rwxrw-r-- 1 donnie donnie    48 Jun 30 20:26 test.sh
-rw-rw-r-- 1 donnie donnie     0 Jul  5 19:31 test.sh.dec.sh
-rwxr-xr-x 1 donnie donnie 11440 Jul  5 19:12 test.sh.x
ubuntu2404% 
```

在这里，我使用`|`符号作为*或*操作符，查看所有文件，其文件名的前部分是`ex`或`test`。

#### 使用文件模式匹配标志

正如你所知道的，Linux 和 Unix 操作系统通常区分大小写。因此，如果你使用 `ls *.jpg` 命令来查看所有 JPEG 类型的图像文件，你将完全错过任何可能有 `*.JPG` 扩展名的文件。使用 `zsh` 时，你可以使用**通配符标志**来使命令不区分大小写。下面是一个例子：

```
ubuntu2404% touch graphic1.jpg graphic2.JPG graphic3.jpg file3.jpg
ubuntu2404% ls *.jpg
file3.jpg  graphic1.jpg  graphic3.jpg
ubuntu2404% ls (#i)*.jpg
file3.jpg  graphic1.jpg  graphic2.JPG  graphic3.jpg
ubuntu2404% 
```

第二个 `ls` 命令中的 `(#i)` 就是通配符标志。`#` 表示这是一个标志，它会出现在你想要匹配的模式前面。

`(#l)` 标志会匹配小写或大写的模式，如果你为搜索指定小写模式，它就会匹配小写模式。如果你指定大写模式，那么它只会匹配大写模式。下面是它的样子：

```
ubuntu2404% ls (#l)*.jpg
file3.jpg  graphic1.jpg  graphic2.JPG  graphic3.jpg
ubuntu2404% ls (#l)*.JPG
graphic2.JPG
ubuntu2404% 
```

你可以看到，当我指定 `*.jpg` 作为模式时，它也找到了 `.JPG` 文件。但是，当我指定 `.JPG` 作为模式时，它只找到了 `.JPG` 文件。

现在，让我们扩展一些目录。

#### 扩展目录

你可以使用**通配符限定符**查看不同类型的目录或文件。我们将从目录的限定符开始，具体包括：

+   `*(F)`：这个扩展了非空目录。

+   `*(^F)`：这个扩展了普通文件和空目录。

+   `*(/^F)`：这个仅扩展空目录。

首先，让我们查看我主目录中的非空目录。

```
ubuntu2404% ls *(F)            
bashlib-0.4:
bashlib     config.cache  config.status  COPYING  Makefile     README
bashlib.in  config.log	  configure	 INSTALL  Makefile.in
. . .
. . .
My_Files:
afile.txt			somefile.txt	 yetanotherfile.txtx
anotherfile.txt			somepicture.jpg  yetanotheryetanotherfile.txt
haveicreatedenoughfilesyet.txt	somescript.sh
ubuntu2404% 
```

你可以看到，这个命令显示了非空目录及其内容。但它没有显示我顶层主目录中的任何文件。所以，一切正常。

现在，我想查看我顶层主目录中的所有文件，以及可能存在的任何空目录。我会这样做：

```
ubuntu2404% ls *(^F)
acl_demo.sh		   rsync_password.sh
diskspace.sh		   shutdown-update.sh
. . .
. . .
rootlock_2.sh		   while_demo.sh.x.c
Documents:
empty_directory:
ubuntu2404% 
```

这两个空目录出现在输出的底部。

最后，我只想看到空目录，如下所示：

```
ubuntu2404% ls *(/^F)
Documents:
empty_directory:
ubuntu2404% 
```

很酷吧？

接下来，让我们看看文件扩展。

#### 扩展文件

扩展文件的方式有几种。首先，我们将根据文件或目录的文件类型进行扩展。

##### 扩展文件和目录

如果你想查看目录中的文件，但又不想看到任何目录，可以这样做：

```
ubuntu2404% ls *(.)
acl_demo.sh		   rsync_password.sh
diskspace.sh		   shutdown-update.sh
encrypted_password	   sudo_demo1.sh
. . .
. . .
rootlock_1.sh		   while_demo.sh
rootlock_2.sh		   while_demo.sh.x.c
ubuntu2404% 
```

如果你只想看到你的目录，无论是空目录还是非空目录，可以这样做：

```
ubuntu2404% ls *(/)
bashlib-0.4:
bashlib     config.cache  config.status  COPYING  Makefile     README
bashlib.in  config.log	  configure	 INSTALL  Makefile.in
. . .
. . .
Documents:
empty_directory:
FallOfSudo:
fallofsudo.py  LICENSE	main.png  README.md
My_Files:
afile.txt			somefile.txt	 yetanotherfile.txtx
anotherfile.txt			somepicture.jpg  yetanotheryetanotherfile.txt
haveicreatedenoughfilesyet.txt	somescript.sh
ubuntu2404% 
```

好的，我想你已经明白这里发生了什么。让我们继续看一下文件权限。

##### 通过文件权限扩展

在*第二十章，Shell 脚本安全性*中，我解释了文件和目录权限是如何工作的。让我们快速回顾一下。

权限设置分为*用户*（即文件或目录的所有者）、*组*和*其他*。每一项都可以设置任何组合的读取、写入和执行权限。如果你需要查找具有特定权限设置的文件或目录，你可以使用 `find` 工具。在 `zsh` 中，你可以使用**通配符限定符**代替 `find`。这里是你将使用的限定符列表：

对于用户：

+   `r:` 用户具有读取权限。

+   `w`: 这表示用户具有写权限的文件。

+   `x`: 用户具有可执行权限。

+   `U`: 这会查找属于当前用户的文件或目录。

+   `s`: 这会查找具有 SUID 权限设置的文件。

对于组：

+   `A`: 该组具有读取权限。

+   `I`: 该组具有写权限。

+   `E`: 该组具有可执行权限。

+   `G`: 这会查找属于当前用户组的文件或目录。

+   `S`: 这会查找具有 SGID 权限设置的文件或目录。

对于其他人：

+   `R`: 其他人具有读取权限。

+   `W`: 其他人具有写权限。

+   `X`: 其他人具有可执行权限。

+   `t`: 这会查找具有粘滞位的目录。

那么，所有这些是如何工作的呢？假设您在`/bin/`目录中，并且想查看所有具有 SUID 权限的文件。只需像这样做：

```
ubuntu2404% cd /bin      
ubuntu2404% ls -ld *(s)  
-rwsr-xr-x 1 root root  72792 Apr  9 07:01 chfn
-rwsr-xr-x 1 root root  44760 Apr  9 07:01 chsh
-rwsr-xr-x 1 root root  39296 Apr  8 15:57 fusermount3
-rwsr-xr-x 1 root root  76248 Apr  9 07:01 gpasswd
-rwsr-xr-x 1 root root  51584 Apr  9 14:02 mount
-rwsr-xr-x 1 root root  40664 Apr  9 07:01 newgrp
-rwsr-xr-x 1 root root  64152 Apr  9 07:01 passwd
-rwsr-xr-x 1 root root  55680 Apr  9 14:02 su
-rwsr-xr-x 1 root root 277936 Apr  8 14:50 sudo
-rwsr-xr-x 1 root root  39296 Apr  9 14:02 umount
ubuntu2404% 
```

如果您在自己的家目录中并且想查看整个文件系统中的所有 SUID 文件，只需使搜索递归，如下所示：

```
ubuntu2404% ls -ld /**/*(s)
-rwsr-xr-x 1 root root        85064 Feb  6  2024 /snap/core20/2264/usr/bin/chfn
-rwsr-xr-x 1 root root        53040 Feb  6  2024 /snap/core20/2264/usr/bin/chsh
. . .
. . .
-rwsr-xr-x 1 root root       154824 Jul 26 02:32 /usr/lib/snapd/snap-confine
ubuntu2404% 
```

与`find`不同，`ls`并非本质上是递归的。因此，如果您想在当前目录的所有下级子目录中进行搜索，只需这样做：

```
ubuntu2404% pwd
/home/donnie
ubuntu2404% ls -l **/*(s)
zsh: no matches found: **/*(s)
ubuntu2404% 
```

这与完整的文件系统搜索之间的唯一区别是，我使用了`**/*(s)`而不是`/**/*(s)`。 （当然，在我的家目录中没有 SUID 文件，这是可以预期的。）

如果您需要实际处理这个`ls`输出，您可以创建一个`zsh`脚本，像这样创建一个`find_suid.sh`脚本：

```
#!/bin/zsh
ls -ld /**/*(s) > suid_files.txt 
```

这只是一个简单的小脚本，它创建一个包含您系统中所有 SUID 文件的文本文件。但嘿，谁说我们不能把它弄得更花哨一些呢？让我们看看如何在`find_suid2.sh`脚本中实现：

```
#!/bin/zsh
suid_results_file=$1
if [[ -f "$suid_results_file" ]]; then
        echo "This file already exists."
        exit
fi
ls -ld /**/*(s) > "$suid_results_file" 
```

使用此脚本时，您需要指定要保存结果的文件名。如果该文件名已经存在，脚本会给出警告信息并退出。

您可以以相同的方式使用其他任何通配符限定符。例如，假设您需要搜索整个文件系统中属于您的文件。让我们看看是如何工作的：

```
ubuntu2404% ls -ld /**/*(U) | less 
```

这会生成一个非常长的列表，因此我必须将其管道输出到`less`以防止输出内容滚出屏幕顶部。以下是您将看到的一部分内容：

```
crw--w----  1 donnie tty    136, 0 Sep  3 20:04 /dev/pts/0
crw-------  1 donnie tty      4, 1 Sep  3 19:24 /dev/tty1
drwxr-x--- 13 donnie donnie   4096 Sep  3 20:02 /home/donnie
-rw-rw-r--  1 donnie donnie     44 Jul 24 20:53 /home/donnie/acl_demo.sh
drwxr-xr-x  2 donnie donnie   4096 Jun 29 21:10 /home/donnie/bashlib-0.4
. . .
. . .
-rw-r--r--  1 donnie donnie      0 Sep  3 19:24 /sys/fs/cgroup/user.slice/user-1000.slice/user@1000.service/memory.reclaim
-rwxrwxrwx  1 donnie donnie   2155 Jul  5 19:31 /usr/bin/deshc 
```

正如您所见，典型的 Linux 系统中到处都有属于您的文件。

这是`zsh`相对于其他所有 shell（包括`bash`）的一个明显优势。`zsh`的扩展文件通配符功能可以替代`find`的大部分功能，但语法更简单。（当然，您也可以使用这个技巧与除了`ls`之外的其他实用程序一起使用，例如`cp`，`mv`，`rm`，`chmod`或`chown`。）

你还可以使用`zsh`文件通配符来替代`grep`、`awk`和`tr`等工具。老实说，用于替代这些工具的通配符命令语法复杂到让人头疼，因此你最好还是使用这些工具。不过，使用`zsh`文件通配符会使你的脚本变得稍微轻便和更快。而且，在那些可能没有安装这些工具的罕见情况下，你还可以使用文件通配符。

无论如何，如果你想了解`zsh`的额外文件通配符功能，这里有一个链接：

Zsh 原生脚本手册：

[`github.zshell.dev/docs/zsh/Zsh-Native-Scripting-Handbook.html`](https://github.zshell.dev/docs/zsh/Zsh-Native-Scripting-Handbook.html)

你还可以通过阅读`zshexpn`的手册页来进一步了解文件通配符和变量扩展的相关内容。

好的，我们简要谈谈数组。

## 理解`zsh`数组

这会很快，因为我只想指出`zsh`数组的三点。第一点是，`zsh`数组的索引编号从 1 开始，而不像`bash`数组那样从 0 开始。下面是`bash`的示例：

```
donnie@fedora:~$ mybasharray=(vicky cleopatra horatio)
donnie@fedora:~$ echo ${mybasharray[0]}
vicky
donnie@fedora:~$ echo ${mybasharray[1]}
cleopatra
donnie@fedora:~$ 
```

你看到使用索引号 0 时，返回的是列表中的第一个元素，而索引号 1 则返回第二个元素。现在，在`zsh`中是这样的：

```
ubuntu2404% myzsharray=(vicky cleopatra horatio)
ubuntu2404% echo ${myzsharray[0]}
ubuntu2404% echo ${myzsharray[1]}
vicky
ubuntu2404% echo ${myzsharray[2]}
cleopatra
ubuntu2404% 
```

这次，索引号为 0 什么也不会返回，因为索引 0 不存在。

第二个大的区别在于如何查看整个数组的内容。使用`bash`时，你需要像这样做：

```
donnie@fedora:~$ echo ${mybasharray[@]}
vicky cleopatra horatio
donnie@fedora:~$ 
```

如果你愿意，你仍然可以在`zsh`中这样做，但你不必这样做。你可以像查看普通变量的值一样查看数组的内容，像这样：

```
ubuntu2404% echo $myzsharray  
vicky cleopatra horatio
ubuntu2404% 
```

最后，我想向你展示如何消除数组中的重复条目。为了演示，让我们创建一个包含水果列表的数组，像这样：

```
ubuntu2404% fruits=(orange apple orange banana kiwi banana apple orange)
ubuntu2404% echo $fruits
orange apple orange banana kiwi banana apple orange
ubuntu2404% 
```

如你所见，列表中有几个重复项。要去除这些重复项，只需这样做：

```
ubuntu2404% echo ${(u)fruits}
orange apple banana kiwi
ubuntu2404% 
```

将`(u)`替换为`(U)`会将整个列表以大写字母打印出来。

```
ubuntu2404% echo ${(U)fruits}
ORANGE APPLE ORANGE BANANA KIWI BANANA APPLE ORANGE
ubuntu2404% 
```

将`u`和`U`结合使用，你就能去除重复项，并将列表以大写字母打印出来。

```
ubuntu2404% echo ${(uU)fruits}
ORANGE APPLE BANANA KIWI
ubuntu2404% 
```

为了实际展示，让我们看看`fruit_array.sh`脚本：

```
#!/bin/zsh
fruits=(apple orange banana apple orange kiwi)
echo "Let's look at the entire list of fruits."
for fruit in $fruits; do
        echo $fruit
done
echo "*****************"
echo "*****************"
echo "Now, let's eliminate the duplicate fruits."
printf "%s\n" ${(u)fruits} 
```

在第一个代码段中，我使用`for`循环打印出`fruits`数组中的每个水果。在第二个代码段中，我想打印出每个独立的水果，但不包括重复项。使用`echo`会将所有水果打印在同一行，就像我在上面的命令行示例中展示的那样。因此，我没有使用`echo`，而是用了`printf`。这里，`%s`参数告诉`printf`打印指定的文本字符串，在本例中就是数组中的每个成员。`\n`选项在每个水果后插入换行符。运行脚本的效果如下：

```
ubuntu2404% ./fruit_array.sh
Let's look at the entire list of fruits.
apple
orange
banana
apple
orange
kiwi
*****************
*****************
Now, let's eliminate the duplicate fruits.
apple
orange
banana
kiwi
ubuntu2404% 
```

其实你可以在数组上做更多的事情，比我在这里所能展示的要多。要了解它们，看看 `zshexpn` 手册页。

数组部分到此为止。那么现在，让我们来谈谈数学。

## 增强的数学功能

在 *第十一章—执行数学运算* 中，我解释了 `bash` 只有有限的内建数学功能，并且只能处理整数。对于更复杂的运算，你需要在 `bash` 脚本中使用外部程序，例如 `bc`。另一方面，`zsh` 内建了更强大的数学功能。

对于我们的第一个例子，让我们尝试在 `zsh` 命令行中将 5 除以 2：

```
ubuntu2404% echo $((5/2))
2
ubuntu2404% 
```

好吧，那不是我想要的。我应该看到最终答案是 2.5，而不是 2。发生了什么？答案是，为了执行浮动点数学，你必须明确地将其中一个数字表示为浮动点数。所以，我们来修正这个问题。

```
ubuntu2404% echo $((5.0/2))
2.5
ubuntu2404% 
```

好得多。通过将第一个数字表示为 5.0 而不是 5，浮动点数学得以正常工作。不幸的是，`zsh` 的数学功能并不完美。我的意思是，它在这个除法例子中工作得很好，但看看当我尝试其他任何操作时会发生什么：

```
ubuntu2404% echo $((2.1+2.1))
4.2000000000000002
ubuntu2404% echo $((2.1*2))
4.2000000000000002
ubuntu2404% echo $((2.1-2))
0.10000000000000009
ubuntu2404% echo $((2.1%2))
0.10000000000000009
ubuntu2404% 
```

出于某种我不理解的原因，`zsh` 在小数点后添加了额外的尾随数字。如果它们全是零倒还无所谓，但尾随的 2 和 9 让我们的计算结果出现了问题。幸运的是，我们可以通过简单地将结果赋给一个变量来轻松修正这个问题，就像这样：

```
ubuntu2404% ((sum=2.1+2.1))
ubuntu2404% echo $sum
4.2000000000
ubuntu2404% 
```

尾随的零仍然存在，但至少多余的 2 和 9 数字不见了。

即使经过了一些相当广泛的研究，我也从未找到可以用来指定小数点后显示数字位数的命令或选项开关。

分组数学运算，以指定它们的优先级，像这样写：

```
ubuntu2404% ((product=(2.1+2.1)*8))
ubuntu2404% echo $product        
33.6000000000
ubuntu2404% 
```

如果简单的数学运算不足以满足你的需求，你可以加载 `mathfunc` 模块，我将在接下来的章节中展示给你。

# 使用 zsh 模块

很多 `zsh` 功能包含在可加载的 `zsh` 模块中。我无法在本章中涵盖所有的 `zsh` 模块，因此我们只看一些更有用的模块。

首先，让我们来看一下默认情况下哪些模块会加载到 `zsh` 会话中：

```
ubuntu2404% zmodload
zsh/complete
zsh/computil
zsh/main
zsh/parameter
zsh/stat
zsh/terminfo
zsh/zle
zsh/zutil
ubuntu2404% 
```

还有很多更多可选的模块，你可以通过使用 `zmodload` 命令自行加载，正如我们接下来要看到的。

要查看所有可用的 `zsh` 模块，请参阅 `zshmodules` 手册页。

现在，让我们通过查看 `mathfunc` 模块来更具体一些。

## 使用 mathfunc 模块

如果你想创建一些非常复杂的数学脚本，而不需要使用像 `bc` 这样的外部工具，只需加载 `mathfunc` 模块，如下所示：

```
ubuntu2404% zmodload zsh/mathfunc
ubuntu2404% 
```

这个模块提供了三角函数、对数、指数运算以及其他一些数学函数。你可以这样查看模块提供的所有函数：

```
ubuntu2404% zmodload -lF zsh/mathfunc
+f:abs
+f:acos
+f:acosh
+f:asin
+f:asinh
. . .
. . .
+f:y0
+f:y1
+f:yn
ubuntu2404% 
```

所以现在，假设你想查看 9 的平方根。可以这样做：

```
ubuntu2404% ((squareroot=sqrt(9)))
ubuntu2404% echo $squareroot
3.0000000000
ubuntu2404% 
```

现在，让我们把这个放入 `squareroot.sh` 脚本中：

```
#!/bin/zsh
zmodload zsh/mathfunc
your_number=$1
((squareroot=sqrt(your_number)))
echo $squareroot 
```

请注意，我必须在此脚本的开头放置一个`zmodload zsh/mathfunc`命令。这是因为每次关闭终端或注销计算机时，您通过命令行加载的任何模块都会自动卸载。因此，您需要将此`zmodload`命令添加到脚本中。

在这里，我使用`your_number`变量来保存用户为`$1`参数指定的数字值。让我们看看它是否有效。

```
ubuntu2404% ./squareroot.sh 9
3.0000000000
ubuntu2404% ./squareroot.sh 25
5.0000000000
ubuntu2404% ./squareroot.sh 2
1.4142135624
ubuntu2404% 
```

哦，没错，它运行得很好，而且完全不需要修改任何外部程序，比如`bc`。

其他所有`mathfunc`函数也以类似方式工作。运用你的想象力，以及我为`bash`展示的脚本概念，来创建你自己的`zsh`数学脚本。

这就是你对`mathfunc`模块的简介。接下来，我们简要看看`datetime`模块。

## `datetime`模块

`datetime`模块的功能几乎与`date`命令相同。你可以使用其中任何一个做一些酷的事情，比如在文件名中创建带时间戳的文件。它默认没有加载，因此我们现在来加载它：

```
ubuntu2404% zmodload zsh/datetime
ubuntu2404% 
```

现在，让我们看看它提供了什么：

```
ubuntu2404% zmodload -lF zsh/datetime
+b:strftime
+p:EPOCHSECONDS
+p:EPOCHREALTIME
+p:epochtime
ubuntu2404% 
```

下面是你正在查看的内容的详细说明：

+   `strftime`：这是此模块中唯一的函数。你可以像使用`date`命令一样使用它。

+   `EPOCHSECONDS`：这个环境变量是一个整数值，表示自 Unix 纪元开始以来经过的秒数。这个变量在`bash`中也可用。

+   `EPOCHREALTIME`：这个环境变量是一个浮动值，表示自 Unix 纪元开始以来经过的秒数。根据系统的不同，这个值的精确度可能达到纳秒或微秒级别。这个变量在`bash`中也可用。

+   `epochtime`：这个变量包含两个组件。它像`EPOCHREALTIME`变量，只不过小数点被空格代替。这个变量在`bash`中不可用。

Unix 纪元从 1970 年 1 月 1 日开始。早期的 Unix 开发者决定，设置 Unix 计算机时间的最简单方法就是以自该日期以来经过的秒数来表示时间。

你可以通过以下方式查看当前三个变量的值：

```
ubuntu2404% echo $EPOCHSECONDS
1725470539
ubuntu2404% echo $EPOCHREALTIME
1725470545.8938724995
ubuntu2404% echo $epochtime
1725470552 124436538
ubuntu2404% 
```

现在，让我们创建一个使用`strftime`函数的`zsh_time.sh`脚本：

```
#!/bin/zsh
zmodload zsh/datetime
timestamp=$(strftime '%A-%F_%T')
echo "I want to create a file at $timestamp." > "timefile_$timestamp.txt" 
```

`strftime`函数使用与`date`命令相同的日期和时间格式选项。在这种情况下，我们有：

+   `%A`：这是星期几。

+   `%F`：这是以 YYYY-MM-DD 格式表示的日期。

+   `%T`：这是以 HH:MM:SS 格式表示的时间。

使用`date`和`strftime`的唯一区别是，使用`date`时，必须在格式选项字符串前加上`+`，像这样：

```
date +'%A-%F-%T' 
```

`strftime`函数有自己的手册页，你可以通过运行`man strftime`来查看。

在我运行脚本之后，我应该会得到我的文件：

```
ubuntu2404% ./zsh_time.sh                             
ubuntu2404% ls -l timefile_*                               
-rw-rw-r-- 1 donnie donnie 58 Sep  4 18:16 timefile_Wednesday-2024-09-04_18:16:37.txt
ubuntu2404% cat timefile_Wednesday-2024-09-04_18:16:37.txt
I want to create a file at Wednesday-2024-09-04_18:16:37.
ubuntu2404% 
```

我已经介绍了我认为最有用的模块。如果你想阅读其他`zsh`模块的内容，可以查看`zshmodules`手册页。

我认为这差不多可以作为我们对`zsh`脚本的介绍了。让我们总结一下，继续前进。

# 总结

在这一章中，我向你介绍了在`zsh`中编写脚本的概念，而不是在`bash`中编写。我从展示如何安装`zsh`开始，如果它还没有安装的话。我展示了`bash`和`zsh`之间的差异，这些差异可能会影响你编写`zsh`脚本的方式。这些差异包括文件通配符、变量扩展、数组索引和数学运算的不同。在过程中，我向你展示了这些`zsh`特性如何帮助你编写比`bash`脚本更简单、更有功能的脚本。

`zsh`脚本的主要问题，除了`bash`更加普及之外，就是缺乏文档。关于设置`zsh`用户环境的教程和文章有很多，但关于`zsh`脚本的很少。希望你能利用我在这里提供的知识编写一些酷炫的`zsh`脚本。

在下一章，也是最后一章中，我将向你介绍如何在 Linux 上使用 PowerShell。（是的，你没看错。）我们在那里见。

# 问题

1.  你会使用哪个命令来查看已加载到你的`zsh`会话中的所有`zsh`模块？

    1.  `zmodls`

    1.  `zmod -l`

    1.  `zmodload -l`

    1.  `zmodload`

1.  你想加载数学函数模块。你会使用以下哪个命令？

    1.  `zmod mathfunc`

    1.  `zmod zsh/mathfunc`

    1.  `zmodload -l mathfunc`

    1.  `zmodload -l zsh/mathfunc`

    1.  `zmodload zsh/mathfunc`

1.  `bash`和`zsh`处理数组的主要区别是什么？

    1.  没有区别。

    1.  `bash`数组索引从 1 开始，`zsh`数组索引从 0 开始。

    1.  `bash`数组索引从 0 开始，`zsh`数组索引从 1 开始。

    1.  `bash`可以处理数组，但`zsh`不能。

1.  你会使用哪个`zsh`命令来查看目录中所有的`.png`文件和`.PNG`文件？

    1.  `ls *.png`

    1.  `ls (#i).png`

    1.  `ls (#l).PNG`

    1.  `ls *.PNG`

1.  你只想查看当前工作目录中的空目录。你会使用以下哪个`zsh`命令？

    1.  `ls -l *(/^F)`

    1.  `ls -l *(^F)`

    1.  `ls -l *(F)`

    1.  `ls -ld`

# 进一步阅读

+   Z Shell 手册：[`zsh-manual.netlify.app/`](https://zsh-manual.netlify.app/)

+   什么是 ZSH，为什么你应该使用它而不是 Bash？：[`www.howtogeek.com/362409/what-is-zsh-and-why-should-you-use-it-instead-of-bash/`](https://www.howtogeek.com/362409/what-is-zsh-and-why-should-you-use-it-instead-of-bash/)

+   ZSH 入门：探索 Linux 的优雅 Shell：[`www.fosslinux.com/133167/zsh-for-beginners-exploring-linuxs-elegant-shell.htm`](https://www.fosslinux.com/133167/zsh-for-beginners-exploring-linuxs-elegant-shell.htm)

+   如何为 ZSH 编写脚本：[`www.linuxtoday.com/blog/writing-scripts-for-zsh/`](https://www.linuxtoday.com/blog/writing-scripts-for-zsh/)

# 答案

1.  d

1.  e

1.  c

1.  b

1.  a

# 加入我们的 Discord 社区！

与其他用户、Linux 专家以及作者本人一起阅读这本书。

提出问题，为其他读者提供解决方案，通过“问我任何问题”（Ask Me Anything）环节与作者交流，等等。扫描二维码或访问链接加入社区。

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code10596186092701843.png)
