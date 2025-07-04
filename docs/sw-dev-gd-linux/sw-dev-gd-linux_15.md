

# 使用 Docker 对应用程序进行容器化

在过去的十年里，Docker 容器化已经成为 Web 应用和现代微服务的默认打包格式。在容器中，您的程序处于一个非常轻量级、与宿主环境安全隔离的 Linux 文件系统、进程、用户和网络抽象的壳层中。容器镜像也非常便携——它们可以轻松地从开发者的笔记本电脑转移到测试或预发布环境，再到生产服务器。这解决了过去几十年中困扰软件和基础设施的许多问题。

从某种意义上说，容器与您从软件仓库中学习如何安装的 Linux 包非常相似。容器镜像大致上是您应用程序的压缩档案（例如 `.tar.gz` 文件），以及应用程序所需的所有配置文件和依赖项。这个小包——镜像——可以通过 Docker 执行。关于这种容器的革命性之处在于，它将一切都整齐地打包在一个单一的工件中，并且可以在任何安装了容器运行时（如 Docker）的 Linux 系统上运行。

本章内容本身就足以成为一本书——Docker 和 Linux 容器总体上是非常庞大的主题。然而，像本书中的其他内容一样，我们只关注让您能够舒适地与基于 Docker 的基础设施进行交互所需的基本理论和实践技能。

在本章中，您将了解以下内容：

+   容器解决的应用开发和运营问题

+   容器是什么，它们与 Linux 包的相似之处

+   Docker 镜像与 Docker 容器的区别

+   在开发工作流中使用 Docker 的所有实用基础

+   如何通过 Dockerfile 构建自己的容器镜像（您将容器化一个真实的 Python Web 应用）

+   一些更高级的话题，比如虚拟机和容器的区别，以及 Linux 如何通过命名空间实现容器抽象

+   一些经过实践验证的容器技巧、窍门和最佳实践

让我们开始吧。

# 容器如何作为包工作

Docker 成为打包软件的标准工具，目的是包括一个已知的、能够正常工作的系统。Docker 容器通常包含您想要运行的软件以及一个完整的，虽然常常经过精简的，Linux 系统作为其执行环境。这个执行环境提供了库和工具，以及一些其他内容，如基本的系统配置，使得容器可以作为一个独立实体运行，不依赖于运行容器的系统。主要目标是确保应用程序能够在开发者的机器、生产和测试环境以及其他地方成功运行，而不需要处理操作系统版本、已安装的库等细节问题。

需要记住的是，操作系统和库并不会消失。库中的漏洞仍然存在，任何打包的依赖项都应因安全等原因进行更新。然而，软件的使用者，无论是最终用户、系统运维人员，还是任何编排软件，现在都获得了一个通用的软件包，不需要关心系统依赖项。虽然软件的运行和配置细节仍然取决于软件本身，但它的执行方式（在容器中）在某种程度上是标准化的。

总结来说，这意味着环境的任何特定设置（例如安装依赖项）现在都在 Dockerfile 中描述，一旦创建了有效的容器镜像，除了特定配置之外，容器应该能在任何支持 Docker 运行的系统上运行，或者更广泛地说，**OCI** 镜像。

**OCI**（**Open Container Initiative**的缩写）提供了指定镜像格式、执行 Linux 容器等标准。它有时与 Docker 同义使用，意思是开发者可能使用 Docker 创建镜像，但在编排器上执行时可能完全不使用 Docker。

没有比在你的机器上安装 Docker 并开始尝试一些命令更好的入门方式了，来，我们开始吧。

# 前提：安装 Docker

首先，下载并安装 Docker Desktop。你可以在这里找到安装说明：[`docs.docker.com/get-docker/`](https://docs.docker.com/get-docker/)。

另外，官方提供了一个出色的入门教程，链接在这里：[`docs.docker.com/get-started/`](https://docs.docker.com/get-started/)，但是我们建议在完成本章内容后再阅读它。我们会介绍一些基础内容，但重点不在于特定的命令行标志，而是如何作为应用开发者使用这些命令和工作流程。

现在你已经安装了 Docker，接下来让我们开始启动第一个容器吧！

# Docker 快速入门

**Docker 镜像**是我们比喻中的“包”——它是一个静态的工件，保存、存储并传输。当它在机器上执行时，就变成了一个**容器**。这点很重要，因为你有时会听到这些术语被误用：Docker 镜像是容器的不可变基础——容器是一个正在运行的、有命名空间的进程。镜像是预构建的模板，从中生成运行时的活容器。

镜像设计为不可变的：如果你下载一个 nginx 网页服务器镜像并运行它，所做的任何更改都不会影响基础镜像。这是大多数开发者容易犯错的地方，因为他们习惯了长时间运行的虚拟机，虚拟机通常是一次性配置，然后多次启动和停止，并在整个过程中保持其内部状态。

Docker 容器是不同的。理想情况下，它们被设计为短暂和无状态的，而它们所基于的镜像则充当长期存在的蓝图，可以在多个执行环境中启动无限多个容器。

接下来是一个基本的 Docker 工作流示例，旨在帮助你熟悉最重要的 Docker 命令。无需担心记住命令；我们稍后将在本章中深入讲解它们。目前，我们只会解释每一步发生了什么，这样你就能适应下次微服务工作中会看到的内容。

首先，让我们启动 nginx 容器（`docker run`）并交互式地（`-it`）运行 Bash shell（`/bin/bash`）：

```
→  ~ docker run -it nginx /bin/bash 
```

这为我们提供了一个唯一容器的 shell 提示符，该容器是从`nginx`镜像启动的。现在，容器的 Bash shell 已连接到我们的终端：

```
root@e96107c9a58e:/# 
```

让我们写一个名为`test.txt`的文件并验证它是否存在：

```
root@e96107c9a58e:/# echo "I am immutable" >> test.txt
root@e96107c9a58e:/# cat test.txt
I am immutable 
```

我们可以通过常规命令`ctrl-d`退出 shell：

```
root@e96107c9a58e:/#
exit 
```

容器退出后，我们回到了常规的 shell。这是大多数第一次使用 Docker 的用户感到困惑的地方：让我们重新运行第一个命令，再次从 nginx Docker 镜像启动容器，并检查我们的文件：

```
→  ~ docker run -it nginx /bin/bash
root@c3b4d95ab9e6:/# cat test.txt
cat: test.txt: No such file or directory 
```

文件 A 错误报告！Docker 坏了！

其实不是的。这并不是你写入`test.txt`文件的那个容器。如果你仔细看，你可能注意到第二个容器的 shell 提示符中的主机名不同。这是因为每次运行`docker run`命令时，都会从指定的镜像启动一个新的容器。容器的设计是运行、退出并永远消失。

实际上，原始容器仍然存在：

```
→  ~ docker ps -a
CONTAINER ID   IMAGE             COMMAND                  CREATED          STATUS                      PORTS                    NAMES
c3b4d95ab9e6   nginx             "/docker-entrypoint.…"   14 minutes ago   Exited (1) 6 minutes ago                             agitated_hofstadter
e96107c9a58e   nginx             "/docker-entrypoint.…"   14 minutes ago   Exited (0) 14 minutes ago                            nervous_gould 
```

要删除它们，你可以使用`docker rm`并指定你要删除的容器的 ID：

```
→  ~ docker rm c3b4d95ab9e6
c3b4d95ab9e6 
```

你可以使用`docker start`启动一个停止的容器：

```
→  ~ docker start e96107c9a58e
e96107c9a58e 
```

此时，你将看到容器在 Docker 进程列表中运行：

```
→  ~ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS        PORTS     NAMES
e96107c9a58e   nginx     "/docker-entrypoint.…"   18 minutes ago   Up 1 second   80/tcp    nervous_gould 
```

然后你可以使用`docker exec`在容器内执行命令，再次通过`–it`启动并附加到 Bash shell 程序。然后，你可以查看我们修改过的文件系统状态（`test.txt`）：

```
→  ~ docker exec -it e96107c9a58e /bin/bash
root@e96107c9a58e:/# cat test.txt
I am immutable 
```

然而，长时间保留容器——修改它们的状态、停止并重新启动它们，而不是每次都从镜像启动新容器——是不被鼓励的，这会导致很多 Docker 最初帮助解决的相同问题。

让我们避免所有这些错误，通过强制删除它来永远删除这个正在运行的容器：

```
→ ~ docker rm -f e96107c9a58e
e96107c9a58e 
```

你也可以先运行`docker stop`，然后运行`docker rm`，但是使用`docker rm -f`强制删除将会一举停止并删除一个正在运行的容器。

你可以看到 Docker 鼓励使用不可变容器来保持状态不偏离镜像，而镜像是应用程序初始环境的“真理源”。如果你想修改镜像，不能直接修改——镜像是不可变的。

变更应该是*明确的*和*有意图的*，这对于创建和运行可靠的软件至关重要。我们怎么做呢？我们从原始镜像开始，以可控和可复现的方式做出更改（而不是“SSH 登录到服务器并尝试运行这些命令”），然后通过创建新镜像来保存这些更改。这里就需要强大的 Dockerfile。

# 使用 Dockerfile 创建镜像

如果你曾经被要求构建新的 Docker 镜像，或修改现有镜像——可能是为你正在开发的 Web 应用程序——你将会大量使用 Dockerfile（请参阅官方的 Dockerfile 文档：[`docs.docker.com/engine/reference/builder/`](https://docs.docker.com/engine/reference/builder/)）。

大部分新的软件都已经有了官方的（或者至少是开源的、第三方的）Dockerfile。即使你需要在使用这些 Dockerfile 之前做一些自定义，查找你所使用软件或框架的文档也是一个不错的选择。这些示例在软件发布重大升级时通常不会像你自己编写的自定义 Dockerfile 那样容易出错。

此外，一些框架或开发环境，例如 Spring Boot（Java），可以在构建过程中生成 Docker 镜像。

所以，即使你有可能永远不需要自己接触 Dockerfile，但这种可能性不大，你应该对它们的工作原理有基本的了解。

让我们看一下一个非常简单的 Dockerfile，来自开源的 HTTP 回显服务器项目（[`github.com/hashicorp/http-echo`](https://github.com/hashicorp/http-echo)）。它创建了一个 Docker 镜像，将一个作为简单 Web 服务器的 Go 二进制文件打包在内：

```
FROM alpine
ADD "https://curl.haxx.se/ca/cacert.pem" "/etc/ssl/certs/ca-certificates.crt"
ADD "./pkg/linux_amd64/http-echo" "/"
RUN apk add curl
ENTRYPOINT ["/http-echo"] 
```

基本上，这通过以下方式创建一个新的容器镜像：

1.  使用 `alpine` Linux 镜像作为基础进行构建。

1.  下载一些证书并将它们添加到镜像层中（本质上是将某些东西添加到最终容器的文件系统中）。

1.  将构建目录中的 `http-echo` 二进制文件复制到容器镜像中。

1.  运行 Alpine 包管理命令来安装 `curl` 程序。

1.  定义在从这个镜像启动的容器启动时运行的可执行文件或命令。

这些步骤中的每一个都由 Dockerfile 解析器知道如何执行的（大写）*指令*触发。这个特定的 Dockerfile 只使用了 Dockerfile 中可用指令的子集（`FROM`、`ADD`、`RUN` 和 `ENTRYPOINT`）。

这是创建新 Docker 镜像时，Dockerfile 中可用的完整指令集：

+   `ARG`：声明一个构建时参数；基本上是一个在构建过程中稍后使用的变量。

+   `ENV`：在构建过程中设置的环境变量，这些变量将在你的运行容器环境中持久存在（*不仅仅在构建期间*）。采用 `key=value` 格式。

+   `FROM`：基础镜像。

+   `CMD`：为容器启动时运行的默认命令（或默认 `ENTRYPOINT` 参数）。可以被覆盖，但在 Dockerfile 中应有一个。每个 Dockerfile 只能有一个 `CMD`，如果有多个 `CMD`，只有最后一个会生效。

+   `ADD`：一个灵活的指令，复制文件和目录，将它们添加到镜像的文件系统中。它也可以用于从镜像外部或远程 URL（通过 HTTP）复制文件，并进行复杂操作，如扩展、解压、解档等。你在上面的示例 Dockerfile 中看到它作为 `curl` 命令的替代，用于下载 CA 证书文件。

+   `COPY`：仅复制文件和目录，比 `ADD` 更简单、魔法性和功能性更强。

+   `LABEL`：添加镜像元数据，格式为 `key=value`。

+   `EXPOSE`：通知使用此镜像的消费者，该容器将监听哪些网络协议和端口。

+   `ENTRYPOINT`：告诉容器启动时运行哪个命令。使用 *exec 形式* (`ENTRYPOINT ["executable", "param1", "param2"]`) 以确保容器能够接收并响应来自容器外部的信号。

+   `RUN command arg1 arg2`：在镜像的 shell 中运行 `command`，并传入参数 `arg1` 和 `arg2`：

    +   `RUN ["command", "arg1", "arg2", "argN"]`：与上面相同，但有助于避免 shell 字符串混乱。

    +   每个 `RUN` 指令都会在新的镜像层中执行（我们在这里不深入探讨镜像层，但知道这一点可能会有帮助）。

    +   `RUN --mount` 可用于在构建过程中暂时将文件系统挂载到容器中，而无需将文件本身复制到镜像层中。

    +   `RUN --network` 和 `RUN --security` 也存在，用于分别管理网络上下文和特权容器。

+   `WORKDIR`：为 Dockerfile 中随后的指令设置工作目录。相当于类 Unix 操作系统中的 `cd`。

+   `SHELL`：覆盖在 Docker 构建过程中用于解释命令的默认 shell。命令必须使用 exec 形式。

+   `STOPSIGNAL`：设置此容器应解释为退出信号的系统调用信号。默认情况下，这是 SIGTERM，像其他 Linux 进程一样。

+   `VOLUME`：定义将从主机挂载进来的卷。

+   `USER`：将构建命令使用的（容器）用户更改为此后的用户（可以在构建过程中多次切换用户）。

+   `ONBUILD`：定义一个指令，当该镜像作为基础镜像用于另一个构建时触发。

+   `HEALTHCHECK`：一些健康检查功能，你可能不会使用它，因为你的容器调度器有自己的健康检查功能。

我们将通过一个实际的、端到端的示例来演示如何将这些内容结合起来，但首先让我们更加深入地回顾一下我们刚才使用过的命令。

# 容器命令

现在让我们深入探讨一些较为复杂但重要的命令和命令调用，这些命令你在使用 Docker 时可能会遇到。

## docker run

让我们来看一下之前使用过的`docker run`命令的一个更复杂的调用：

```
 docker run --rm --name mywebcontainer -p 80:80 -v /tmp:/usr/share/nginx/html:ro -d nginx 
```

+   `--rm`：容器退出时清理（删除）此容器。

+   `--name mywebcontainer`：给这个容器起一个友好的名字——`mywebcontainer`。

+   `-p 80:80`：将主机的`80`端口映射到容器的`80`端口。左边的端口号代表“外部”（运行容器的环境），右边的端口号代表“内部”（容器）端口。例如，`-p 4000:80`将容器的`80`端口映射到`localhost:4000`。

+   `-v /tmp:/usr/share/nginx/html:ro`：挂载一个卷——主机环境中的`/tmp`目录将被挂载到容器中的`/usr/share/nginx/html`目录；`:ro`确保这是一个只读挂载（挂载的文件不能从容器内修改）。

+   `-d`：以分离模式（在后台）运行容器。

+   `nginx`：用于启动此容器的镜像。

如果你想在`http://localhost:80`上查看一些 HTML 内容，你可以将一个`index.html`文件添加到你的`/tmp`目录中：

```
cat <<EOF > /tmp/index.html
<!doctype html>
<h1>Hello World</h1>
<p>This is my container</p>
</html>
EOF 
```

因为我们的`/tmp`目录映射到了容器的`/usr/share/nginx/html`目录（nginx 会在此目录中查找 HTML 文件），nginx 会立即识别并开始提供该文件。

卷是有状态应用程序可以使用无状态容器运行的机制。

### docker image list

要查看你本地下载的镜像，可以运行`docker images`（或如果你更喜欢，可以运行`docker image list`）。

如果你已经构建并使用了很多 Docker 镜像，列表可能会很长！

```
$ docker image list
REPOSITORY                                                 TAG                  IMAGE ID       CREATED         SIZE
nginx                                                      latest               51086ed63d8c   10 days ago     142MB
vault                                                      latest               22fdc6314051   2 months ago    207MB
golang                                                     1.19-alpine          d0f5238dcb8b   2 months ago    352MB 
```

## docker ps

`docker ps`有点像 Linux 中的`ps`命令。它让你查看哪些容器正在主机上运行，并显示一些信息，比如它们的 ID、正在运行的命令、创建时间和运行时间、端口映射等。

运行命令

```
$ docker ps 
```

将产生类似以下的输出：

```
CONTAINER ID   IMAGE          COMMAND                  CREATED              STATUS              PORTS                NAMES
2aca849eef73   nginx          "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   mywebcontainer 
```

## docker exec

在容器镜像的开发过程中，通常会进入容器并运行命令。要在运行中的容器中启动一个交互式的 shell，可以使用`docker exec`：

```
docker exec -it mywebcontainer /bin/bash 
```

对于我们之前启动的 nginx 容器，这将会在容器环境内启动一个 Bash shell。你所做的任何状态更改（如文件创建、内核设置等）将在容器停止时丢失——下一个`docker run`会简单地从相同的基础镜像状态启动一个新的容器。

## docker stop

要停止一个容器，运行`docker stop $CONTAINERNAME`——如果容器没有友好名称，你也可以使用容器 ID：

```
docker stop mywebcontainer 
```

如果容器是使用`--rm`选项启动的，就像我们启动的 nginx 容器一样，容器将在停止后被删除，并且其状态（如果与基础镜像有差异）将丢失。

如果容器没有使用`--rm`选项启动，它的状态将保留在你的文件系统中，你可以通过`docker start $CONTAINERNAME`再次启动该容器。其状态将被保留。

# Docker 项目：Python/Flask 应用容器

我们将容器化一个使用 Flask Web 框架的小型 Python Web 服务。这是一个非常常见的模式，Python 非常适合容器化，因为很多 Python 项目的打包和依赖管理非常混乱。你将自己创建所有文件——试着使用命令行文本编辑器进行练习！

## 1\. 设置应用程序

首先，创建一个新目录并进入该目录：

```
mkdir dockerpy && cd dockerpy 
```

创建一个小型的 Python Web 应用程序。在这个示例中，我使用的是 vim 编辑器，但你可以使用任何你喜欢的编辑器：

```
vim echo_server.py 
```

将以下文本粘贴进去：

```
from flask import Flask, request
import os
app = Flask(__name__)
@app.route('/')
def echo():
    return {
        "method": request.method,
        "headers": dict(request.headers),
        "args": request.args
    }
@app.route('/health')
def health():
    return {"status": "healthy"}

if __name__ == "__main__":
    env_port = os.environ.get("PORT", 8080)
    app.run(host='0.0.0.0', port=env_port) 
```

这就是整个 Web 应用程序——它只是读取一些来自传入请求的信息，并利用这些信息将响应推送回客户端。

保存并退出文件（`esc`，`:x`）。

创建一个名为 `requirements.txt` 的文件，文件内容仅包含以下一行：

```
Flask>=3.0.0 
```

接下来，创建你的 Dockerfile：

```
vim Dockerfile 
```

输入以下内容：

```
# Use an official Python base image
FROM python:3.12-slim

# Set the working directory inside the container
WORKDIR /app

# Copy our list of dependencies into the container
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the script into the container
COPY echo_server.py .

# Set a healthcheck to kill the container if it's not listening on the internal port
HEALTHCHECK --interval=30s --timeout=5s \
  CMD curl --fail http://localhost:8080/health || exit 1

# Expose port for the application
EXPOSE 8080

ENV PORT=8080
CMD ["python", "echo_server.py"] 
```

目前你只需要这些。你的 `dockerpy` 目录现在应该包含三个文件：

+   Dockerfile

+   `echo_server.py`

+   `requirements.txt`

## 2\. 创建 Docker 镜像

使用 `docker build` 命令构建一个新的 Docker 镜像。`-t` 用于为容器打标签并命名：

```
docker build -t dockerpy . 
```

请注意末尾的 `.` 字符，它指示 Docker 使用当前目录作为其构建上下文。

## 3\. 从你的镜像启动一个容器

你已经在本章之前使用过 `docker run` 命令。现在用它从你新构建的镜像启动一个容器：

```
docker run --rm -d -p 8080:8080 --name my-dockerpy dockerpy
(the command will print out the ID of your new container) 
```

这里有一些新的参数：

+   `--rm` 告诉 Docker 在容器退出时删除该容器。这防止了旧的容器在文件系统中滞留，就像你在本章的第一个示例中看到的那样。

+   `-d` 告诉 Docker 将容器后台运行，这样它就不会在前台附加到你的终端。

+   `-p` 设置端口映射：冒号左边是容器端口，右边是它映射到的主机端口。如果容器应用程序在端口 `1234` 上运行，并且你希望它映射到主机的端口 `80`，那么命令会写成 `–p 1234:80`。

+   `--name` 为你的容器标记一个名字，方便你在 `docker ps` 的输出中找到它。

现在，你的容器化应用程序已经运行，并且可以通过浏览器或命令行访问它。让我们使用 `curl` 命令连接并向 web 服务发送请求：

```
curl localhost:8080
{"args":{},"headers":{"Accept":"*/*","Host":"localhost:8080","User-Agent":"curl/8.1.2"},"method":"GET"} 
```

对于那些经历过无法重现的依赖地狱（Python、Ruby 等因这类问题而出名）的人来说，这应该是一个启示。你过去需要与应用程序一起拖动的所有复杂性——从本地开发环境到 CI 和测试，再到预生产，最后到生产——现在都浓缩成一个单一的制品，保证无论在哪里运行，它都包含相同的内容。

我们之前没有用过的一个命令是 `docker exec`，它允许你在正在运行的容器内执行命令。如果由于某些原因，你必须检查或修改一个正在运行的容器，这个命令非常有用：

```
docker exec -it my-dockerpy /bin/sh 
```

这会启动并连接到容器中的 `/bin/sh`（大多数生产容器中只会有一个最小的 shell，在 `/bin/sh`，并且不会配备像 Bash 这样功能全面的 shell）。

让我们通过接下来要讲解的最后一个命令 `docker kill` 停止服务器：

```
docker kill my-dockerpy 
```

这会向进程发送 `SIGKILL`（信号 9），而不是 `SIGTERM`（信号 15），并立即停止进程，而不给它机会优雅地关闭。

# 容器与虚拟机

现在你已经初步了解了创建和使用 Docker 镜像的工作流程。然而，了解容器和虚拟机之间的基本区别也是非常有益的。当你在排除操作问题时，这些知识可能会派上用场，同时它也是面试中常见的一个问题，用来评估你对容器化原理的理解程度。

**虚拟机**（**VMs**）允许你在另一个主机操作系统上运行完整的操作系统，如 Linux、Windows 或 DragonFly BSD。虚拟机独立于宿主系统运行。实际上，在 macOS 上运行 Docker 会透明地使用虚拟机来提供 Docker 所需的 Linux 操作系统。

因此，虚拟机会运行完整的操作系统，如 Linux，这又使用像 `systemd` 这样的初始化系统。由于这一点，你管理服务和进程的方式就像管理物理机器一样。就日常使用而言，适用于物理机器的所有操作也适用于虚拟机。然而，容器的使用方式通常并非如此。

**Docker 容器**通常只包含单个应用程序；实际上，它们通常只包含一个进程。如果容器内有多个进程，通常是因为一个多进程的应用程序生成了子进程（例如，网页服务器或命令执行程序通常会这样做）。由于广泛认同的最佳实践是让容器只运行一个进程，并且在该进程退出时容器也会退出，所以任何形式的内部进程监督和管理在这里都是多余的。

相反，你会发现通常由操作系统的初始化系统完成的工作已经转移到容器运行时环境之外，转移到外部系统中，这些系统负责*管理*容器，例如 Kubernetes、Nomad 等。

在这种新模型中，容器类似于操作系统进程，而容器编排器则扮演各种操作系统和调度器的角色。

在 Docker 容器中，PID1（一个完整 Linux 操作系统中的初始化系统）是你的 CMD 或 ENTRYPOINT。通常，这就是你正在运行的软件的主进程。通常情况下，容器应该运行一个单一的进程。虽然在某些场景中，人们故意以不同的方式运行容器，但运行一个单独的进程，并在该进程停止时让容器停止，是预期的行为。尤其是在将服务容器化并运行在生产环境中时，应该确保遵循这种方式。这个规则有例外，特别是当运行一些早于 Docker 容器普及的旧软件时，但在这种情况下，你通常会有所了解，并且往往是基于专为此目的制作的容器。

# 关于 Docker 镜像仓库的快速说明

在本章中，我们一直在使用 `nginx` 镜像。但这个镜像到底来自哪里呢？默认情况下，Docker 会尝试从 Docker Hub 下载镜像（[`hub.docker.com/`](https://hub.docker.com/)），Docker Hub 是一个公共 Docker 镜像的中央仓库。Docker Hub 的工作方式类似于 Linux 软件包仓库，包含了可以随时使用的上传 Docker 镜像。大多数流行的服务器软件都可以在那里找到，像你刚刚看到的 `nginx` 一样，下载并使用非常简单。

然而，并不是所有的应用都是公开的，使用私有仓库来存储 Docker 镜像是很常见的。Docker 镜像仓库提供商的名单不断变化，所以我们不会在这里列出它们，但了解它们都与 Docker Hub 的工作方式相同就足够了。

# 痛苦的容器化经验教训

当你开始构建自己的容器时，记住 Docker 官方文档中讨论的最佳实践可以帮助你避免许多问题，详细内容请参考：[`docs.docker.com/get-started/09_image_best/`](https://docs.docker.com/get-started/09_image_best/)。

话虽如此，我们列出了一些我们注意到的最严重的容器化错误，以及如何避免它们。这个部分是许多不眠之夜、故障和艰难学习的结果。

## 镜像大小

从最小的镜像开始，比如 Scratch 或 Alpine。为了部署大多数应用程序，尽量避免使用像 Ubuntu 这样的大型镜像和发行版是一个好主意。如果构建过程中需要依赖项，建议在构建较大或多容器项目时，删除这些依赖项或使用中间构建容器。

小巧、精简的镜像不仅意味着更快的下载速度和更少的资源消耗，还使得你更容易管理它们。如果一个镜像没有包含你不需要的软件和库，那么你需要更新的内容就更少，攻击者可以利用的漏洞面更小，容器安全扫描工具发出的噪声警告也更少。

## C 标准库

需要了解你正在使用的**C 标准库**（也叫做 *libc*）。许多 Linux 发行版使用 `glibc`；一些像 Alpine Linux 则使用 `musl` 或其他库。这些库及其生成的二进制文件可能在不同的系统间不兼容。例如，在 Alpine 上，你可能需要自己编译一些不太常见的工具。如果你的项目依赖于基础镜像中通过包提供的某些库，你可能会遇到不兼容的问题。当然，升级、降级或完全切换基础镜像可能会引发类似的问题。

然而，由于 Alpine 和 `musl` 已稳步获得普及，这些问题变得不太可能发生（至少，更容易通过 Google 搜索找到解决办法！）。如果你不依赖任何 C 库，通常这不会是个问题。另外，静态编译你的代码可以使你更独立于底层系统，从而不再依赖基础镜像。

## 生产环境不是你的笔记本：外部依赖

不要依赖本地挂载或其他本地容器。已部署容器的环境通常与你的笔记本电脑大不相同。仅仅因为你在笔记本上把数据库容器和 Web 应用容器放在一起，并不意味着这些容器在生产环境中会被调度到同一台机器上。

数据卷也是如此——这些容器外的接触点是你需要与 Ops/DevOps 同事进行规划的地方。你可能会通过调度器或其他 DevOps 工具集成服务发现、健康检查和共享卷。

# 容器理论：命名空间

如果你在想这些容器魔法是如何在底层工作的，或者只是担心有一天在压力下需要排查一个容器环境，那么了解命名空间的概念会很有帮助。如果你不关心容器抽象是如何在 Linux 上构建的，可以跳过这一部分。

命名空间是一个多重含义的术语，在不同的技术领域有不同的解释。在 Linux 容器的上下文中，命名空间的概念最好通过 `chroot`（更改根目录）来解释。`chroot` 是一个老旧的 Unix 和类 Unix 操作系统工具，允许用户更改文件系统的根路径（即 `/` 路径）。

这个工具的使用其实非常简单：`chroot /some/path`将设置`/some/path`为新的根目录`/`。除了允许操作系统安装程序切换到当前正在安装的系统以执行命令外，它还允许进行基本的命名空间隔离。事实上，许多软件和不同 Linux 发行版的配置已经在利用`chroot`来增强安全性，因为使用`chroot`本质上将文件系统的某些部分排除在当前可读范围之外——它使得新根目录外的任何内容都无法访问。因此，如果攻击者利用某个漏洞在运行在`chroot`环境中的 web 服务器上执行远程代码，那么系统和此目录外的任何文件将保持不受影响。

用于在 Linux 和其他操作系统上实现容器的技术原语在过去十年中发生了显著变化，未来可能还会继续变化。幸运的是，低级实现对你作为主要使用容器化技术的软件工程师并不至关重要，而对于容器技术的操作员或实施者则非常重要。

“容器”抽象依赖于像以下技术：

+   文件系统命名空间（例如，使用`chroot`）。

+   用户和进程的命名空间，使得容器外部的进程在容器内部不可见。换句话说，容器中的 root 和 PID 5 将分别映射到容器命名空间外的非特权用户和另一个进程 ID。

+   资源分组和计费技术，如`cgroups`。

+   网络虚拟化/命名空间，使得容器无法直接访问网络接口，同时也可以处理端口号重叠问题。例如，你可以运行两个不同的容器并暴露端口`8080`；由于容器的网络栈相互独立，因此不会出现端口已被占用的错误。

# 我们如何通过容器进行运维？

虽然本书不是面向系统管理员或站点可靠性工程师的，但你应该了解容器通常运行的基本背景。主要思想是，容器大致上是无状态的“函数”，它们处理输入（来自其他服务的 web 请求或 HTTP 消息）并生成输出（web 响应、侧面效应和流向 STDOUT 的日志）。在运行良好的操作环境中，容器可以视为 Linux 进程或编程中的函数的类比。

容器通常通过像 Kubernetes、Nomad 等第三方工具层“调度”到主机上。如果容器类似于进程，那么这些工具就充当操作系统调度器的角色（整体系统是一个分布式系统，而不是单一主机）。

容器的输出通常由相同的工具捕获，并重定向到日志解决方案，如 Logstash、Graylog 和 Datadog。所有运行中的容器的度量指标可以被提取并输入像 Prometheus 这样的工具进行分析和故障排除。

# 结论

在这一章中，你对与 Docker 及容器工作相关的最重要的事项进行了快速浏览。虽然单独的技术可能会发生变化——比如哪个容器调度器流行，或如何最好地处理日志流——但我们始终专注于每个现代软件开发者应该掌握的核心理论和技能。

我们希望你从这一章中带走几个主要观点。首先，我们希望你能直观地理解容器化为人们解决了哪些问题，主要通过控制复杂性并将依赖打包成一个单一的工件。

同样重要的是要记住图像和容器之间的区别，并且通过使用官方文档，练习从零开始构建自己的 Dockerfile。

我们希望访问一些更高级的话题，比如虚拟机和容器的区别，以及命名空间是如何工作的，这些会在故障排除或面试时派上用场。我们讨论的最佳实践也会在这些情况下有所帮助。

最后，为了巩固你的学习，我们建议你通过将自己的应用程序容器化来练习这些技能。你将学到很多东西，而且在本章的所有信息仍然鲜活在你脑海中的时候开始会更加容易。

# 在 Discord 上了解更多

要加入本书的 Discord 社区——在这里你可以分享反馈、向作者提问并了解新版本——请扫描下面的二维码：

[`packt.link/SecNet`](https://packt.link/SecNet)

![](img/QR_Code1768422420210094187.png)
