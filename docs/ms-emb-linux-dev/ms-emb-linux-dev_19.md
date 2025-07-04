

# 第十五章：打包 Python

**Python**是最流行的**机器学习**（**ML**）编程语言。再加上机器学习在我们日常生活中的普及，Python 在边缘设备上运行的需求不断增加也就不足为奇了。即使在转译器和 WebAssembly 的时代，将 Python 应用程序打包部署依然是一个未解决的问题。在本章中，你将了解有哪些选择可以将 Python 模块打包在一起，并在何时使用某种方法而非另一种方法。

我们从回顾今天 Python 打包解决方案的起源开始，从内建的标准`distutils`到其继任者`setuptools`。接下来，我们将审视`pip`包管理器，然后转到用于 Python 虚拟环境的`venv`，最后是`conda`，这个通用的跨平台解决方案。

由于 Python 是解释型语言，你不能像使用 Go 等语言那样将程序编译为独立的可执行文件。这使得 Python 应用程序的部署变得复杂。运行 Python 应用程序需要安装 Python 解释器和若干运行时依赖项。这些要求需要与代码兼容才能使应用程序正常运行。这就需要软件组件的精确版本管理。解决这些部署问题正是 Python 打包的核心内容。

在本章中，我们将覆盖以下主题：

+   回顾 Python 打包的起源

+   使用`pip`安装 Python 包

+   使用`venv`管理 Python 虚拟环境

+   使用`conda`安装预编译的二进制文件

# 技术要求

为了跟随示例进行操作，请确保在你的 Linux 主机系统上安装了以下软件：

+   Python：Python 3 解释器和标准库

+   `pip`：Python 3 的包管理器

+   `venv`：用于创建和管理轻量级虚拟环境的 Python 模块

+   Miniconda：`conda`包和虚拟环境管理器的最小安装程序

我推荐使用 Ubuntu 24.04 LTS 或更高版本来进行本章的操作。尽管 Ubuntu 24.04 LTS 可以在 Raspberry Pi 4 上运行，但我仍然更喜欢在 x86-64 桌面 PC 或笔记本电脑上进行开发。Ubuntu 还预装了 Python 3 和`pip`，因为 Python 在系统中被广泛使用。不要卸载`python3`，否则你将无法使用 Ubuntu。要在 Ubuntu 上安装`venv`，请输入以下命令：

```
$ sudo apt install python3-venv 
```

**重要说明**

在到达`conda`部分之前，不要安装 Miniconda，因为它会干扰依赖于系统 Python 安装的早期`pip`练习。

# 回顾 Python 打包的起源

Python 打包的生态系统是一个庞大的失败尝试和被遗弃工具的墓地。Python 社区内关于依赖管理的最佳实践经常变化，一年的推荐解决方案可能在下一年就变成了一个行不通的方案。在研究这个话题时，记得查看信息发布的时间，并且不要相信任何可能已经过时的建议。

大多数 Python 库都使用`setuptools`进行分发，包括在**Python 软件包索引**（**PyPI**）上找到的所有包。此分发方法依赖于`setup.py`项目规范文件，**Python 包安装器**（**pip**）使用该文件来安装包。`pip`还可以在项目安装后生成或*冻结*一个精确的依赖关系列表。这个可选的`requirements.txt`文件由`pip`与`setup.py`一起使用，以确保项目安装是可重复的。

## distutils

**distutils**是 Python 的原始打包系统。从 Python 2.0 开始，直到 Python 3.12 版本，它一直被包含在 Python 标准库中。`distutils`提供了一个同名的 Python 包，可以被你的`setup.py`脚本导入。现在由于`distutils`已被弃用，直接使用该包不再受支持。`setuptools`已成为其首选替代品。

尽管`distutils`可能仍适用于简单项目，但社区已经向前发展。如今，`distutils`仅因遗留原因而存在。许多 Python 库最初是在`distutils`是唯一选择的时候发布的。将它们迁移到`setuptools`现在将需要大量工作，并可能破坏现有用户的使用。

## setuptools

**setuptools**通过增加对复杂结构的支持，扩展了`distutils`，使得基于 Flask 和 FastAPI 等 Web 框架构建的大型应用程序更容易分发。它已成为 Python 社区事实上的打包系统。像`distutils`一样，`setuptools`也提供了一个同名的 Python 包，你可以在`setup.py`脚本中导入它。`distribute`是`setuptools`的一个雄心勃勃的分支，最终合并回`setuptools 0.7`，巩固了`setuptools`作为 Python 打包的终极选择的地位。

`setuptools`引入了一个命令行工具，称为`easy_install`（现在已弃用），以及一个名为`pkg_resources`的 Python 包，用于运行时的包发现和资源文件访问。`setuptools`还可以生成充当其他可扩展包（例如框架和应用程序）插件的包。你可以通过在`setup.py`脚本中为其他主包注册入口点来实现这一点，以便让其他包导入。

在 Python 的上下文中，术语*distribution*有着不同的含义。一个 distribution 是一个包含包、模块和其他资源文件的版本化存档，用于分发一个发布版本。*release*是某一时刻对 Python 项目的一个版本化快照。更糟糕的是，*package*和*distribution*这两个术语是多重含义的，Pythonista 们常常交替使用这两个词。为了方便起见，我们可以把 distribution 看作你下载的内容，而 package 是被安装并导入的模块。

切割一个发行版可能会产生多个分发文件，例如源分发和一个或多个构建分发文件。不同平台可能会有不同的构建分发文件，比如包含 Windows 安装程序的文件。*构建分发* 这个术语意味着在安装之前不需要构建步骤，但不一定意味着预编译。某些构建分发格式（如 **Wheel**（`.whl`））会排除已编译的 Python 文件。例如，包含已编译扩展的构建分发称为 *二进制分发*。

**扩展模块** 是用 C 或 C++ 编写的 Python 模块。每个扩展模块都会编译成一个单独的动态加载库，例如 Linux 上的共享对象（`.so`）和 Windows 上的动态链接库（`.pyd`）。与此相比，纯模块必须完全用 Python 编写。`setuptools` 引入的 Egg（`.egg`）构建分发格式支持纯模块和扩展模块。由于 Python 源代码（`.py`）文件在 Python 解释器导入模块时会编译成字节码（`.pyc`）文件，你可以理解像 Wheel 这样的构建分发格式可能会排除预编译的 Python 文件。

## setup.py

假设你正在开发一个小型的 Python 程序，可能是查询远程 REST API 并将响应数据保存到本地 SQL 数据库。如何将你的程序与其依赖项打包，以便进行部署呢？你可以从定义一个 `setup.py` 脚本开始，`setuptools` 将使用该脚本来安装你的程序。使用 `setuptools` 进行部署是实现更复杂的自动化部署方案的第一步。

即使你的程序足够小，可以舒适地放在一个模块中，它也很可能不会一直保持这种状态。假设你的程序由一个名为 `follower.py` 的单个文件组成，像这样：

```
$ tree follower
follower
└── follower.py 
```

然后，你可以通过将 `follower.py` 拆分成三个独立的模块，并将它们放置在一个名为 `follower` 的嵌套目录中，将这个模块转换为一个包：

```
$ tree follower/
follower/
└── follower
 ├── fetch.py
 ├── __main__.py
 └── store.py 
```

`__main__.py` 模块是你的程序开始的地方，因此它包含主要的顶层、面向用户的功能。`fetch.py` 模块包含向远程 REST API 发送 HTTP 请求的函数，`store.py` 模块包含将响应数据保存到本地 SQL 数据库的函数。要将这个包作为脚本运行，你需要向 Python 解释器传递 `-m` 选项，如下所示：

```
$ PYTHONPATH=follower python -m follower 
```

`PYTHONPATH` 环境变量指向目标项目的包目录所在的目录。`-m` 选项后的 `follower` 参数告诉 Python 运行 `follower` 包下的 `__main__.py` 模块。像这样将包目录嵌套在项目目录中，为你的程序发展成由多个具有独立命名空间的包组成的大型应用程序铺平了道路。

在项目的各个部分都已正确放置后，我们现在准备创建一个最小化的`setup.py`脚本，`setuptools`可以用它来打包和部署项目：

```
from setuptools import setup
    setup(
    name='follower',
    version='0.1',
    packages=['follower'],
    include_package_data=True,
    install_requires=['requests', 'sqlalchemy']
) 
```

`install_requires`参数是一个外部依赖列表，这些依赖需要在项目运行时自动安装。请注意，在我的示例中，我并没有指定这些依赖需要什么版本，也没有指定从哪里获取它们。我只要求使用看起来和行为类似于`requests`和`sqlalchemy`的库。像这样将策略与实现分开，使你可以轻松地将官方 PyPI 版本的依赖替换为你自己的版本，以防你需要修复一个 bug 或添加一个功能。

**信息提示**

在依赖声明中添加可选的版本说明符是可以的，但在`setup.py`中硬编码分发网址作为`dependency_links`在原则上是错误的。

`packages`参数告诉`setuptools`要分发项目发布版本中的哪些内部包。由于每个包都在父项目目录的子目录中定义，因此在这种情况下唯一被打包的包是`follower`。我将数据文件与 Python 代码一起包含在此分发中。为了做到这一点，你需要将`include_package_data`参数设置为`True`，这样`setuptools`就会查找`MANIFEST.in`文件并安装其中列出的所有文件。以下是`MANIFEST.in`文件的内容：

```
include data/events.db 
```

如果数据目录包含我们想要包含的嵌套目录，我们可以使用`recursive-include`将它们及其内容一起包含：

```
recursive-include data * 
```

这是最终的目录布局：

```
$ tree follower
follower
├── data
│   └── events.db
├── follower
│   ├── fetch.py
│   ├── __init__.py
│   └── store.py
├── MANIFEST.in
└── setup.py 
```

`setuptools`擅长构建和分发依赖于其他包的 Python 包。它之所以能够做到这一点，是因为它具备了入口点和依赖声明等特性，而这些特性在`distutils`中根本没有。`setuptools`与`pip`配合得很好，并且`setuptools`的新版本会定期发布。Wheel 构建分发格式是为了替代`setuptools`引入的 Egg 格式而创建的。这一努力已经取得了很大成功，得益于一个流行的`setuptools`扩展用于构建 wheels，并且`pip`对安装 wheels 的良好支持。

# 使用 pip 安装 Python 包

现在你已经知道如何在`setup.py`脚本中定义项目的依赖关系。那么，如何安装这些依赖呢？如何在找到更好的依赖时升级或替换它？当你不再需要某个依赖时，如何判断何时可以安全地删除它？

管理项目依赖是件棘手的事情。幸运的是，Python 自带了一个叫做**pip**的工具，可以在项目初期阶段提供帮助。这个名字代表了**pip 安装 Python**，这是一个递归首字母缩略词。`pip`是 Python 的官方包管理器。

`pip`的初始 1.0 版本于 2011 年 4 月 4 日发布，正好与 Node.js 和`npm`的兴起同时发生。在成为`pip`之前，该工具名为`pyinstall`。`pyinstall`于 2008 年创建，是作为`easy_install`的替代工具，后者当时与`setuptools`一起捆绑。`easy_install`现已弃用，`setuptools`推荐使用`pip`代替。

由于`pip`与 Python 安装程序一起提供，而且你可以在系统上安装多个版本的 Python（例如，2.7 和 3.13），了解你正在运行的`pip`版本是很有帮助的：

```
$ pip --version 
```

如果系统中找不到`pip`可执行文件，那可能意味着你使用的是 Ubuntu 20.04 LTS 或更高版本，并且没有安装 Python 2.7。这没问题。我们将仅在本节其余部分中将`pip3`替代为`pip`，`python3`替代为`python`：

```
$ pip3 --version 
```

如果有`python3`但没有`pip3`可执行文件，则可以按如下方式在基于 Debian 的发行版（如 Ubuntu）中安装它：

```
$ sudo apt install python3-pip 
```

`pip`将包安装到名为`site-packages`的目录中。要找到你的`site-packages`目录的位置，请运行以下命令：

```
$ python3 -m site | grep ^USER_SITE 
```

**重要提示**

现在 Python 2 已经被弃用，`pip3`和`python3`命令在像 Ubuntu 这样的流行 Linux 发行版上可用。如果你的 Linux 发行版没有`pip3`和`python3`命令，则改用`pip`和`python`命令。

要获取系统中已安装的包的列表，使用以下命令：

```
$ pip3 list 
```

列表显示`pip`只是另一个 Python 包，因此你可以使用`pip`来升级它自己，但我建议你不要这样做，至少长期来说不要这么做。我将在下一节中介绍虚拟环境时解释原因。

要获取安装在`site-packages`目录中的包的列表，使用以下命令：

```
$ pip3 list --user 
```

这个列表应该是空的，或者比系统包列表要短得多。

返回到上一节中的示例项目。`cd`进入包含`setup.py`的父级`follower`目录。然后运行以下命令：

```
$ pip3 install --ignore-installed --user --break-system-packages . 
```

`pip`将使用`setup.py`来获取并安装由`install_requires`声明的包到你的`site-packages`目录中。`--user`选项指示`pip`将包安装到`site-packages`目录而不是全局目录。`--ignore-installed`选项强制`pip`重新安装系统中已经存在的任何必需包到`site-packages`，以确保没有依赖项丢失。`--break-system-packages`选项在基于 Debian 的 Linux 发行版（如 Ubuntu）中是必需的，因为这些发行版不鼓励用户在系统范围内安装非 Debian 打包的包。

现在再次列出`site-packages`目录中的所有包：

```
$ pip3 list --user
Package            Version
------------------ ---------
certifi            2025.1.31
charset-normalizer 3.4.1
follower           0.1
greenlet           3.1.1
idna               3.10
requests           2.32.3
SQLAlchemy         2.0.38
typing_extensions  4.12.2
urllib3            2.3.0 
```

这次你应该看到`requests`和`SQLAlchemy`都在包列表中。

要查看刚刚安装的`SQLAlchemy`包的详细信息，发出以下命令：

```
$ pip3 show sqlalchemy 
```

显示的详细信息包含`Requires`和`Required-by`字段。这两个字段都是相关包的列表。你可以使用这些字段中的值和连续调用`pip show`来追踪你项目的依赖树。但使用`pip install`一个名为`pipdeptree`的命令行工具并用它来代替可能更容易。

当`Required-by`字段为空时，这是一个很好的指标，说明现在可以安全地从系统中卸载该包。如果没有其他包依赖于已删除包的`Requires`字段中的包，那么也可以安全地卸载这些包。这就是如何使用`pip`卸载`sqlalchemy`的方式：

```
$ pip3 uninstall sqlalchemy -y --break-system-packages 
```

末尾的`-y`会抑制确认提示。要一次卸载多个包，只需在`-y`前添加更多包名称。这里省略了`--user`选项，因为`pip`足够智能，能够在全局安装的包也存在时，优先从`site-packages`中卸载包。

**提示**

从你的`site-packages`目录中卸载`follower`包及其所有依赖项，以免污染你的 Python 安装或 Linux 发行版，避免安装非 Debian 打包的包。

有时你需要一个执行某个特定功能或使用某项技术的包，但你不知道它的名称。你可以使用`pip`在命令行中对 PyPI 进行关键字搜索，但这种方法通常会返回太多结果。通过在 PyPI 网站上搜索包（[`pypi.org/search/`](https://pypi.org/search/)）要容易得多，它允许你按各种分类器筛选结果。

## requirements.txt

`pip install`会安装包的最新发布版本，但通常你会想要安装与你的项目代码兼容的特定版本包。最终，你会想要升级你项目的依赖项。但在我展示如何做之前，我首先需要展示如何使用`pip freeze`来固定你的依赖项。

`requirements`文件允许你明确指定`pip`应为你的项目安装哪些包和版本。按照惯例，项目的**requirements 文件**始终命名为`requirements.txt`。requirements 文件的内容仅仅是一个列出项目依赖项的`pip install`参数列表。这些依赖项有精确的版本号，以确保当其他人尝试重新构建和部署你的项目时不会有意外。将`requirements.txt`文件添加到项目的代码库中是一个良好的实践，以确保构建的可复现性。

返回到我们的`follower`项目，现在我们已经安装了所有依赖项并验证代码按预期工作，我们可以冻结`pip`为我们安装的包的最新版本了。`pip`有一个`freeze`命令，输出已安装包及其版本。你将该命令的输出重定向到`requirements.txt`文件：

```
$ pip3 freeze --user > requirements.txt 
```

现在你有了一个`requirements.txt`文件，克隆你项目的人可以使用`-r`选项和要求文件的名称来安装所有依赖项：

```
$ pip3 install --user -r requirements.txt 
```

自动生成的要求文件格式默认使用精确版本匹配（`==`）。例如，像`requests==2.32.3`这样的行告诉`pip`，要安装的`requests`版本必须是精确的`2.32.3`。在要求文件中，你可以使用其他版本说明符，例如最小版本（`>=`）、版本排除（`!=`）和最大版本（`<=`）。最小版本（`>=`）匹配任何大于或等于右侧版本的版本。版本排除（`!=`）匹配任何除了右侧版本之外的版本。最大版本（`<=`）匹配任何小于或等于右侧版本的版本。

你可以在一行中结合多个版本说明符，用逗号分隔它们：

```
requests >=2.32.3,<3.0 
```

当`pip`安装要求文件中指定的包时，默认行为是从 PyPI 获取它们。你可以通过在`requirements.txt`文件的顶部添加如下行，将 PyPI 的 URL（[`pypi.org/simple/)`](https://pypi.org/simple/))替换为备用 Python 包索引的 URL：

```
--index-url http://pypi.mydomain.com/mirror 
```

建立并维护自己的私有 PyPI 镜像所需的努力不可小觑。当你只需要修复一个 bug 或为项目依赖项添加一个特性时，覆盖包源而非整个包索引更为合理。

我之前提到过，硬编码分发 URL 到`setup.py`中是错误的。你可以在要求文件中使用`-e`参数形式来覆盖单独的包源：

```
-e git+https://github.com/myteam/flask.git#egg=flask 
```

在这个示例中，我指示`pip`从我们团队的 GitHub 分支`pallets/flask.git`获取`flask`包源。`-e`参数形式还可以接受 Git 分支名称、提交哈希或标签名称：

```
-e git+https://github.com/myteam/flask.git@master
-e git+https://github.com/myteam/flask.git@5142930ef57e2f0ada00248bdaeb95406d18eb7c
-e git+https://github.com/myteam/flask.git@v1.0 
```

使用`pip`将项目的依赖项升级到 PyPI 上发布的最新版本非常简单：

```
pip3 install --user --upgrade -r requirements.txt 
```

在你验证了安装最新版本的依赖项不会破坏你的项目后，你可以将它们重新写入要求文件：

```
$ pip3 freeze --user > requirements.txt 
```

确保冻结操作没有覆盖要求文件中的任何覆盖或特殊版本处理。撤销任何错误并将更新后的`requirements.txt`文件提交到版本控制系统。

在某个时刻，升级你的项目依赖项可能会导致代码出现问题。新版本的包可能会引入回归或与项目不兼容的情况。要求文件格式提供了应对这些情况的语法。假设你在项目中使用的是`requests`的 2.32.3 版本，而版本 3.0 发布了。根据语义版本控制的实践，增加主版本号表示`requests`的版本 3.0 包含了对该库 API 的破坏性更改。

你可以这样表达新的版本要求：

```
requests ~= 2.32.3 
```

兼容版本说明符（`~=`）依赖于语义版本控制。兼容意味着大于或等于右侧的版本，并且小于下一个版本的主版本号（例如，`>= 1.1` 和 `== 1.*`）。你已经看到我用更明确的方式表示了对 `requests` 的相同版本要求，如下所示：

```
requests >=2.32.3,<3.0 
```

如果你一次只开发一个 Python 项目，这些 `pip` 依赖管理技巧是有效的。但很有可能你会使用同一台机器同时进行多个 Python 项目的开发，每个项目可能需要不同版本的 Python 解释器。仅使用 `pip` 处理多个项目的最大问题是，它将所有包安装到特定 Python 版本的相同用户 `site-packages` 目录中。这使得将一个项目的依赖与另一个项目隔离开来变得非常困难。

正如我们将在下一章看到的，`pip` 与 Docker 配合得很好，用于部署 Python 应用程序。你可以将 `pip` 添加到基于 Buildroot 或 Yocto 的 Linux 镜像中，但这仅能实现设备上的快速实验。像 `pip` 这样的 Python 运行时包安装程序并不适合 Buildroot 和 Yocto 环境，在这些环境中你希望在构建时定义嵌入式 Linux 镜像的所有内容。`pip` 在像 Docker 这样的容器化环境中运行良好，因为在这些环境中，构建时和运行时之间的界限往往是模糊的。

在 *第七章*中，你学习了如何在 `meta-python` 层中使用 Python 模块以及如何为自己的应用定义自定义层。你可以使用 `pip freeze` 生成的 `requirements.txt` 文件来指导从 `meta-python` 中选择依赖项，以便用于你的自定义层配方。Buildroot 和 Yocto 都是以系统范围的方式安装 Python 包，因此我们接下来要讨论的虚拟环境技术并不直接适用于嵌入式 Linux 构建。不过，它们有助于你为嵌入式 Python 应用程序组装完整的依赖项列表。

# 使用 venv 管理 Python 虚拟环境

**虚拟环境**是一个自包含的目录树，包含特定版本 Python 的 Python 解释器、用于管理项目依赖项的 `pip` 可执行文件以及本地的 `site-packages` 目录。在虚拟环境之间切换会让 shell 误认为当前唯一可用的 Python 和 `pip` 可执行文件是活动虚拟环境中存在的那些。最佳实践是为每个项目创建一个不同的虚拟环境。这种隔离方式解决了两个项目依赖于同一包的不同版本的问题。

虚拟环境对于 Python 来说并不新鲜。Python 安装的系统范围性质决定了它们的必要性。虚拟环境不仅可以让你安装同一包的不同版本，还为你提供了一种简单的方式来运行多个版本的 Python 解释器。管理 Python 虚拟环境的工具有很多。大约在 2019 年非常流行的工具`pipenv`如今已经逐渐衰落。流行的`conda`包管理器自 2014 年底起就支持 Python 虚拟环境。而 Python 3 内建的虚拟环境支持（`venv`）从 2012 年引入以来，逐渐成熟并被广泛采用。

**venv**自 Python 3.3 版本以来就已经随 Python 一起发布。由于它仅与 Python 3 安装捆绑，因此`venv`与需要 Python 2.7 的项目不兼容。由于 Python 2.7 的官方支持已于 2020 年 1 月 1 日结束，因此这个 Python 3 的限制问题现在已经不那么严重了。`venv`基于流行的`virtualenv`工具，而后者仍在维护并可以从 PyPI 获取。如果你有一个或多个仍然需要 Python 2.7 的项目，你可以使用`virtualenv`而不是`venv`来处理这些项目。

默认情况下，`venv`会安装系统中找到的最新版本的 Python。如果你的系统中有多个 Python 版本，你可以在创建每个虚拟环境时，通过运行`python3`或你想要的版本来选择特定的 Python 版本（*Python 教程*，[`docs.python.org/3/tutorial/venv.html`](https://docs.python.org/3/tutorial/venv.html)）。对于新项目来说，使用最新版本的 Python 通常没问题，但对于大多数遗留和企业软件来说却是不可接受的。我们将使用与你的 Ubuntu 系统一起提供的 Python 3 版本来创建和使用虚拟环境。

要创建虚拟环境，首先决定放置虚拟环境的目录，然后运行`venv`模块并指定目标目录路径：

1.  确保`venv`已安装在你的 Ubuntu 系统上：

    ```
    $ sudo apt install python3-venv 
    ```

1.  为你的项目创建一个新目录：

    ```
    $ mkdir myproject 
    ```

1.  切换到那个新目录：

    ```
    $ cd myproject 
    ```

1.  在名为`venv`的子目录中创建虚拟环境：

    ```
    $ python3 -m venv ./venv 
    ```

现在你已经创建了虚拟环境，下面是如何激活并验证它：

1.  如果你还没有进入你的项目目录，请切换到项目目录：

    ```
    $ cd myproject 
    ```

1.  检查你的系统上`pip3`可执行文件的安装位置：

    ```
    $ which pip3
    /usr/bin/pip3 
    ```

1.  激活项目的虚拟环境：

    ```
    $ source ./venv/bin/activate 
    ```

1.  检查你的项目中`pip3`可执行文件的安装位置：

    ```
    (venv) $ which pip3
    /home/frank/myproject/venv/bin/pip3 
    ```

1.  列出随虚拟环境一起安装的包：

    ```
    (venv) $ pip3 list
    Package Version
    ------- -------
    pip     24.0 
    ```

如果你在虚拟环境中运行`which pip`命令，你会看到`pip`现在指向一个可执行文件。此时，你可以在虚拟环境中运行`pip`或`python`时省略`3`。

接下来，让我们将一个名为`hypothesis`的基于属性的测试库安装到现有的虚拟环境中：

1.  如果你还没有进入你的项目目录，请切换到项目目录：

    ```
    $ cd myproject 
    ```

1.  如果项目的虚拟环境尚未激活，请重新激活它：

    ```
    $ source ./venv/bin/activate 
    ```

1.  安装 `hypothesis` 包：

    ```
    (venv) $ pip install hypothesis 
    ```

1.  列出当前已安装在虚拟环境中的包：

    ```
    (venv) $ pip list
    Package          Version
    ---------------- -------
    attrs            25.1.0
    hypothesis       6.125.2
    pip              24.0
    sortedcontainers 2.4.0 
    ```

注意，除了 `hypothesis` 之外，列表中还新增了两个包（`attrs` 和 `sortedcontainers`）。`hypothesis` 依赖这两个包。假设你有另一个 Python 项目依赖于 `sortedcontainers` 版本 1.5.10，而不是版本 2.4.0。这两个版本不兼容，因此会发生冲突。虚拟环境允许你为每个项目安装同一个包的不同版本。

你可能注意到，退出项目目录并不会停用其虚拟环境。别担心，停用虚拟环境就这么简单：

```
(venv) $ deactivate
$ 
```

这将把你带回全局系统环境，你必须再次输入 `python3` 和 `pip3`。你现在已经了解了开始使用 Python 虚拟环境所需的所有知识。创建和切换虚拟环境现在已经是 Python 开发中的常见做法。隔离的环境让你更容易跟踪和管理多个项目中的依赖关系。

# 使用 conda 安装预编译的二进制文件

**conda** 是由 **Anaconda** 发行版为 PyData 社区提供的包和虚拟环境管理系统。Anaconda 发行版包括 Python 以及一些难以构建的开源项目的二进制文件，如 PyTorch 和 TensorFlow。`conda` 可以在没有完整 Anaconda 发行版的情况下安装，Anaconda 非常庞大，或者可以选择更小的 **Miniconda** 发行版，尽管它仍然超过 256 MB。

尽管 `conda` 在 `pip` 之后不久就为 Python 所创建，但它已经发展成一个通用的包管理器，类似于 APT 或 Homebrew。现在，它可以用来为任何语言打包和分发软件。因为 `conda` 下载的是预编译的二进制文件，所以安装 Python 扩展模块非常轻松。`conda` 的另一个重要卖点是它是跨平台的，完全支持 Linux、macOS 和 Windows。

除了包管理，`conda` 还是一个完整的虚拟环境管理器。`conda` 虚拟环境具备我们从 Python `venv` 环境中期待的所有好处，甚至更多。像 `venv` 一样，`conda` 允许你使用 `pip` 从 PyPI 安装包到项目的本地 `site-packages` 目录。如果你愿意，你也可以使用 `conda` 自带的包管理功能，从不同的渠道安装包。渠道是由 Anaconda 和其他软件发行版提供的包源。

## 环境管理

与`venv`不同，`conda`的虚拟环境管理器可以轻松处理多个版本的 Python，包括 Python 2.7。你需要在 Ubuntu 系统上安装 Miniconda 才能进行接下来的练习。你希望使用 Miniconda 而不是 Anaconda 来管理虚拟环境，因为 Anaconda 环境自带许多预安装的包，其中很多你根本不需要。Miniconda 环境经过精简，可以让你轻松安装任何 Anaconda 的包，必要时使用。

在 Ubuntu 24.04 LTS 上安装和更新 Miniconda：

1.  下载 Miniconda：

    ```
    $ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh 
    ```

1.  安装 Miniconda：

    ```
    $ bash Miniconda3-latest-Linux-x86_64.sh 
    ```

1.  更新根环境中所有已安装的包：

    ```
    (base) $ conda update --all 
    ```

你刚安装的 Miniconda 自带了`conda`和一个根环境，里面有一个 Python 解释器和一些基本包。默认情况下，`conda`根环境的`python`和`pip`可执行文件会安装在你的主目录中。`conda`的根环境被称为`base`。你可以通过运行以下命令查看它的安装位置，以及其他任何可用的`conda`环境的位置：

```
(base) $ conda env list 
```

在创建自己的`conda`环境之前，验证一下这个根环境：

1.  安装 Miniconda 后，打开一个新的终端。

1.  检查根环境中`python`可执行文件的安装位置：

    ```
    (base) $ which python 
    ```

1.  检查 Python 的版本：

    ```
    (base) $ python --version 
    ```

1.  检查根环境中`pip`可执行文件的安装位置：

    ```
    (base) $ which pip 
    ```

1.  检查`pip`的版本：

    ```
    (base) $ pip --version 
    ```

1.  列出根环境中已安装的包：

    ```
    (base) $ conda list 
    ```

接下来，创建并使用你自己的名为`py311`的`conda`环境：

1.  创建一个名为`py311`的新虚拟环境：

    ```
    (base) $ conda create --name py311 python=3.11.9 
    ```

1.  激活你新的虚拟环境：

    ```
    (base) $ source activate py311 
    ```

1.  检查你的环境中`python`可执行文件的安装位置：

    ```
    (py311) $ which python 
    ```

1.  检查 Python 的版本是否为 3.11.9：

    ```
    (py311) $ python --version 
    ```

1.  列出你环境中已安装的包：

    ```
    (py311) $ conda list 
    ```

1.  退出你的环境：

    ```
    (py311) $ conda deactivate 
    ```

使用`conda`创建一个安装了 Python 2.7 的虚拟环境就像下面这样简单：

```
(base) $ conda create --name py27 python=2.7.18 
```

再次查看你的`conda`环境，看看`py311`和`py27`是否现在出现在列表中：

```
(base) $ conda env list 
```

最后，让我们删除`py27`环境，因为我们不会再使用它：

```
(base) $ conda remove --name py27 --all 
```

现在你知道如何使用`conda`来管理虚拟环境了，接下来让我们用它来管理这些环境中的包。

## 包管理

由于`conda`支持虚拟环境，我们可以像使用`venv`一样，使用`pip`以隔离的方式在不同项目之间管理 Python 依赖项。作为一个通用的包管理器，`conda`有自己的依赖管理功能。我们知道，`conda list`会列出在活动虚拟环境中安装的所有包。我还提到过`conda`使用的软件包源，称为通道（channels）：

1.  你可以通过输入以下命令获取`conda`配置的通道 URL 列表：

    ```
    (base) $ conda info 
    ```

1.  在继续之前，让我们重新激活你在上一个练习中创建的`py311`虚拟环境：

    ```
    (base) $ source activate py311
    (py311) $ 
    ```

1.  现在大多数 Python 开发都是在 Jupyter notebook 中进行的，所以我们先安装这些软件包：

    ```
    (py311) $ conda install jupyter notebook 
    ```

1.  在提示时输入*y*。这将安装`jupyter`和`notebook`软件包及其所有依赖项。当你输入`conda list`时，你会看到安装的软件包列表比之前长得多。现在，让我们安装一些在计算机视觉项目中需要的 Python 软件包：

    ```
    (py311) $ conda install opencv matplotlib 
    ```

1.  再次在提示时输入*y*。这次安装的依赖项数量较少。`opencv`和`matplotlib`都依赖于`numpy`，所以`conda`会自动安装该包，而不需要你指定。如果你想指定`opencv`的较旧版本，可以通过以下方式安装所需版本：

    ```
    (py311) $ conda install opencv=4.6.0 
    ```

1.  `conda`会尝试为该依赖项*解决*活动环境。由于当前虚拟环境中没有其他软件包依赖`opencv`，目标版本很容易解决。如果有依赖项，那你可能会遇到软件包冲突，重新安装会失败。解决后，`conda`会在降级`opencv`及其依赖项之前提示你。输入*y*将`opencv`降级到版本 4.6.0。

1.  假设你改变了主意，或者发布了一个新的版本的`opencv`，解决了你之前的担忧。下面是如何将`opencv`升级到 Anaconda 发行版提供的最新版本：

    ```
    (py311) $ conda update opencv 
    ```

1.  这次，`conda`会提示你是否想更新`opencv`及其依赖项到最新版本。这次，输入*n*取消软件包更新。与其单独更新软件包，通常一次性更新活动虚拟环境中所有已安装的软件包更为方便：

    ```
    (py311) $ conda update --all 
    ```

1.  移除已安装的软件包也很简单：

    ```
    (py311) $ conda remove jupyter notebook 
    ```

当`conda`移除`jupyter`和`notebook`时，它还会移除它们所有悬挂的依赖项。悬挂的依赖项是指那些没有其他已安装软件包依赖的已安装包。像大多数通用的包管理器一样，`conda`不会移除其他已安装包仍依赖的任何依赖项。

1.  有时候你可能不知道想要安装的软件包的确切名称。亚马逊提供了一个名为 Boto 的 AWS SDK for Python。像许多 Python 库一样，Boto 有一个适用于 Python 2 的版本和一个适用于 Python 3 的较新版本（Boto3）。要在 Anaconda 中搜索包含`boto`名称的包，可以输入以下命令：

    ```
    (py311) $ conda search '*boto*' 
    ```

1.  你应该在搜索结果中看到`boto3`和`botocore`。在撰写本文时，Anaconda 上可用的最新版本的`boto3`是 1.36.3。要查看该特定版本`boto3`的详细信息，请输入以下命令：

    ```
    (py311) $ conda search boto3=1.36.3 --info 
    ```

包详情显示`boto3`版本 1.36.3 依赖于`botocore`（`botocore >=1.36.3,<1.37.0`），所以安装`boto3`会同时安装`botocore`。

假设你已经在 Jupyter notebook 中安装了所有开发 OpenCV 项目所需的包。你如何与他人分享这些项目需求，以便他们能够重现你的工作环境呢？答案可能会让你吃惊：

1.  你将当前活动的虚拟环境导出为 YAML 文件：

    ```
    (py311) $ conda env export > my-environment.yaml 
    ```

1.  就像 `pip freeze` 生成的需求列表一样，`conda` 导出的 YAML 文件是你虚拟环境中所有已安装包的列表，并附带它们的固定版本。从环境文件创建 `conda` 虚拟环境需要使用 `-f` 选项和文件名：

    ```
    $ conda env create -f my-environment.yaml 
    ```

1.  环境名称已包含在导出的 YAML 文件中，因此创建环境时无需使用 `--name` 选项。任何从 `my-environment.yaml` 创建虚拟环境的人，在执行 `conda env list` 时，都能在环境列表中看到 `py311`。

`conda` 是开发人员工具库中一个非常强大的工具。通过将通用包安装与虚拟环境结合使用，它提供了一个引人注目的部署方案。`conda` 实现了 Docker（接下来介绍）所做的许多相同目标，但没有使用容器。由于其专注于数据科学社区，它在 Python 上比 Docker 更具优势。因为主要的 ML 框架（如 PyTorch 和 TensorFlow）大多基于 CUDA，寻找 GPU 加速的二进制文件通常是困难的。`conda` 通过提供多个预编译的包二进制版本解决了这个问题。

将 `conda` 虚拟环境导出为 YAML 文件，以便在其他机器上安装，是另一种部署选项。这种解决方案在数据科学社区中很受欢迎，但在嵌入式 Linux 生产环境中不起作用。`conda` 不是 Yocto 支持的三大包管理器之一。即使 `conda` 可用，考虑到资源限制，将 Miniconda 安装到 Linux 镜像中的存储空间对大多数嵌入式系统来说并不合适。

如果你的开发板配备了像 NVIDIA Jetson 系列这样的 NVIDIA GPU，那么你确实希望使用 `conda` 来进行设备端开发。幸运的是，有一个名为 **Miniforge** ([`github.com/conda-forge/miniforge)`](https://github.com/conda-forge/miniforge)) 的 `conda` 安装程序，已知在像 Jetson 这样的 64 位 ARM 机器上能够正常工作。使用设备上的 `conda`，你可以安装 `jupyter`、`numpy`、`pandas`、`scikit-learn` 和大多数其他流行的 Python 数据科学库。

# 总结

到现在为止，你可能会问自己：“这些关于 Python 包管理的内容和嵌入式 Linux 有什么关系？”答案是“关系不大”，但请记住，这本书的标题中也包含了*开发*一词，而这一章与现代软件开发息息相关。作为开发人员，要成功，你需要能够快速、频繁并可重复地将代码部署到生产环境。这意味着你需要小心地管理依赖项，并尽可能地自动化整个过程。现在，你已经知道如何使用 Python 来实现这一点。

# 进一步学习

+   *Python 包管理用户指南*，PyPA – [`packaging.python.org`](https://packaging.python.org)

+   *setup.py 与 requirements.txt*，Donald Stufft 编写 – [`caremad.io/posts/2013/07/setup-vs-requirement`](https://caremad.io/posts/2013/07/setup-vs-requirement)

+   *pip 用户指南*，PyPA – [`pip.pypa.io/en/latest/user_guide/`](https://pip.pypa.io/en/latest/user_guide/)

+   *Conda 用户指南*，Anaconda 公司 – [`docs.conda.io/projects/conda/en/latest/user-guide`](https://docs.conda.io/projects/conda/en/latest/user-guide)
