# 13

# 使用 SSH 进行安全远程访问

**安全外壳协议**（**SSH**）是一把瑞士军刀——一个可以做任何事情的工具——用于创建安全连接并通过这些连接传输数据。在你的职业生涯中，你将使用 SSH 来做几乎所有事情：

+   安全登录到远程系统

+   克隆你的私人 Git 仓库

+   从你的笔记本电脑向服务器传输文件，或在服务器之间传输文件

+   将通过 VPN 连接的网络服务映射到你笔记本电脑的本地端口，以便你家庭网络中的某人可以使用它

+   其他涉及通过多个网络连接隧道传输流量或发送文件的任务

在本章中，我们将为你提供你所需的所有基础知识，让你感到得心应手。你将学习公钥加密是如何工作的，这是理解这些工具及其使用的基础。你将创建 SSH 密钥，并使用它们登录到远程服务器。为了巩固这些基础知识，我们甚至为你创建了一个小项目，你将在其中为你经常使用的远程主机设置基于密钥的登录。

在你使用 SSH 时难免会遇到一些问题，我们整理了在实际应用中最常见的 SSH 错误。你将学习这些常见错误信息的含义，以及如何使用 SSH 内置的“调试”选项来解决它们。

在本章中，我们将涵盖以下主题：

+   公钥加密入门

+   消息加密

+   消息签名

+   SSH 密钥

+   将 SSH2 密钥转换为 OpenSSH 格式

+   文件传输

+   SSH 隧道

+   配置文件

让我们直接从公钥加密的基础知识开始，让你能理解本章的其余内容，而不至于觉得它像是深奥的黑魔法。

# 公钥加密入门

根据你职业发展的路径，你可能已经接触过公钥加密的主题。虽然加密学是一个独立的领域，而这本书并不是关于加密学的，但*了解其基础知识*是很重要的。幸运的是，核心概念非常简单，掌握了这些就能走得很远。我们会尽量简短地介绍这一部分，然后直接进入你需要使用的命令，帮助你配置并使用 SSH 进行安全访问。

公钥加密是一种使用两个独立密钥的系统，分别称为公钥和私钥。合起来，这些密钥组成了所谓的密钥对。正如名字所示，公钥是可以与任何人共享的密钥，而私钥则必须保持私密，绝对不能泄露。

## 消息加密

一旦你创建了密钥对，任何使用该密钥对公钥加密的消息都可以用相应的私钥解密。

假设有一个名叫 Alice 的人，她想要发送一条加密消息给 Bob。为此，Alice 下载了 Bob 的公钥，并用它来加密消息。然后，Alice 将加密后的消息发送给 Bob。因为 Bob 拥有匹配的私钥，他可以解密并读取这条消息。即使其他人看到加密后的消息，也无法解密和读取，因为只有 Bob 拥有私钥。

因此，*绝对不要*将你的私钥与第三方共享。这样做会违反安全原则。

## 消息签名

还有一种使用这两个密钥的方法：它们可以用来签名消息。对消息进行“签名”是一种加密学方法，用来证明消息确实是由拥有该密钥的人所写。其原理是，虽然用公钥加密的消息可以用私钥解密，但反之亦然——用私钥加密的消息可以用公钥解密。

当 Alice 想要签名一条消息时，可以使用私钥来加密消息（或者加密消息的安全哈希）。任何拥有 Alice 公钥的人都可以用它解密消息，如果解密成功，就可以确定这条消息一定是用 Alice 的私钥加密的。

加密和签名这两种机制通常是一起使用的。此外，签名本身常常用于确保安全（通过验证作者身份），例如，在通过包管理器或应用商店下载软件时。

值得注意的是，这些核心的公钥加密和签名机制被广泛应用于从加密电子邮件到保护 HTTPS 网络流量，再到现代世界中的成千上万种应用。换句话说，Alice 和 Bob 不一定是人类；他们也可以是计算机、服务等。

现在你已经了解了 SSH 如何利用基本的加密构建块来保护你的远程访问，接下来你可以实际应用这些先进的技术了！

# SSH 密钥

在使用 SSH 时，你可能会首先做的事情就是创建你自己的密钥对。这将允许你通过 SSH 认证到服务器。创建密钥对的经典命令如下：

```
ssh-keygen -t ed25519 -C 'John Doe <john.doe@example.org>' 
```

这将创建一个 `ed25519`（一种现代椭圆曲线加密算法）密钥对，并以 `John Doe <john.doe@example.org>` 作为注释。注释类似于你在编程语言中熟悉的注释，可以是任意文本字符串，并不会在功能上产生干扰。在 SSH 的情况下，这个注释将被附加到你的公钥上，方便在将密钥上传到服务器时区分各个密钥，例如在 `authorized_keys` 文件中。稍后在本章中，我们将更深入地讨论 `authorized_keys` 文件以及如何使用它们来设置无缝、安全的远程服务器访问。

运行此命令后，OpenSSH 会询问你一些关于存储它创建的密钥文件位置以及你希望用于加密私钥的密码的问题。由于这将是用来访问远程系统的密钥，确保设置一个强密码。

**注意**

默认情况下，密钥将被放置在你的 `~/.ssh` 目录中。

现在你已经创建了密钥对，是时候重申最重要的实用点：永远不要共享私钥。这样做会让第三方冒充你。没有任何服务应该要求你共享私钥。公共密钥应该是共享的，并且可以公开的，它会带有 `.pub` 后缀，这样你就能区分它。

幸运的是，这些文件的内容看起来非常不同，因此如果你感到困惑，可以查看它们来判断哪一个是哪个：

+   公钥的格式是 `<algorithm> <key> <comment>`。

+   私钥以类似 `-----BEGIN OPENSSH PRIVATE KEY-----` 的行开始，接着是密钥和类似的结束行。

小心不要覆盖这些密钥文件，并确保你将它们都存储在一个安全的备份中。同样，密码管理器是一个不错的选择。许多密码管理器甚至有专门选项来存储私钥文件或通用文本。

## 这些规则的例外

对于个人使用——例如在你的笔记本电脑上——你总是希望加密你的私钥（在创建密钥时指定一个密码），然后像之前提到的那样永远不要共享它。

然而，在某些情况下，打破这些规则是可以接受的——特别是在为自动化系统设置密钥时。如果你希望构建服务器在检出代码库之前通过 GitHub 进行身份验证，你可能会使用一个私钥未加密的密钥对（除非你希望在每次构建服务运行时手动输入密码）。

机器与机器之间的身份验证和加密是使用加密密钥对的一个重要原因。只要确保总是为这些任务创建专用的、单一用途的密钥对，并且不要在不同的机器或服务之间重复使用或共享这些密钥对。

## 登录和身份验证

使用基于 SSH 的身份验证登录远程系统的过程类似于这样：

```
ssh user@example.org 
```

`user` 是你希望登录的用户名，`example.org` 是你想连接的任何远程系统的代号。它通常是一个 IP 地址，而不是一个完全限定的域名。

如果你正在使用 SSH 密钥登录，或者需要指定特定的密钥（身份，或 `-i`），它看起来像这样：

```
ssh -i ~/.ssh/id_ecdsa user@example.org 
```

当你访问一个从未连接过的 SSH 服务器时，你将看到远程服务器的指纹。这使你能够确保你连接的服务器确实是你想连接的服务器，并且没有发生中间人攻击。你应该确保这个指纹是正确的。

一旦您输入`yes`并将该指纹标记为受信任，它将被保存到一个文件中。如果它发生了变化——例如，如果有人在相同 IP 地址上设置了一个恶意服务器，您的 SSH 客户端将通知您，并且身份验证将不可能成功。

在您将服务器标记为受信任后，您的本地客户端和服务器将协商使用哪种身份验证方式。OpenSSH 提供了多种选项，其中最常见的两种涉及密码或密钥对。根据选择的选项，OpenSSH 将要求您输入密码（或用于解密私钥的密码）。一旦身份验证步骤成功，您将登录。

# 实际项目：设置基于密钥的登录到远程服务器

假设您有权限访问长时间运行的 Linux 服务器，并希望允许基于密钥的登录，请按照以下步骤操作。

## Step 1: 在 SSH 客户端（而非服务器）上打开您的终端

您将在接下来的步骤中使用本地命令行环境。

## Step 2: 生成密钥对

如果您已经设置了密钥对，因为您之前在本章中跟随了这些步骤，那么太棒了！您可以跳过此步骤。

如果您还没有 SSH 密钥对，请输入以下命令并按*Enter*键创建一个：

```
ssh-keygen -t ed25519
# ensure that the public key is not world-readable
chmod 600 ~/.ssh/id_ed25519 
```

正如之前提到的，我们强烈建议添加一个密码短语以增强安全性。

## Step 3: 将公钥复制到您的服务器

生成密钥后，您需要将公钥放置在您的服务器上。公钥通常具有扩展名`.pub`，默认情况下将位于您的`~/.ssh`目录中。

您可以将其手动复制到远程用户的`authorized_keys`文件中（该文件包含该用户的所有授权公钥，每行一个密钥），或者您可以使用`ssh-copy-id`程序将所有这些操作压缩为单个命令：

```
ssh-copy-id username@example.org 
```

将`username`替换为远程服务器上的用户，将`remote_server_address`替换为服务器的 IP 地址或域名。

此命令将要求您在远程服务器上输入用户密码。输入后，公钥将追加到远程用户主目录下的`~/.ssh/authorized_keys`文件中。这允许您登录并在远程机器上执行命令，而无需提示输入密码。

## Step 4: 测试一下吧！

现在尝试登录服务器：

```
ssh username@example.org 
```

如果一切顺利，您应该能够登录而无需输入密码（除非您为 SSH 密钥设置了一个密码短语）。现在，您将使用一个*更*安全的加密密钥来进行身份验证，而不是使用可能被攻击者猜到的小密码字符串。

欢迎来到安全且无密码的 SSH 访问的美好世界！

# 将 SSH2 密钥转换为 OpenSSH 格式

当不使用基于 Unix 的操作系统时，你通常会遇到 SSH2 公钥格式。PuTTY 可能是使用这种格式的最著名软件，许多使用 Windows 的人也使用它通过 SSH 进行连接。要连接到**SSH 文件传输协议**（**SFTP**）服务器、Git 仓库或其他使用 OpenSSH 密钥格式的系统，你需要将 SSH2 公钥转换为 OpenSSH 格式。下面是如何进行转换。

## 我们想要实现的目标

我们从一个 SSH2 格式的公钥开始，它看起来像这样：

```
---- BEGIN SSH2 PUBLIC KEY ----
Comment: "rsa-key-20160402"
AAAAB3NzaC1yc2EAAAABJQAAAgEAiL0jjDdFqK/kYThqKt7THrjABTPWvXmB3URI
pGKCP/jZlSuCUP3Oc+IxuFeXSIMvVIYeW2PZAjXQGTn60XzPHr+M0NoGcPAvzZf2
u57aX3YKaL93cZSBHR97H+XhcYdrm7ATwfjMDgfgj7+VTvW4nI46Z+qjxmYifc8u
VELolg1TDHWY789ggcdvy92oGjB0VUgMEywrOP+LS0DgG4dmkoUBWGP9dvYcPZDU
F4q0XY9ZHhvyPWEZ3o2vETTrEJr9QHYwgjmFfJn2VFNnD/4qeDDHOmSlDgEOfQcZ
Im+XUOn9eVsv//dAPSY/yMJXf8d0ZSm+VS29QShMjA4R+7yh5WhsIhouBRno2PpE
VVb37Xwe3V6U3o9UnQ3ADtL75DbrZ5beNWcmKzlJ7jVX5QzHSBAnePbBx/fyeP/f
144xPtJWB3jW/kXjtPyWjpzGndaPQ0WgXkbf8fvIuB3NJTTcZ7PeIKnLaMIzT5XN
CR+xobvdC8J9d6k84/q/laJKF3G8KbRGPNwnoVg1cwWFez+dzqo2ypcTtv/20yAm
z86EvuohZoWrtoWvkZLCoyxdqO93ymEjgHAn2bsIWyOODtXovxAJqPgk3dxM1f9P
AEQwc1bG+Z/Gc1Fd8DncgxyhKSQzLsfWroTnIn8wsnmhPJtaZWNuT5BJa8GhnzX0
9g6nhbk=
---- END SSH2 PUBLIC KEY ---- 
```

目标是将其转换为以下格式的 OpenSSH 公钥：

```
ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAgEAiL0jjDdFqK/kYThqKt7THrjABTPWvXmB3URIpGK
CP/jZlSuCUP3Oc+IxuFeXSIMvVIYeW2PZAjXQGTn60XzPHr+M0NoGcPAvzZf2u57aX3YKaL93cZSBHR
97H+XhcYdrm7ATwfjMDgfgj7+VTvW4nI46Z+qjxmYifc8uVELolg1TDHWY789ggcdvy92oG
jB0VUgMEywrOP+LS0DgG4dmkoUBWGP9dvYcPZDUF4q0XY9ZHhvyPWEZ3o2vETTrEJr9QHYwgjmFf
Jn2VFNnD/4qeDDHOmSlDgEOfQcZIm+XUOn9eVsv//dAPSY/yMJXf8d0ZSm+VS29QShMjA4R+7yh5Wh
sIhouBRno2PpEVVb37Xwe3V6U3o9UnQ3ADtL75DbrZ5beNWcmKzlJ7jVX5QzHSBAnePbBx/fyeP/
f144xPtJWB3jW/kXjtPyWjpzGndaPQ0WgXkbf8fvIuB3NJTTcZ7PeIKnLaMIzT5XNCR+xobvdC8
J9d6k84/q/laJKF3G8KbRGPNwnoVg1cwWFez+dzqo2ypcTtv/20yAmz86EvuohZoWrtoWvkZLCoyxdqO93ymE
jgHAn2bsIWyOODtXovxAJqPgk3dxM1f9PAEQwc1bG+Z/Gc1Fd8DncgxyhKSQzLsfWroTn
In8wsnmhPJtaZWNuT5BJa8GhnzX09g6nhbk= 
```

## 如何将 SSH2 格式的密钥转换为 OpenSSH

我们用来创建新密钥的`ssh-keygen`命令，也可以通过这个非常简单的命令进行转换：

```
ssh-keygen -i -f ssh2.pub > openssh.pub 
```

上述命令将从文件`ssh2.pub`中获取密钥，并将其写入`openssh.pub`。

如果你只想查看 OpenSSH 密钥内容，或者准备好进行复制和粘贴，那么不必担心将标准输出重定向到文件（与上面的命令相同，只是去掉了最后一部分）：

```
ssh-keygen -i -f ssh2.pub 
```

这将简单地显示 OpenSSH 格式的公钥。

一个更实用的例子可能是将同事的密钥转换并追加到服务器的`authorized_keys`文件中。可以使用以下命令实现：

```
ssh-keygen -i -f coworker.pub >> ~/.ssh/authorized_keys 
```

之后，使用相关私钥的同事将能够以运行此命令的用户身份登录系统。

## 另一个方向：将 SSH2 密钥转换为 OpenSSH 格式

反向操作——将 OpenSSH 转换为 SSH2 密钥——也是可能的。只需使用`-e`（用于导出）标志，而不是`-i`（用于导入）：

```
ssh-keygen -e -f openssh.pub > ssh2.pub 
```

# SSH 代理

当你频繁使用 SSH 密钥登录服务器时，必须每次连接（或重新连接）到主机时反复输入私钥密码，可能会让人觉得烦人。SSH-Agent 允许你在本地会话中存储一个身份（私钥）——换句话说，它让你解密一次私钥，然后将其保存在内存中，直到你注销或启动新的 shell 会话。这意味着你只需添加一个身份（密钥对），然后可以反复使用它，而无需重新解密私钥。

**注意**

SSH 代理并不总是在你的本地 shell 会话中运行——各种 IDE、窗口管理器、桌面管理器和密码管理器也可以为你运行代理。当你只需输入一次身份密码时，你就知道这是在运行代理。

要将密钥添加到代理中，只需使用`ssh-add`命令——参数是该身份的私钥路径：

```
ssh-add ~/.ssh/id_ecdsa 
```

我们建议养成使用`–t`选项的习惯，该选项会设置一个时间限制，指定密钥在内存中保持解密状态的时间。以下命令与上面相同，唯一的不同是它设置了 30 秒的时间限制，之后代理将从内存中删除密钥：

```
ssh-add –t 30 ~/.ssh/id_ecdsa 
```

要查看已添加到代理中的密钥，可以使用以下命令：

```
ssh-add -L 
```

要从 SSH 代理中删除所有身份，使用`-D`：

```
ssh-add -D 
```

**注意**

如果您已经向代理中添加了超过三个身份，您可能仍然需要在通过 SSH 登录时指定`-i $YOUR_IDENTITY`。这是因为大多数服务器在三次错误尝试后会拒绝登录，而 SSH 在登录时会逐一尝试代理中存储的每个密钥。如果第一个密钥无法工作，它将尝试第二个，以此类推。如果服务器在三次尝试后中止登录，您将永远无法使用代理中的第四个密钥。

在使用 SSH 登录远程机器时，您可以启用代理转发（`-A`）；但我们建议您谨慎且小心地使用它，原因如下：

```
ssh -A user@remotehost 
```

使用此命令登录`remotehost`后，SSH 将把您的密钥转发到该主机，以便您可以从那里跳转到其他主机。我们推荐谨慎使用这一功能，因为它允许已被攻破的主机看到您的私钥，而我们希望您始终保持这些密钥的私密性，最好保存在您的计算机上，或者在您最狂野的时刻，保存在密码管理器中。换句话说，如果`remotehost`已经被黑客攻击，那么您的 SSH 密钥也会受到威胁。

关于安全性的一点说明：一些 Linux 桌面环境，如 GNOME/MATE，当您使用并解密 SSH 密钥时，会将密钥一直保存在内存中。这是默认行为，且可能带来安全风险，您应该注意这一点。

# 常见的 SSH 错误和-v（详细模式）参数

SSH 命令中的`-v`标志启用详细模式，会输出连接过程的逐步日志。这一功能对于诊断常见问题非常有用，例如身份验证失败、连接超时和密钥不匹配。使用方法如下：

```
ssh -v username@example.org 
```

您将获得有关 SSH 握手和连接的逐步信息，便于识别和解决可能出现的任何问题。

以下是一些您可能遇到的常见错误，详细输出可以帮助您诊断这些错误：

+   **权限被拒绝（公钥/密码）**：这表示服务器拒绝了您的登录尝试。详细日志将显示尝试过的密钥，帮助您确定是否使用了正确的密钥，甚至是否提供了密钥。这是一个*极其*常见的问题，尤其是在客户端存储了超过三个密钥对，而服务器仅允许三次尝试时。

+   **连接超时**：如果连接时间过长，可能是网络问题或 IP 地址/端口错误。`-v`标志会显示连接过程卡住的位置，帮助您了解客户端是否到达了服务器。

+   **连接被拒绝**：这通常意味着 SSH 没有在服务器的目标端口上运行（或无法访问）。详细输出会清晰地指出连接尝试被拒绝，帮助您集中精力检查防火墙规则或 SSH 服务器设置。

+   **主机密钥验证失败**：服务器的密钥与系统的`known_hosts`文件中保存的密钥不匹配。`-v`标志会显示不匹配的密钥，此时你可以重点查看**为什么**（例如，这个 IP 地址或主机名是否有新服务器？）。

+   **无法解析主机名**：通常与 DNS 或网络问题有关。

+   **无法连接到主机**：这通常表示网络问题，可能涉及防火墙或路由设置错误。

+   **身份验证失败次数过多**：已达到最大身份验证尝试次数。详细模式将显示所有已尝试的方法，其中可能包括不需要的或意外的密钥提议。

+   **密钥加载错误**：这些通常表示你的 SSH 密钥的格式或权限存在问题。`-v`标志会显示 SSH 客户端正在尝试加载的密钥，你可以检查格式或权限问题。

使用`-v`标志将帮助你理解到底出了什么问题，以及如何修复它。至少，它会帮助你开始朝正确的方向查找。

# 文件传输

在下面的章节中，我们将探索用于文件传输的`sftp`和`scp`命令。通过几个示例，你将理解如何在大多数情况下处理文件。也就是说，我们还将介绍不使用 SFTP 或 SCP 的文件传输方式，以防它们在服务器上被禁用。

## SFTP

虽然 OpenSSH 通常用于登录远程系统，但它也允许在不需要登录会话的情况下进行文件传输。这通常是通过 SFTP 子系统实现的。虽然 SFTP 类似于**文件传输协议**（**FTP**），它实际上是一个完全自定义的协议。像 FTP 一样，SFTP 允许经过身份验证的用户向远程服务器上传和下载文件。与不安全的 FTP 不同，SFTP 的身份验证和文件传输是安全的，并且完全加密。

有许多 FTP 客户端也支持 SFTP。一个著名的例子是 Filezilla，它有一个出色的图形用户界面。然而，由于这是一本关于 Linux 命令行的书，我们将为你提供如何在命令行中使用 SFTP 的基本概述。

身份验证几乎与`ssh`相同：

```
sftp user@example.org 
```

完成后，你将看到一个类似 FTP 的界面。它接受一些我们在前几章中已讨论的简化/修改版的 Shell 命令，并引入了一些新的命令。以下是最常用的命令：

+   `help`：列出所有命令并提供简短总结

+   `ls`：列出远程目录的内容

+   `lls`：列出本地目录的内容

+   `cd`：更改远程目录

+   `lcd`：更改本地目录

+   `pwd`：显示你所在的远程目录

+   `lpwd`：显示你所在的本地目录

+   `get`：从远程服务器下载文件

+   `put`：将文件上传到远程服务器

+   `chmod`：更改远程文件或目录的权限

+   `chown`：更改远程文件或目录的所有者

+   `quit`，`exit`，`bye`：退出 SFTP（*CTRL*+*D* 也有效）

## SCP

`scp` 命令通常比 `sftp` 更实用，用于上传和下载文件和目录。虽然历史上它独立于 SFTP，但今天它使用了 SFTP 子系统。它作为 `cp` 命令的替代品，除了它可以实现从远程服务器上传和下载。

命令格式如下：

```
scp $SOURCE:filepath $DESTINATION:filepath 
```

`$SOURCE`，`$DESTINATION`，或者两者都可以是远程系统；`scp` 使用你已经看到的 SSH `user@example.org` 语法。

下面是实际操作的样子：

```
scp user@example.org:/home/user/my_remote_file /home/user/my_local_file 
```

这将执行以下操作：

1.  连接到 `example.org`。

1.  作为 `user` 进行身份验证（如果你没有使用 SSH 密钥和 SSH 代理，它会提示你输入密码）。

1.  将文件复制到本地路径 `/home/user/my_local_file`。

如你所见，参数的顺序与 Linux 的 `cp`（复制）命令相同。

反转源和目标参数——上传文件，而不是下载文件——看起来应该是你预期的样子：

```
scp /home/user/my_local_file user@example.org:/home/user/my_remote_file 
```

就像使用 `cp` 一样，你可以指定相对路径。以下命令会将远程文件复制到当前（本地）目录：

```
scp user@example.org:/home/user/my_remote_file . 
```

与 `cp` 命令一样，也可以使用 `-r`（递归，某些系统上是 `-R`）标志递归地复制整个目录：

```
scp -r user@example.org:/home/user/directory local_directory 
```

## 巧妙的示例

与所有命令和工具一样，`ssh` 和 `scp` 也可以在脚本中使用；例如，你可以使用 ssh 快速备份数据库：

```
ssh username@example.org "pg_dump databasename | gzip -c" > database_backup.sql.gz 
```

如果在脚本中使用类似的巧妙 shell 技巧，请注意在出现错误时进行提示；否则，程序可能在遇到问题时默默卡住。对于开发环境来说，这种命令确实非常实用。

### 如果没有 SFTP 或 SCP

在一些罕见的情况下，你可能会发现 SFTP 在服务器上被禁用，且只能登录到交互式 shell。你仍然希望将文件传输到远程系统，尽管可能可以在编辑器中打开文件，但并不总是可行（例如，整个目录或二进制文件）。下面是一些你可以用来实现目标的技巧（它们都利用了 Unix 管道与 SSH 会话的结合）。

最简单的情况是从远程服务器下载文件：

```
ssh user@example.org "cat /path/to/file" > local_target_file 
```

此命令将执行以下操作：

1.  登录到服务器。

1.  在远程服务器上运行命令 `cat /path/to/file`，这将导致该文件的内容被流式传输到 `stdout`。

1.  `stdout` 可以被本地重定向到我们选择的文件中。

像大多数行为规范的 Unix 软件一样，错误信息和其他可能干扰的输出会发送到 `STDERR`，所以你无需担心密码提示会阻碍文件内容的传输并破坏通过 SSH 隧道传输的文件内容。

### 目录上传和 .tar.gz 压缩

让我们做点不同的事情，上传整个目录。由于这可能是相当大的数据，我们也将使用`tar`程序进行压缩。`tar`是一个将多个文件和/或目录合并为一个文件（即“归档”）的命令。

在归档后添加压缩步骤是很常见的——例如，你可能见过以`tar.gz`或`tar.bz2`结尾的文件。这意味着文件首先使用`tar`归档成一个单一文件，然后使用`gzip`或`bzip2`进行压缩：

```
tar czf - /home/user/directory_to_upload | ssh user@example.org "tar -xvzf -C /home/user/" 
```

在这里，我们首先使用`tar`归档`/home/user/directory`，将结果变成字节流。

`-`是目标文件。如果你想立即存储它，而不是将其传输到另一个程序，它可能是像`/home/user/directory.tar.gz`这样的路径。就像许多 Unix 命令一样，破折号意味着该内容应写入标准输出（stdout）。

然后，生成的字节流会被传递到`ssh`进程中，作为远程系统上的`tar`命令的输入（`stdin`），该命令会解压并解档该流，将结果目录写入`/home/user/directory_to_upload`。

# 隧道

SSH 隧道用于通过 SSH 连接传输数据。在接下来的部分中，我们将探讨两种隧道方法：本地转发和代理。

## 本地转发

SSH 可以创建到远程系统的安全加密隧道。这个功能类似于 VPN，能够让你访问远程系统可以到达的服务。

这是一个强大的功能，实际上通过 SSH 可以很容易地实现。你只需要在建立 SSH 会话时，指定一个额外的参数`-L`，以及目标和要绑定的本地端口。

假设有一个远程系统在`8080`端口运行 HTTP 服务器。你想在你的笔记本电脑上通过`3000`端口访问它。以下是使用简单命令实现这一目标的方法：

```
ssh -L 3000:localhost:8080 user@example.org 
```

现在，你可以打开浏览器并访问`http://localhost:3000/`，像在远程系统上打开浏览器并访问`http://localhost:8080`一样，访问 Web 服务器。

## 代理

如果你想访问一个你没有权限访问的服务器，但远程服务器有权限的情况，也是一样的。假设`app.example.org`是你运行 Web 应用程序的地方。这个 Web 应用程序连接到像 PostgreSQL 这样的数据库服务器`db.example.org`，而这个数据库服务器仅能从`www.example.org`内部网络访问。像大多数生产数据库一样，它被防火墙保护，防止外部直接连接。

从网络的角度来看，它是这样的：

`(localhost) ——-> (app.example.org) ——-> (db.example.org)`

你可能想使用本地的`psql` PostgreSQL 客户端连接到那个数据库。你可以像这样创建一个隧道：

```
ssh -L 5000:db.example.org:5432 user@app.example.org 
```

这将打开一个新的 SSH 会话连接到`app.example.org`，从那里连接到数据库服务器，并将`db.example.org:5432`映射到`localhost:5000`。

现在，在你的笔记本电脑上运行 `psql --port=5000 --host=localhost dbname` 将通过 `app.example.org` 进行中转，从而连接到 `db.example.org:5432` 上的 Postgres 数据库。

# 配置文件

可以在 `.ssh/config` 中指定主机配置。这在多种情况下都非常有用，因为它允许你指定：

+   为主机设置自定义（友好的）名称

+   默认用户

+   端口

+   在连接前打开的隧道

+   身份文件（密钥）

以及许多其他内容。

**注意**

如果你连接的服务器有一个永久的 IP 地址，那么在 SSH 配置文件中指定它是有意义的，这样可以避免在灾难恢复情况下依赖 DNS 或 CDN。

SSH 配置文件并不特别复杂，因此我们将在这里展示一个使用许多可用功能的示例：

```
# Set Defaults for all hosts using the glob character (*)Host *
  ServerAliveInterval 30     # Check if the connection is alive every 30 seconds
  ForwardAgent yes           # Forward SSH agent to the remote host
  Compression yes            # Enable compression
  IdentityFile ~/.ssh/id_rsa # Default identity file

# Specific settings for host "example1.com"
Host example1
  HostName example1.com       # The real hostname to connect to
  User john                   # Default username for this host
  Port 22                     # SSH port (default is 22)
  IdentityFile ~/.ssh/id_ecdsa # Different identity file for this host

# Settings for another specific host "example2.com"
Host example2
  HostName example2.com
  User jane
  Port 22000                  # Different SSH port for this host
  IdentityFile ~/.ssh/id_ed25519

# Using a jump host to connect to a host behind a firewall
Host target-behind-fw
  HostName 192.168.0.2         # Private IP of the target host
  User alice
  Port 22
  ProxyJump jump-host          # Use 'jump-host' as a jump host

# Configuration for the jump host
Host jump-host
  HostName jump.example.com    # Public IP or domain of the jump host
  User jumpuser
  Port 22
  IdentityFile ~/.ssh/jump_key

# Using a SOCKS proxy to connect to a host
Host proxy-host
  HostName proxy-target.com    # The real hostname to connect to
  User proxyuser
  Port 22
  ProxyCommand nc -X 5 -x localhost:1080 %h %p  # Using SOCKS proxy on localhost port 1080 
```

# 结论

OpenSSH 是一个非常多功能的工具，我们希望你在这一章中获得的介绍能够激发你进行实验并学习更多。想想我们涵盖的所有内容：

你已经学会了公钥密码学的基础知识，这对于理解这类工具及其使用至关重要。你了解了如何创建 SSH 密钥，并将它们用于远程 shell 会话。

希望你通过跟随本章并为你经常使用的远程主机设置基于密钥的登录，也获得了一些实践经验。如果该远程主机恰好位于 **Amazon Web Services**（**AWS**）或其他使用 `.pem` 密钥的平台，你还学会了如何在密钥格式之间进行转换（单凭这一技巧，就一定能让你的同事印象深刻）。

即使你自己没有遇到这些问题，我们也向你展示了人们在实际操作中常遇到的 SSH 错误，以及如何使用 `–v` 选项来追踪这些问题。

我们甚至讨论了远超远程交互式 shell 会话的 SSH 用法——文件传输、隧道网络流量以及为不同服务器设置自定义配置。令人惊讶的是，OpenSSH 还能做更多的事情：

+   使用 `ssh-keygen` 加密和签名文件

+   通过 FIDO/U2F 添加双因素认证，并将密钥存储在外部设备上

+   强制在登录后运行某些命令，这既可以限制攻击面，又能让 SSH 成为服务接口的一部分

OpenSSH 项目在其手册页和网站上提供了优秀的文档。如果你遇到需要在机器之间建立安全连接的问题，并且希望使用经过实战检验的技术，那么 OpenSSH 值得一试。现在，去加密吧！

# 在 Discord 上了解更多

要加入本书的 Discord 社区——在这里你可以分享反馈、向作者提问并了解新版本——请扫描下面的二维码：

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code1768422420210094187.png)
