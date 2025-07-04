# 第十一章：增强容器化工作负载的安全性

容器平台和管理框架为管理员和开发者提供了应用层抽象。轻量级容器框架允许快速开发和部署新应用，而较重的容器平台则可以实现资源的最优消耗和高度弹性的托管平台。

SELinux 在许多框架和平台中发挥着重要作用，确保不受信任的容器无法逃脱或与它们不应交互的资源进行交互。在本章中，我们将探讨 SELinux 如何得到支持，从`systemd-nspawn`到`podman`（以及 Docker），最后是在更大的环境中与 Kubernetes 结合使用。我们还将学习如何使用`udica`工具为容器创建自定义的 SELinux 域。

在本章中，我们将涵盖以下主要内容：

+   使用 SELinux 与 systemd 的容器支持

+   配置 podman

+   利用 Kubernetes 的 SELinux 支持

# 技术要求

查看以下视频，观看代码实际运行： [`bit.ly/34aHOfl`](https://bit.ly/34aHOfl)

# 使用 SELinux 与 systemd 的容器支持

在*第七章*，*配置特定应用的 SELinux 控制*中，我们介绍了 systemd 作为一个 SELinux 感知的应用套件，能够以可配置的 SELinux 上下文启动不同的服务。除了服务支持外，systemd 还拥有其他不少功能。其中之一就是`systemd-nspawn`。

使用`systemd-nspawn`，systemd 提供容器功能，允许管理员以集成的方式与 systemd 管理的容器进行交互，几乎就像这些容器本身就是服务一样。它使用与 Linux Containers 项目（现代容器框架的前身）和 Docker 相同的原语，基于命名空间（因此有`nspawn`中的`n`）。

信息性说明

**Linux Containers 项目**有一个名为**LXC**的产品，它结合了 Linux 内核中的多个隔离和资源管理服务，如**控制组**（**cgroups**）和命名空间隔离。cgroups 允许对 CPU、内存和 I/O 的资源消耗进行限制或调节，而命名空间则允许隐藏信息并限制对系统资源的访问视图。Docker 的早期版本是建立在 LXC 之上的，尽管 Docker 后来直接使用了 Linux 服务本身，而不再使用 LXC。

在 SELinux 方面，容器内运行的软件可能无法正确地查看 SELinux 的状态（这取决于容器配置），因为容器与主机本身是隔离的。SELinux 目前尚不支持命名空间，无法让容器或其他隔离进程拥有自己的 SELinux 视图，因此，如果容器能够查看 SELinux 状态，它绝不应该被允许修改它。

现在，与可以使用我们在*第九章*中看到的 sVirt 方法的 Docker、`podman`和 Kubernetes 不同，*安全虚拟化*，`systemd-nspawn`方法不支持此技术。

说明性备注

`systemd-nspawn`命令可能没有默认安装。在 CentOS、Debian 及相关发行版中，提供此工具的软件包名为`systemd-container`。其他发行版如 Gentoo 和 Arch Linux 则将其作为默认的 systemd 安装的一部分。

让我们看看`systemd-nspawn`是如何工作的，以及它的 SELinux 支持情况。

## 初始化 systemd 容器

要创建一个 systemd 容器，我们需要在文件系统中创建一个存放其文件的位置，然后使用正确的参数调用`systemd-nspawn`。为了准备文件系统，我们可以下载预构建的容器镜像，或自行创建一个。让我们使用 Jailkit 软件，正如在*第七章*中使用的那样，*配置应用程序特定的 SELinux 控制*，并从中构建一个容器：

1.  首先，创建容器运行时将托管的目录：

    ```
    # mkdir /srv/ctr
    ```

1.  编辑`/etc/jailkit/jk_init.ini`文件，并包含以下部分：

    ```
    [nginx]
    comment = nginx runtime
    paths = /usr/sbin/nginx, /etc/nginx, /var/log/nginx, /var/lib/nginx, /usr/share/nginx, /usr/lib64/nginx, /usr/lib64/perl5/vendor_perl
    users = root,nginx
    groups = root,nginx
    includesections = netbasics, uidbasics, perl
    ```

    本节告诉 Jailkit 应该将哪些内容复制到目录中，并支持哪些用户。

1.  执行`jk_init`命令以填充目录：

    ```
    # jk_init -v -j /srv/ctr/nginx nginx
    ```

1.  最后，使用`systemd-nspawn`启动容器：

    ```
    # systemd-nspawn -D /srv/ctr/nginx /usr/sbin/nginx \
      -g "daemon off;"
    ```

由于 Nginx 默认会尝试作为守护进程运行，因此容器会立即停止，因为它没有活动进程。通过使用`daemon off`选项启动，`nginx`将保持在前台，容器可以继续工作。

## 使用特定的 SELinux 上下文

当我们直接启动容器时，该容器将以用户的 SELinux 上下文运行。然而，我们可以通过命令行参数传递容器的目标上下文：

+   `--selinux-context=`选项（简写为`-Z`）允许管理员为容器的运行时进程定义 SELinux 上下文。

+   `--selinux-apifs-context=`选项（简写为`-L`）允许管理员为容器的文件和文件系统定义 SELinux 上下文。

然而，必须仔细选择可用的 SELinux 类型。容器内运行的进程不能执行任何类型的转换，因此通常无法使用常规的 SELinux 域。以我们的 Nginx 示例为例，`httpd_t`域不能用于该容器。

我们可以使用发行版为容器工作负载提供的 SELinux 类型。最近的 CentOS 版本将使用如`container_t`（以前称为`svirt_lxc_net_t`）这样的域，以及面向文件的 SELinux 类型`container_file_t`。虽然这个域不具备容器所需的所有可能权限，但它为容器提供了一个良好的基准。

让我们为我们的容器使用这种类型：

1.  首先，我们需要通过一些附加权限扩展`container_t`权限，以支持`nginx`守护进程。创建一个包含以下内容的 CIL 策略文件：

    ```
    (typeattributeset cil_gen_require container_t)
    (typeattributeset cil_gen_require container_file_t)
    (typeattributeset cil_gen_require http_port_t)
    (typeattributeset cil_gen_require node_t)
    (allow container_t container_file_t (chr_file (read open getattr ioctl write)))
    (allow container_t self (tcp_socket (create setopt bind listen accept read write)))
    (allow container_t http_port_t (tcp_socket (name_bind)))
    (allow container_t node_t (tcp_socket (node_bind)))
    (allow container_t self (capability (net_bind_service setgid setuid)))
    ```

1.  将此文件作为新的 SELinux 模块加载：

    ```
    # semodule -i custom_container.cil
    ```

1.  使用`container_file_t` SELinux 类型重新标记容器的文件：

    ```
    # chcon -R -t container_file_t /srv/ctr/nginx
    ```

1.  使用适当的标签启动容器：

    ```
    # systemd-nspawn -D /srv/ctr/nginx \
    -Z system_u:system_r:container_t:s0 \
    -L system_u:object_r:container_file_t:s0 \
    /usr/sbin/nginx -g "daemon off;"
    ```

每当容器启动时，它都会保持附加到当前会话。当然，我们可以创建服务文件来在后台启动容器，或者使用`screen`或`tmux`等会话管理服务。然而，更加用户友好的方法是使用`machinectl`。

## 使用 machinectl 简化容器管理

`machinectl`命令允许管理员通过 systemd 更轻松地管理容器甚至虚拟机。对于容器，`machinectl`将使用`systemd-nspawn`。

让我们使用这个`machinectl`命令来下载、启动和停止一个容器：

1.  首先，使用`pull-tar`参数下载一个准备好的容器镜像，并将其准备好在系统上使用：

    ```
    # machinectl pull-tar https://nspawn.org/storage/archlinux/archlinux/tar/image.tar.xz archlinux
    # machinectl import-tar archlinux.tar.xz
    ```

1.  使用`list-images`参数列出可用的镜像：

    ```
    # machinectl list-images
    ```

1.  现在我们可以克隆这个镜像并启动容器：

    ```
    # machinectl clone archlinux test
    # machinectl start test
    ```

1.  要访问容器环境，请使用`shell`参数：

    ```
    # machinectl shell test
    ```

1.  我们可以使用`poweroff`参数关闭容器：

    ```
    # machinectl poweroff test
    ```

当我们使用`machinectl`时，容器将运行在`unconfined_service_t` SELinux 域中。目前无法覆盖此设置。幸运的是，我们还有其他工具可用于简化容器管理，这些工具在内置 SELinux 支持方面更强大，例如 Docker 和`podman`。

# 配置 podman

`podman`工具是 CentOS 8 及其他源自 Red Hat Enterprise Linux 的发行版的默认容器管理工具。其他发行版，如 Gentoo，也可以通过安装`libpod`轻松获取`podman`。

## 选择 podman 而非 Docker

当我们将`podman`与 Docker 进行比较时，如果只是用它来进行基本的容器管理操作，可能看不出太大区别。命令非常相似，`podman`甚至具有一个 Docker 兼容层，这使得习惯使用 Docker 的管理员也能方便地使用`podman`。

但是，从底层来看，存在许多差异。首先，`podman`是一个无守护进程的容器管理系统，允许最终用户轻松地在其受限空间内运行容器。`libpod`项目还采用了不同的设计原则，并支持不同的容器运行时，支持基于**开放容器倡议**（**OCI**）定义的容器运行时接口（**CRI-O**）。

让我们使用`podman`在系统上部署一个 PostgreSQL 容器：

1.  首先，我们需要找到合适的容器。我们可以使用`podman search`命令来查找：

    ```
    # podman search postgresql
    ```

1.  在列出的各种 PostgreSQL 容器中，我们选择了 Bitnami 版本：

    ```
    /var/lib/containers/storage. 
    ```

1.  现在我们可以启动一个容器，为 PostgreSQL 超级用户（`postgres`）设置密码，并确保 PostgreSQL 端口（`5432`）对系统可用：

    ```
    # podman run -dit --name postgresql-test \
      -e POSTGRESQL_PASSWORD="pgsqlpass" \
      -p 5432:5432 postgresql
    ```

    此命令将基于我们刚刚下载的容器基础创建一个容器定义，并在系统上启动它。

1.  我们可以使用`psql`来验证数据库是否运行：

    ```
    # psql -U postgres -h localhost
    ```

1.  当我们完成容器操作时，可以停止它（使用`podman stop`），这样会保留当前的容器信息，允许我们稍后重新启动它（使用`podman start`）或完全从系统中删除容器：

    ```
    # podman rm postgresql-test
    ```

1.  删除容器会移除容器运行时，但基础容器镜像仍然保留在系统上：

    ```
    # podman images
    ```

    这样，我们可以快速启动另一个容器，而无需重新下载文件。

使用容器是一种快速有效的方式，可以迅速在系统上安装和部署软件。此外，SELinux 提供了一些额外的保护措施，确保这些容器不会出现不当行为。

## 使用 SELinux 的容器

当我们查看活动的运行时时，我们会注意到 SELinux 已经以我们理解的方式限制了这些容器：

```
# ps -efZ | grep postgres
system_u:system_r:container_t:s0:c182,c609 ... /opt/bitnami/postgres -D ...
```

正在运行的进程被分配到两个类别，并在`container_t` SELinux 域中执行。这是我们在*第九章*中看到的 sVirt 方法，*安全虚拟化*。然而，与虚拟机不同，容器通常以更为临时的方式使用：当容器基础的新版本发布时，容器会被废弃，新的容器会启动。虚拟机通常会进行系统内升级，因此具有更长的生命周期。

容器的临时性方式还意味着我们需要以不同的方式提供数据持久性。大多数容器使用的方法是允许从主机将位置映射到容器环境中。

让我们使用`podman`将容器外部的位置映射到容器内部的`/bitnami/postgresql`位置，这是 PostgreSQL 容器所需要的：

1.  首先，在主机上创建我们想要存储 PostgreSQL 数据（库）的位置：

    ```
    # mkdir -p /srv/db/postgresql-test
    ```

1.  接下来，将此位置的所有权更改为具有用户 ID `1001` 的用户（这是容器内部使用的用户 ID）：

    ```
    # chown -R 1001:1001 /srv/db/postgresql-test
    ```

1.  现在启动容器，创建从此位置到容器的映射：

    ```
    /srv/db/postgresql-test. If we later delete the container and create a new one (for instance, because an update for the container base has been made available), the database itself is not affected.
    ```

映射本身包含一个 SELinux 特定的变量，即尾部的`:Z`。如果我们省略此映射中的`：Z`，该位置仍然可以在容器内部访问。然而，PostgreSQL 运行时将无法使用它。

重要提示

在目录映射中使用`:Z`（或稍后我们会看到的`:Z`（或`:z`））要比不使用好！

容器仍然是主机操作系统的一部分。当我们创建`/srv/db/postgresql-test`这个位置时，默认会收到`var_t`的 SELinux 类型。希望使用该位置的容器将需要`var_t`的写入权限。然而，这个权限并不是我们希望提供的。毕竟，容器应该尽可能与主机隔离——这正是 sVirt 技术的核心所在。

因此，我们需要相应地重新标记这个位置。通用容器使用的 SELinux 类型是`container_file_t`。此外，我们希望确保只有正确的容器能够访问这个位置。限制和隔离访问就是命令中`:Z`（大写的`Z`）所做的：用`container_file_t`类型标记该目录，并为其关联正确的类别。

如果我们希望某个位置可以被*多个容器*访问，我们可以告诉`podman`*共享*这个位置，但仍然使用`container_file_t`的 SELinux 类型进行标记。为了实现这一点，我们需要使用`:z`参数（小写的`z`），如下所示：

```
# podman run -dit --name postgresql-test \
 -e POSTGRESQL_PASSWORD="pgsqlpass" \
 -v /srv/db/postgresql-test:/bitnami/postgresql:z \
 -p 5432:5432 postgresql
```

创建适当的映射并不是 SELinux 配置中的唯一方法。如果需要，我们还可以告诉`podman`为容器使用不同的 SELinux 域。

## 更改容器的 SELinux 域

要控制容器启动时的 SELinux 上下文，我们可以在`podman`命令中使用`--security-opt`参数。例如，要使用`container_logreader_t` SELinux 域运行 Nginx 容器，我们可以使用如下命令：

```
# podman run -dit --name nginx-test -p 80:80 \
  --security-opt label=type:container_logreader_t nginx
```

该域比默认的`container_t`域权限稍高，因为它还具有对日志文件的读取权限。例如，我们可以用它来让 Web 服务器公开日志文件。

我们可以传递的其他标记选项如下：

+   SELinux 用户，使用`label=user:<SELinux user>`参数。

+   SELinux 角色，使用`label=role:<SELinux role>`参数。

+   SELinux 敏感度级别，使用`label=level:<SELinux level>`参数。

+   文件的 SELinux 类型，使用`label=filetype:<SELinux type>`参数。这会设置带有`:Z`和`:z`后缀的定位映射的 SELinux 上下文。所选类型必须是容器 SELinux 域的入口点。

我们还可以使用另一个选项，即`label=disable`。设置这个参数后，容器将运行时不使用任何 SELinux 隔离。这个选项并不禁用容器的 SELinux，而是将一个名为`spc_t`的未受限域与容器关联：

```
# podman run -dit --name nginx-test -p 80:80 \
  --security-opt label=disable \
  -v /srv/web/localhost:/usr/share/nginx/html nginx
# ps -efZ | grep nginx
unconfined_u:system_r:spc_t:s0 ... nginx: worker process
```

对于大多数使用场景，默认的`container_t`域已经具有足够的权限，但对于某些情况，它的权限可能过于宽泛。幸运的是，我们可以轻松创建与我们使用场景相匹配的新的 SELinux 域。

## 使用 udica 创建自定义域

`container_t` 域被配置为广泛可重用，这意味着它对常见用例有许多特权，这些特权您可能不想给每个容器。此外，如果我们启动一个容器，但需要为其关联更多特权，那么我们将不得不扩展 `container_t` 以获取更多特权，结果是所有容器都会收到此特权扩展。

为了快速建立新策略，可以使用名为 `udica` 的工具。`udica` 工具读取容器定义并从中创建自定义 SELinux 策略。然后，我们可以将此自定义策略用于此特定容器，允许其他容器保持不变。

让我们将其用于 Jupyter Notebook，我们希望为（共享的）用户主目录位置授予读/写权限：

1.  首先，我们创建容器的定义：

    ```
    # podman run -dit --name notebook-test -p 8888:8888 \
      -v /home/lisa/work:/home/jovyan/shared scipy-notebook
    ```

1.  接下来，使用 `podman inspect` 检查此容器并将结果存储在文件中：

    ```
    # podman inspect notebook-test > notebook-test.json
    ```

1.  使用 `udica` 为其生成 SELinux 策略：

    ```
    # udica -j notebook-test.json custom-notebook-test
    Policy custom-notebook-test created!
    Please load these modules using:
    # semodule -i custom-notebook-test.cil /usr/share/udica/templates/{base_container.cil,net_container.cil}
    Restart the container with: "--security-opt label=type:custom-notebook-test.process" parameter
    ```

1.  根据 `udica` 输出加载自定义策略：

    ```
    # semodule -i custom-notebook-test.cil \
        /usr/share/udica/templates/base_container.cil \
        /usr/share/udica/templates/net_container.cil
    ```

1.  停止并删除容器，然后根据 `udica` 输出重新创建它：

    ```
    # podman stop notebook-test
    # podman rm notebook-test
    # podman run -dit --name notebook-test -p 8888:8888 \
      -v /home/lisa/work:/home/jovyan/shared \
      --security-opt \
      label=type:custom-notebook-test.process scipy-notebook
    ```

自定义 SELinux 策略具有写入主目录的特权，因为容器从 `/home/lisa/work` 映射，并且 `udica` 自动为其创建了权限。如果我们希望容器仅具有只读特权，我们可以使用带有尾部 `:ro` 的映射（而不是 `:Z` 或 `:z` 用于 SELinux 特定更改）。这将映射容器内部的位置，并且 `udica` 仅为关联的 SELinux 类型创建读权限。

如果创建自定义策略有点过于具体，我们还可以使用适当的 SELinux 布尔值来微调 `container_t` 域的特权。

## 使用 SELinux 布尔值切换 `container_t` 特权

`container_t` SELinux 域与 `svirt_sandbox_domain` 属性关联，通过这种关联，将自动由我们在*第九章*中看到的 `virt_*` SELinux 布尔值管理。

也有一些特定于容器的 SELinux 布尔值：

+   使用 `container_use_cephfs`，容器可以使用基于 CephFS 的存储。这主要用于容器由较大的容器集群软件（如 Kubernetes）管理时。

+   使用 `container_manage_cgroup`，容器可以管理 cgroups。当容器内部托管有 systemd 时，这是必需的，这种情况通常适用于完整的容器运行时（而不是特定进程的容器）。这些容器托管几乎完整的 Linux 系统。

+   使用 `container_connect_any`，`container_t` SELinux 域可以连接到任何 TCP 端口。

请注意，这些布尔值影响 `container_t` 域的特权，因此对所有容器都有效。

## 调整容器托管环境

`podman` 工具默认会将其容器卷和基础镜像存储在`/var/lib/containers`目录中。管理员可以通过位于`/etc/containers`的`storage.conf`配置文件添加更多存储位置。然而，你还需要相应地调整 SELinux 配置。

假设我们将使用`/srv/containers`位置，那么我们需要创建一个等价规则，确保该位置的标签被正确设置：

```
# semanage fcontext -a -e /var/lib/containers \
  /srv/containers
# restorecon -R -v /srv/containers
```

如果该位置是网络挂载点，你可能还需要更改相应的 SELinux 布尔值。

# 利用 Kubernetes 的 SELinux 支持

当容器在更大的环境中使用时，它们通常通过容器编排框架进行管理，这些框架允许跨多个系统扩展容器部署和管理。Kubernetes 是一个受欢迎的容器编排框架，拥有良好的社区支持和商业支持。

Kubernetes 使用机器上找到的容器软件。例如，当我们在 Fedora 的 CoreOS 上安装 Kubernetes 时，它会检测到 Docker 可用，并使用 Docker 引擎来管理容器。

## 配置 Kubernetes 以支持 SELinux

安装 Kubernetes 可能是一项艰巨的任务，存在多种方法，范围从单节点的实验部署到商业支持的安装方法。在 Kubernetes 网站上，有一种经过充分记录的安装方法是使用`kubeadm`来引导 Kubernetes 集群的创建。

重要提示

Kubernetes 的安装文档可以在 Kubernetes 官网找到，链接为[`kubernetes.io/docs/setup/production-environment/tools/kubeadm`](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm)。在本节中，我们不会逐步讲解如何设置一个工作的 Kubernetes 实例，而是提供一些指引，说明需要进行哪些更改以支持 SELinux。

当初始化 Kubernetes 集群时，`kubeadm`命令会下载并运行各种 Kubernetes 服务作为容器。不幸的是，Kubernetes 的服务使用了多个从主机系统到容器的映射，以便其正常运行。这些映射没有使用`:Z`或`:z`选项——甚至这样做是错误的，因为这些位置是系统范围内的，应该保留当前的 SELinux 标签。

结果，Kubernetes 的服务将以默认的`container_t` SELinux 域运行（因为 Docker 会愉快地应用 sVirt 保护），而无法访问这些位置。我们可以应用的最明显更改是让服务暂时使用特权更高的`spc_t`域。然而，在安装过程中进行此更改是困难的，因为我们需要在安装失败之前快速更改域。

虽然我们可以为 Kubernetes 创建部署配置，这会立即使用`spc_t`配置服务，但也可以采取另一种方法：

1.  在安装开始之前，将 `container_t` 类型标记为宽松域。虽然这会防止 SELinux 对容器的控制，但我们可以认为 Kubernetes 的安装是以封闭和受监督的方式进行的：

    ```
    # semanage permissive -a container_t
    ```

1.  运行 `kubeadm init`，它将在系统上安装服务：

    ```
    # kubeadm init
    ```

1.  安装服务后，转到 `/etc/kubernetes/manifests`。在此目录中，您将找到四个清单，每个清单代表一个 Kubernetes 服务：

    ```
    # cd /etc/kubernetes/manifests
    ```

1.  编辑每个清单文件（`etcd.yaml`、`kube-apiserver.yml`、`kube-controller-manager.yml` 和 `kube-scheduler.yml`），并添加一个安全上下文定义，使服务以 `spc_t` 域运行。此操作作为 `containers` 部分下的配置指令进行：

    ```
    apiVersion: v1
    kind: Pod
    metadata:
      name: etcd
    spec:
      containers:
      - command: …
      securityContext:
        seLinuxOptions:
          type: spc_t
        image: k8s.gcr.io/etcd:3.4.3-0
        …
    ```

1.  在 Kubernetes 安装过程中，将安装 `kubelet` 服务，它会检测到这些文件已被更改，并会自动重启容器。如果没有，您可以关闭并删除 Docker 中的容器定义，`kubelet` 将自动重新创建它们：

    ```
    # docker ps
    CONTAINER ID      ...     NAMES
    548f0c3ed18e          k8s_POD_etcd-ppubssa3ed_kube…
    b7b1df2d0027          k8s_POD_kube-apiserver-…
    eecd4d4ad108          k8s_POD_kube-scheduler-…
    76da4910b927          k8s_POD_kube-controller-…
    # for n in 548f0c3ed18e b7b1df2d0027 eecd4d4ad108 76da4910b927; do docker stop $n; docker rm $n; done
    ```

1.  验证服务是否现在以特权 `spc_t` 域运行：

    ```
    # ps -ef | grep spc_t
    ```

1.  移除 `container_t` 的宽松状态，使其恢复到强制模式：

    ```
    # semanage permissive -d container_t
    ```

在安装过程中进行这些微小调整后，Kubernetes 现在可以正常运行，并启用了 SELinux 支持。

## 设置 SELinux 上下文以用于 pods

在 Kubernetes 中，容器是 pod 的一部分。`podman` 工具也能够使用 pod 的概念（因此得名）。例如，我们可以将 Nginx 容器放入一个名为 `webserver` 的 pod 中，如下所示：

```
# podman pod create -p 80:80 --name webserver
# podman pull docker.io/library/nginx
# podman run -dit --pod webserver --name nginx-test nginx
```

与 `podman` 不同，Kubernetes 不依赖命令行交互来创建和管理如 pods 之类的资源。相反，它使用清单文件（正如我们在 *配置带 SELinux 支持的 Kubernetes* 部分简要提到的那样）。Kubernetes 管理员或 DevOps 团队将创建清单文件，并将其应用于环境。

例如，为了让 Nginx 容器在 Kubernetes 上运行，可以使用以下清单：

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test-deployment
spec:
  selector:
    matchLabels:
      app: nginx-test
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx-test
    spec:
      containers:
      - name: nginx-test
        image: nginx:latest
        ports:
        - containerPort: 80
```

这个清单是一个 Kubernetes 部署，它告诉 Kubernetes 我们想要运行两个 Nginx 容器。要将其应用于环境，使用 `kubectl apply`：

```
$ kubectl apply -f simple-nginx.yml
```

与 Kubernetes 服务的清单一样，我们可以告诉 Kubernetes 使用特定的 SELinux 类型：

```
…
spec:
  containers: ...
  securityContext:
    seLinuxOptions:
      type: "container_logreader_t"
```

`seLinuxOptions` 块可以包含 `user`、`role`、`type` 和 `level`，用于定义 SELinux 用户、SELinux 角色、SELinux 类型和 SELinux 敏感级别。

与常规容器管理服务（如 Docker 或 CRI-O）不同，Kubernetes 不允许更改映射卷上的 SELinux 标签（单节点部署除外）：当我们将卷映射到容器时，它们会保留系统上当前的 SELinux 标签。因此，如果您希望确保资源可以从常规的 `container_t` 域访问，您需要确保这些位置被标记为 `container_file_t`。

Kubernetes 确实提供了先进的访问控制。容器内的卷启用也是通过插件架构来处理的，已有多个插件可用。当插件启用 SELinux 标签时，Kubernetes 会尝试重新标记资源并分配类别（与 sVirt 相同）。然而，目前该功能仅在单节点部署（使用本地主机存储插件）中可用——对于这样的部署，使用 `podman` 要简单得多。

# 总结

容器化工作负载允许管理员快速且轻松地为系统添加功能，同时保持容器内可能的依赖关系。每个容器都托管自己的依赖项，允许容器从系统中移除或添加，而不会影响其他容器。通过 SELinux，这些工作负载进一步与主机隔离，并且在启用 sVirt 保护的情况下，相互之间也得到了隔离。

我们已经看到 systemd 支持容器，但缺乏基于 sVirt 的保护，也了解了 `podman` 如何在其容器环境中应用 sVirt 保护。我们了解到，Docker 和 `podman` 在使用上非常相似，但在底层实现上有所不同。这两个框架都允许我们将不同的 SELinux 类型应用于容器和资源，并且通过 `udica`，我们学会了如何在不进行大量开发的情况下创建自定义策略。最后，我们还了解了如何配置 Kubernetes 使用 SELinux 标签。

在所有这些支持 SELinux 的技术背后，我们已经准备好迎接 SELinux 策略的开发了。在接下来的章节中，我们将深入学习如何与 SELinux 策略打交道，并根据需要调整策略。

# 问题

1.  为什么在 SELinux 中，`docker` 或 `podman` 比 `machinectl` 更受欢迎？

1.  我们如何确保主机数据在容器内得到正确映射？

1.  我们如何从容器定义中创建自定义策略？

1.  在 Kubernetes 的清单中，我们可以将 SELinux 设置放在哪里？
