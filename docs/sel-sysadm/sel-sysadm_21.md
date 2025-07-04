# 您可能喜欢的其他书籍

如果您喜欢本书，您可能对 Packt 出版的其他书籍感兴趣：

![](https://www.packtpub.com/product/mastering-linux-security-and-hardening-second-edition/9781838981778)

**Linux 安全与加固技术精通**

Donald Tevault

ISBN: 978-1-83898-177-8

+   创建具有强密码的受限用户账户

+   使用 iptables、UFW、nftables 和 firewalld 配置防火墙

+   使用不同的加密技术保护您的数据

+   加固安全外壳服务，防止安全入侵

+   使用强制访问控制保护系统免受漏洞利用

+   加固内核参数并设置内核级审计系统

+   应用 OpenSCAP 安全配置文件并设置入侵检测

+   安全配置 GRUB 2 启动加载程序和 BIOS/UEFI

![](https://www.packtpub.com/product/cybersecurity-attacks-red-team-strategies/9781838828868)

**网络安全攻击 – 红队策略**

Johann Rehberger

ISBN: 978-1-83882-886-8

+   了解与安全漏洞相关的风险

+   实施构建高效渗透测试团队的策略

+   使用知识图谱绘制主场图

+   使用索引和其他实用技术进行凭证猎取

+   获取蓝队工具的见解，提升您的红队技能

+   使用适当的数据沟通结果并影响决策者

# 留下评论 - 告诉其他读者您的看法

请通过在您购买书籍的网站上留下评论，与他人分享您对本书的想法。如果您是通过亚马逊购买本书，请在本书的亚马逊页面上留下真实的评论。这对其他潜在读者至关重要，他们可以利用您的客观意见做出购买决策；我们可以了解客户对我们产品的看法；我们的作者也能看到您对他们与 Packt 合作编写的书籍的反馈。它只需花费您几分钟时间，但对其他潜在客户、我们的作者以及 Packt 都非常有价值。谢谢！

## 目录

1.  SELinux 系统管理 第三版

1.  为什么订阅？

1.  贡献者

1.  关于作者

1.  关于审阅者

1.  Packt 正在寻找像您这样的作者

1.  前言

    1.  本书适用对象

    1.  本书涵盖内容

    1.  最大化本书价值

    1.  下载示例代码文件

    1.  代码实战

    1.  下载彩色图像

    1.  使用的约定

    1.  联系我们

    1.  评审

1.  第一章：使用 SELinux

1.  第一章：基本的 SELinux 概念

    1.  技术要求

    1.  为 Linux 提供更多安全性

        1.  介绍 Linux 安全模块（LSM）

        1.  使用 SELinux 扩展常规 DAC

        1.  限制 root 权限

        1.  减少漏洞影响

        1.  启用 SELinux 支持

    1.  标记所有资源和对象

        1.  剖析 SELinux 上下文

        1.  通过类型强制访问

        1.  通过角色授予域访问

        1.  通过用户限制角色

        1.  通过敏感性控制信息流

    1.  定义和分发策略

        1.  编写 SELinux 策略

        1.  通过模块分发策略

        1.  在策略存储中捆绑模块

    1.  区分不同的策略

        1.  支持 MLS

        1.  处理未知权限

        1.  支持无约束域

        1.  限制跨用户共享

        1.  递增策略版本

        1.  不同的策略内容

    1.  总结

    1.  问题

1.  第二章：理解 SELinux 决策和日志记录

    1.  技术要求

    1.  开关 SELinux

        1.  设置全局 SELinux 状态

        1.  切换到宽容或强制模式

        1.  使用内核启动参数

        1.  为单个服务禁用 SELinux 保护

        1.  理解 SELinux-aware 应用程序

    1.  SELinux 日志记录与审计

        1.  跟踪审计事件

        1.  调优 AVC

        1.  揭示更多日志记录

        1.  配置 Linux 审计

        1.  配置本地系统日志记录器

        1.  阅读 SELinux 拒绝信息

        1.  其他 SELinux 相关事件类型

        1.  使用 ausearch

    1.  获取拒绝帮助

        1.  使用 setroubleshoot 进行故障排除

        1.  当 SELinux 拒绝发生时发送电子邮件

        1.  使用 audit2why

        1.  与 systemd-journal 交互

        1.  运用常识

    1.  总结

    1.  问题

1.  第三章：管理用户登录

    1.  技术要求

    1.  面向用户的 SELinux 上下文

    1.  SELinux 用户与角色

        1.  列出 SELinux 用户映射

        1.  将登录映射到 SELinux 用户

        1.  为服务定制登录

        1.  创建 SELinux 用户

        1.  列出可访问的域

        1.  管理分类

    1.  处理 SELinux 角色

        1.  定义允许的 SELinux 上下文

        1.  使用 getseuser 验证上下文

        1.  使用 newrole 切换角色

        1.  通过 sudo 管理角色访问

        1.  使用 runcon 访问其他域

        1.  切换到系统角色

    1.  SELinux 与 PAM

        1.  通过 PAM 分配上下文

        1.  在宽容模式下禁止访问

        1.  多重实例化目录

    1.  总结

    1.  问题

1.  第四章：使用文件上下文和进程域

    1.  技术要求

    1.  SELinux 文件上下文简介

        1.  获取上下文信息

        1.  解释 SELinux 上下文类型

    1.  保留或忽略上下文

        1.  继承默认上下文

        1.  查询过渡规则

        1.  复制和移动文件

        1.  临时更改文件上下文

        1.  将类别放置在文件和目录上

        1.  在文件上使用多级安全

        1.  备份和恢复扩展属性

        1.  使用挂载选项设置 SELinux 上下文

    1.  SELinux 文件上下文表达式

        1.  使用上下文表达式

        1.  注册文件上下文更改

        1.  优化递归上下文操作

        1.  使用可自定义类型

        1.  编译不同的 file_contexts 文件

        1.  交换本地修改

    1.  修改文件上下文

        1.  使用 setfiles、rlpkg 和 fixfiles

        1.  重新标记整个文件系统

        1.  使用 restorecond 自动设置上下文

        1.  使用 tmpfiles 在启动时设置 SELinux 上下文

    1.  进程的上下文

        1.  获取进程上下文

        1.  过渡到域

        1.  验证目标上下文

        1.  其他支持的过渡

        1.  查询初始上下文

        1.  调整内存保护

    1.  限制过渡的范围

        1.  过渡期间的环境清理

        1.  禁用不受约束的过渡

        1.  使用 Linux 的 NO_NEW_PRIVS

    1.  类型、权限和约束

        1.  理解类型属性

        1.  查询域权限

        1.  学习约束

    1.  总结

    1.  问题

1.  第五章：控制网络通信

    1.  技术要求

    1.  控制进程间通信

        1.  使用共享内存

        1.  通过管道进行本地通信

        1.  通过 UNIX 域套接字进行通信

        1.  理解 netlink 套接字

        1.  处理 TCP、UDP 和 SCTP 套接字

        1.  列出连接上下文

    1.  Linux 防火墙和 SECMARK 支持

        1.  介绍 netfilter

        1.  实施安全标记

        1.  为数据包分配标签

        1.  过渡到 nftables

        1.  评估 eBPF

    1.  保护高速 InfiniBand 网络

        1.  直接访问内存

        1.  保护 InfiniBand 网络

        1.  管理 InfiniBand 子网

        1.  控制对 InfiniBand 分区的访问

    1.  理解带标签的网络

        1.  使用 NetLabel 进行回退标签化

        1.  基于网络接口限制流量

        1.  接受来自选定主机的对等通信

        1.  验证点对点流量

        1.  使用旧式控制

    1.  使用带标签的 IPsec 与 SELinux

        1.  设置常规 IPsec

        1.  启用带标签的 IPsec

    1.  使用 NetLabel 和 SELinux 支持 CIPSO

        1.  配置 CIPSO 映射

        1.  添加特定领域的映射

        1.  使用本地 CIPSO 定义

        1.  支持 IPv6 CALIPSO

    1.  总结

    1.  问题

1.  第六章：通过基础设施即代码编排配置 SELinux

    1.  技术要求

    1.  目标设置和策略介绍

        1.  操作的幂等性

        1.  策略和状态管理

        1.  SELinux 配置设置

        1.  设置文件上下文

        1.  从错误中恢复

        1.  比较框架

    1.  使用 Ansible 进行 SELinux 系统管理

        1.  Ansible 的工作原理

        1.  安装和配置 Ansible

        1.  创建和测试 Ansible 角色

        1.  使用 Ansible 为文件系统资源分配 SELinux 上下文

        1.  使用 Ansible 加载自定义 SELinux 策略

        1.  使用 Ansible 的即插即用 SELinux 支持

    1.  利用 SaltStack 配置 SELinux

        1.  SaltStack 的工作原理

        1.  安装和配置 SaltStack

        1.  使用 SaltStack 创建和测试我们的 SELinux 状态

        1.  使用 SaltStack 为文件系统资源分配 SELinux 上下文

        1.  使用 SaltStack 加载自定义 SELinux 策略

        1.  使用 SaltStack 的即插即用 SELinux 支持

    1.  使用 Puppet 自动化系统管理

        1.  Puppet 的工作原理

        1.  安装和配置 Puppet

        1.  使用 Puppet 创建和测试 SELinux 类

        1.  使用 Puppet 为文件系统资源分配 SELinux 上下文

        1.  使用 Puppet 加载自定义 SELinux 策略

        1.  使用 Puppet 的即插即用 SELinux 支持

    1.  利用 Chef 进行系统自动化

        1.  Chef 的工作原理

        1.  安装和配置 Chef

        1.  创建 SELinux 配方书

        1.  使用 Chef 为文件系统资源分配 SELinux 上下文

        1.  使用 Chef 加载自定义 SELinux 策略

        1.  使用 Chef 的即插即用 SELinux 支持

    1.  总结

    1.  问题

1.  第二部分：SELinux 感知平台

1.  第七章：配置特定应用程序的 SELinux 控制

    1.  技术要求

    1.  调整 systemd 服务、日志记录和设备管理

        1.  systemd 中的服务支持

        1.  使用 systemd 进行日志记录

        1.  处理设备文件

    1.  通过 D-Bus 通信

        1.  理解 D-Bus

        1.  使用 SELinux 控制服务获取

        1.  管理消息流

    1.  配置 PAM 服务

        1.  Cockpit

        1.  Cron

        1.  OpenSSH

    1.  与 Apache 一起使用 mod_selinux

        1.  介绍 mod_selinux

        1.  配置 Apache 的通用 SELinux 灵敏度

        1.  将最终用户映射到特定领域

        1.  根据源更改领域

    1.  总结

    1.  问题

1.  第八章：SEPostgreSQL – 用 SELinux 扩展 PostgreSQL

    1.  技术要求

    1.  介绍 PostgreSQL 和 sepgsql

        1.  使用 sepgsql 重新配置 PostgreSQL

        1.  创建测试账户

        1.  调整 PostgreSQL 中的 sepgsql

        1.  故障排除 sepgsql

    1.  理解 SELinux 的数据库特定对象类和权限

        1.  理解 sepgsql 权限

        1.  使用默认支持的类型

        1.  创建可信程序

        1.  使用 sepgsql 特定函数

    1.  使用 MCS 和 MLS

        1.  基于类别限制对列的访问

        1.  限制用户领域用于敏感度范围操作

    1.  将 SEPostgreSQL 集成到网络中

        1.  为远程会话创建回退标签

        1.  调优 SELinux 策略

    1.  总结

    1.  问题

1.  第九章：安全虚拟化

    1.  技术要求

    1.  理解 SELinux 安全虚拟化

        1.  介绍虚拟化

        1.  审查虚拟化风险

        1.  重用现有虚拟化域

        1.  微调虚拟化支持的 SELinux 策略

        1.  理解 sVirt 使用 MCS

    1.  增强 libvirt 与 SELinux 支持

        1.  区分共享资源与专用资源

        1.  评估 libvirt 架构

        1.  为 sVirt 配置 libvirt

        1.  更改客户机的 SELinux 标签

        1.  自定义资源标签

        1.  控制可用类别

        1.  更改存储池位置

    1.  使用 Vagrant 与 libvirt

        1.  部署 Vagrant 和 libvirt 插件

        1.  安装兼容 libvirt 的框

        1.  配置 Vagrant 框

    1.  总结

    1.  问题

1.  第十章：使用 Xen 安全模块与 FLASK

    1.  技术要求

    1.  理解 Xen 和 XSM

        1.  介绍 Xen 虚拟机监控器

        1.  安装 Xen

        1.  创建无特权客户机

        1.  理解 Xen 安全模块

    1.  运行启用 XSM 的 Xen

        1.  重建带有 XSM 支持的 Xen

        1.  使用 XSM 标签

        1.  操作 XSM

    1.  应用自定义 XSM 策略

    1.  总结

    1.  问题

1.  第十一章：增强容器化工作负载的安全性

    1.  技术要求

    1.  使用 SELinux 和 systemd 的容器支持

        1.  初始化 systemd 容器

        1.  使用特定的 SELinux 上下文

        1.  使用 machinectl 促进容器管理

    1.  配置 podman

        1.  选择 podman 而不是 Docker

        1.  使用 SELinux 与容器

        1.  更改容器的 SELinux 域

        1.  使用 udica 创建自定义域

        1.  使用 SELinux 布尔值切换 container_t 特权

        1.  调整容器托管环境

    1.  利用 Kubernetes 的 SELinux 支持

        1.  配置带有 SELinux 支持的 Kubernetes

        1.  为 pod 设置 SELinux 上下文

    1.  总结

    1.  问题

1.  第三部分：策略管理

1.  第十二章：调整 SELinux 策略

    1.  技术要求

    1.  使用 SELinux 布尔值

        1.  列出 SELinux 布尔值

        1.  更改布尔值

        1.  检查布尔值的影响

    1.  处理策略模块

        1.  列出策略模块

        1.  加载和移除策略模块

    1.  替换和更新现有策略

        1.  使用 audit2allow 创建策略

        1.  使用合理的模块名称

        1.  使用 audit2allow 生成参考策略风格模块

        1.  构建参考策略风格模块

        1.  构建传统风格模块

        1.  替换默认的分发策略

    1.  总结

    1.  问题

1.  第十三章：分析策略行为

    1.  技术要求

    1.  执行单步分析

        1.  使用不同的 SELinux 策略文件

        1.  显示策略对象信息

        1.  理解 sesearch

        1.  查询允许规则

        1.  查询类型转换规则

        1.  查询其他类型规则

        1.  查询与角色相关的规则

        1.  使用 apol 浏览

        1.  使用 apol 工作区

    1.  调查域转换

        1.  使用 apol 进行域转换分析

        1.  使用 sedta 进行域转换分析

        1.  使用 sepolicy 进行域转换分析

    1.  分析信息流

        1.  使用 apol 进行信息流分析

        1.  使用 seinfoflow 进行信息流分析

        1.  使用 sepolicy communicate 进行简单的信息流分析

    1.  比较策略

        1.  使用 sediff 比较策略

    1.  总结

    1.  问题

1.  第十四章：处理新应用程序

    1.  技术要求

    1.  无任何限制地运行应用程序

        1.  理解无约束域的工作原理

        1.  将新应用程序作为无约束域运行

        1.  扩展无约束域

        1.  将域标记为宽松模式

    1.  使用沙箱化的应用程序

        1.  理解 SELinux 沙箱

        1.  使用沙箱命令

    1.  为新应用程序分配常见策略

        1.  理解领域复杂性

        1.  在特定策略中运行应用程序

    1.  扩展生成的策略

        1.  理解生成策略的局限性

        1.  介绍 sepolicy generate

        1.  使用 sepolicy generate 生成策略

    1.  总结

    1.  问题

1.  第十五章：使用参考策略

    1.  技术要求

    1.  引入参考策略

        1.  浏览策略

        1.  构建策略模块结构

    1.  使用和理解策略宏

        1.  使用单类权限组

        1.  调用权限组

    1.  创建应用程序级策略

        1.  构建面向网络的服务策略

        1.  解决用户应用程序问题

    1.  添加用户级策略

    1.  使用支持工具获取帮助

        1.  使用 selint 验证代码

        1.  本地查询接口和宏

    1.  总结

    1.  问题

1.  第十六章：使用 SELinux CIL 开发策略

    1.  技术要求

    1.  引入 CIL

        1.  将 .pp 文件转换为 CIL

        1.  理解 CIL 语法

    1.  创建精细粒度的定义

        1.  根据角色或类型

        1.  定义新的端口类型

        1.  向策略添加约束

    1.  构建完整的应用程序策略

        1.  使用命名空间

        1.  通过属性分配扩展策略

        1.  添加入口点信息

        1.  逐步扩展策略

        1.  引入权限集

        1.  添加宏

    1.  总结

    1.  问题

1.  评估

    1.  第一章

    1.  第二章

    1.  第三章

    1.  第四章

    1.  第五章

    1.  第六章

    1.  第七章

    1.  第八章

    1.  第九章

    1.  第十章

    1.  第十一章

    1.  第十二章

    1.  第十三章

    1.  第十四章

    1.  第十五章

    1.  第十六章

1.  你可能会喜欢的其他书籍

    1.  留下评论 - 让其他读者知道你的想法

## 地标

1.  封面

1.  目录
