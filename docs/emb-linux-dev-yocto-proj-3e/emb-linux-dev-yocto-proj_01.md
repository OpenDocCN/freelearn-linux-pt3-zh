

# 第一章：认识 Yocto 项目

本章介绍 **Yocto 项目**。这里讨论的项目的主要概念在全书中都会不断使用。此外，我们还将简要讨论 Yocto 项目的历史、OpenEmbedded、Poky、BitBake、元数据和版本管理方案。所以，请系好安全带，欢迎登机！

# 什么是 Yocto 项目？

Yocto 项目是 Linux 基金会的一个工作组，定义如下：

Yocto 项目是一个开源协作项目，帮助开发人员创建专为嵌入式产品设计的定制 Linux 系统，无论产品的硬件架构如何。Yocto 项目提供了一个灵活的工具集和开发环境，允许全球的嵌入式设备开发人员通过共享的技术、软件堆栈、配置和最佳实践进行协作，从而创建这些定制的 Linux 镜像。

全球成千上万的开发人员发现，Yocto 项目在系统和应用开发、存档与管理收益、以及在速度、占用空间和内存利用方面的定制化上提供了优势。该项目在交付嵌入式软件堆栈方面已经成为标准。该项目支持多种硬件平台的软件定制和构建交换，并且软件堆栈可以进行维护和扩展。

- Yocto 项目概览与概念手册

Yocto 项目是一个开源协作项目。它提供模板、工具和方法，帮助我们为嵌入式产品创建定制的基于 Linux 的系统，无论硬件架构如何。它可以基于 `glibc` 和 `musl` C 标准库以及为裸机开发提供的 **实时操作系统** (**RTOS**) 工具链，生成定制的 Linux 发行版，正如 Zephyr 项目所做的那样。

由 Linux 基金会成员管理，该项目保持独立于成员组织，这些成员组织通过多种方式参与并为项目提供资源。

它成立于 2010 年，由许多硬件制造商、开源操作系统供应商、电子公司共同合作，旨在减少工作重复，提供资源和信息，以满足新手和经验丰富用户的需求。其中的资源之一是 OpenEmbedded Core，这是 OpenEmbedded 项目提供的核心系统组件。

Yocto 项目聚集了多个公司、社区、项目和工具，目标一致——构建基于 Linux 的嵌入式产品。这些利益相关者同舟共济，由共同的社区需求驱动，共同协作。

# 描述 Yocto 项目

为了帮助我们理解 Yocto 项目的职责和成果，我们可以用计算机的类比来说明。输入是一组描述我们需求的数据，即我们的规格说明。作为输出，我们得到所需的基于 Linux 的嵌入式产品。

输出由操作系统的各个部分组成，涵盖了 Linux 内核、引导加载程序和根文件系统（`rootfs`），这些组件被打包并组织在一起以便协同工作。

Yocto 项目的工具在所有中间步骤中都参与到生成最终 `rootfs` 包和其他交付物的过程。先前构建的软件组件会在不同的构建中重用——无论是应用程序、库，还是任何软件组件。

当无法进行重用时，软件组件会按照正确的顺序和所需的配置进行构建，包括从各自的代码库中提取所需的源代码，如 Linux 内核档案（[www.kernel.org](http://www.kernel.org)）、GitHub、BitBucket 和 GitLab。

Yocto 项目的工具准备其构建环境、实用工具和工具链，减少了对主机软件的依赖。这些实用工具、版本和配置选项独立于主机 Linux 发行版，从而最小化了在生成相同结果时对主机工具的依赖。一个微妙但重要的好处是，确定性大大增强，减少了构建主机的依赖，但首次构建的时间增加。

BitBake 和 OpenEmbedded Core 都在 OpenEmbedded 项目的框架下，而一些项目，如 Poky，则在 Yocto 项目的框架下。它们是互为补充的，在系统中扮演着各自特定的角色。我们将在本章以及全书中详细了解它们如何协同工作。

# OpenEmbedded 项目与 Yocto 项目的联盟

OpenEmbedded 项目大约在 2003 年 1 月成立，当时一些来自 **OpenZaurus** 项目的核心开发者开始使用新的构建系统。从一开始，OpenEmbedded 构建系统便是一个受 **Gentoo Portage** 包管理系统启发并基于该系统的任务调度器，名为 BitBake。因此，该项目迅速扩展了其软件库和支持的机器列表。

由于混乱和缺乏协调的开发，使用 OpenEmbedded 在需要更稳定和精致代码库的产品中变得具有挑战性，这就是 Poky 发行版诞生的原因。Poky 起初是 OpenEmbedded 构建系统的一个子集，并且在有限的架构集上拥有更精致和稳定的代码库。此外，Poky 的体积较小，允许其开发出一些突出的技术，如 IDE 插件和**快速仿真器**（**QEMU**）集成，这些技术至今仍在使用。

Yocto 项目和 OpenEmbedded 项目整合了他们的工作，形成了一个核心构建系统，称为 OpenEmbedded Core。它融合了 Poky 和 OpenEmbedded 的精华，强调更多地使用附加组件、元数据和子集。大约在 2010 年 11 月，Linux 基金会宣布 Yocto 项目将在一个由 Linux 基金会资助的项目下继续这一工作。

# 了解 Poky

Poky 是默认的 Yocto 项目参考发行版，使用 OpenEmbedded 构建系统技术。它由工具、配置文件和配方数据（称为元数据）组成。它是平台独立的，并使用 BitBake 工具、OpenEmbedded Core 以及一组默认的元数据进行交叉编译，如下图所示。此外，它提供了构建和组合成千上万个分布式开源项目的机制，以形成一个完全可定制、完整且一致的 Linux 软件栈。

Poky 的主要目标是提供嵌入式开发者所需的所有功能。

![图 1.1 – Poky 主要组件](img/B19361_01_01.jpg)

图 1.1 – Poky 主要组件

## BitBake

BitBake 是一个任务调度器和执行系统，解析 Python 和 Shell 脚本代码。解析后的代码生成并运行任务，这些任务是按代码的依赖关系排列的一系列步骤。

BitBake 评估所有可用的元数据，管理动态变量展开、依赖关系和代码生成。此外，它跟踪所有任务以确保它们完成，最大化处理资源的利用率，从而减少构建时间并提高可预测性。BitBake 的开发在 [`lists.openembedded.org/g/bitbake-devel`](https://lists.openembedded.org/g/bitbake-devel) 邮件列表中进行，源代码位于 Poky 的 `bitbake` 子目录中。

## OpenEmbedded Core

OpenEmbedded Core 元数据集合提供了 Poky 构建系统的引擎。它提供了核心功能，旨在尽可能地通用和精简。它支持七种不同的处理器架构（ARM、ARM64、x86、x86-64、PowerPC、PowerPC 64、MIPS、MIPS64、RISC-V32 和 RISC-V 64），仅支持通过 QEMU 模拟的平台。

开发工作集中在 [`lists.openembedded.org/g/openembedded-core`](https://lists.openembedded.org/g/openembedded-core) （mailto:openembedded-core@lists.openembedded.org）邮件列表中，并将其元数据存储在 Poky 的 `meta` 子目录中。

## 元数据

该元数据包含配方和配置文件。它由 Python 和 Shell 脚本文本文件混合组成，提供了一个极其灵活的工具。Poky 使用它来扩展 OpenEmbedded Core，并包括两个不同的层次，它们是其他元数据子集，如下所示：

+   `meta-poky`：此层提供默认和支持的发行版策略、视觉品牌和元数据跟踪信息（维护者、上游状态等）。这是作为一个精心策划的模板，可供发行版构建者用于创建他们自定义的发行版。

+   `meta-yocto-bsp`：提供了作为 Yocto 项目开发的参考硬件使用的 **板级支持包** (**BSP**) 和 **质量保证** (**QA**) 过程。

*第九章*，*与 Yocto 项目的开发*，更详细地探讨了元数据，并且当我们编写食谱时作为参考。

# Yocto 项目发布

Yocto 项目每六个月发布一次，分别在 4 月和 10 月。这个发布周期确保了持续的开发流，同时在稳定性方面提供了更多的测试和重点关注。当一个版本准备好时，它会成为**稳定**版本或**长期支持**（**LTS**）版本。

支持期在稳定版本和长期支持（LTS）版本之间有显著差异。稳定版本的支持期为 7 个月，每个稳定版本提供 1 个月的重叠支持。LTS 版本的最短支持期为 2 年，并可选择延长。官方支持期结束后，转为**社区**支持，最终进入**生命周期结束**（**EOL**）。

当官方发布支持期结束后，如果社区成员接手并成为社区维护者，发布版本可以转为社区支持。最终，当源代码在 2 个月内没有更改，或社区维护者不再是活跃成员时，版本将进入生命周期结束（EOL）阶段。

以下图表显示了两个发布周期：

![图 1.2 – 稳定版本或 LTS 版本的发布周期](img/B19361_01_Table_1.jpg)

图 1.2 – 稳定版本或 LTS 版本的发布周期

*表 1.1*提供了 Yocto 项目版本、代号、发布日期和当前支持级别，内容如下所示。更新后的表格可在[`wiki.yoctoproject.org/wiki/Releases`](https://wiki.yoctoproject.org/wiki/Releases)查看：

| **代号** | **版本** | **发布日期** | **支持级别** |
| --- | --- | --- | --- |
| Mickledore | 4.2 | 2023 年 4 月 | 未来（直到 2023 年 10 月） |
| Langdale | 4.1 | 2022 年 10 月 | 稳定（直到 2023 年 5 月） |
| Kirkstone | 4.0 | 2022 年 5 月 | LTS（最短至 2024 年 4 月） |
| Honister | 3.4 | 2021 年 10 月 | 生命周期结束（EOL） |
| Hardknott | 3.3 | 2021 年 4 月 | 生命周期结束（EOL） |
| Gatesgarth | 3.2 | 2020 年 10 月 | 生命周期结束（EOL） |
| Dunfell | 3.1 | 2020 年 4 月 | LTS（直到 2024 年 4 月） |
| Zeus | 3.0 | 2019 年 10 月 | 生命周期结束（EOL） |
| Warrior | 2.7 | 2019 年 4 月 | 生命周期结束（EOL） |
| Thud | 2.6 | 2018 年 11 月 | 生命周期结束（EOL） |
| Sumo | 2.5 | 2018 年 4 月 | 生命周期结束（EOL） |
| Rocko | 2.4 | 2017 年 10 月 | 生命周期结束（EOL） |
| Pyro | 2.3 | 2017 年 5 月 | 生命周期结束（EOL） |
| Morty | 2.2 | 2016 年 11 月 | 生命周期结束（EOL） |
| Krogoth | 2.1 | 2016 年 4 月 | 生命周期结束（EOL） |
| Jethro | 2.0 | 2015 年 11 月 | 生命周期结束（EOL） |
| Fido | 1.8 | 2015 年 4 月 | 生命周期结束（EOL） |
| Dizzy | 1.7 | 2014 年 10 月 | 生命周期结束（EOL） |
| Daisy | 1.6 | 2014 年 4 月 | 生命周期结束（EOL） |
| Dora | 1.5 | 2013 年 10 月 | 生命周期结束（EOL） |
| Dylan | 1.4 | 2013 年 4 月 | 生命周期结束（EOL） |
| Danny | 1.3 | 2012 年 10 月 | 生命周期结束（EOL） |
| Denzil | 1.2 | 2012 年 4 月 | 生命周期结束（EOL） |
| Edison | 1.1 | 2011 年 10 月 | 生命周期结束（EOL） |
| Bernard | 1.0 | 2011 年 4 月 | 生命周期结束（EOL） |
| Laverne | 0.9 | 2010 年 10 月 | 生命周期结束（EOL） |

表 1.1 – Yocto 项目版本列表

# 总结

本章概述了 OpenEmbedded 项目与 Yocto 项目之间的关系，构成 Poky 的组件以及该项目的起源。下一章将介绍 Poky 工作流，包括如何下载、配置和准备 Poky 构建环境，以及如何使用 QEMU 构建并运行第一个镜像。
