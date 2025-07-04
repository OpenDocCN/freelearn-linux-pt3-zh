24. 使用 virsh 命令行工具管理 KVM

在前几章中，我们已经讲解了在 CentOS 8 上安装和配置基于 KVM 的来宾操作系统所需的步骤。本章将介绍 virsh 工具中一些之前未涉及的额外功能，以及如何从命令行使用它来管理基于 KVM 的来宾操作系统。

24.1 virsh Shell 和命令行

virsh 工具既是一个命令行工具，也是一个交互式 shell 环境。在命令行模式下使用时，只需在命令提示符下输入命令，并附上适当的参数以执行相应任务。

要将选项作为命令行参数使用，请在终端命令提示符下输入，如下示例所示：

# 第二十三章：virsh <option>

virsh 工具在 shell 模式下使用时，提供了一个交互式环境，从中可以发出一系列命令。

要在 virsh shell 中运行命令，请运行以下命令：

# virsh

欢迎使用 virsh，虚拟化交互式终端。

输入：‘help’以获取命令帮助

输入‘quit’退出

virsh #

在 virsh # 提示符下，输入您希望运行的选项。例如，下面的 virsh 会话列出了当前的虚拟机，启动了一个名为 FedoraVM 的虚拟机，然后获取另一个列表以验证虚拟机是否正在运行：

# virsh

欢迎使用 virsh，虚拟化交互式终端。

输入：‘help’以获取命令帮助

输入‘quit’退出

virsh # list

Id 名称 状态

----------------------------------------------------

8 RHEL8VM 正在运行

9 CentOS7VM 正在运行

virsh # start FedoraVM

域 FedoraVM 已启动

virsh # list

Id 名称 状态

----------------------------------------------------

8 RHEL8VM 正在运行

9 CentOS7VM 正在运行

10 FedoraVM 正在运行

virsh#

virsh 工具支持多种命令，您可以使用 help 选项获取完整的命令列表：

# virsh help

通过在 help 指令后指定命令，可以获得每个命令语法的更多详细信息：

# virsh help restore

名称

restore - 从文件中恢复一个域到保存的状态

概述

restore <file> [--bypass-cache] [--xml <string>] [--running] [--paused]

描述

恢复一个域。

选项

[--file] <string> 要恢复的状态

--bypass-cache 避免恢复时使用文件系统缓存

--xml <string> 文件名，包含目标的更新 XML

--正在恢复域到运行状态

--paused 恢复域到暂停状态

在本章的剩余部分，我们将更详细地探讨一些这些命令。

24.2 列出来宾系统状态

CentOS 8 虚拟化主机上的来宾系统状态可以随时通过 virsh 工具的 list 选项查看。例如：

# virsh list

上述命令将显示包含每个来宾的行，类似如下内容：

virsh # list

Id 名称 状态

----------------------------------------------------

8 RHEL8VM 正在运行

9 CentOS7VM 正在运行

10 FedoraVM 正在运行

24.3 启动来宾系统

可以使用 virsh 工具配合 start 选项启动客户操作系统，并在后面跟上要启动的客户操作系统名称。例如：

# virsh start myGuestOS

24.4 关闭客户系统

virsh 工具的关闭选项，顾名思义，用于关闭客户操作系统：

# virsh shutdown guestName

请注意，关闭选项允许客户操作系统在收到关闭指令时执行有序关机。要立即停止客户操作系统，可以使用 destroy 选项（但会伴随文件系统损坏和数据丢失的风险）：

# virsh destroy guestName

24.5 暂停和恢复客户系统

可以使用 virsh 工具的 suspend 和 resume 选项暂停和恢复客户系统。例如，要暂停特定的系统：

# virsh suspend guestName

同样，要恢复暂停的系统：

# virsh resume guestName

请注意，如果主机系统重新启动，挂起的会话将会丢失。此外，挂起的系统仍然会占用内存。为了保存会话，使其不再占用内存并且可以恢复到原始状态（即使在重启后），需要保存并恢复虚拟机。

24.6 保存和恢复客户系统

可以使用 virsh 工具保存并恢复正在运行的客户操作系统。当系统被保存时，客户操作系统的当前状态将写入磁盘并从系统内存中移除。保存的系统可以在任何时候恢复（包括主机系统重启后）：

要保存客户：

# virsh save guestName path_to_save_file

要恢复保存的客户操作系统会话：

# virsh restore path_to_save_file

24.7 重启客户系统

要重启客户操作系统：

# virsh reboot guestName

24.8 配置分配给客户操作系统的内存

要配置分配给客户操作系统的内存，请使用 virsh 命令的 `setmem` 选项。例如，以下命令将分配给客户系统的内存减少到 256Mb：

# virsh setmem guestName 256

请注意，接受的内存设置必须在当前域可用的内存范围内。可以使用`setmaxmem`选项增加内存。

24.9 总结

virsh 工具提供了广泛的选项，用于创建、监控和管理客户虚拟机。如本章所述，该工具可以在命令行或交互模式下使用。
