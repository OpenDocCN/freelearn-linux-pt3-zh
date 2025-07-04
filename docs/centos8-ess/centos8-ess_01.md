2. Linux 简史

CentOS 是 Linux 操作系统的众多变种之一（也称为发行版）。它基于由美国公司 Red Hat, Inc. 开发的 Red Hat 企业版 Linux 发行版的源代码，总部位于北卡罗来纳州的罗利市。该公司成立于 1990 年代中期，由当时由 Marc Ewing 和 Bob Young 拥有的两家公司合并而成。然而，Linux 的起源更为悠久。本章将概述 Linux 操作系统和 Red Hat, Inc. 的历史，然后解释 CentOS 如何融入这一背景。

2.1 Linux 究竟是什么？

Linux 是一种操作系统，就像 Windows 是操作系统一样（Linux 和 Windows 之间的相似之处到此为止）。操作系统这个术语用于描述充当计算机硬件和我们每天使用的应用程序之间的层的软件。当程序员编写应用程序时，他们通过操作系统接口来执行诸如将文件写入硬盘驱动器、在屏幕上显示信息等任务。如果没有操作系统，每个程序员都必须编写代码直接访问系统的硬件。此外，程序员还必须支持每一个曾经创造的硬件，以确保应用程序能够在所有可能的硬件配置上运行。由于操作系统处理了所有这些硬件复杂性，应用程序开发变得容易得多。Linux 只是当今可用的多种操作系统之一。

2.2 UNIX 的起源

要了解 Linux 的历史，我们首先需要回到 1960 年代末的 AT&T 贝尔实验室。那时，AT&T 停止了参与名为 Multics 的新操作系统的开发。两位 AT&T 工程师 Ken Thompson 和 Dennis Ritchie 决定将他们从 Multics 项目中学到的知识应用于创建一个新的操作系统，名为 UNIX，并迅速在企业和学术机构中获得了广泛的认可和采用。

各种专有的 UNIX 实现最终进入了市场，包括 IBM（AIX）、惠普（HP-UX）和 Sun Microsystems（SunOS 和 Solaris）等公司创建的版本。此外，一种名为 MINIX 的类 UNIX 操作系统由 Andrew S. Tanenbaum 创建，旨在用于教育用途，并向大学提供源代码访问。

2.3 谁创造了 Linux？

Linux 的起源可以追溯到两个人的工作和理念。Linux 操作系统的核心是所谓的内核。内核是操作系统功能所需的核心功能集合。它管理系统资源并处理硬件与应用程序之间的通信。Linux 内核由 Linus Torvalds 开发，Linus 对 MS-DOS 不感兴趣，并且由于 Intel 80386 微处理器的 MINIX 版本迟迟未发布，他决定编写自己的类似 UNIX 的内核。当他完成内核的第一个版本时，他将其以开源许可证发布，允许任何人下载源代码并自由使用和修改，而无需支付 Linus 任何费用。

大约在同一时期，Free Software Foundation 的 Richard Stallman，这位强烈支持自由和开源软件的人，正在开发自己的开源操作系统。然而，Stallman 并未一开始就专注于内核，而是决定从开发所有必要的 UNIX 工具、实用程序和编译器的开源版本开始，用于使用和维护操作系统。当他完成了这些基础设施的开发后，似乎最明显的解决方案是将他的工作与 Linus 编写的内核结合，创建一个完整的操作系统。这个组合被称为 GNU/Linux。纯粹主义者坚持认为 Linux 应该始终被称为 GNU/Linux（事实上，一度 Richard Stallman 拒绝接受任何没有将 Linux 称为 GNU/Linux 的媒体采访）。考虑到 GNU 工具是由 Free Software Foundation 开发的，且它们构成了 GNU/Linux 的重要和必要部分，这种立场并非不合理。不幸的是，大多数人和媒体仅称其为 Linux，这种情况可能会一直持续下去。

2.4 红帽的早期历史

1993 年，Bob Young 创建了一家公司，名为 ACC Corporation，按照 Young 的说法，他是从“妻子的缝纫室”里经营这家公司。ACC 这个名字本来是为了代表一个目录业务，但也同时是他妻子经营的一家小型企业的缩写，名为“康涅狄格州的古董和收藏品”。在 ACC 目录业务中销售的物品包括 Linux CD 和相关的开源软件。

大约在同一时期，Marc Ewing 创建了自己的 Linux 发行版公司，并将其命名为 Red Hat Linux（因为他在卡内基梅隆大学时习惯戴红色棒球帽）。

1995 年，ACC 收购了 Red Hat，并采用了 Red Hat, Inc. 的名称，经历了快速和显著的增长。Bob Young 在公司于 1999 年 8 月上市后不久辞去了 CEO 职位，之后投身于多个商业和慈善事业，包括一家名为 Lulu 的按需印刷图书出版公司和拥有两支加拿大职业体育队的股份。2018 年，IBM 宣布计划以 340 亿美元收购 Red Hat, Inc.

2.5 红帽支持

Red Hat Linux 的早期版本通过软盘和光盘发货给客户（这当然是在宽带互联网普及之前）。当用户遇到软件问题时，他们只能通过电子邮件联系 Red Hat。事实上，Bob Young 常常开玩笑说，这种方式有效地限制了支持请求，因为当客户意识到需要帮助时，他们的计算机通常已经无法使用，因此无法通过电子邮件向 Red Hat 支持团队寻求帮助。后来，Red Hat 提供了更好的支持服务，配套有付费订阅，并且现在提供了多种支持等级，从“自助帮助”（无支持）到高级支持不等。

2.6 开源

Red Hat Enterprise Linux 8 是 Red Hat 目前的商业产品，主要面向企业和关键任务的安装。它也是 Red Hat 不断扩展的产品和服务生态系统的基石。RHEL 是一个开源产品，您可以免费下载源代码并自行构建软件（这项任务不可小觑）。然而，如果您希望下载一个预构建的、可安装的二进制版本（无论是否有支持），则需要付费。

2.7 Fedora 项目

Red Hat 还赞助了 Fedora 项目，其目标是提供一个免费的 Linux 操作系统（包括源代码和二进制发行版），即 Fedora Linux。Fedora Linux 还作为许多新功能的试验场，这些功能最终会被纳入 Red Hat 企业 Linux 操作系统家族。例如，CentOS 8.0 在很大程度上基于 Fedora 29。

2.8 CentOS - 免费的替代品

对于那些无法负担 Red Hat Enterprise Linux 订阅的用户，CentOS 操作系统提供了另一种选择。CentOS 项目最初是一个社区驱动的项目，但现在已与 Red Hat 合作，它将 Red Hat Enterprise Linux 的源代码去除 Red Hat 品牌和订阅要求，进行编译并提供下载。虽然 CentOS 在许多方面提供了与 RHEL 完全相同的操作系统，但它不包括 Red Hat 的技术支持。

2.9 总结

Linux 操作系统的起源可以追溯到 Linus Torvalds 和 Richard Stallman 的工作，他们结合了 Linux 内核和 GNU 项目所构建的工具及编译器。

多年来，Linux 的开源特性促使了各种不同 Linux 发行版的发布。其中一个发行版是 Red Hat Enterprise Linux，由 Bob Young 和 Mark Ewing 创立的公司 Red Hat, Inc.创建。Red Hat 专注于提供企业级 Linux 软件解决方案，并结合广泛的技术支持服务。CentOS 本质上是基于与 Red Hat Enterprise Linux 相同的源代码构建的，但去除了 Red Hat 的品牌标识。
