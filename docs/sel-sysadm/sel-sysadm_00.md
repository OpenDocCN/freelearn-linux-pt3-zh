# 前言

**增强安全 Linux**（**SELinux**）是 Linux 最完整的安全解决方案之一，默认情况下大多数主要 Linux 发行版都提供该解决方案，例如 Red Hat Enterprise Linux、CentOS、Fedora 和 Gentoo，同时它也可以轻松地在其他发行版中启用，如 SUSE、Debian 和 Ubuntu。

SELinux 使管理员能够进一步加强其 Linux 系统和应用程序的安全性，使得入侵者和恶意行为者更难滥用系统。

*SELinux 系统管理 – 第三版*提供了 SELinux 在 Linux 系统中的端到端覆盖，从理解 SELinux 是什么以及它如何作用，到调整 SELinux 控制和它在 Linux 和应用平台中的集成，再到定义和维护自定义策略。

# 本书适用于谁

本书适用于希望控制其系统安全状态的 Linux 管理员。它包含有关 SELinux 操作和管理程序的最新信息，您将能够通过**强制访问控制**（**MAC**）进一步加强您的系统——这是一种已经塑造 Linux 安全多年的安全策略。

本书对于 IT 架构师也具有启发性，可以帮助他们理解如何将 SELinux 定位为增强 Linux 系统和组织内 Linux 托管服务的安全性。

读者应具备合理的 Linux 系统维护经验，包括用户管理、软件安装与维护、常规 Linux 安全控制和网络配置。

# 本书涵盖的内容

*第一章*，*SELinux 基本概念*，提供了有关 SELinux 技术的基本见解，并使您理解 SELinux 实现之间的差异。

*第二章*，*理解 SELinux 决策与日志记录*，教您如何分析 SELinux 事件，以及如何配置系统日志以便于 SELinux 故障排除。

*第三章*，*管理用户登录*，使您能够管理 Linux 用户并将其与正确的 SELinux 上下文关联。

*第四章*，*使用文件上下文和进程域*，解释了 SELinux 标签如何在系统上暴露，以及如何更改文件和资源的 SELinux 上下文。

*第五章*，*控制网络通信*，介绍了 SELinux 在网络级别的访问控制保护，从基于套接字的保护措施到使用 SELinux 进行的包过滤。

*第六章*，*通过基础设施即代码编排配置 SELinux*，展示了如何使用自动化和编排工具在大规模环境中配置 SELinux 设置。

*第七章*，*配置特定应用程序的 SELinux 控制*，解释了 SELinux 是如何被多个应用程序采纳，以进一步增强它们的安全性。

*第八章*，*SEPostgreSQL——使用 SELinux 扩展 PostgreSQL*，帮助你了解如何在常规 PostgreSQL 部署中启用 SEPostgreSQL，以及如何在数据库引擎内使用 SELinux 控制。

*第九章*，*安全虚拟化*，结合 libvirt 和其他虚拟化技术以及 SELinux，进一步保护并隔离虚拟来宾之间的相互影响。

*第十章*，*使用 Xen 安全模块与 FLASK*，教你如何使用类似 SELinux 的方法，通过 Xen 安全模块来隔离其来宾，以及管理员如何进一步调整和优化隔离。

*第十一章*，*增强容器化工作负载的安全性*，展示了容器平台如 Docker、podman 和 Kubernetes 如何使用 SELinux 来保护宿主系统，防止潜在不可信的容器，并在容器之间提供隔离。

*第十二章*，*调优 SELinux 策略*，扩展了 SELinux 布尔值及其对系统的影响，并展示了如何处理不同的 SELinux 策略模块。

*第十三章*，*分析策略行为*，解释了管理员和分析师如何解读 SELinux 策略，并使用策略分析工具来了解策略的允许行为。

*第十四章*，*处理新应用程序*，介绍了如何在当前 SELinux 策略不支持的新应用程序上应用 SELinux。

*第十五章*，*使用参考策略*，解释了如何使用参考策略创建和调整 SELinux 策略。

*第十六章*，*使用 SELinux CIL 开发策略*，介绍了通用中间语言，并讲解如何运用它来开发自定义策略。

# 为了充分利用本书

本书专注于 SELinux 技术，SELinux 在许多 Linux 发行版中默认启用。虽然本书使用 CentOS 8 版本作为大部分示例，但任何基于参考策略（所有主要 Linux 发行版都使用这种策略）的 Linux 发行版都能跟随本书的内容。

在*第五章*，*控制网络通信*，有一节讲解了如何使用 SELinux 进行 InfiniBand 基础设施的配置，如果你想严格按照示例操作，需要专门的硬件。然而，本章的大部分内容无需额外的硬件要求即可跟随。

**如果你正在使用本书的数字版本，我们建议你自己输入代码，或通过 GitHub 仓库访问代码（下节中会提供链接）。这样可以帮助你避免因复制和粘贴代码而产生的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，链接：[`github.com/PacktPublishing/SELinux-System-Administration-Third-Edition`](https://github.com/PacktPublishing/SELinux-System-Administration-Third-Edition)。

如果代码有更新，相关更新会发布在现有的 GitHub 仓库中。

我们还有其他代码包，来自我们丰富的书籍和视频目录，可以在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快去看看吧！

# 《实战中的代码》

本书的《实战中的代码》视频可以在[`bit.ly/3o4paOb`](https://bit.ly/3o4paOb)观看。

# 下载彩色图片

我们还提供了一份 PDF 文件，包含本书中使用的截图/图表的彩色版本。你可以在这里下载：`static.packt-cdn.com/downloads/9781800201477_ColorImages.pdf`。

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。例如：“在激活模块时，`semodule`命令会将这些模块复制到一个专门的目录中。”

代码块如下所示：

```
  chain input {
    type filter hook input priority 0;
    ct state new meta secmark set tcp dport map @secmapping_in
    ct state new ct secmark set meta secmark
    ct state established,related meta secmark set ct secmark
  }
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目会用粗体显示：

```
(roleattributeset cil_gen_require system_r)
(block pgpool
  (type domain)
  (roletype .system_r domain)
)
```

任何命令行的输入或输出都如下所示：

```
$ cat /etc/passwd
cat: /etc/passwd: Permission denied
```

**粗体**：表示新术语、重要的词汇或你在屏幕上看到的词汇。例如，菜单或对话框中的词汇会以这种方式出现在文本中。示例：“这可以通过点击**开放策略**按钮，或通过导航至**文件** | **开放策略**来实现。”

提示或重要说明

以这种方式显示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请在邮件主题中提到书名，并通过 customercare@packtpub.com 与我们联系。

**勘误**：虽然我们已尽一切努力确保内容的准确性，但错误有时会发生。如果你在本书中发现错误，请向我们报告。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)，选择你的书籍，点击“勘误提交表单”链接，并填写相关信息。

**盗版**：如果你在互联网上发现任何形式的非法复制本书内容，我们将非常感激你能提供相关的网址或网站名称。请通过 copyright@packt.com 联系我们，并附上该材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域拥有专业知识，并且有兴趣撰写或参与编写书籍，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 评论

请留下评论。当您阅读并使用本书后，为什么不在您购买本书的网站上留下评论呢？潜在的读者可以看到并参考您的公正意见来做出购买决策，我们在 Packt 能了解您对我们产品的看法，而我们的作者也能看到您对他们书籍的反馈。感谢您！

如需了解更多有关 Packt 的信息，请访问[packt.com](http://packt.com)。
