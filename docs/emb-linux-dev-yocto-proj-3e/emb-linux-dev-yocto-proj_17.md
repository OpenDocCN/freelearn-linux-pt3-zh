

# 第十七章：最佳实践

本章旨在提供我们（作者）多年来在嵌入式设备和嵌入式 Linux 开发方面的个人经验。我们收集了一些常常被低估或完全忽视的方面，以便为你下一个项目提供灵感。

我们将本章分为两个独立部分，一部分涉及与 Yocto 项目相关的指导原则，另一部分涉及项目的一些更一般性的方面。这样，你就不需要按照特定顺序学习这两个部分。

# Yocto 项目遵循的指导原则

本节旨在收集一些关于 Yocto 项目元数据和项目组织的指导原则，这些原则可以让我们在短期和长期维护方面更加轻松。

## 管理层次

随着我们在产品开发中的进展，我们自然会使用多个仓库来满足我们面临的需求。跟踪这些仓库是一个复杂的挑战，因为我们需要做以下几件事：

+   确保我们将来能够复现之前的构建

+   允许多个团队成员在同一代码库中工作

+   使用持续集成工具验证我们所做的更改

+   避免我们使用的层次中出现细微的变化

这些目标令人畏惧，但目前已有一些工具在使用，并且采取了不同的策略来克服这些挑战。

最简单的解决方案使用`image-buildinfo`类（[`docs.yoctoproject.org/4.0.4/ref-manual/classes.html#image-buildinfo-bbclass`](https://docs.yoctoproject.org/4.0.4/ref-manual/classes.html#image-buildinfo-bbclass)），该类默认将一个包含构建信息和层次修订的纯文本文件写入目标文件系统的`${sysconfdir}/buildinfo`。一些工具已经被开发出来，可以帮助这一过程。以下是讨论这些工具的内容：

+   Google 为 Android 开发开发了**repo**工具（[`source.android.com/docs/setup/download#repo`](https://source.android.com/docs/setup/download#repo)）。它已经被其他项目采纳。repo 的一个关键特点是，它需要一些工具与基于 Yocto 项目的项目集成，以自动化构建目录和环境配置。可以参考*O.S. Systems 嵌入式 Linux 项目*（[`github.com/OSSystemsEmbeddedLinux/ossystems-embedded-linux-platform`](https://github.com/OSSystemsEmbeddedLinux/ossystems-embedded-linux-platform)）作为使用 repo 的灵感来源。

+   Siemens 开发了**kas**（[`github.com/siemens/kas`](https://github.com/siemens/kas)），提供了一种简便机制，用于下载源代码、自动化构建目录和环境配置等。

+   Garmin 开发了**Whisk** ([`github.com/garmin/whisk`](https://github.com/garmin/whisk))，用于通过 OpenEmbedded 和 Yocto 项目管理复杂的产品配置。其主要特点包括单一源树、多个配置轴、多个产品构建、隔离层配置等。

+   Agilent 开发了**Yocto Buddy** ([`github.com/Agilent/yb`](https://github.com/Agilent/yb))。该设计旨在简化设置，并保持基于 Yocto 项目的环境同步。Yocto Buddy 的灵感来源于前面提到的所有工具，当前仍处于早期开发阶段。

这是现有工具的一个子集，不应被视为完整列表。理想情况下，您应该在做出决定前先使用它们，因为选择取决于项目用例和团队的专业知识。

## 避免创建过多的层

Yocto 项目的一个显著优势是，它能够使用和创建多个层。这使我们能够做到以下几点：

+   重用半导体供应商的 BSP 层

+   通过共享可重用的模块来减少工作重复，从而支持新的或特定的应用程序、编程语言等。

然而，在开发项目或一组产品时，创建多个层可能没有成效。例如，以下情况中开发仅包含 BSP 的层是有意义的：

+   在**系统模块**（**SoM**）供应商的案例中，板卡即是产品

+   当外部需要访问某个层时，然而我们希望限制非 BSP 源的访问

使用单一层来为产品甚至公司提供支持，有许多优势，例如：

+   促进可重用组件的开发，例如多个产品共享的`packagegroup`开发工具或网络工具包

+   降低由于为特定产品或板卡做更改而引发的意外副作用的风险

+   增加跨多个产品的 bug 修复复用，并重用 BSP 低级组件，如 Linux 内核或引导加载程序

+   在多个产品之间推动标准化，减少新团队成员的学习曲线

是否使用一个或多个层取决于多个方面；然而，我们建议从简单开始，未来如有需要再拆分层。

## 为新版本的 Yocto 项目发布准备产品元数据

随着我们产品的增长，元数据也在增长，并且需要良好的组织结构。以下是在产品开发过程中常见的一些用例：

+   需要回溯一个新配方版本，以修复 bug 或增加新特性

+   Yocto 项目配方中尚未提供缺失的包配置或 bug 修复

我们使用两个配方目录来组织这类内容：

+   `recipes-backport`: 从新的 Yocto 项目版本回溯的配方

+   `recipes-staging`: 新的配方或`bbappend`文件，添加缺失的包配置或 bug 修复

我们持续将新的配方或错误修复从`recipes-staging`发送到相应的上游项目（例如，OpenEmbedded Core）。然后，当补丁被接受后，我们将此更改从`recipes-staging`移动到`recipes-backport`目录。这种方法使我们能够跟踪待上游提交的任务，并轻松将我们的元层升级到新的 Yocto Project 版本。此外，我们可以快速处理回移目录并将其删除。

## 创建您的自定义发行版

在使用 Yocto Project 时，我们通常会在`build/conf/local.conf`中添加许多配置。然而，正如书中所讨论的，这样做不好，因为它不在源代码管理中，并且可能在开发者之间有所不同。使用自定义发行版有许多好处，以下是其中一些重点：

+   允许多个开发者之间的一致使用

+   提供清晰的视图，展示我们与基础发行版（例如`poky`）相比所使用的不同`DISTRO_FEATURES`

+   提供一个集中地点，让我们可以全局查看产品所需的所有配方配置，从而减少配置配方所需的`bbappend`文件数量（例如，`PACKAGECONFIG:pn-<myrecipe>:append = "` `myfeature"`）

除了那些更技术性的方面，使用自定义发行版还允许对 SDK 或其他 Yocto Project 生成的工件进行适当的品牌化。

我们在*第十二章*的*使用自定义发行版*部分中学会了如何创建自定义发行版，*创建* *自定义层*。

## 避免为您的产品重复使用现有的镜像

镜像是所有内容汇集的地方。当我们开发产品时，出于多种原因，重要的是最小化镜像中安装的包的数量：

+   减少 rootfs 大小

+   减少构建时间

+   减少需要处理的许可证数量

+   降低安全漏洞的攻击面

一个典型的起点是将`core-image-base.bb`文件复制到我们的自定义层，并命名为`myproduct-image.bb`，然后对其进行扩展，添加我们需要的产品镜像内容。此外，我们创建一个名为`myproduct-image-dev.bb`的镜像，用于开发过程中，并确保它依赖于`myproduct-image.bb`以及仅用于开发的工件，避免代码重复。这样，我们就有了两个镜像，一个用于生产，一个用于开发，但它们共享相同的核心特性和包。

## 标准 SDK 通常被低估

应用程序开发意味着一个互动过程，主要是因为我们通常会持续构建应用程序，直到实现我们所期望的目标。这个用例不太适合 Yocto Project，主要是由于以下原因：

+   每次开始构建配方时，它都会丢弃之前的构建对象

+   部署应用程序或镜像所需的时间更长

+   缺乏在 IDE 环境中的适当集成

对于一些主题，有一些替代方案，例如使用`devtool`来重用构建对象并帮助部署应用程序。我们在《使用 devtool 部署到目标》和《使用 devtool 构建配方》章节中看到了如何使用`devtool`，但开发体验仍然很繁琐。

对于应用程序和其他组件（如 Linux 内核和引导程序）的开发，使用标准 SDK 仍然更可取。这样我们可以专注于更快的开发，推迟或并行化 Yocto 项目的集成任务。

## 避免对 Linux 内核和引导程序进行过多的补丁修改

在嵌入式 Linux 开发中，内核和引导程序的补丁需求是内在的，因为我们很少在没有任何更改的情况下使用硬件。这些组件的修改程度与您的硬件设计有关，例如：

+   使用**单板计算机**（**SBC**），变更数量应尽量少

+   在使用**系统级模块**（**SOM**）与自定义基板时，变更数量可能会因供应商基板硬件设计的修改数量而异

+   最终，使用自定义硬件设计意味着开发自定义 BSP，因此会有大量修改

这些并非一成不变。例如，考虑使用 SBC 启动项目。后来，我们发现供应商没有提供良好的参考 BSP，因此 BSP 的修改数量和工作量将大幅增加。

当我们有小的变更时，最好将这些变更作为补丁文件添加到组件配方中处理。但是当维护组件的工作量增加时，有必要将这些变更保留在单独的组件分支中。使用存储库分支具有以下优势：

+   变更历史记录

+   不同的开发和生产分支或标签

+   与其他提供者合并的可能性

+   这允许使用更简单的配方，因为我们不需要进行个别的补丁维护

总之，我们应该使用适合项目的策略。最终会有所变化，但采用正确的方法可以减少支持所用硬件的总体工作量。

## 避免使用 AUTOREV 作为 SRCREV

在开发产品时通常会将`AUTOREV`作为`SRCREV`。我们必须交互式地更改代码，并在 Yocto 项目中尝试该代码。话虽如此，这也伴随着几个缺点：

+   由于每次重建映像时都可能使用不同的配方修订版，因此很难重现先前的构建。

+   当 BitBake 使特定配方的缓存无效时，才会应用`AUTOREV`值。这发生在我们修改配方本身或更改触发 BitBake 缓存重建的任何`.conf`文件时。

这些缺点使得`AUTOREV`非常脆弱，而其他替代方案可以更一致地覆盖交互式代码更改。通常使用`devtool`，因为它允许我们直接在工作区中更改代码，并强制食谱使用此作为源代码。另一个替代方法是使用`externalsrc.bbclass`类（[`docs.yoctoproject.org/4.0.4/index.html#ref-classes-externalsrc`](https://docs.yoctoproject.org/4.0.4/index.html#ref-classes-externalsrc)），它允许我们配置一个食谱，以使用一个目录作为构建源。

## 创建软件物料清单

Poky 构建系统可以描述镜像中使用的所有组件，包括每个软件组件的许可证。这个描述会生成一个**软件物料清单**（**SBOM**），使用**软件包数据交换**（**SPDX**）标准（[`spdx.dev/`](https://spdx.dev/)）。使用 SPDX 格式的好处是能够利用现有工具，从而实现额外的自动化，这是使用 Poky 的标准许可证输出格式无法做到的。

SBOM 对确保开源许可证合规性至关重要。然而，SBOM 默认不会生成。你可以参考*《Yocto 项目开发任务手册》*中的*创建软件物料清单*部分（[`docs.yoctoproject.org/4.0.4/dev-manual/common-tasks.html#creating-a-software-bill-of-materials`](https://docs.yoctoproject.org/4.0.4/dev-manual/common-tasks.html#creating-a-software-bill-of-materials)）。

# 通用项目的指南

本节讨论了一些与项目相关的指南，帮助减少一般项目风险，避免常见的陷阱。

## 持续监控项目的许可证约束

根据我们所从事的项目，许可证合规性可能是一个大问题，也可能是一个小问题。一些项目有非常严格的许可证约束，例如以下内容：

+   无法使用 GPLv3 发布的软件

+   项目特定知识产权的复制左传播

+   公司范围内的许可证约束

建议在项目开始时就启动这一过程，从而减少整个项目中的返工量。然而，项目的许可证约束和项目组件的许可证可能会发生变化，这要求我们持续监控许可证合规性。

## 安全问题可能会危害你的项目

在我们这个超连接的时代，每个连接的设备都是潜在的安全攻击目标。作为嵌入式设备开发人员，我们应当为创建一个更安全的环境做出贡献。我们应该做以下几点：

+   扫描我们的嵌入式 Linux 软件以查找已知的安全漏洞

+   监控关键软件的安全修复

+   实施修复现场设备的过程

我们可以使用 Yocto 项目的基础设施，正如*《Yocto 项目开发任务手册》*中的*检查漏洞*部分所讨论的那样([`docs.yoctoproject.org/4.0.4/dev-manual/common-tasks.html#checking-for-vulnerabilities`](https://docs.yoctoproject.org/4.0.4/dev-manual/common-tasks.html#checking-for-vulnerabilities))，来扫描我们配方中的已知**常见漏洞和暴露**(**CVE**)信息。我们不应仅限于此，因为我们的 BSP 组件也可能需要安全修复，而 BSP 供应商通常会忽视这些修复。不过，过度谨慎的程度取决于项目的细分领域。

## 不要低估维护成本

起初，上游我们的更改可能看起来并不具备战略意义，原因如下：

+   上游工作使用资源来适应修改

+   上游审查反馈可能需要额外的互动和返工

+   需要完成与产品无直接关联的开发工作

通常，开发和管理团队低估了维护的总成本。但不幸的是，这往往是项目中最昂贵的部分，因为它会持续多年。将我们的更改上游到相应的项目，可以让我们做到以下几点：

+   避免多年来的工作重复

+   减少升级新 Yocto 项目版本时的摩擦

+   接受关于我们正在上游的更改的关键性和建设性反馈

+   减少与安全更新和漏洞修复相关的工作量

+   减少我们需要维护的代码量

上游工作是持续性的。每当我们添加一个新特性时，我们可能会增加代码与上游之间的差距。因此，我们可以推迟上游工作，但当你开始更新到下一个 Yocto 项目版本时，上游工作的成本将成倍增加。

## 尽早解决项目的风险点和限制

由于软件和硬件必须协同工作，因此有些方面直接依赖于我们的硬件设计。为了降低项目风险，我们应该尽可能预见到许多关键的软件和硬件需求，以便能够验证一些方面，例如：

+   我们打算使用的内存量是否足够或过多？

+   硬件使用的功率是否足够满足我们的限制？

+   目标 GPU 是否能够渲染我们需要的动画？

+   所有计划的外设设备是否都已准备好可用的 Linux 内核驱动，还是我们需要规划开发这些驱动？

前述问题可以通过使用参考板或已准备好的 BSP 板来回答。这使我们能够在无需设计自定义硬件的情况下，生产出**最小可行产品**(**MVP**)。在验证项目的风险和限制后，这些板仍然是宝贵的资产，用于以下内容：

+   继续开发我们的软件，直到自定义板和 BSP 准备好使用

+   作为与我们自定义设计进行比较的基础

+   作为验证某个错误是否仅限于我们定制板和 BSP 的参考

考虑到我们可以使用参考板或知名板开发软件，我们应尽可能推迟设计定制板。推迟设计为我们提供了改变项目多个方面的自由，例如由于特定驱动程序需要更换外设，甚至在应用程序和功能成熟后更改计划中的 CPU 和内存能力。

当我们最终决定使用定制设计时，应该尽量保持它与我们选择的参考板相似。但当然，有时我们需要偏离参考设计。然而，这样做有引入设计问题和增加定制 BSP 成本的风险。

# 摘要

呼！在这一最终章节中，您已了解一系列作者在实际项目中使用的最佳实践。我们希望这些内容能为您在规划下一个项目时提供一些值得考虑的要点。

在本书中，我们已经涵盖了必要的背景知识，以便您能够独立学习 Yocto 项目的其他任何方面。因此，您现在对当您请求 BitBake 构建食谱或镜像时发生的幕后过程有了大致了解。从现在开始，您已经准备好释放思维，尝试新事物。球现在在您这一方——真正的乐趣才刚刚开始！
