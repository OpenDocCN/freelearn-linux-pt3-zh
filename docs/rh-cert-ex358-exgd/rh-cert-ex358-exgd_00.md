# 前言

本书探讨了 Red Hat Linux 服务器管理和自动化的世界，旨在帮助你在参加 EX358 认证考试时取得成功。它详细讲解了 Red Hat 列出的每个主要目标，帮助你全面准备，完成职业生涯中的这一重要步骤。

结合多年在 Red Hat 和 Linux 领域的经验，我将深入讲解你需要了解的不同应用，帮助你理解 Red Hat Enterprise Linux 和 Ansible 自动化。我在 Red Hat 工作超过 6 年，使用 Red Hat 系统已经超过 10 年。我拥有多项认证，包括 Red Hat 认证架构师，这一认证涵盖了多个 Red Hat 产品和领域。

本书将帮助你面对在测试环境中遇到的所有细节，并覆盖一些有用的领域，节省你的时间和精力，从而帮助你以优异的成绩完成认证考试。每个目标都被分解并以有意义的方式呈现，帮助你记住成功通过考试所需的知识。

# 本书适合谁阅读

本书适合那些希望提升职业发展的 Linux 系统管理员。它将帮助你掌握额外的技能，使你能够通过这项行业考试。在开始本书中的任务之前，你应当具备扎实的 Red Hat Linux 和 Ansible 自动化背景。

# 本书的内容

*第一章*，*块存储 – 学习如何在 Red Hat Enterprise Linux 上配置块存储*，详细介绍了 iSCSI 配置中的块存储设置及其连接性。

*第二章*，*网络文件存储 – 扩展你对如何共享数据的知识*，解释了 NFS 和 Samba 的设置，以及对共享驱动器的权限和连接性。

*第三章*，*自动化网络服务 – Red Hat Linux 网络概述*，标志着开始使用 Red Hat Enterprise Linux 进行网络配置，如 IP 地址设置。

*第四章*，*链路聚合创建 – 创建自己的链路并掌握网络领域*，解释了如何在 Red Hat Enterprise Linux 服务器上设置网络团队。

*第五章*，*DNS、DHCP 和 IP 地址配置 – 深入了解 Red Hat Linux 网络*，详细介绍了如何从 Red Hat Enterprise Linux 服务器为基础设施提供 DNS 和 DHCP 服务。

*第六章*，*打印机和电子邮件 – 在 Linux 服务器上设置打印机和电子邮件服务*，涵盖了如何设置打印服务以及空白电子邮件客户端。

*第七章*，*数据库 – 设置与使用 MariaDB SQL 数据库*，教您如何使用 MySQL 命令在 Red Hat Enterprise Linux 服务器上安装、保护和配置 MariaDB。

*第八章*，*Web 服务器和网络流量 – 学习如何创建和控制流量*，讲解了如何安装和配置 Apache 与 Nginx web 服务器。我们还会讲解如何控制连接到我们基础设施的 web 服务器的流量。

*第九章*，*全面回顾与考试测试题*，提供了一些示例问题，让我们可以将全书所学内容应用到实际中。

*第十章*，*帮助通过考试的小窍门和技巧*，介绍了一些简单的窍门和技巧，可以帮助您节省时间，并提高您在考试中的成功率。

# 要想最大化地利用本书

在阅读本书之前，您应该具备 Red Hat Enterprise Linux 的工作知识，并且应该有使用 Ansible 自动化编写剧本的经验。如果没有这些技能，这本书和考试可能会让您感到有些困难。

| **本书涉及的软件/硬件** | **操作系统要求** |
| --- | --- |
| Ansible 2.9 | Red Hat Enterprise Linux 8.1 |
| VirtualBox | OSX 或 Windows |

您需要从您的工作区连接互联网，以便下载所需的软件。您还需要使用 VirtualBox 来跟随实验室布局，并理解如果偏离布局，可能会有所不同。您还需要为 Red Hat 设置开发者许可，相关说明见*第一章*。

**如果您使用的是本书的数字版，建议您自己输入代码，或者从本书的 GitHub 仓库访问代码（链接将在下节提供）。这样可以帮助您避免与代码复制和粘贴相关的潜在错误。**

您需要有运行 Red Hat Enterprise Linux 的经验，并且熟悉 Ansible 自动化。建议您有这些系统和应用的实际经验，以便更容易跟随和理解考试内容。

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件，链接为[`github.com/PacktPublishing/Red-Hat-Certified-Specialist-in-Services-Management-and-Automation-EX358-Exam-Guide`](https://github.com/PacktPublishing/Red-Hat-Certified-Specialist-in-Services-Management-and-Automation-EX358-Exam-Guide)。如果代码有更新，GitHub 仓库中的代码会随之更新。

我们还提供了其他来自我们丰富的图书和视频目录中的代码包，您可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)查看。赶紧去看看吧！

# 下载彩色图片

我们还提供了包含本书中截图和图表的彩色 PDF 文件。您可以在此下载：[`packt.link/Yrj1x`](https://packt.link/Yrj1x)。

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入以及 Twitter 用户名。例如：“请记住，您的`/etc/hosts`文件将根据您的 IP 地址有所不同。”

代码块设置如下：

```
tasks:
    - name: Install Samba and Samba-Client
      package:
        name:
          - samba
          - samba-client
          - cifs-utils
        state: latest
```

任何命令行输入或输出均按如下方式书写：

```
$ ssh-keygen
```

**粗体**：表示新术语、重要词汇或屏幕上显示的词语。例如，菜单或对话框中的词汇通常以**粗体**显示。示例如下：“选择**编辑** **连接**。”

提示或重要注意事项

如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何内容有疑问，请发送电子邮件至 customercare@packtpub.com，并在邮件主题中注明书名。

**勘误**：尽管我们已尽最大努力确保内容的准确性，但错误仍有可能发生。如果您发现本书中的错误，我们将非常感激您向我们报告。请访问[www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)并填写表单。

**盗版**：如果您在互联网上发现我们作品的任何非法复制形式，我们将非常感激您提供相关的网址或网站名称。请通过版权@packtpub.com 与我们联系，并附上相关材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有意写书或参与书籍的编写，请访问[authors.packtpub.com](https://authors.packtpub.com)。

# 分享您的想法

阅读完*《红帽认证服务管理与自动化 EX358 考试指南》*后，我们希望听到您的想法！请[点击这里直达 Amazon 评论页面](https://packt.link/r/1803235497)并分享您的反馈。

您的评论对我们和技术社区都很重要，帮助我们确保提供优质内容。

# 下载本书的免费 PDF 副本

感谢您购买本书！

您喜欢在旅途中阅读，但无法随身携带纸质书籍吗？

您的电子书购买是否与您选择的设备不兼容？

别担心，现在购买每本 Packt 书籍时，您将免费获得该书的无 DRM PDF 版本。

随时随地，任何设备上都能阅读。您可以直接将代码从您喜爱的技术书籍中复制并粘贴到您的应用程序中。

优惠不止于此，您还可以获得独家折扣、新闻通讯和每日发送到您邮箱的精彩免费内容。

按照这些简单的步骤获取福利：

1.  扫描二维码或访问以下链接

![](img/B18607_QR_Free_PDF.jpg)

https://packt.link/free-ebook/9781803235493

1.  提交您的购买凭证

1.  就是这样！我们将直接通过电子邮件发送您的免费 PDF 和其他福利

# 第一部分：Red Hat Linux 8 —— 使用自动化配置和维护存储

在本部分中，您将学习如何手动和自动设置与维护块存储和文件存储。这将满足通过 Red Hat EX358 考试的目标。

本部分包含以下章节：

+   *第一章*，*块存储——学习如何在 Red Hat 企业版 Linux 上配置块存储*

+   *第二章*，*网络文件存储——扩展您对数据共享方式的知识*
