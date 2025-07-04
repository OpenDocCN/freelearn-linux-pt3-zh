26\. 在 CentOS 8 上使用容器

现在，上一章已涵盖 Linux 容器的基础知识，本章的目标是演示如何使用 CentOS 8 中包含的 Podman、Skopeo 和 Buildah 工具创建和管理容器。本章结束时，你将更清楚地理解如何在 CentOS 8 上创建和管理容器，并为继续探索 Linux 容器的强大功能打下知识基础。

26.1 拉取容器镜像

在这个例子中，CentOS 8 基础镜像将从仓库中拉取。然而，在拉取镜像之前，可以使用 skopeo 工具获取有关镜像仓库的信息，例如：

# 第二十五章：skopeo inspect docker://docker.io/library/centos

{

"名称": "docker.io/library/centos",

"摘要": "sha256:f94c1d992c193b3dc09e297ffd54d8a4f1dc946c37cbeceb26d35ce1647f88d9",

"RepoTags": [

"5.11",

"5",

"6.10",

"6.6",

"6.7",

"6.8",

"6.9",

"6",

"7.0.1406",

"7.1.1503",

"7.2.1511",

"7.3.1611",

"7.4.1708",

"7.5.1804",

"7.6.1810",

"7.7.1908",

"7",

"8",

"centos5.11",

"centos5",

"centos6.10",

"centos6.6",

"centos6.7",

"centos6.8",

"centos6.9",

"centos6",

"centos7.0.1406",

"centos7.1.1503",

"centos7.2.1511",

"centos7.3.1611",

"centos7.4.1708",

"centos7.5.1804",

"centos7.6.1810",

"centos7.7.1908",

"centos7",

"centos8",

"最新"

],

"创建时间": "2019-10-01T23:19:57.105928163Z",

"Docker 版本": "18.06.1-ce",

"标签": {

"org.label-schema.build-date": "20190927",

"org.label-schema.license": "GPLv2",

"org.label-schema.name": "CentOS 基础镜像",

"org.label-schema.schema-version": "1.0",

"org.label-schema.vendor": "CentOS"

},

"架构": "amd64",

"操作系统": "linux",

"层": [

"sha256:729ec3a6ada3a6d26faca9b4779a037231f1762f759ef34c08bdd61bf52cd704"

]

}

在验证了仓库中包含我们需要的镜像后，可以使用 podman 命令通过以下语法下载所需 CentOS 版本的镜像，其中<RepoTag>应替换为 skopeo 输出中显示的 CentOS 版本标签：

# podman pull docker://docker.io/library/centos:<RepoTag>

例如，要拉取 CentOS 7 镜像：

# podman pull docker://docker.io/library/centos:centos7

另外，为了默认拉取 CentOS 的最新版本：

# podman pull docker://docker.io/library/centos:latest

尝试拉取 docker://docker.io/library/centos:latest... 获取镜像源签名

跳过 blob 729ec3a6ada3（已存在）：68.21 MiB / 68.21 MiB [=======] 0s

正在复制配置 0f3e07c0138f: 2.13 KiB / 2.13 KiB [==========================] 0s

正在将清单写入镜像目标

存储签名

0f3e07c0138fbe05abcb7a9cc7d63d9bd4c980c3f61fea5efa32e7c4217ef4da

通过要求 podman 列出所有本地镜像来验证镜像是否已存储：

# podman images

仓库 标签 镜像 ID 创建时间 大小

docker.io/library/centos latest 0f3e07c0138f 8 周前 227 MB

可以通过运行 podman inspect 命令获取本地镜像的详细信息：

# podman inspect centos:latest

26.2 在容器中运行镜像

从注册表中拉取的镜像是一个完全可以运行的镜像，准备好在容器中运行，无需修改。要运行镜像，可以使用`podman run`命令。在这种情况下，将指定`–rm`选项，以指示我们希望在容器中运行镜像，执行一个命令，然后让容器退出。在此情况下，将使用`cat`工具输出容器根文件系统中`/etc/passwd`文件的内容：

# podman run --rm centos:latest cat /etc/passwd

root:x:0:0:root:/root:/bin/bash

bin:x:1:1:bin:/bin:/sbin/nologin

daemon:x:2:2:daemon:/sbin:/sbin/nologin

adm:x:3:4:adm:/var/adm:/sbin/nologin

lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

sync:x:5:0:sync:/sbin:/bin/sync

shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown

halt:x:7:0:halt:/sbin:/sbin/halt

mail:x:8:12:mail:/var/spool/mail:/sbin/nologin

operator:x:11:0:operator:/root:/sbin/nologin

games:x:12:100:games:/usr/games:/sbin/nologin

ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin

nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin

dbus:x:81:81:System message bus:/:/sbin/nologin

systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin

systemd-resolve:x:193:193:systemd Resolver:/:/sbin/nologin

比较容器内的`/etc/passwd`文件与宿主系统中的`/etc/passwd`文件，注意容器内缺少宿主系统中所有的附加用户，这证明了`cat`命令是在容器环境中执行的。同时注意到容器在几秒钟内启动、执行命令并退出。与启动一个完整的操作系统、执行任务并关闭虚拟机所需的时间相比，你会开始欣赏容器的速度和效率。

要启动一个容器，保持它运行并访问 shell，可以使用以下命令：

$ podman run --name=mycontainer -it centos:latest /bin/bash

[root@965acf617e6e /]#

请注意，使用了一个额外的命令行选项来为容器指定名称“mycontainer”。虽然这是可选的，但这使得容器更容易识别和引用，作为使用自动生成的容器 ID 的替代方法。

当容器正在运行时，在另一个终端窗口运行 podman，以查看系统中所有容器的状态

# podman ps -a

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES

965acf617e6e docker.io/library/centos:latest /bin/bash 大约一分钟前 启动 大约一分钟前 mycontainer

要从宿主机在运行中的容器中执行命令，只需使用`podman exec`命令，指定运行中的容器名称和要执行的命令。例如，以下命令会在名为`mycontainer`的容器中启动第二个 bash 会话：

# podman exec -it mycontainer /bin/bash

bash-4.4#

请注意，尽管上面的示例引用了容器名称，但也可以通过 `podman ps -a` 命令列出的容器 ID 来实现相同的结果：

# podman exec -it 965acf617e6e /bin/bash

bash-4.4#

或者，`podman attach` 命令也可以附加到一个正在运行的容器并访问 shell 提示符：

# podman attach mycontainer

bash-4.4#

一旦容器启动并运行，就可以像任何其他 CentOS 8 系统一样进行额外的配置更改并安装软件包。

26.3 管理容器

一旦启动，容器将继续运行，直到通过 podman 停止，或者当启动容器时运行的命令退出。例如，在主机上运行以下命令将使容器退出：

# podman stop mycontainer

或者，在容器最后一个剩余的 bash shell 中按下 Ctrl-D 键盘组合，将会使得 shell 和容器都退出。退出后，容器的状态会相应地变化：

# podman ps -a

容器 ID 镜像 命令 创建时间 状态 端口 名称

965acf617e6e docker.io/library/centos:latest /bin/bash 10 分钟前 已退出 (0) 21 秒前 mycontainer

尽管容器已经不再运行，但它仍然存在，并且包含了所有对配置和文件系统所做的更改。如果你安装了软件包、做了配置更改或添加了文件，这些更改将在“mycontainer”中保持不变。要验证这一点，只需按如下方式重新启动容器：

# podman start mycontainer

启动容器后，使用 `podman exec` 命令再次在容器内执行命令，具体方法如前所述。例如，再次获得 shell 提示符：

# podman exec -it mycontainer /bin/bash

正在运行的容器也可以通过 `podman pause` 和 `podman unpause` 命令进行暂停和恢复，如下所示：

# podman pause 我的容器

# podman unpause 我的容器

26.4 将容器保存为镜像

一旦容器的客系统配置完成，并符合你的要求，你很可能想要创建并运行多个此类型的容器。为此，需要将容器保存为镜像到本地存储中，以便用作额外容器实例的基础。可以通过 `podman commit` 命令结合容器的名称或 ID 以及镜像存储的名称来实现，例如：

$ podman commit mycontainer mycentos_image

一旦镜像保存完毕，检查它是否出现在本地仓库的镜像列表中：

$ podman images

仓库 标签 镜像 ID 创建时间 大小

localhost/mycentos_image latest c32c45218143 5 秒前 227 MB

docker.io/library/centos latest 0f3e07c0138f 8 周前 227 MB

保存的镜像现在可以用来创建与原始容器完全相同的额外容器：

$ podman run --name=mycontainer2 -it localhost/mycentos_image /bin/bash

26.5 从本地存储中删除镜像

要从本地存储中删除不再需要的镜像，只需运行 `podman rmi` 命令，并引用 `podman images` 命令输出的镜像名称或 ID。例如，要删除前面部分创建的名为 mycentos_image 的镜像，可以按如下方式运行 podman：

$ podman rmi localhost/mycentos_image

注意：在删除镜像之前，基于该镜像的任何容器必须先被删除。

26.6 删除容器

即使容器已退出或被停止，它仍然存在，并且可以随时重新启动。如果容器不再需要，可以在容器停止后使用 `podman rm` 命令将其删除，方法如下：

# podman rm mycontainer2

26.7 使用 Buildah 构建容器

Buildah 允许从现有容器、镜像或完全从头开始构建新容器。Buildah 还包括挂载容器文件系统的功能，这样就可以从主机访问和修改它。

例如，以下 buildah 命令将从 CentOS 8 Base 镜像构建容器（如果该镜像尚未从注册表拉取，buildah 将在创建容器之前先下载它）：

$ buildah from docker://docker.io/library/centos:centos8

运行此命令的结果将是一个名为 centos-working-container 的容器，它已准备好运行：

$ buildah run centos-working-container cat /etc/passwd

26.8 从头开始构建容器

从头开始构建容器本质上是创建一个空容器。创建后，可以安装软件包以满足容器的需求。当创建一个仅需要安装最少软件包的容器时，这种方法特别有用。

从头开始构建的第一步是运行以下命令来构建空容器：

# buildah from scratch

working-container

构建完成后，将创建一个名为 working-container 的新容器：

$ buildah containers

CONTAINER ID BUILDER IMAGE ID IMAGE NAME CONTAINER NAME

00f81b68f03f * 0f3e07c0138f docker.io/library/centos:centos8 centos-working-container

d7dd4b652379 * scratch working-container

空容器现在已准备好安装一些软件包。不幸的是，这不能在容器内执行，因为此时连 bash 或 dnf 工具都不存在。相反，需要将容器的文件系统挂载到主机系统上，并使用 dnf 安装软件包，系统根目录设置为挂载的容器文件系统。通过以下方式开始挂载容器的文件系统：

# buildah mount working-container

/var/lib/containers/storage/overlay/20b46cf0e2994d1ecdc4487b89f93f6ccf41f72788da63866b6bf80984081d9a/merge

如果文件系统成功挂载，buildah 会输出容器文件系统的挂载点。现在我们已经可以访问容器文件系统，使用 dnf 命令可以通过--installroot 选项将软件包安装到容器中，指向已挂载的容器文件系统。例如，以下命令会在容器文件系统中安装 bash、CoreUtils 和 dnf 包（其中<container_fs_mount>是之前通过 buildah mount 命令输出的挂载路径）：

# dnf install --releasever=8 --installroot <container_fs_mount> bash coreutils dnf

请注意，--releasever 选项用于告知 dnf 在容器中安装 CentOS 版本 8 的包。

安装完成后，按照以下方式卸载临时文件系统：

# buildah umount working-container

一旦 dnf 完成了包的安装，就可以运行容器并访问 bash 命令提示符，方法如下：

# buildah run working-container bash

bash-4.4#

26.9 容器桥接网络

如前一章所述，容器网络是通过容器网络接口（CNI）桥接网络栈实现的。以下命令展示了运行容器的主机系统上的典型网络配置：

# ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000

link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

inet 127.0.0.1/8 scope host lo

valid_lft 永久 preferred_lft 永久

inet6 ::1/128 scope host

valid_lft 永久 preferred_lft 永久

2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000

link/ether 08:00:27:20:dc:2f brd ff:ff:ff:ff:ff:ff

inet 192.168.0.33/24 brd 192.168.0.255 scope global dynamic noprefixroute enp0s3

valid_lft 3453 秒 preferred_lft 3453 秒

inet6 2606:a000:4307:f000:aa6:6da1:f8a9:5f95/64 scope global dynamic noprefixroute

valid_lft 3599 秒 preferred_lft 3599 秒

inet6 fe80::4275:e186:85e2:d81f/64 scope link noprefixroute

valid_lft 永久 preferred_lft 永久

3: cni0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000

link/ether 7e:b6:04:22:4f:22 brd ff:ff:ff:ff:ff:ff

inet 10.88.0.1/16 scope global cni0

valid_lft 永久 preferred_lft 永久

inet6 fe80::7cb6:4ff:fe22:4f22/64 scope link

valid_lft 永久 preferred_lft 永久

12: veth2a07dc55@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master cni0 state UP group default

link/ether 42:0d:69:13:89:af brd ff:ff:ff:ff:ff:ff link-netns cni-61ba825e-e596-b2ef-a59f-b0743025e448

inet6 fe80::400d:69ff:fe13:89af/64 scope link

valid_lft 永久 preferred_lft 永久

在上面的示例中，主机有一个名为 enp0s3 的接口，连接到外部网络，IP 地址为 192.168.0.33。 此外，还创建了一个名为 cni0 的虚拟接口，并分配了 IP 地址 10.88.0.1。 在主机上运行相同的 ip 命令时，容器可能会显示以下输出：

# ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000

link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

inet 127.0.0.1/8 scope host lo

valid_lft forever preferred_lft forever

inet6 ::1/128 scope host

valid_lft forever preferred_lft forever

3: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default

link/ether 3e:52:22:4b:e0:d8 brd ff:ff:ff:ff:ff:ff link-netnsid 0

inet 10.88.0.28/16 scope global eth0

valid_lft forever preferred_lft forever

inet6 fe80::3c52:22ff:fe4b:e0d8/64 scope link

valid_lft forever preferred_lft forever

在这种情况下，容器的 IP 地址是 10.88.0.28。 在主机上运行 ping 命令可以验证主机和容器确实在同一子网内：

# ping 10.88.0.28

PING 10.88.0.28 (10.88.0.28) 56(84) 字节的数据。

64 字节来自 10.88.0.28：icmp_seq=1 ttl=64 时间=0.056 ms

64 bytes from 10.88.0.28: icmp_seq=2 ttl=64 time=0.039 ms

.

.

CNI 配置设置可以在主机系统的 /etc/cni/net.d/87-podman-bridge.conflist 文件中找到，默认情况下，文件内容如下：

{

"cniVersion": "0.3.0",

"name": "podman",

"plugins": [

{

"type": "bridge",

"bridge": "cni0",

"isGateway": true,

"ipMasq": true,

"ipam": {

"type": "host-local",

"subnet": "10.88.0.0/16",

"routes": [

{ "dst": "0.0.0.0/0" }

]

}

},

{

"type": "portmap",

"capabilities": {

"portMappings": true

}

}

]

}

可以修改此文件以更改子网地址范围，也可以更改插件类型（此示例中设置为 bridge）以实现不同的网络配置。

26.10 总结

本章介绍了如何在 CentOS 8 上使用 skopeo 和 buildah 工具创建和管理 Linux 容器，包括使用从仓库中获取的容器镜像和完全从头开始构建的新镜像。
