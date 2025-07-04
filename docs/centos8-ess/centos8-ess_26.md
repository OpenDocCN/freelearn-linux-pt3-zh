28\. 配置 CentOS 8 Postfix 邮件服务器

除了作为 Web 服务器，电子邮件是 CentOS 8 系统的主要用途之一，特别是在商业环境中。鉴于电子邮件的重要性和普及，许多人对 Linux 系统上电子邮件结构的复杂性感到惊讶，这种复杂性往往会让 CentOS 8 的新手感到有些不知所措。

好消息是，大部分复杂性存在的原因是为了让有经验的电子邮件管理员能够为大规模企业安装配置复杂的设置。事实上，对于大多数 Linux 管理员来说，设置一个基本的电子邮件系统以便用户能够发送和接收电子邮件相对简单。

在《CentOS 8 Essentials》这一章中，我们将解释基于 Linux 的电子邮件配置基础，并逐步介绍如何配置一个基本的电子邮件环境。为了提供基础知识，我们将把电子邮件系统的复杂性留给更高级的书籍进行详细讨论。

28.1 电子邮件系统的结构

一个完整的电子邮件系统由多个组件组成。以下是对每个组件的简要描述：

28.1.1 邮件用户代理

这是系统中典型用户最熟悉的部分。邮件用户代理（MUA），即邮件客户端，是用于编写、发送和阅读电子邮件的应用程序。任何在计算机上编写并发送邮件的人，都使用过某种类型的邮件用户代理。

Linux 上典型的图形化 MUA 包括 Evolution、Thunderbird 和 KMail。对于那些偏好基于文本的邮件客户端的用户，还有传统的 Pine 和 mail 工具。

28.1.2 邮件传输代理

邮件传输代理（MTA）是电子邮件系统的一部分，负责将电子邮件从一台计算机传输到另一台计算机（无论是在同一局域网内还是通过互联网传输到远程系统）。一旦配置正确，大多数用户通常不会直接与他们选择的 MTA 交互，除非他们出于某种原因希望重新配置它。Linux 系统上有许多 MTA 的选择，包括 sendmail、Postfix、Fetchmail、Qmail 和 Exim。

28.1.3 邮件投递代理

电子邮件系统的另一个基础设施部分，邮件投递代理（MDA），通常对用户是隐藏的，它在后台执行邮件传输代理与邮件客户端（MUA）之间的邮件过滤。最常见的 MDA 形式是垃圾邮件过滤器，它会在邮件到达用户的收件箱之前，从系统中移除所有不需要的电子邮件。流行的 MDA 包括 Spamassassin 和 Procmail。需要注意的是，一些邮件用户代理应用程序（如 Evolution、Thunderbird 和 KMail）自带 MDA 过滤功能，而其他一些应用程序（如 Pine 和 Basla）则没有。这可能会让 Linux 初学者感到困惑。

28.1.4 SMTP

SMTP 是 Simple Mail Transport Protocol（简单邮件传输协议）的缩写。它是电子邮件系统用于将邮件从一台服务器传输到另一台服务器的协议。这个协议本质上是 MTA 之间用来相互通信并传输邮件消息的语言。

28.1.5 SMTP Relay

SMTP Relay 是一种协议，允许使用外部 SMTP 服务器发送电子邮件，而不是托管本地 SMTP 服务器。通常，这将涉及使用如 MailJet、SendGrid 或 MailGun 等服务。这些服务避免了配置和维护自己的 SMTP 服务器的必要性，并且通常提供额外的好处，如分析功能。

28.2 配置 CentOS 8 电子邮件服务器

许多系统使用 Sendmail MTA 来传输电子邮件消息，并且在许多 Linux 发行版中，这是默认的邮件传输代理。然而，Sendmail 是一个复杂的系统，对于初学者和有经验的用户来说，它都可能难以理解和配置。由于处理电子邮件消息的速度较慢，Sendmail 也逐渐失宠，相比之下，许多较新的 MTA 被认为更高效。

现在，许多系统管理员使用 Postfix 或 Qmail 来处理电子邮件。与 Sendmail 相比，它们的配置更快且更容易。

因此，本章将重点介绍 Postfix 作为 MTA，因为它简洁且流行。如果您更愿意使用 Sendmail，有许多专门讲解此主题的书籍，比我们在本章中能够讲解的要深入得多。

作为第一步，本章将介绍如何配置 CentOS 8 系统作为完整的电子邮件服务器。之后，本章还将介绍如何使用 SMTP Relay 服务。

28.3 Postfix 安装前步骤

在安装 Postfix 之前，第一步是确保系统上没有正在运行 Sendmail。您可以使用以下命令进行检查：

# 第二十七章：systemctl status sendmail

如果未安装 sendmail，该工具将显示类似以下的消息：

单元 sendmail.service 未找到。

如果系统上正在运行 sendmail，则必须在安装和配置 Postfix 之前停止它。要停止 sendmail，请运行以下命令：

# systemctl stop sendmail

下一步是确保 sendmail 在系统重启时不会自动重新启动：

# systemctl disable sendmail

Sendmail 现在已关闭，并已配置为在系统启动时不自动启动。可选地，要完全从系统中删除 sendmail，请运行以下命令：

# dnf remove sendmail

28.4 防火墙/路由器配置

由于发送和接收电子邮件涉及网络连接，因此需要使用 firewall-cmd 工具将 smtp 服务添加到防火墙中，命令如下：

# firewall-cmd --permanent --add-service=smtp

同时，配置任何位于服务器与互联网之间的防火墙或路由器，以允许端口 25、143 和 587 的连接也非常重要。如果有必要，还需要为这些端口配置端口转发，将其转发到邮件服务器上的相应端口。

完成这些初步步骤后，我们可以继续安装 Postfix。

28.5 在 CentOS 8 上安装 Postfix

默认情况下，CentOS 8 安装过程会为大多数配置安装 postfix。要验证是否已经安装 postfix，请使用以下 rpm 命令：

# rpm -q postfix

如果 rpm 报告 postfix 未安装，可以通过以下方式安装：

# dnf install postfix

dnf 工具将下载并安装 postfix，并在 /etc/passwd 文件中配置一个特殊的 postfix 用户。

28.6 配置 Postfix

postfix 的主要配置设置位于 /etc/postfix/main.cf 文件中。网上有许多资源提供关于 postfix 的详细信息，因此本节将重点介绍启动邮件系统所需的基本选项。

main.cf 文件中的关键选项如下：

myhostname = mta1.domain.com

mydomain = domain.com

myorigin = $mydomain

mydestination = mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

inet_interfaces = $myhostname

mynetworks = subnet

其他设置可能已经在安装过程中为您配置，或者除非您有冒险精神并且希望配置一个更复杂的电子邮件系统，否则不需要进行任何更改。

myhostname 的格式为 host.domain.extension。如果您的 Linux 系统名为 MyLinuxHost，您的互联网域名为 MyDomain.com，则应将 myhostname 选项设置如下：

myhostname = mylinuxhost.mydomain.com

mydomain 设置只是上述设置中的域名部分。例如：

mydomain = mydomain.com

myorigin 设置定义了输出邮件从哪个域名看似发送，当邮件到达收件人的收件箱时应该显示为您的域名：

myorigin = $mydomain

其中一个最重要的参数是 mydestination，它与传入邮件有关，声明该服务器是哪些域的最终投递目的地。任何发送到不在此列表中的域名的传入邮件将被视为中继请求，通常会导致投递失败，这也受 mynetworks 设置（见下文）的影响。

inet_interfaces 设置定义了允许 postfix 接收电子邮件的系统网络接口，一般设置为 all：

inet_interfaces = all

mynetworks 设置定义了哪些外部系统被信任可以将该服务器用作 SMTP 中继。此设置的可能值如下：

•host - 仅本地系统被信任。所有外部客户端尝试使用该服务器作为中继将被拒绝。

•subnet - 只有在同一网络子网中的系统才被允许将服务器用作中继。例如，如果服务器的 IP 地址为 192.168.1.29，则 IP 地址为 192.168.1.30 的客户端系统可以使用服务器作为中继。

•class - 同一 IP 地址类别（A、B 和 C）中的任何系统都可以将服务器用作中继。

可信 IP 地址也可以通过手动指定子网、地址范围或引用模式文件来定义。以下示例声明本地主机和子网 192.168.0.0 为可信 IP 地址。

mynetworks = 192.168.0.0/24, 127.0.0.0/8

对于此示例，将属性设置为子网，这样与服务器在同一局域网中的其他系统就能通过 SMTP 中继发送电子邮件，而外部系统则无法发送邮件。

mynetworks = subnet

28.7 配置 DNS MX 记录

当您通过注册商注册并配置域名时，DNS 设置中将配置一些默认值。其中之一是所谓的邮件交换器（MX）记录。该记录本质上定义了邮件应发送到域的哪个位置，通常默认设置为由注册商提供的邮件服务器。如果您托管自己的邮件服务器，则 MX 记录应设置为您的域名或邮件服务器的地址。如何更改此记录取决于您的域名注册商，但通常需要编辑域名的 DNS 信息，并添加或编辑现有的 MX 记录，以使其指向您的邮件服务器。

28.8 在 CentOS 8 系统上启动 Postfix

一旦/etc/postfix/main.cf 文件配置了正确的设置，就可以启动 Postfix。可以通过以下命令从命令行启动：

# systemctl start postfix

要配置 Postfix 在系统启动时自动启动，请运行以下命令：

# systemctl enable postfix

此时，Postfix 进程应该已经启动。验证一切正常的最佳方法是检查邮件日志。该日志通常位于/var/log/maillog 文件中，并且应该包含类似以下输出的条目：

Mar 25 11:21:48 demo-server postfix/postfix-script[5377]: starting the Postfix mail system

Mar 25 11:21:48 demo-server postfix/master[5379]: daemon started -- version 3.3.1, configuration /etc/postfix

只要没有记录错误消息，您就已成功安装并启动了 Postfix，并准备好测试 Postfix 配置。

28.9 测试 Postfix

测试 Postfix 配置的一种简单方法是发送本地用户之间的电子邮件消息。要进行快速测试，可以使用以下 mail 工具（其中 name 和 mydomain 分别替换为系统中的用户名和您的域名）：

# mail name@mydomain.com

当提示时，输入电子邮件消息的主题，然后输入消息正文。要发送电子邮件，只需按 Ctrl-D。例如：

# mail neil@mydomain.com

主题：测试邮件消息

这是一个测试消息。

EOT

再次运行邮件命令，这次以另一个用户身份执行，验证消息是否已发送和接收：

$ mail

Heirloom Mail 版本 12.5 7/5/10\. 输入?获取帮助。

"/var/spool/mail/neilsmyth": 1 message 1 new

>N 1 root Mon Mar 25 13:36 18/625 "测试邮件消息"

&

如果消息没有出现，检查日志文件（/var/log/maillog）以查找错误。成功的邮件投递将在日志文件中如下所示：

Mar 25 13:41:37 demo-server postfix/pickup[7153]: 94FAF61E8F4A: uid=0 from=<root>

Mar 25 13:41:37 demo-server postfix/cleanup[7498]: 94FAF61E8F4A: message-id=<20190325174137.94FAF61E8F4A@demo-server.mydomain.com>

Mar 25 13:41:37 demo-server postfix/qmgr[7154]: 94FAF61E8F4A: from=<root@mydomain.com>, size=450, nrcpt=1 (queue active)

Mar 25 13:41:37 demo-server postfix/local[7500]: 94FAF61E8F4A: to=<neil@mydomain.com>, relay=local, delay=0.12, delays=0.09/0.01/0/0.02, dsn=2.0.0, status=sent (delivered to mailbox)

Mar 25 13:41:37 demo-server postfix/qmgr[7154]: 94FAF61E8F4A: removed

本地邮件功能正常后，尝试向外部地址（例如 GMail 账户）发送邮件。同时，测试通过外部账户向你域名中的用户发送邮件，验证来邮功能是否正常。每次操作后，检查/var/log/maillog 文件中的错误说明。

28.10 通过 SMTP 中继服务器发送邮件

配置邮件服务器以处理外发邮件的替代方案是使用 SMTP 中继服务。如前所述，有许多此类服务可用，大多数可以通过搜索“SMTP Relay Service”找到。大部分服务要求你以某种方式验证域名，并提供 MX 记录以更新 DNS 设置。你还将获得一个用户名和密码，这些信息需要添加到 postfix 配置中。本节假设你已经在系统上安装了 postfix，并且你选择的 SMTP 中继提供商要求的所有初始步骤都已完成。

首先编辑/etc/postfix/main.cf 文件，并使用你的域名配置 myhostname 参数：

myhostname = mydomain.com

接下来，在/etc/postfix 目录下创建一个名为 sasl_passwd 的新文件，并添加一行，内容包括中继服务提供的邮件服务器主机以及用户名和密码。例如：

[smtp.myprovider.com]:587 neil@mydomain.com:mypassword

请注意，上述条目中也指定了端口 587。如果没有此设置，postfix 将默认为使用端口 25，而大多数 SMTP 中继服务提供商默认会封锁该端口。

创建密码文件后，使用 postmap 工具生成包含邮件凭证的哈希数据库：

# postmap /etc/postfix/sasl_passwd

在继续之前，采取一些额外步骤来保护你的 postfix 凭证：

# chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db

# chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db

再次编辑 main.cf 文件，并添加一条配置项来指定中继服务器：

relayhost = [smtp.myprovider.com]:587

在 main.cf 文件中，添加以下行以配置 SMTP 服务器的身份验证设置：

smtp_use_tls = yes

smtp_sasl_auth_enable = yes

smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd

smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt

smtp_sasl_security_options = noanonymous

smtp_sasl_tls_security_options = noanonymous

最后，重新启动 Postfix 服务：

# systemctl restart postfix

一旦服务重新启动，尝试使用邮件工具或你喜欢的邮件客户端发送和接收邮件。

28.11 总结

一个完整的端到端邮件系统包括邮件用户代理（MUA）、邮件传输代理（MTA）、邮件投递代理（MDA）和 SMTP 协议。CentOS 8 提供了多种 MTA 解决方案，其中较为流行的是 Postfix。本章概述了如何在 CentOS 8 系统上安装、配置和测试 Postfix，使其既能充当邮件服务器，又能使用第三方 SMTP 中继服务器发送和接收邮件。
