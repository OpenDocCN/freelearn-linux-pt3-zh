# 15

# 启动我们自定义的嵌入式 Linux

是时候了！我们已经准备好启动我们的自定义嵌入式 Linux，因为我们已经掌握了所需的概念，并对 Yocto 项目和 Poky 有了足够的了解。在本章中，我们将实践迄今为止学到的内容，使用外部 BSP 层与 Poky 一起生成镜像，并使用 SD 卡启动以下机器：

+   BeagleBone Black

+   Raspberry Pi 4

+   VisionFive

本章中的概念适用于任何其他开发板，只要该厂商提供了一个可以与 Yocto 项目一起使用的 BSP 层。

# 寻找合适的 BSP 层

在 *第十一章*《探索外部层》中，我们了解到 Yocto 项目允许将其元数据分散到不同的层中。它组织这些元数据，使我们可以选择将哪个具体的元层添加到我们的项目中。

找到开发板的 BSP 层的方式各不相同，但通常我们可以通过访问[`layers.openembedded.org`](https://layers.openembedded.org)来找到。我们可以搜索机器名称，网站会在其数据库中查找包含该层的内容。

## 回顾影响硬件使用的各个方面

本章使用的开发板维护良好且易于使用。然而，使用其他开发板也是一个有效的选择，但实际效果可能有所不同。

当我们选择一块开发板时，第一步是验证其软件支持的质量。低级组件包括以下内容：

+   引导加载程序（如 U-Boot、GRUB 或 systemd-boot）

+   Linux 内核（包括其他必需的驱动程序，如 GPU 或 WiFi）

+   硬件加速所需的用户空间软件包

这些都是至关重要的，但并不是唯一需要考虑的方面。Yocto 项目中的集成通常以 BSP 层的形式出现，减少了开发板使用时的摩擦，因为它通常提供以下内容：

+   可重用的磁盘分区布局（例如，WIC `.wks` 模板）

+   开箱即用的机器定义

+   为硬件加速集成的用户空间软件包（开箱即用）

软件启用的成熟度以及 Yocto 项目 BSP 对使用开发板时的摩擦和使用 Poky 时的开箱体验有着重要影响。

## 看一看广泛使用的 BSP 层

在本章中，我们将看到一个广泛使用的 BSP 层列表。请注意，这不是一个完整的列表，也不是一个权威的列表。我们希望能帮助你在身边有特定厂商的开发板时，方便地找到所需的层。以下是按字母顺序排列的列表：

+   *全志科技*: 这有`meta-allwinner`层

+   *AMD*: 这有`meta-amd`层

+   *英特尔*: 这有`meta-intel`层

+   *NXP*: 这有`meta-freescale`和`meta-freescale-3rdparty`层

+   *树莓派*: 这有`meta-raspberrypi`层

+   *RISC-V*: 这有`meta-riscv`层

+   *德州仪器*: 这有`meta-ti`层

在接下来的章节中，我们将开始使用示例开发板。

# 使用物理硬件

为了更好地探索 Yocto 项目的功能，拥有一个真实的开发板是很有帮助的，这样我们可以享受启动定制嵌入式系统的体验。为此，我们尽力收集了最广泛可用的开发板，以便你拥有其中之一的可能性更高。

接下来的章节将介绍以下开发板的操作步骤：

+   *BeagleBone Black*：BeagleBone Black 是一个基于社区的开发板，拥有来自全球的成员。更多信息请访问[`beagleboard.org/black/`](https://beagleboard.org/black/)。

+   *Raspberry Pi 4*：全球最著名的 ARM64 开发板，拥有最广泛的社区支持。查看更多详情，请访问[`www.raspberrypi.org/`](https://www.raspberrypi.org/)。

+   *VisionFive*：世界上首个旨在运行 Linux 的经济型 RISC-V 开发板。查看更多详情，请访问[`www.starfivetech.com/en`](https://www.starfivetech.com/en)。

所有列出的开发板都由非营利性教育和辅导组织维护，使得社区成为探索嵌入式 Linux 世界的肥沃土壤。下表总结了这些开发板及其主要特点：

| **开发板版本** | **特点** |
| --- | --- |
| BeagleBone Black | TI AM335x（单核）512 MB 内存 |
| Raspberry Pi 4 | Broadcom BCM2711 64 位 CPU（四核）1 GB 至 8 GB 内存 |
| VisionFive | U74 双核 8 GB 内存 |

表 15.1 – 涵盖的开发板硬件规格

在接下来的章节中，我们将为每个建议的开发板烘焙和启动 Yocto 项目映像。建议你只阅读适用于自己所拥有的开发板的章节。确保参考开发板的文档，以便了解如何为操作做好准备。

# BeagleBone Black

在接下来的两个章节中，我们将介绍为*BeagleBone* *Black*开发板烘焙和启动映像的步骤。

## 为 BeagleBone Black 构建映像

要使用此开发板，我们可以依赖`meta-yocto-bsp`层，该层默认包含在 Poky 中。可以通过[`git.yoctoproject.org/meta-yocto/tree/meta-yocto-bsp?h=kirkstone`](https://git.yoctoproject.org/meta-yocto/tree/meta-yocto-bsp?h=kirkstone)访问该 meta 层。

为了创建源结构，请使用以下命令行下载 Poky：

```
git clone git://git.yoctoproject.org/poky -b kirkstone
```

完成此步骤后，我们需要创建用于构建的目录。可以使用以下命令行来实现：

```
source oe-init-build-env build
```

在我们正确设置了`build`目录和 BSP 层之后，就可以开始构建了。在`build`目录中，我们必须运行以下命令：

```
MACHINE=beaglebone-yocto bitbake core-image-full-cmdline
```

`MACHINE`变量可以根据我们希望使用的开发板进行更改，或者在`build/conf/local.conf`中设置。

## 启动 BeagleBone Black

构建过程完成后，映像文件将保存在`build/tmp/deploy/images/beaglebone-yocto/`目录中。我们想要使用的文件是`core-image-full-cmdline-beaglebone-yocto.wic`。

确保指向正确的设备，并再次确认不要在硬盘上进行写入操作。

要将`core-image-full-cmdline`镜像复制到 SD 卡上，我们应该使用`dd`工具，如下所示：

```
sudo dd if=core-image-full-cmdline-beaglebone-yocto.wic of=/dev/<media>
```

将内容复制到 SD 卡后，插入 SD 卡插槽，连接 HDMI 电缆，然后开机。系统应该能正常启动。

注意

BeagleBone Black 的启动序列首先会尝试从 eMMC 启动，只有当 eMMC 启动失败时才会尝试从 SD 卡启动。开机时按住**USER**/**BOOT**按钮会暂时改变启动顺序，确保从 SD 卡启动。要进一步为您的板卡定制这些说明，请参考[`www.beagleboard.org/black`](http://www.beagleboard.org/black)的文档。

# Raspberry Pi 4

在接下来的两个章节中，我们将介绍为*Raspberry Pi* *4*板卡制作和启动镜像的步骤。

## 为 Raspberry Pi 4 制作镜像

要将此板卡支持添加到我们的项目中，我们需要包含`meta-raspberrypi`元层，它是支持 Raspberry Pi 板卡（包括 Raspberry Pi 4，但不限于它）的 BSP 层。可以通过[`git.yoctoproject.org/cgit.cgi/meta-raspberrypi/log/?h=kirkstone`](http://git.yoctoproject.org/cgit.cgi/meta-raspberrypi/log/?h=kirkstone)访问该元层。

要创建源结构，请使用以下命令行下载 Poky：

```
git clone git://git.yoctoproject.org/poky -b kirkstone
```

完成此步骤后，我们需要创建一个用于构建的`build`目录并添加 BSP 层。可以使用以下命令行进行操作：

```
source oe-init-build-env build
bitbake-layers layerindex-fetch meta-raspberrypi
```

在我们正确设置了`build`目录和 BSP 层后，可以开始构建。在`build`目录下，我们必须运行以下命令：

```
MACHINE=raspberrypi4 bitbake core-image-full-cmdline
```

`MACHINE`变量可以根据我们想要使用的板卡进行修改，或者在`build/conf/local.conf`中设置。

## 启动 Raspberry Pi 4

在构建过程完成后，镜像文件将位于`build/tmp/deploy/images/raspberrypi4/`目录下。我们需要使用的文件是`core-image-full-cmdline-raspberrypi4.wic.bz2`。

确保指向正确的设备，并仔细检查不要写入硬盘。

要将`core-image-full-cmdline`镜像复制到 SD 卡上，我们应该使用`dd`工具，如下所示：

```
bzcat core-image-full-cmdline-raspberrypi4.wic.bz2 | sudo dd of=/dev/<media>
```

将内容复制到 SD 卡后，插入 SD 卡插槽，连接 HDMI 电缆，然后开机。系统应该能正常启动。

# VisionFive

在接下来的两个章节中，我们将介绍为*VisionFive*板卡制作和启动镜像的步骤。

## 为 VisionFive 制作镜像

要将此板卡支持添加到我们的项目中，我们需要包含`meta-riscv`元层，它是支持基于 RISC-V 的板卡（包括 VisionFive，但不限于它）的 BSP 层。可以通过[`github.com/riscv/meta-riscv/tree/kirkstone`](https://github.com/riscv/meta-riscv/tree/kirkstone)访问该元层。

要创建源结构，请使用以下命令行下载 Poky：

```
git clone git://git.yoctoproject.org/poky -b kirkstone
```

完成此步骤后，我们必须创建一个用于构建的 `build` 目录，并添加 BSP 层。我们可以使用以下命令行来实现：

```
source oe-init-build-env build
bitbake-layers layerindex-fetch meta-riscv
```

在我们正确设置了 `build` 目录和 BSP 层后，就可以开始构建了。在 `build` 目录中，我们必须调用以下命令：

```
MACHINE=visionfive bitbake core-image-full-cmdline
```

`MACHINE` 变量可以根据我们要使用的板卡进行更改，或者在 `build/conf/local.conf` 中设置。

## 启动 VisionFive

构建过程结束后，镜像将出现在 `build/tmp/deploy/images/visionfive/` 目录中。我们要使用的文件是 `core-image-full-cmdline-visionfive.wic.gz`。

确保指向正确的设备，并仔细检查不要写入你的硬盘。

要将 `core-image-full-cmdline` 镜像复制到 SD 卡，我们应该使用 `dd` 工具，命令如下：

```
zcat core-image-full-cmdline-visionfive.wic.gz | sudo dd of=/dev/<media>
```

将内容复制到 SD 卡后，插入 SD 卡槽，连接 HDMI 电缆，并开机启动。

注意

VisionFive 没有默认的启动目标，需要手动干预才能启动。请使用以下命令在 U-Boot 提示符下通过串口控制台进行操作：

**setenv bootcmd “****run distro_bootcmd”**

**saveenv**

**启动**

命令 `saveenv` 是可选的，用于使新配置持久化，以便在重启后能够直接使用。

查看如何获取串口控制台，请参阅 *快速入门* *指南* ([`doc-en.rvspace.org/VisionFive/Quick_Start_Guide/`](https://doc-en.rvspace.org/VisionFive/Quick_Start_Guide/))。

# 采取下一步行动

呼！我们完成了！现在你应该掌握 Yocto 项目构建系统的基本知识，并能够扩展你在其他领域的知识。我们尝试涵盖了使用 Yocto 项目的最常见日常任务。有一些事情你可能需要练习：

+   创建 `bbappend` 文件以应用补丁或对配方进行其他更改

+   创建你的自定义镜像

+   更改 Linux 内核配置文件（`defconfig`）

+   更改 BusyBox 配置并包括配置片段以在层中添加或删除功能

+   为软件包添加新的配方

+   创建一个包含特定产品机器、配方和镜像的产品层

记住，源代码是最终的知识来源，所以要善用它。

在查找如何做某件事时，找到一个相似的配方可以节省你测试不同方法解决问题的时间。

最终，你很可能会发现自己处于修复或增强 OpenEmbedded Core、一个 meta 层或一个 BSP 的位置。所以，不要害怕——提交补丁，并将反馈和修改请求视为学习和改进解决问题方式的机会。

# 总结

我们学习了如何发现我们项目中需要使用的板卡的 BSP。通过添加外部 BSP 层并在实际板卡上使用这些层生成镜像，我们巩固了 Yocto 项目的知识。我们还巩固了学习 Yocto 项目其他相关方面所需的背景信息。

在下一章中，我们将探讨如何通过使用 QEMU 来加速产品开发，避免每次开发周期都依赖硬件。
