# 序言

Linux 系统管理是一项要求你始终保持在技术前沿的工作。因此，你需要拥有合适的工具来确保工作得当。

Fedora Linux 是基于 Red Hat Enterprise Linux 开发的发行版，提供了能够帮助我们完成任务的工具。

在本书中，我将与您分享如何将 Fedora Linux 用作工作站操作系统来管理 Linux 系统。

通过我的 20 年系统管理员经验，结合建议、最佳实践、技巧，甚至一些窍门，我将帮助你建立一个能够优化系统管理员任务的工作站。

# 本书适合谁阅读

本书适合所有希望开始使用 Fedora Linux 作为工作站，执行系统管理员日常任务的人。它还将帮助你学习如何优化该发行版的工具以进行管理任务。

你需要了解 Linux 和系统管理的基础知识，但不需要深入的知识。

本书提供了一个实际的背景，展示如何使用工作站进行最常见的系统管理任务。

# 本书的内容

*第一章*，*Linux 与开源项目*，介绍了当今最流行的开源项目和 Linux 发行版，重点突出它们的主要用途和区别。

*第二章*，*安装最佳实践*，研究了安装 Fedora Linux 以及优化其作为工作站使用的最佳实践。

*第三章*，*调整桌面环境*，概述了增强工作环境可用性的不同小程序和插件。

*第四章*，*优化存储使用*，分析了不同类型的本地存储及其配置，以优化性能。

*第五章*，*网络与连接性*，概述了网络连接管理以及性能监控工具。

*第六章*，*沙箱应用程序*，探讨了桌面沙箱应用程序的使用与配置。

*第七章*，*文本编辑器*，总结了 Fedora Linux 中最流行和广泛使用的文本编辑器的特点。

*第八章*，*LibreOffice 办公套件*，概述了 LibreOffice 办公套件的办公工具，并总结了每个应用程序的主要选项——Writer 用于文字处理，Calc 用于电子表格，Impress 用于幻灯片，Draw 用于图像。

*第九章*，*邮件客户端与浏览器*，探讨了 Fedora Linux 中包含的互联网生产力工具、邮件客户端和浏览器，如 Evolution、Thunderbird、Firefox 和 Google Chrome。

*第十章*，*系统管理*，提供了系统管理的基础知识，以及一些有用的技巧和快捷键。它还探讨了应用最佳实践的基础。

*第十一章*，*性能调优最佳实践*，探讨了操作系统调优的最佳实践，作为提高系统管理性能的一种方法。

*第十二章*，*SELinux*，介绍了基于策略的访问控制基础，作为 Fedora Linux 中的安全执行模块。

*第十三章*，*虚拟化与容器*，概述了不同的 Fedora Linux 虚拟化资源。它提供了虚拟化的基础以及 Fedora Linux 中可用的方法——基于 KVM/libvirt 的虚拟化或使用 Podman 的容器。

# 最大化使用本书的效果

尽管管理员需要具备基本的 Linux 知识，但要跟随每章中的安装和配置指南并不需要深入的知识。

| **操作系统** | **下载链接** |
| --- | --- |
| Fedora Linux 工作站 | [`fedoraproject.org/workstation/download/`](https://fedoraproject.org/workstation/download/) |

**如果你正在使用本书的数字版本，我们建议你亲自输入代码或从本书的 GitHub 仓库获取代码（链接在下一节提供）。这样做有助于避免与复制和粘贴代码相关的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，网址为[`github.com/PacktPublishing/Fedora-Linux-System-Administration`](https://github.com/PacktPublishing/Fedora-Linux-System-Administration)。如果代码有更新，它将会在 GitHub 仓库中更新。

我们还在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上提供了其他代码包，来自我们丰富的书籍和视频目录。赶紧去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如：“将下载的`WebStorm-10*.dmg`磁盘镜像文件作为另一个磁盘挂载到你的系统中。”

一块代码块如下所示：

```
for <variable> in <list>
do
command <variable>
done
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会用粗体标出：

```
if <condition>;
then
<statement 1>
...
<statement n>
else
<statement alternative>
fi
```

任何命令行输入或输出都按以下方式书写：

```
$ sudo grep -E 'svm|vmx' /proc/cpuinfo
$ sudo dnf install qemu-kvm virt-manager virt-viewer guestfstools virt-install genisoimage
```

**粗体**：表示新术语、重要单词或屏幕上显示的文字。例如，菜单或对话框中的词语以**粗体**显示。以下是一个例子：“从**管理**面板中选择**系统信息**。”

提示或重要备注

看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何部分有问题，请发送邮件至 customercare@packtpub.com，并在邮件主题中提到书名。

**勘误表**：尽管我们已尽最大努力确保内容的准确性，但仍可能出现错误。如果你在本书中发现错误，我们将非常感谢你向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表格。

**盗版**：如果你在互联网上发现任何形式的非法复制品，我们将非常感谢你提供该资料的地址或网站名称。请通过 copyright@packtpub.com 与我们联系并附上资料的链接。

**如果你有兴趣成为作者**：如果你在某个领域拥有专长，并且有兴趣撰写或贡献一本书，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享你的想法

阅读完 *Fedora Linux 系统管理* 后，我们很希望听到你的想法！请 [点击这里直接访问亚马逊评论页面](https://packt.link/r/1804618403) 并分享你的反馈。

你的评价对我们和技术社区都非常重要，帮助我们确保提供优质内容。

## 下载这本书的免费 PDF 版本

感谢购买本书！

你喜欢随时随地阅读，但又无法随身携带印刷版书籍吗？你的电子书购买是否与所选设备不兼容？

不用担心，现在购买每本 Packt 书籍时，你都会免费获得该书的无 DRM PDF 版本。

在任何地方、任何设备上阅读。直接从你最喜爱的技术书籍中搜索、复制并粘贴代码到你的应用程序中。

这里的优惠还不止这些，你可以获得独家的折扣、新闻通讯，并且每天将有精彩的免费内容直接发送到你的邮箱

按照以下简单步骤享受这些福利：

1.  扫描二维码或访问下面的链接

![](img/B19121_QR_Free_PDF.jpg)

[`packt.link/free-ebook/978-1-80461-840-0`](https://packt.link/free-ebook/978-1-80461-840-0)

1.  提交你的购买凭证

1.  就是这样！我们会直接将你的免费 PDF 以及其他福利发送到你的邮箱
