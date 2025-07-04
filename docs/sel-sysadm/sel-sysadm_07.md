# 第六章：通过基础设施即代码编排配置 SELinux

随着大型分布式应用平台、云服务的出现，以及虚拟化基础设施的广泛采用，系统管理员正通过**基础设施即代码（Infrastructure-as-Code）**框架来积极管理他们的系统：这些框架使用类似源代码的信息来管理系统，包括编排和配置工具。

在本章中，管理员将学习如何分发和加载自定义 SELinux 策略模块，设置文件上下文定义并将其应用到系统，设置系统或 SELinux 域的宽松状态，配置系统上的 SELinux 设置，以及如何在工具不支持的情况下自定义 SELinux 操作。我们将使用四个流行的自动化框架来进行应用：Ansible、Chef、Puppet 和 SaltStack。

本章将涵盖以下主题：

+   介绍目标设置和策略

+   使用 Ansible 进行 SELinux 系统管理

+   利用 SaltStack 配置 SELinux

+   使用 Puppet 自动化系统管理

+   使用 Chef 进行系统自动化

# 技术要求

本章的代码文件可以在我们的 Git 仓库中找到：[`github.com/PacktPublishing/SELinux-System-Administration-Third-Edition`](https://github.com/PacktPublishing/SELinux-System-Administration-Third-Edition)。

查看以下视频，了解代码的实际操作：[`bit.ly/2T4Fksv`](https://bit.ly/2T4Fksv)

# 介绍目标设置和策略

在开始使用这四个自动化框架之前，我们需要明确我们希望实现的目标。毕竟，要真正比较自动化框架，我们需要每次使用相同的测试来测试每个框架。

## 操作的幂等性

每当我们创建一个带有中央存储库的远程管理环境时，需要考虑运行远程管理活动对系统的影响。一个非常常见的最佳实践，所有这些框架都强烈采纳的，就是幂等性。

**幂等**任务是指如果系统的状态已经是所需的状态，那么执行该任务不会修改系统。换句话说，重复执行一个任务不会影响系统或正在运行的进程，前提是无需进行更改。例如，考虑加载 SELinux 模块：如果模块已经加载，则不应重新加载该模块。如果模块尚未加载，则会加载适当的模块。

虽然自动化框架支持的大多数操作都是幂等的，但如果框架不支持我们需要的功能，我们需要自己创建自定义操作。例如，如果框架不支持加载 SELinux 模块，我们就需要编写自己的代码来完成此操作。

大多数编排框架会将非幂等任务封装在更为幂等的定义中。例如，如果配置文件的更改需要系统重启，那么封装后的定义可能是*在文件更改后重启*。引擎可以检查文件的状态（更改时间）和系统的状态（重启时间），从而推断是否需要重启，即使系统重启本身是一个非幂等的任务。

## 策略和状态管理

我们希望通过自动化框架支持的第一个场景是确保 SELinux 处于活动（强制）模式，并且正确的 SELinux 策略已加载，这是机器的包管理系统通常执行的任务。虽然让包管理系统处理这一任务比较方便，但它只能使用发行版特定的默认策略。具有不同安全需求的系统的系统管理员在使用默认策略时会受到限制，因此他们需要创建自定义策略和策略处理程序。因此，我们将研究如何分发和加载自定义策略。

我们将在示例中使用的自定义策略是 CIL 策略，这是一种非常新的策略，自动化框架通常不直接支持它。然而，它为我们提供了一个很好的重复性场景，允许我们在自动化框架中创建自定义规则。我们将策略本身存储在一个名为`test.cil`的文件中，文件内容如下：

```
(auditallow staff_sudo_t sysadm_t (process (transition)))
```

这个简单的策略将启用日志记录任何从`staff_sudo_t`域到`sysadm_t`域的过渡，并且可以通过`sudo`快速进行测试。在我们的示例中，它的唯一作用是快速验证策略是否已正确加载。

在状态管理方面，我们将确保系统处于强制模式，但将`zoneminder_t` SELinux 域标记为宽松模式。

## SELinux 配置设置

我们希望采取的第二组操作是根据之前章节中讨论的各种 SELinux 设置来配置系统。我们之前通过`semanage`命令看到过大多数这些设置，我们将学习各种自动化框架如何支持这些设置，并且支持到什么程度。

我们不会逐一讲解每个设置，而是集中讨论每个自动化框架中支持的配置集。如果某个框架不支持某个特定配置（例如`semanage ibpkey`配置，这个配置相对较新），我们需要为此创建自定义操作。在这种情况下，我们将演示如何处理这一问题，因为这种处理方法在其他场景中也会重复出现，且类似。

## 设置文件上下文

我们希望看到的第三组操作是自动化框架如何支持将文件上下文应用于资源。这可以是应用`semanage fcontext`配置，之后进行恢复操作（例如使用`restorecon`），也可以是验证框架是否支持直接应用上下文。

直接应用上下文允许管理员直接使用框架，而无需费力地创建和重新应用文件上下文定义（这可能会带来一定的性能开销）。然而，只有在自动化框架是唯一可以进行系统更改的方法时，才应考虑这样做。在任何其他情况下，缺失的文件上下文定义可能会导致管理员将上下文重置为错误的状态。

## 从错误中恢复

在本章中，我们深入探讨了允许在多个系统上管理 SELinux 的各种框架。本章的目的是介绍这些框架的核心支持功能，而不是详细解释框架本身或框架本身的安全配置。我们不建议在没有测试的情况下立即将其应用于生产系统，务必确保做好备份！

话虽如此，本章中应用的许多设置如果出现问题是可以轻松纠正的。我们参考*第二章*，*理解 SELinux 决策和日志记录*，在需要时选择性地将 SELinux 设置为宽容模式。

此外，每个框架都可以轻松暂停，允许管理员在不受框架立即覆盖更改影响的情况下修正问题。

## 比较框架

我们进一步讨论的每个框架都有自己对基础设施自动化和配置的处理方式。本书的目的不是详细探讨每个框架的细节，而是专注于它们的核心支持功能，以及它们如何处理 SELinux。我们还将抽象化如何处理不同的 Linux 发行版，并且所有示例都基于 CentOS。

此外，这些框架不断改进和发展。在本章中，我们仅探讨它们的常见用法，而不是它们如何在特定部署中发挥作用。例如，如果一个框架默认使用基于代理的架构，但也支持基于 SSH 的连接，我们只会考虑基于代理的架构，因为这是这些框架的默认设置，我们希望专注于 SELinux 配置支持功能。

但不要让这阻止你进一步实验这些框架，并根据自己的喜好进行调整！话说回来，让我们深入了解第一个引擎——Ansible。

# 使用 Ansible 进行 SELinux 系统管理

我们将首先考虑的编排和自动化工具是 Ansible，它是一个非常流行的开源解决方案，用于远程管理系统。Ansible 得到了 Red Hat 的商业支持，但并不限制其支持 Red Hat 或甚至 Linux 系统。其他环境，如 Windows 环境甚至网络设置，也有显著的基于 Ansible 的支持。

## Ansible 的工作原理

**Ansible** 通常使用一个托管配置并解析设置的中央服务器。然后，Ansible 运行时通过 SSH 连接到远程系统，将必要的数据发送到临时位置，并在本地执行步骤。

使用 SSH 作为主要连接方式有显著的优势：管理员知道该协议是如何工作的以及如何进行配置和控制。此外，Ansible 不需要在目标机器上进行额外的部署，除了 Python 和 libselinux 的 Python 绑定（这些通常在启用了 SELinux 的机器上默认安装）。

Ansible 通过其模块知道如何访问各种资源。**Ansible 模块**包含 Ansible 用于正确执行任务的逻辑。模块代码会分发到目标机器，并在远程系统上执行。

管理员配置系统时使用的定义存储在 Ansible 剧本中。**剧本**定义了系统应如何配置，Ansible 将读取和解释剧本，以了解它必须在每个系统上执行什么。

为了便于在更大的环境中管理 Ansible 剧本，Ansible 使用 **Ansible 角色** 来捆绑一致的定义。管理员可以在剧本中为系统分配角色，以自动提升这些系统的状态。例如，可以创建一个角色来创建一个正确配置的 Web 服务器、数据库等。

在本章中，我们将创建一个名为 `packt_selinux` 的角色，并将其应用于远程系统。在该角色中，我们将展示如何使用 Ansible 配置和执行各种 SELinux 任务。

## 安装和配置 Ansible

要安装和设置 Ansible，大多数 Linux 发行版都提供对该框架的开箱即用支持。在 CentOS 上，可以执行以下步骤。其他发行版的用户可以轻松推导出适用于其平台的步骤：

1.  你需要启用 **企业 Linux 的附加包**（**EPEL**），然后就可以轻松安装 Ansible。请在主节点（你希望管理其他系统的节点）上执行以下命令：

    ```
    # yum install epel-release
    # yum install ansible
    ```

1.  安装完成后，创建一对 SSH 密钥，用于主系统和目标系统之间的通信。使用 `ssh-keygen` 命令在主系统上创建密钥对，然后将公钥（`~/.ssh/id_rsa.pub`）复制到远程系统，保存为 `~/.ssh/authorized_keys`：

    ```
    # ssh-keygen
    # scp ~/.ssh/id_rsa.pub rem1:/root/.ssh/authorized_keys
    ```

1.  测试远程连接是否正常工作，例如，通过远程执行 `id` 命令：

    ```
    # ssh rem1 id
    ```

1.  如果测试成功，我们可以配置 Ansible，将此远程系统视为它将管理的节点之一。为此，请编辑 `/etc/ansible/hosts` 并将主机名添加到列表中：

    ```
    # cat /etc/ansible/hosts
    rem1
    ```

1.  为了查看 Ansible 是否能够正确管理远程系统，我们可以让它收集有关远程系统的所有信息。Ansible 中的事实代表了发现的远程系统设置，可用于稍后调整 playbook 和角色。例如，Ansible 发现的发行版信息可以用来选择安装时使用的包名称：

    ```
    all, reflecting all entries in /etc/ansible/hosts) to execute the tasks in the setup module. 
    ```

最后一项任务的输出是一组大量发现的事实，显示我们连接成功，并且 Ansible 已经准备好管理远程系统。

## 创建和测试 Ansible 角色

为了允许在多个系统间共享可重用的配置，Ansible 建议使用其 Ansible 角色。我们将创建一个名为`packt_selinux`的角色，让它创建一个自定义目录，然后将这个角色分配给远程系统：

1.  使用`ansible-galaxy`创建一个空的、但已准备好使用的角色：

    ```
    packt_selinux/tasks/main.yml, which will host all the settings and definitions we want to apply when we assign the packt_selinux role to a system. The other directories are, for our brief introduction to Ansible, less relevant, but play an important role in making sufficiently modular roles.
    ```

1.  编辑`main.yml`文件，并创建一个自定义目录。文件的内容应如下所示：

    ```
    ---
    - name: Create /usr/share/selinux/custom directory
       file:
            path: /usr/share/selinux/custom
            owner: root
            group: root
            mode: '0755'
            state: directory
    ```

    在后续步骤中，该文件将通过更多的块进行扩展。每个块都以一个名称开始来标识该块，然后是状态定义。在当前块中，我们使用了 Ansible 的`file`模块来断言在给定参数下文件或目录是可用的。

1.  将角色分配给远程系统并应用 playbook。我们通过首先创建一个`/etc/ansible/site.yml`文件，并添加以下内容来实现：

    ```
    ---
    - hosts: all 
      roles:
            - packt_selinux
    ```

1.  运行此 playbook，将我们角色中定义的设置应用于远程系统：

    ```
    # ansible-playbook /etc/ansible/site.yml
    ```

    Ansible 将显示其进度，以及执行更改的任务。在我们的案例中，更改意味着目录已被创建。

现在我们已经测试了我们的角色并将其分配给远程系统，接下来要做的就是逐步更新角色，直到它包含我们所需的所有逻辑。不需要其他配置，且在每次更改后，我们可以从主服务器重新运行`ansible-playbook`命令。

## 使用 Ansible 为文件系统资源分配 SELinux 上下文

在当前角色中，我们在`/usr/share/selinux`内创建了一个自定义目录。这个父目录已设置为`usr_t` SELinux 类型，因此新创建的子目录也会继承该类型。然而，该目录的 SELinux 用户会有所不同，因为 Ansible 在远程登录到系统后创建了这个目录。在默认的 CentOS 配置中，这意味着目标目录的上下文将具有`unconfined_u`作为其 SELinux 用户组件。

更新`main.yml`中的定义，并显式设置 SELinux 用户和类型：

```
---
- name: Create /usr/share/selinux/custom directory
  file:
        path: /usr/share/selinux/custom
        owner: root
        group: root
        mode: '0755'
        state: directory
        setype: 'usr_t'
        seuser: 'system_u'
```

应用更改后（使用`ansible-playbook`），更新后的定义将为该目录正确设置 SELinux 用户和 SELinux 类型。

在这种情况下，我们为文件定义添加了两个参数：`setype`和`seuser`。Ansible 文件模块支持以下与 SELinux 相关的参数：

+   `seuser` 是资源的 SELinux 用户。对于系统资源，设置为 `system_u`，如示例中所使用的。

+   `serole` 是资源的 SELinux 角色。通常不使用此参数，因为系统上的角色继承通常会导致资源被标记为 `object_r` 角色，这在大多数情况下是正确的。

+   `setype` 是资源的 SELinux 类型，是文件模块中最常用的 SELinux 参数。

+   `selevel` 是资源的 SELinux 敏感度级别，默认值为 `s0`。

正如我们从示例中已经学到的，如果继承的上下文是正确的，你不需要声明类型。

## 使用 Ansible 加载自定义 SELinux 策略

Ansible 当前的版本不支持加载自定义的 SELinux 模块。虽然在 Ansible galaxy（一个贡献者可以添加更多模块的生态系统）中可以找到自定义模块，但我们来看一下如何将自定义策略分发到 Ansible 控制的系统上，并在模块尚未加载时加载它。

虽然我们可以开始创建自定义模块，但我们将使用现有角色中的任务组合来实现这一目标。我们将按顺序尝试完成以下任务：

1.  将名为 `test.cil` 的自定义策略上传到远程系统。

1.  检查此自定义策略是否已经加载。

1.  加载自定义策略，但仅在前面的检查失败时。

这三个任务通过三个模块处理：`copy` 模块、`shell` 模块和 `command` 模块。我们将在不同的步骤中使用每个模块：

1.  在本章前面提到的自定义策略，通过将 `test.cil` 文件放入 `packt_selinux` 角色的 `files/` 文件夹中来创建。

1.  在角色的 `main.yml` 文件中创建一个新的代码块，内容如下：

    ```
    - name: Upload test.cil file to /usr/share/selinux/custom
      copy:
            src: test.cil
            dest: /usr/share/selinux/custom/test.cil
            owner: root
            group: root
            mode: '0644'
    ```

    这将确保 `test.cil` 文件（当前在主机上）被分发到目标节点，存放在我们之前创建的目录中。

1.  接下来，我们检查策略是否已经加载。为此，我们使用 `shell` 模块，并稍后使用失败或成功的状态。因此，我们将返回值存储在 `test_is_loaded` 变量中，并明确告诉 Ansible 忽略失败，因为我们将其作为检查，而不是状态定义：

    ```
    - name: Check if test SELinux module is loaded
      shell: /usr/sbin/semodule -l | grep -q ^test$
      register: test_is_loaded
      ignore_errors: True
    ```

1.  `command` 模块加载策略文件，且仅在前一个任务失败时加载：

    ```
    - name: Load test.cil if not loaded yet
      command: /usr/sbin/semodule -i /usr/share/selinux/custom/test.cil
      when: test_is_loaded is failed
    ```

这种方法展示了如何利用我们对 SELinux 的了解来定义和设置状态。这个方法也可以应用于其他 SELinux 设置，例如，在定义或调整设置之前，通过验证列表输出（例如，使用 `semanage`）来进行检查。

## 使用 Ansible 开箱即用的 SELinux 支持

Ansible 提供了许多模块，原生支持多种与 SELinux 相关的设置，以下是我们简要介绍的：

+   `selinux` 模块可用于设置或更改 SELinux 状态（强制或宽容），以及选择适当的 SELinux 策略类型（例如，targeted）：

    ```
    - name: Set SELinux to enforcing mode
      selinux:
              policy: targeted
              state: enforcing
    ```

+   使用`seboolean`模块，可以随意调整 SELinux 布尔值：

    ```
    - name: Set httpd_builtin_scripting to true
      seboolean:
              name: httpd_builtin_scripting
              state: yes
    ```

+   `sefcontext`模块允许我们更改 SELinux 文件上下文定义：

    ```
    - name: Set the context for /srv/web
      sefcontext:
              target: '/srv/web(/.*)?'
              setype: httpd_sys_content_t
              state: present
    ```

+   使用`selinux_permissive`，我们可以选择性地将某些 SELinux 策略域标记为宽松模式：

    ```
    - name: Set zoneminder_t as permissive domain
      selinux_permissive:
              name: zoneminder_t
              permissive: true
    ```

+   `selogin`模块可以用来将登录映射到 SELinux 用户，就像`semanage login`一样：

    ```
    - name: Map taylor's login to the unconfined_u user
      selogin:
              login: taylor
              seuser: unconfined_u
              state: present
    ```

+   `seport`可以用来创建 SELinux 端口映射：

    ```
    - name: Set port 10122 to ssh_port_t
      seport:
              ports: 10122
              proto: tcp
              setype: ssh_port_t
              state: present
    ```

其他 SELinux 设置可能通过自定义模块得到支持，但通过前面介绍的方法，管理员已经可以开始在其环境中的所有系统上配置 SELinux。

# 使用 SaltStack 配置 SELinux

我们要考虑的第二个编排和自动化框架是 SaltStack，它得到了 SaltStack 公司商业支持。SaltStack 使用类似于 Ansible 的声明式语言，并且也是用 Python 编写的。在本章中，我们将使用开源的 SaltStack 框架，但也有 SaltStack 的企业版，它在开源版本的基础上增加了更多的功能。

## SaltStack 的工作原理

**SaltStack**，通常也被称为 Salt，是一个自动化框架，使用代理/服务器模型进行集成。与 Ansible 不同，SaltStack 通常要求在目标节点（称为**minions**）上安装代理，并激活 minion 守护进程以实现与主服务器的通信。此通信是加密的，minion 认证使用公钥验证，必须在主服务器上进行批准，以确保没有恶意 minion 参与 SaltStack 环境。

尽管 SaltStack 也支持无代理安装，但我们将重点介绍基于代理的部署。在这种配置下，minions 会定期与主服务器检查是否需要应用任何更新。但管理员无需等到 minion 拉取最新的更新：你还可以从主服务器触发更新，从而将更改推送到节点。

minion 应该处于的目标状态被写入以`.sls`为后缀的文件中。这些 Salt State 文件可以引用其他状态文件，从而实现模块化设计并在多台机器上重用。

如果我们需要更复杂的编码，SaltStack 支持创建和分发模块，称为**Salt 执行模块**。然而，与 Ansible 的 Galaxy 不同，目前没有社区仓库来寻找更多的执行模块。

## 安装和配置 SaltStack

SaltStack 的安装在不同的 Linux 发行版中类似。让我们看看在 CentOS 机器上是如何安装的：

1.  我们首先需要启用包含 SaltStack 软件的仓库。该项目通过 RPM 文件来维护仓库定义，这些文件可以立即安装：

    ```
    # yum install https://repo.saltstack.com/py3/redhat/salt-py3-repo-latest.el8.noarch.rpm
    ```

1.  一旦我们在所有系统上启用了软件仓库，就可以在主服务器上安装`salt-master`，并在远程系统上安装`salt-minion`：

    ```
    master ~# yum install salt-master
    remote ~# yum install salt-minion
    ```

1.  在启动系统上的守护进程之前，我们首先更新 minion 配置以指向主节点。默认情况下，minion 会尝试连接到主机名为 `salt` 的主节点，但可以通过编辑 `/etc/salt/minion` 并设置正确的主机名来轻松更改：

    ```
    remote ~# vim /etc/salt/minion
    master: ppubssa3ed
    ```

1.  配置好 minion 后，我们现在可以启动 SaltStack 的主节点（`salt-master`）和 minion（`salt-minion`）守护进程：

    ```
    master ~# systemctl start salt-master
    remote ~# systemctl start salt-minion
    ```

1.  Minion 将连接到主节点并展示其公钥。要列出当前连接的代理，可以使用 `salt-key -L`：

    ```
    master ~# salt-key -L
    Accepted Keys:
    Denied Keys:
    Unaccepted Keys:
    rem1.internal.genfic.local
    Rejected Keys:
    master ~# salt-key -a rem1.internal.genfic.local
    The following keys are going to be accepted:
    Unaccepted Keys:
    rem1.internal.genfic.local
    Proceed? [n/Y] y
    Key for minion rem1.internal.genfic.local accepted.
    ```

1.  一旦我们接受了密钥，主节点就会知道并控制该 minion。让我们看看是否能够与远程系统正确交互：

    ```
    master ~# salt '*' service.get_all
    ```

    这个命令将列出 minion 上的所有系统服务。

`salt` 命令是用于从主节点查询和与远程 minion 交互的主要命令。如果上一个命令成功返回了所有系统服务，那么 SaltStack 就已经正确配置并准备好管理远程系统。

## 创建并测试我们的 SELinux 状态与 SaltStack

让我们创建一个名为 `packt_selinux` 的 SELinux 状态，并将其应用到远程 minion 上：

1.  我们首先需要创建顶级文件。此文件是 SaltStack 的主配置文件，通过它来配置整个环境：

    ```
    master ~# mkdir /srv/salt
    master ~# vim /srv/salt/top.sls
    base:
      '*':
        - packt_selinux
    ```

1.  接下来，我们为 `packt_selinux` 创建状态定义：

    ```
    init.sls file is the main state file for this packt_selinux state. So, when SaltStack reads the top.sls file, it sees a reference to the packt_selinux state and then searches for the init.sls file inside this state.
    ```

1.  将本章前面定义的 SELinux `test.cil` 模块放置在 `/srv/salt/packt_selinux` 目录下，这也是我们在状态定义中引用的位置。放置后，我们可以将此状态应用到环境中：

    ```
    master ~# salt '*' state.apply
    ```

`salt` 命令的 `state.apply` 子命令用于将状态应用到整个环境。每次修改状态定义时，可以使用此命令强制更新 minions。否则，minions 将默认每 60 分钟更新一次其状态。这些计划的状态更新称为 mine 更新，并在 `/etc/salt/minion` 中的代理上进行配置。

## 使用 SaltStack 为文件系统资源分配 SELinux 上下文

在写作时，SaltStack 尚未在稳定版本中完全支持在资源中处理 SELinux 类型。然而，SaltStack 支持运行命令，但只有在某个测试成功（或失败）之后才能执行。

更新 `init.sls` 文件，并在其中添加以下代码：

```
{%- set path = '/usr/share/selinux/custom/test.cil' %}
{%- set context = 'system_u:object_r:usr_t:s0' %}
set {{ path }} context:
  cmd.run:
    - name: chcon {{ context}} {{ path }}
    - unless: test $(stat -c %C {{ path }}) == {{ context }}
path and context) so that we do not need to iterate the path and context multiple times, and then use these variables in a cmd.run call.
```

`cmd.run` 方法允许我们通过本书前面所见的命令轻松创建自定义 SELinux 支持。`unless` 检查包含一个测试，以判断是否需要执行该命令，从而允许我们创建幂等的状态定义。

## 使用 SaltStack 加载自定义 SELinux 策略

让我们在远程系统上加载我们的自定义 SELinux 模块。SaltStack 支持通过 `selinux.module` 状态加载 SELinux 模块：

```
load test.cil:
  selinux.module:
    - name: test
    - source: /usr/share/selinux/custom/test.cil
    - install: True
    - unless: "semodule -l | grep -q ^test$"
```

与前一部分一样，我们需要添加一个 `unless` 语句，否则每次应用状态时，SaltStack 都会尝试重复加载 SELinux 模块。

## 使用 SaltStack 内置的 SELinux 支持

SaltStack 对 SELinux 的原生支持正在逐步扩展，但仍有很大的改进空间：

+   使用`selinux.boolean`，可以在目标机器上设置 SELinux 布尔值：

    ```
    httpd_builtin_scription:
      selinux.boolean:
        - value: True
    ```

+   文件上下文（由`semanage fcontext`管理）可以使用`selinux.fcontext_policy_present`状态进行定义：

    ```
    "/srv/web(/.*)?":
      selinux.fcontext_policy_present:
        - sel_type: httpd_sys_content_t
    ```

+   若要删除定义，请使用`selinux.fcontext_policy_absent`定义。

+   使用`selinux.mode`，我们可以将系统置于强制模式或宽容模式：

    ```
    enforcing:
      selinux.mode
    ```

+   端口映射通过`selinux.port_policy_present`状态来处理：

    ```
    tcp/10122:
      selinux.port_policy_present:
        - sel_type: ssh_port_t
    ```

使用前面提到的`cmd.run`方法，我们可以以可重复的方式将 SELinux 配置更新应用于不支持的设置的系统。

# 使用 Puppet 自动化系统管理

Puppet 是我们将要检查的第三个自动化框架。它是我们列表中最古老的一个，首次发布于 2005 年，通常被视为其他自动化框架的基准。它得到了 Puppet 公司（也常被称为 Puppet Labs）的商业支持。

## Puppet 如何工作

与 SaltStack 类似，**Puppet**使用基于代理/服务器的模型，并通过公钥认证代理，确保环境中没有恶意代理活跃。

**Puppet 主节点**可以访问**Puppet 清单**，这是 Puppet 希望实现的状态声明。这些清单使用一种受到 Ruby 启发的特定语言，并可以引用由模块提供的类，以确保在环境中具有可重用性。

**Puppet 模块**因此是 Puppet 的工作马，而 Puppet 拥有一个重要的社区，称为 Puppet Forge，允许您下载和安装社区创建的模块，以更轻松地管理您的环境。

Puppet 代理会定期连接到主节点，向主节点报告远程机器的当前详细信息。这些当前详细信息被称为**事实**，Puppet 可以利用这些信息动态处理环境中的变化。主节点随后将目标状态编译成所谓的**目录**并将该目录发送到代理。代理然后应用此目录并报告结果。

## 安装和配置 Puppet

Puppet 公司提供多个 Linux 发行版的集成包。以下说明重点介绍与 RPM 兼容的发行版，但其他平台也有非常相似的说明：

1.  Puppet 公司通过 RPM 文件提供存储库定义。在建立存储库后，您可以安装`puppetserver`和`pdk`（在主节点上）以及`puppet-agent`（在远程系统上），使软件随时可用：

    ```
    # yum install https://yum.puppet.com/puppet6-release-el-8.noarch.rpm
    master ~# yum install puppetserver pdk
    remote ~# yum install puppet-agent
    ```

1.  配置主节点，使其证书命名正确。编辑`/etc/puppetlabs/puppet`中的`puppet.conf`文件，并在`[master]`部分内更新或添加以下设置：

    ```
    master ~# vim /etc/puppetlabs/puppet/puppet.conf
    certname = ppubssa3ed.internal.genfic.local
    server = ppubssa3ed.internal.genfic.local
    environment = production
    ```

1.  启动 Puppet 服务器，以便客户端可以开始连接到它：

    ```
    master ~# systemctl start puppetserver
    ```

1.  在远程系统上，编辑相同的配置文件，并在`[main]`部分更新或添加以下设置：

    ```
    remote ~# vim /etc/puppetlabs/puppet/puppet.conf
    [main]
    certname = rem1.internal.genfic.local
    server = ppubssa3ed.internal.genfic.local
    environment = production
    runinterval = 1h
    ```

1.  接下来，启动 Puppet 代理：

    ```
    remote ~# systemctl start puppet
    ```

1.  在主节点上，我们现在可以查询待处理的证书请求。它应该显示我们最近启动的代理的请求：

    ```
    master ~# /opt/puppetlabs/bin/puppetserver ca list
    Requested Certificates:
      rem1.internal.genfic.local   (SHA256) ...
    ```

1.  我们可以通过以下方式接受此请求（签署证书）：

    ```
    master ~# /opt/puppetlabs/bin/puppetserver ca sign --certname rem1.internal.genfic.local
    Successfully signed certificate request for rem1.internal.genfic.local
    ```

1.  为了验证连接是否正常工作，请登录到远程机器并触发代理应用（当前为空的）目录：

    ```
    remote ~# /opt/puppetlabs/bin/puppet agent --test
    ```

与 SaltStack 不同，Puppet 依赖代理频繁轮询服务器，而不是将更改推送到代理。在我们之前做的配置中，我们将代理配置为每小时检查一次。通过`puppet agent --test`命令，我们可以立即指示代理运行状态检查。

## 使用 Puppet 创建和测试 SELinux 类

让我们创建`packt_selinux`类，通过它我们将配置远程机器的 SELinux 设置：

1.  调用`/etc/puppetlabs/code/modules`目录：

    ```
    master ~# cd /etc/puppetlabs/code/modules
    master ~# pdk new module packt_selinux --skip-interview
    ```

    结果是一个空模块，包含许多默认文件和目录。我们将主要与模块的清单文件一起工作。

1.  在`packt_selinux/manifests`目录中，创建一个名为`init.pp`的新文件，内容如下：

    ```
    class packt_selinux {
      file { "/usr/share/selinux/custom":
        ensure => directory,
        mode => "0755",
      }
    }
    ```

1.  接下来，在`/etc/puppetlabs/code/environments/production/manifests`位置创建一个名为`site.pp`的文件，内容如下：

    ```
    node 'rem1.internal.genfic.local' {
      include packt_selinux
    }
    ```

    `site.pp`文件为 Puppet 提供了顶级层次结构，用于将其环境与适当的定义关联。在此示例中，主机名为`rem1.internal.genfic.local`的节点通过引用我们之前创建的`packt_selinux`模块进行配置。

    在`packt_selinux`模块内部，我们创建了`packt_selinux`类，该类目前由一个指令组成，用于创建`/usr/share/selinux/custom`。

1.  设置这些定义后，让远程代理更新其状态：

    ```
    remote ~# puppet agent -t
    ```

    在产品环境中，通常会定期安排此命令执行，或者持续运行 Puppet 代理作为守护进程。

在将类正确分配给节点后，我们可以通过更多的 SELinux 细节扩展我们的配置。

## 使用 Puppet 为文件系统资源分配 SELinux 上下文

让我们通过以下代码段增强当前的类定义：

```
file { 'selinux_custom_module_test':
  path => "/usr/share/selinux/custom/test.cil",
  ensure => file,
  owner => "root",
  group => "root",
  source => "puppet:///modules/packt_selinux/test.cil",
  require => File["/usr/share/selinux/custom"],
  seltype => "usr_t",
}
```

为了使此块正常工作，我们需要将`test.cil` SELinux 模块放置在`packt_selinux`模块位置中的`files/`文件夹内。此块将使 Puppet 将文件上传到该目录，并设置依赖关系，要求该目录必须存在。`require`语句指向先前定义的块。

我们还看到 Puppet 对 SELinux 类型定义提供了开箱即用的支持。文件类具有几个支持 SELinux 的参数，可以使用：

+   `seluser`定义了资源的 SELinux 用户。

+   `selrole`定义了资源的 SELinux 角色。

+   `seltype`定义了资源的 SELinux 类型。

+   `selrange`定义了资源的 SELinux 敏感度范围。

+   `selinux_ignore_defaults`告诉 Puppet 忽略默认的 SELinux 上下文（如从 SELinux 策略查询到的）。

我们之前的示例实际上是多余的，因为 Puppet 会主动查询 SELinux 策略以发现正确的资源上下文并应用。设置`selinux_ignore_defaults`为`true`时，Puppet 将不会查询并调整上下文，这在测试没有正确上下文定义的新配置时非常有用。

## 使用 Puppet 加载自定义 SELinux 策略

Puppet 确实支持加载和管理 SELinux 模块。然而，它的支持目前仅限于传统的 SELinux 策略模块，而不包括基于 CIL 的模块。

所以，让我们在模块定义中创建另一个块，加载`test.cil`文件，但仅在没有已加载测试 SELinux 模块的情况下：

```
exec { '/usr/sbin/semodule -i /usr/share/selinux/custom/test.cil':
  require => File['selinux_custom_module_test'],
  unless => '/usr/sbin/semodule -l | grep -q ^test$',
}
```

这种方法允许我们在本地 Puppet 支持不足的情况下，创建自定义的 SELinux 配置调整。

## 使用 Puppet 的开箱即用的 SELinux 支持

Puppet 有一些开箱即用的 SELinux 相关类，并通过 Puppet Forge 提供更多支持，Puppet Forge 是一个由社区贡献模块组成的生态系统。我们推荐的一个模块是`puppet-selinux`模块，Puppet（公司）在 Puppet Forge 上维护该模块（因此更有可能在 Puppet 的后续版本中保持支持）。

安装新模块非常简单，可以使用`puppet module`命令：

```
master ~# /opt/puppetlabs/bin/puppet module install puppet-selinux
```

然后我们可以在清单中引用通过此模块提供的`selinux`类：

+   可以直接使用`selinux`类来设置系统的强制（或宽松）状态：

    ```
    class { selinux:
      mode => 'enforcing',
      type => 'targeted',
    }
    ```

+   （本地）`selboolean`类可以用于设置 SELinux 布尔值：

    ```
    selboolean { 'httpd_builtin_scripting':
      value => off,
    }
    ```

+   SELinux 文件上下文可以使用`selinux::fcontext`类定义：

    ```
    selinux::fcontext { '/srv/web(/.*)?':
      seltype => 'httpd_sys_content_t',
    }
    ```

+   文件上下文的等效定义由`selinux::fcontext::equivalence`处理，方式如下：

    ```
    selinux::fcontext::equivalence { '/srv/www':
      ensure => 'present',
      target => '/srv/web',
    }
    ```

+   自定义端口映射由`selinux::port`处理：

    ```
    selinux::port { 'set_ssh_custom_port':
      ensure => 'present',
      seltype => 'ssh_port_t',
      protocol => 'tcp',
      port => 10122,
    }
    ```

+   可以使用`selinux::permissive`将单独的 SELinux 域设置为宽松模式：

    ```
    selinux::permissive { 'zoneminder_t':
      ensure => 'present',
    }
    ```

+   如果存在标准的 SELinux 模块，则使用`selmodule`可以加载它。在这种情况下，它将在`selmoduledir`引用的目录中查找以块名称命名的 SELinux 模块：

    ```
    selmodule { 'vlock':
      ensure => 'present',
      selmoduledir => '/usr/share/selinux/custom',
    }
    ```

尽管在 Puppet Forge 上可能会有其他支持 SELinux 的模块，但请确保验证这些模块是否成熟且足够稳定。如果它们的支持不确定，您可能需要采用之前使用过的`exec`方式，如在*使用 Puppet 加载自定义 SELinux 策略*一节中所示。

# 使用 Chef 进行系统自动化

我们将要探讨的最后一个自动化框架是 Chef。Chef 相比于前面提到的框架，更偏向于手动操作和开发导向，但它仍然非常强大。它有类似名称的公司 Chef 提供商业支持。

## Chef 的工作原理

Chef 对自动化有着稍微更为广泛的方法，并且需要更多的工作来启动和运行。然而，一旦设置完成，它提供了一个非常灵活和可编程的环境，可以在其中完成基础设施自动化。

Chef 架构中有三种类型的系统：

+   **Chef 服务器** 充当中央枢纽，用于维护自动化代码，并与远程系统交互以应用更改。

+   **Chef 工作站** 是管理员和工程师开发 Chef recipes（代码）和 cookbooks 并与 Chef 服务器交互的端点。每个 Chef 环境可以有多个 Chef 工作站。

+   **Chef 客户端** 是在由 Chef 环境管理的远程系统（节点）上运行的代理。

开发者在 **recipes** 中创建自动化代码，这些代码类似于任务。多个 recipes 被打包在一个 **cookbook** 中，并上传到 Chef 服务器，然后可以将这些 recipes 应用于一个或多个节点。可以将 cookbooks 与之前的自动化框架中的模块进行比较。

Chef 客户端和服务器使用基于公钥的身份验证和加密进行它们的交互。客户端采取主动措施，连接服务器以下载最新的 cookbooks 和其他资源，然后计算并应用最新的更改，并将这些更改的反馈发送回服务器。

## 安装和配置 Chef

完整的 Chef 安装需要管理员安装几个组件。Chef 工作站和 Chef 服务器需要管理员安装，而 Chef 代理将由 Chef 后续安装。

### 安装 Chef 工作站

要安装和使用 Chef，请先下载 Chef 工作站。所有 Chef 软件都可以从 [`downloads.chef.io`](https://downloads.chef.io) 下载。对于 CentOS，Chef 工作站以 RPM 的形式提供，可以使用 `yum` 进行安装。

但是，与常见的打包软件不同，Chef 工作站的依赖关系没有显式列出为 RPM 依赖关系，导致软件在没有必要的库的情况下安装。安装结束时，RPM 文件将执行一个后安装脚本，检查依赖关系并报告缺少的库：

```
master ~# yum install chef-workstation-0.17.5-1.el7.x86_64.rpm
```

目前，依赖关系需要安装以下 CentOS 软件包：

```
master ~# yum install libX11-xcb libXcomposite libXcursor libXdamage nss gdk-pixbuf2 gtk3 libXScrnSaver alsa-lib git
```

安装完成后，作为常规非根用户运行 `chef -v` 来验证是否满足所有依赖：

```
master ~$ chef -v
```

命令应输出包含的 Chef 组件的版本。

### 安装和配置 Chef 服务器核心

第二个安装是 Chef 服务器核心。此软件再次以 RPM 的形式提供：

1.  使用 `yum` 安装 Chef 服务器核心：

    ```
    master ~# yum install chef-server-core-13.2.0-1.el7.x86_64.rpm
    ```

    安装完成后，我们需要为我们的环境配置它。

1.  创建一个名为 `/var/opt/chef` 的目录。我们将使用这个目录来存储用于身份验证 Chef 服务器的加密密钥：

    ```
    master ~# mkdir /var/opt/chef
    ```

1.  接下来，使用 `chef-server-ctl` 配置 Chef 服务器：

    ```
    master ~# chef-server-ctl reconfigure
    ```

    这将在当前系统上设置 Chef 服务器。此设置可能需要一些时间才能完成，但完成后，我们可以继续在 Chef 中创建用户帐户。

1.  让我们在此系统上为用户`lisa`创建一个名为`chefadmin`的帐户，并为其设置自定义密码：

    ```
    master ~# chef-server-ctl user-create chefadmin Lisa McCarthy lisa@ppubssa3ed.internal.genfic.local pw4chef --filename /var/opt/chef/chefadmin.pem
    ```

1.  在 Chef 配置中创建一个组织单元，并将其与新创建的用户关联：

    ```
    master ~# chef-server-ctl org-create ppubssa3ed "Packt Pub SSA 3rd Edition" --association_user chefadmin --filename /var/opt/chef/ppubssa3ed-validator.pem
    ```

完成这些后，服务器管理已经完成，我们可以开始创建我们的开发环境。

### 准备开发环境

如前所述，Chef 比以前的自动化框架更偏向开发。将与 Chef 交互的用户（使用 Chef 工作站）需要首先建立开发环境：

1.  我们之前为用户`lisa`创建了一个名为`chefadmin`的帐户。现在，以用户`lisa`身份登录并在用户的主目录中创建开发环境：

    ```
    master ~$ mkdir chef
    master ~$ cd chef
    master ~$ git init
    ```

1.  我们创建一个支持 Git 的环境，因为 Chef 工具需要它。如果你还没有激活 Git 配置，你可能需要添加你的电子邮件和名字：

    ```
    master ~$ git config --global user.email "lisa@ppubssa3ed.internal.genfic.local"
    master ~$ git config --global user.name "Lisa McCarthy"
    ```

1.  接下来，在此环境中创建 Chef 刀具配置文件`.chef/knife.rb`（在我们的示例中是`~/chef/.chef/knife.rb`）：

    ```
    chefadmin.pem in our example) and adjust the configuration accordingly.
    ```

1.  下载 Chef 服务器使用的证书（这些证书是自签名证书），然后检查 SSL 连接：

    ```
    master ~$ knife ssl fetch
    master ~$ knife ssl check
    ```

1.  如果检查成功，我们可以提交更改：

    ```
    master ~$ git add -A
    master ~$ git commit -m 'Chef configuration baseline'
    ```

我们现在已经准备好开始我们的食谱和食谱开发了。

## 创建 SELinux 食谱

我们将要开发的食谱将包含各种 SELinux 配置项，然后将这些配置项分配给远程节点：

1.  让我们从创建一个名为`packt_selinux`的食谱开始：

    ```
    metadata.rb and recipes/default.rb. The metadata.rb file contains information about the cookbook and, while it is not necessary for our example, it is sensible to edit and update this file immediately. Later, we will adjust this file to include dependency information toward other cookbooks.
    ```

1.  `recipes/default.rb`文件包含我们希望应用于远程系统的实际逻辑。让我们为`/usr/share/selinux/custom`目录创建一个定义：

    ```
    master ~$ vim recipes/default.rb
    directory '/usr/share/selinux/custom' do
      owner 'root'
      group 'root'
      mode '0755'
      action :create
    end
    ```

1.  现在将食谱上传到 Chef 服务器：

    ```
    master ~$ knife cookbook upload packt_selinux
    ```

1.  我们可以使用`list`子命令查询 Chef 服务器上可用的食谱：

    ```
    master ~$ knife cookbook list
    packt_selinux   0.1.0
    ```

1.  有了食谱可用后，让我们启动目标节点。引导过程只需要执行一次，但必须由 Chef 认证用户触发：

    ```
    master ~$ knife bootstrap rem1 --ssh-user root --node-name rem1
    ```

1.  这确保了 Chef 服务器知道远程系统。我们可以使用`knife node list`查询节点，并使用`show`子命令获取有关节点的更多详细信息：

    ```
    master ~$ knife node show rem1
    ```

1.  使用`run_list add`子命令将`packt_selinux`食谱分配给节点：

    ```
    chef-client binary executes either regularly (through a cron job or similar) or starts as a daemon.
    ```

1.  对于我们的目的，我们将在远程系统上触发`chef-client`命令，以下载并应用最新的更改：

    ```
    chef-client should show how it found and applied the changes listed in the recipe.
    ```

如果此命令成功返回，那么 Chef 已经准备好使用我们开发的食谱来管理远程系统。

## 使用 Chef 为文件系统资源分配 SELinux 上下文

Chef 对 SELinux 上下文的原生支持有限。当被指示在节点上创建或修改文件时，它会根据节点上当前的文件上下文定义重新标记这些文件。然而，我们可以订阅在食谱中定义的事件，并在这些事件发生时触发适当的操作。例如，要显式设置目录的上下文，我们可以创建类似这样的内容：

```
execute 'set_selinux_custom_context' do
  command '/usr/bin/chcon -t usr_t /usr/share/selinux/custom'
  action :nothing
  subscribes :run, 'directory[/usr/share/selinux/custom]', :immediately
end
```

在将其添加到`recipes/default.rb`文件后，我们首先需要将更新后的 cookbook 上传到服务器：

```
master ~$ knife cookbook upload packt_selinux
```

之后，我们可以在远程节点上重新运行`chef-client`以应用此更新的食谱。如果目录之前已经创建，则食谱不会更改任何内容，因为订阅不会被触发。

## 使用 Chef 加载自定义 SELinux 策略

让我们更新我们的食谱，以包括加载自定义策略的逻辑。我们将在食谱中使用两个块，一个用于将`test.cil`文件上传到节点，另一个用于加载它，但仅在之前未加载时：

```
cookbook_file '/usr/share/selinux/custom/test.cil' do
  source 'test.cil'
  owner 'root'
  group 'root'
  mode '0755'
  action :create
end
bash 'load_test_cil' do
  code '/usr/sbin/semodule -i /usr/share/selinux/custom/test.cil'
  not_if '/usr/sbin/semodule -l | grep -q ^test$'
  only_if { ::File.exists?('/usr/share/selinux/custom/test.cil') }
end
```

将`test.cil`文件放入`packt_selinux` cookbook 目录下名为`files`的文件夹中，然后上传更新后的 cookbook 并使用`chef-client`重新应用更改。

## 使用 Chef 的开箱即用 SELinux 支持

虽然 Chef 本身对 SELinux 的原生支持有限，但在 Chef 超市（Chef 社区管理和分发自定义 cookbook 的地方）在线提供了许多 cookbook。Chef（公司）本身维护`selinux` cookbook，它允许管理系统的 SELinux 状态，而`selinux_policy` cookbook 处理一些其他 SELinux 设置。

让我们下载并安装`selinux`和`selinux_policy` cookbooks：

```
master ~$ knife supermarket install selinux_policy
master ~$ knife supermarket install selinux
master ~$ knife cookbook upload selinux_policy
master ~$ knife cookbook upload selinux
```

接下来，调整我们自己的 cookbook 中的`metadata.rb`文件，添加对新添加的 cookbook 的依赖：

```
depends 'selinux_policy'
depends 'selinux'
```

现在，我们可以使用一些预定义的食谱来处理 SELinux 配置设置：

+   使用`selinux_state`，我们可以将系统置于强制模式或宽容模式：

    ```
    selinux_state "SELinux enforcing" do
      action :enforcing
    end
    ```

+   `selinux_policy_boolean`食谱可以配置 SELinux 布尔值：

    ```
    selinux_policy_boolean 'httpd_builtin_scripting' do
      value false
    end
    ```

+   使用`selinux_policy_port`，可以定义自定义的 SELinux 端口映射：

    ```
    selinux_policy_port '10122' do
      protocol 'tcp'
      secontext 'ssh_port_t'
    end
    ```

+   可以使用`selinux_policy_fcontext`设置文件上下文定义：

    ```
    selinux_policy_fcontext '/srv/web(/.*)?' do
      secontext 'httpd_sys_content_t'
    end
    ```

+   可以使用`selinux_policy_permissive`食谱将 SELinux 域设置为宽容模式：

    ```
    selinux_policy_permissive 'zoneminder_t' do
    end
    ```

在远程系统上调用`chef-client`之前，别忘了上传更改后的 cookbook。

# 总结

像 Ansible、SaltStack、Puppet 和 Chef 这样的自动化框架可以轻松用于管理多个系统上的 SELinux 设置。虽然并非所有框架都能原生处理 SELinux 设置，但通过使用社区提供的模块或创建自定义规则来检查和更新设置，可以轻松解决这个问题。在本章中，我们已经看到如何通过安装基于 CIL 的自定义 SELinux 策略来实现这一点。

我们了解到这些框架都有各自的方法。例如，Ansible 不使用远程系统上的任何软件安装，并使用 SSH 与目标系统通信。其他框架都使用代理/服务器模型，但在配置设置（Puppet 和 SaltStack 之间的语法明显不同）或设计（Chef 使用开发人员拥有其开发环境的工作站）上有各自的观点。所有这些框架都很容易安装和配置，并且可以处理大多数 SELinux 设置而不会出现任何问题。所有工具都有一种模块化定义的方式，因此可以轻松地应用到更多的系统中。

现在我们知道如何一致地应用 SELinux 设置，让我们看看还有哪些 SELinux 控制存在，但现在通过用户空间应用程序特定的支持。

# 问题

1.  哪一款工具原生支持在资源上设置 SELinux 上下文？

1.  这些编排工具如何允许超出原生支持的可重用定制？

1.  列出的编排工具之间有哪些明显的区别？
