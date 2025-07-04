# 前言

欢迎来到 *Linux 企业自动化实操*，这本书将是你了解一系列最有价值的流程、方法论和工具的指南，帮助你在企业级规模上精简并高效管理你的 Linux 部署。本书将提供你标准化 Linux 环境并进行大规模管理所需的知识和技能，使用包括 Ansible、AWX（Ansible Tower）、Pulp、Katello 和 OpenSCAP 在内的开源工具。你将学习如何创建标准化操作环境，以及如何使用 Ansible 定义、文档化、管理和维护这些标准。此外，你还将掌握安全加固标准，如 CIS 基准。在整个书中，将提供实际的操作示例供你自己动手尝试，帮助你编写自己的代码，并演示所讲解的原则。

# 本书适用对象

本书适用于任何需要设计、实施和维护 Linux 环境的人。它旨在吸引各种开源专业人士，从基础设施架构师到系统管理员，包括 C 级别的专业人士。假设读者已经具备实施和维护 Linux 服务器的能力，并且熟悉构建、修补和维护 Linux 服务器基础设施的相关概念。对 Ansible 及其他自动化工具的 prior 知识并非必需，但有一定帮助。

# 本书涵盖内容

第一章，*在 Linux 上构建标准化操作环境*，详细介绍了标准化操作环境的概念，这是本书中贯穿始终的核心概念，也是你开始本次学习旅程的必要理解。

第二章，*使用 Ansible 自动化 IT 基础设施*，提供了 Ansible 剧本的详细实操分解，包括清单、角色、变量和开发与维护剧本的最佳实践；这是一个速成课程，让你能学习到足够的 Ansible 知识，开始你的自动化之旅。

第三章，*使用 AWX 精简基础设施管理*，通过实际示例，探讨如何安装和使用 AWX（也可作为 Ansible Tower），从而围绕你的 Ansible 自动化基础设施构建良好的业务流程。

第四章，*部署方法论*，帮助你理解与 Linux 环境中大规模部署相关的各种方法，以及如何最大化这些方法的企业优势。

第五章，*使用 Ansible 构建虚拟机模板进行部署*，探讨了通过构建虚拟机模板在超管中以实际且动手的方式进行大规模部署 Linux 的最佳实践。

第六章，*使用 PXE 启动进行自定义构建*，介绍了 PXE 启动的过程，适用于服务器构建的模板化方法无法使用的情况（例如，仍在使用裸机服务器的场景），以及如何编写脚本通过网络构建标准的服务器镜像。

第七章，*使用 Ansible 进行配置管理*，提供了如何在服务投入使用后管理构建的实际示例，以确保一致性始终如一而不限制创新。

第八章，*使用 Pulp 进行企业存储库管理*，介绍了如何以受控方式执行修补，防止即使是最精心标准化的环境中也会重新引入不一致性，利用 Pulp 工具来实现这一点。

第九章，*使用 Katello 进行修补*，在介绍了 Pulp 工具的工作基础上，引入了 Katello，提供了更多对存储库的控制，同时提供了一个用户友好的图形用户界面。

第十章，*在 Linux 上管理用户*，提供了详细的用户账户管理方法，使用 Ansible 作为编排工具，并结合使用如 LDAP 目录等集中认证系统。

第十一章，*数据库管理*，介绍了如何使用 Ansible 自动化数据库的部署，并在 Linux 服务器上执行常规数据库管理任务。

第十二章，*使用 Ansible 执行常规维护*，探讨了 Ansible 在 Linux 服务器环境中可以执行的一些更高级的持续维护工作。

第十三章，*使用 CIS 基准*，深入探讨了 CIS 服务器加固基准以及如何在 Linux 服务器上应用它们。

第十四章，*使用 Ansible 进行 CIS 加固*，介绍了如何通过 Ansible 高效、可重复地在整个 Linux 服务器环境中部署安全加固策略。

第十五章，*使用 OpenSCAP 审计安全策略*，提供了如何安装和使用 OpenSCAP 持续审计 Linux 服务器的实操示例，因为安全标准可能会被恶意或其他良意的终端用户逆转。

第十六章，*技巧与窍门*，探讨了一些技巧和窍门，以帮助你在企业不断变化的需求面前，保持 Linux 自动化过程的顺畅运行。

# 为了最大程度地利用本书

为了跟随本书中的示例，建议你至少拥有两台 Linux 机器用于测试，尽管更多的机器会更有利于更全面地开发示例。这些可以是物理机或虚拟机——所有示例都是在一组 Linux 虚拟机上开发的，但同样适用于物理机器。在第五章，*使用 Ansible 构建虚拟机模板进行部署*中，我们利用 KVM 虚拟机上的嵌套虚拟化来构建 Linux 镜像。此操作的硬件要求在本章开始时列出。这将需要访问一台具有适当 CPU 的物理机器，或者一台支持嵌套虚拟化的虚拟化管理程序（例如，VMware 或 Linux KVM）。

请注意，本书中的某些示例可能会干扰你网络上的其他服务；如果存在此类风险，会在每章的开头进行说明。我建议你在一个隔离的测试网络中尝试这些示例，除非/直到你确信它们不会对你的操作产生任何影响。

尽管书中提到其他 Linux 发行版，但我们专注于两个关键的 Linux 发行版——CentOS 7.6（不过如果你有 Red Hat Enterprise Linux 7.6 的访问权限，也可以使用，它在大多数示例中也应该同样适用），以及 Ubuntu Server 18.04。所有测试机器都从官方 ISO 镜像构建，使用最小安装配置。

因此，当需要额外的软件时，我们将带你了解安装所需软件的步骤，以便你能够完成示例。如果你选择完成所有示例，你将安装如 AWX、Pulp、Katello 和 OpenSCAP 等软件。唯一的例外是 FreeIPA，它在第十章，*在 Linux 上管理用户*中提到。为你的企业安装目录服务器是一个庞大的主题，不幸的是需要的篇幅超出了本书的范围——因此，你可能需要自行探讨这个话题。

该文本假设你将在你的 Linux 测试机器之一上运行 Ansible，但实际上 Ansible 可以在任何安装了 Python 2.7 或 Python 3（版本 3.5 及以上）的机器上运行（Windows 可以作为控制机运行，但仅通过在较新版本的 Windows 上运行**Windows 子系统 Linux**（**WSL**）层来实现）。Ansible 支持的操作系统包括（但不限于）Red Hat、Debian、Ubuntu、CentOS、macOS 和 FreeBSD。

本书使用的是 Ansible 2.8.x.x 系列版本，尽管部分示例是针对在编写过程中发布的 Ansible 2.9.x.x 版本。Ansible 的安装说明可以在 https:/​/​docs.​ansible.​com/​ansible/​intro_​installation.​html 找到。

# 下载示例代码文件

你可以从你的帐户在[www.packt.com](http://www.packt.com)下载本书的示例代码文件。如果你在其他地方购买了本书，可以访问[www.packtpub.com/support](https://www.packtpub.com/support)，注册后，文件将直接通过电子邮件发送给你。

你可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择支持标签。

1.  点击“代码下载”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用最新版本解压文件或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址是[`github.com/PacktPublishing/Hands-On-Enterprise-Automation-on-Linux`](https://github.com/PacktPublishing/Hands-On-Enterprise-Automation-on-Linux)。如果代码有更新，它将会在现有的 GitHub 仓库中更新。

我们还提供其他来自我们丰富书籍和视频目录的代码包，访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**，查看它们吧！

# 下载彩色图片

我们还提供了一份 PDF 文件，里面有本书使用的截图/图表的彩色图片。你可以在此下载： [`static.packt-cdn.com/downloads/9781789131611_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781789131611_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。例如：“首先，让我们创建一个名为`loadmariadb`的角色。”

代码块设置如下：

```
- name: Ensure PostgreSQL service is installed and started at boot time
  service:
    name: postgresql
    state: started
    enabled: yes
```

任何命令行输入或输出都以如下形式编写：

```
$ mkdir /var/lib/tftpboot/EFIx64/centos7
```

**粗体**：表示新术语、重要词汇或屏幕上显示的词汇。例如，菜单或对话框中的词语会以这种方式出现在文本中。举个例子：“从管理面板中选择系统信息。”

警告或重要说明以这种方式显示。

提示和技巧以这种方式显示。

# 联系我们

我们始终欢迎读者的反馈。

**常见反馈**：如果你对本书的任何部分有疑问，请在邮件主题中提及书名，并通过电子邮件联系我们：`customercare@packtpub.com`。

**勘误**：虽然我们已尽力确保内容的准确性，但错误有时会发生。如果您在本书中发现了错误，我们将非常感激您能向我们报告。请访问 [www.packtpub.com/support/errata](https://www.packtpub.com/support/errata)，选择您的书籍，点击“勘误提交表单”链接，并填写相关信息。

**盗版**：如果您在互联网上发现我们作品的任何非法复制品，我们将非常感激您能提供该内容的地址或网站名称。请通过 `copyright@packt.com` 与我们联系，并附上该资料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专长，并且有兴趣编写或参与编写一本书，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。在您阅读并使用本书后，为什么不在您购买本书的网站上留下评论呢？潜在读者可以看到并利用您的公正意见来做出购买决策，我们在 Packt 能了解您对我们产品的看法，而我们的作者也能看到您对他们书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
