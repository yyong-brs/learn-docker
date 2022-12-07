# 第三章 构建你自己的镜像

在上一章你已经通过 Docker 运行了一些容器，本章将让你了解如何构建自己的镜像。你将接触到学习 Dockerfile语法以及一些构建镜像的技巧。

## 3.1 使用 Docker Hub 的镜像

我们将从您将在本章中构建的镜像的最终版本开始，可以看到它是如何与Docker协同工作的。现在就试试吧，使用一个名为web-ping的简单应用程序，它检查网站是否正常。应用程序
将在容器中运行，并每隔三秒钟执行一次直到容器停止。

你从第2章中知道 docker container run 命令执行时，将会下载你机器上还没有镜像。这是因为软件被分发到了 Docker 平台中，你可以让 Docker 来管理应用，当你需要的时候它会提取镜像或者你可以显式地通过 Docker ClI 运行。

<b>现在就试试</b> 拉取 web-ping 应用的镜像:

```
docker image pull diamol/ch03-web-ping
```

你将会看到 图3.1 类似的输出：

![图3.1](./images/Figure3.1.png)
<center>图3.1 </center>

镜像名称是 `diamol/ch03-web-ping` ,它被存储在 Docker Hub 中。Docker Hub 是 Docker 获取镜像的默认仓库。镜像服务器被称为 registry,然后 Docker Hub 是你可以使用的一个免费的公共 registry。同时 Docker Hub 也提供了 web 界面，你可以通过网站 https://hub.docker.com/r/diamol/ch03-web-ping 来获取上面提到的镜像的信息。

`docker image pull` 命令有一些有趣的输出,它向您展示镜像如何存储。Docker 镜像在逻辑上是可以把它看作一个包含整个应用程序堆栈的大的zip文件。这个镜像包含了 Node.js运行时以及我的应用程序代码。

在拉取过程中，你看不到一个文件被下载;但是你会看到很多下载的信息。这些被称为镜像层。Docker 镜像在物理上被存储为许多小文件，Docker 将它们组装在一起创建容器的文件系统。当所有镜像层都被拉取后，完整的镜像就可用使用了。

<b>现在就试试</b> 让我们基于这个镜像运行一个容器并且看看应用做了什么:

`docker container run -d --name web-ping diamol/ch03-web-ping`

`-d` 参数是 `--detach` 的简写，所以这个容器将在后台运行。这个程序运行时就像一个没有用户界面的批处理任务，不像我们之前在第二章运行的 web 站点容器，这个容器不再接收网络请求，所以你不用公开任何的端口。

这里出现了一个新的参数 `--name`,你知道的你可以使用 Docker 生成的 ID 来管理容器，但是你也可以给容器一个友好的名字。在这个容器中被命名为 web-ping ,你可以使用这个名字来替代随机生成的 id 来指向容器。

我的博客网站现在被你运行的容器中的应用 ping，这个应用运行在一个无限循环中，你可以通过在第二章中熟悉的命令来查看做了些什么。

<b>现在就试试</b> 查看一下容器产生的日志:
`docker container logs web-ping`

你将会看到类似 图 3.2 的输出，显示了应用请求 博客网站的信息：

![图3.2](./images/Figure3.2.png)
<center>图3.2 </center>

一款能够发出网络请求并记录响应时间的应用程序是相当有用的，您可以使用它作为监控网站正常运行时间的基础。但是这个应用程序看起来像是用我的博客的硬编码，所以除了我，它对任何人都没什么用。

但事实并非如此。应用程序实际上可以配置为使用不同的URL、请求之间的不同间隔，甚至是不同类型的HTTP调用。这个应用程序从系统环境中读取它应该使用的配置值变量。

环境变量只是操作系统提供的键/值对。它们在 Windows 和 Linux 上以相同的方式工作，而且它们是一种非常简单的方法来存储小块数据。Docker容器也有环境变量，但是它们不是来自计算机的操作系统，而是由 Docker 设置的，这与 Docker为容器创建主机名和IP地址的方式相同。

web-ping 镜像为环境变量设置了一些默认值。当你运行一个容器，那些环境变量被Docker填充，那是应用程序使用什么来配置网站的URL。可以在创建容器时指定不同的环境变量值，这将改变应用程序的行为。

<b>现在就试试</b> 删除之前运行的容器，通过指定 TARGET 环境变量值来运行一个新的容器：

```
docker rm -f web-ping
docker container run --env TARGET=google.com diamol/ch03-web-ping
```
你可以看到图 3.3 类似的输出:

![图3.3](./images/Figure3.3.png)
<center>图3.3 </center>

这个容器正在做一些不同的事情。首先，它是交互式运行的，因为你没有使用 `--detach` 标志，因此应用程序的输出显示在控制台。容器将继续运行，直到按Ctrl-C结束应用程序。第二,现在是 pinging google.com 而不是 blog.sixeyed.com。

这将是你从本章中学到的主要内容之一——docker 镜像可以打包为应用程序的一组默认配置值，但是您应该能够在运行容器时提供不同的配置设置。

环境变量是实现这一点的一种非常简单的方法。web-ping应用程序代码查找带有 TARGET 键的环境变量，那个键有一个在镜像中的blog.sixeyed.com的值，但您可以用`docker container run `命令使用 `--env` 参数。图3.4显示了如何让容器有自己的设置，它们彼此不同，也不同于镜像。

![图3.4](./images/Figure3.4.png)
<center>图3.4 </center>

主机也有它自己的一组环境变量，但它们和容器的环境变量是分开管理的。每个容器只有 Docker 填充的环境变量。图3.4中重要的一件事是 web-ping 应用程序在每个容器中都是相同的—它们使用相同的镜像，因此应用程序正在运行完全相同的二进制文件集，但由于配置的不同，其行为有所不同。

这取决于 Docker 镜像的作者来提供这种灵活性，而当你从Dockerfile构建你的第一个 Docker 镜像时，我们会看到如何做到这一点。

## 3.2 编写第一个 Dockerfile

Dockerfile 是一个用来打包应用程序的简单脚本，它是一组指令，最终输出 Docker 镜像。Dockerfile 语法简单易学，您可以使用 Dockerfile 打包任何类型的应用程序。作为脚本语言，它很灵活。常见的任务都有自己的命令，您可以使用标准的shell命令（Linux上的Bash或Windows上的PowerShell）。清单3.1显示了如何打包 web-ping 应用程序。

> 展示 3.1 web-ping 应用的 Dockerfile

```
FROM diamol/node
ENV TARGET="blog.sixeyed.com"
ENV METHOD="HEAD"
ENV INTERVAL="3000"
WORKDIR /web-ping
COPY app.js .
CMD ["node", "/web-ping/app.js"]
```

即使这是你见到过的第一个 Dockerfile，你可能也会猜到文件内容发生了什么。Dockerfile 的指令包括 FROM、ENV、WORKDIR、Copy和CMD；它们都是大写的，这是惯例，但是不是必须的。每条指令的详细说明如下：
- FROM—每个镜像都必须基于另一个镜像开始构建，在本例中，web-ping 镜像将使用 diamol/node 镜像作为起点。这个镜像已经安装好了 Node.js，这就是 web-ping 应用程序运行所需的一切。
- ENV—设置环境变量的值。语法为[key]="[value]"，这里有三个ENV指令，设置三个不同的环境变量值。
- WORKDIR—在容器镜像文件系统中创建一个目录，并将其设置为当前工作目录。前斜杠语法适用于Linux和Windows容器，所以这个指令将在 Linux 镜像创建 /web-ping，在 Windows 镜像创建 C:\web-ping。
- COPY—将文件或目录从本地文件系统复制到容器镜像。语法是[源路径] [目标路径]-在本例中，我正在复制我的本地机器的App.js到镜像中的工作目录。
- CMD—指定从镜像启动容器时要运行的命令，此处将运行Node.js，启动 app.js 中的应用程序。

就是这样，在Docker中打包自己的应用程序所需的基本就是这些信息，在这五行代码中已经包好了一些良好的实践。

<b>现在就试试</b>  你无需从此处拷贝 Dockerfile 内容，所有的代码片段都在本书的源码中，在第一章中你已经下载了代码，找到你下载的目录，检查目录文件：

```
cd ch03/exercises/web-ping
ls
```

你可以看到三个文件：
- Dockerfile (没有后缀), 和清单 3.1 内容一致
- app.js,  web-ping 应用的 Node.js 程序代码
- README.md, 使用该镜像的简单说明

你可看到与图 3.5 一样的内容.

![图3.5](./images/Figure3.5.png)
<center>图3.5 </center>

你不需要了解docker 运行的本应用所涉及的 Node.js 或者 javascript 语言知识。如果你查看 app.js 的代码，你会发现它非常基础，它使用了标准的 Node.js 库执行 Http 调用，同时从环境变量获取配置信息。 

在这个目录你拥有构建 web-ping 应用程序镜像的所有文件。

## 3.3 构建镜像

Docker 在基于 Dockerfile 构建镜像之前，需要掌握一些信息，它需要知道目标镜像的名称，以及将要打包到镜像中的所有文件的位置信息。你已经在终端中打开了正确的目录，所以你已经准备好构建镜像了。

<b>现在就试试</b>  通过运行 docker image build 命令根据 Dockerfile 去构建镜像：

`docker image build --tag web-ping .`

`--tag` 参数指定了构建的目标镜像名称，最后的参数指定了 Dockerfile 以及相关文件的目录，Docker 将这个目录称作“上下文”，此处的点号代表“当前目录”。通过这个构建命令你将会看到执行所有 Dockerfile 指令的输出信息。类似图 3.6：

![图3.6](./images/Figure3.6.png)
<center>图3.6 </center>

如果构建命令返回错误信息，你首先需要检查 Docker 引擎是否启动。必须确保 Docker Desktop 程序运行在你的 Windows 或 Mac 机器上。然后检查是否处于正确的目录，你应该位于 ch03-web-ping 目录，该目录包含了 Dockerfile 以及 app.js 文件。最后，检查是否输入了正确的构建命令。

当你看到“successfully built”以及“successfully tagged” 的信息输出，说明你的镜像已构建。它被保存在本地的镜像缓存目录,然后你可以通过 Docker 命令查询镜像。

<b>现在就试试</b> 查询所有以 “w” 开头的镜像名:

`docker image ls 'w*'`

你将会看到 web-ping 镜像被查询出来:

```
> docker image ls w*
REPOSITORY TAG IMAGE ID CREATED SIZE
web-ping latest f2a5c430ab2a 14 minutes ago 75.3MB
```

你可以和通过 Docker Hub 仓库下载的那个镜像一样的方式使用此处构建的镜像，因为应用的内容是一样的，然后配置信息可以通过环境变量进行指定。

<b>现在就试试</b> 基于你自己构建的镜像运行一个每隔 5 秒 ping Docker 网站的容器:

`docker container run -e TARGET=docker.com -e INTERVAL=5000 web-ping`

你将会看到类似图 3.7 的输出，对比一下环境变量信息：

![图3.7](./images/Figure3.7.png)
<center>图3.7 </center>

容器在前台运行，所以需要通过 Ctrl-C 停止它，那样会终止应用程序，最终容器进入 exited 状态。

你已经打包了一个简单的应用程序在 Docker 中运行，该应用进程和那些复杂的应用其实是差不多的。你编写了 Dockerfile 描述了打包应用的步骤，收集了需要包含进镜像的资源，然后配置了用户如何通过配置信息来控制应用程序的行为。

## 3.4 理解 Docker 镜像以及镜像层

在阅读本书的过程中，你将会构建更多的镜像。在本章中，我们将继续使用这个简单的例子，来更好的理解镜像是如何工作的、以及容器和镜像之间的关系。

Docker 镜像包含了你打包的所有文件，这些文件成为了容器的文件系统的一部分，然后同时也包含了很多镜像本身的元数据信息，也就是镜像如何构建的简要过程，你可以通过它来查看镜像的每一层以及每一层的命令。

<b>现在就试试</b> 检查你的 web-ping 镜像的历史信息:

`docker image history web-ping`

你将会看到每个镜像层的输出行信息，下面是我执行时前面几行信息：

```
> docker image history web-ping
IMAGE CREATED CREATED BY
47eeeb7cd600 30 hours ago /bin/sh -c #(nop) CMD ["node" "/web-ping/ap…
<missing> 30 hours ago /bin/sh -c #(nop) COPY file:a7cae366c9996502…
<missing> 30 hours ago /bin/sh -c #(nop) WORKDIR /web-ping
```

CREATED BY 这一列展示了 Dockerfile 指令信息，与 Dockerfile 是一对一的关系，所以 Dockerfile 中的每一行都创建了一个镜像层。我们将稍微深入研究一下，因为理解镜像层是你充分使用 Docker 的关键。

Docker 镜像是镜像层的逻辑集合。层是物理上存储在 Docker 引擎的缓存文件，这就是为什么这很重要:镜像层可以在不同的镜像和不同的容器之间共享。如果你有很多运行 Node.js 应用的容器，它们都将共享同一组包含Node.js 运行时的镜像层。图3.8 显示了其工作原理。

![图3.8](./images/Figure3.8.png)
<center>图3.8 </center>

此处的 diamol/node 镜像包含一个精简的操作系统层，然后就是 Node.js 运行时程序。这个 Linux 的镜像占用大约 75MB 的磁盘空间（windows 类型的容器镜像操作系统层会更大，接近 300MB左右）。你的 web-ping 镜像基于 diamo/node 镜像构建，所以它从该镜像的所有层开始——Dockerfile 中 FROM 指令指示了这么做。在基础镜像之上打包的 app.js 文件只有几千字节大小，所以你认为 web-ping 镜像总共有多大？

<b>现在就试试</b> 你可以使用 docker image ls 查看镜像清单,同时也会显示镜像大小:

`docker image ls`

你的输出将会和图 3.9 类似。

![图3.9](./images/Figure3.9.png)
<center>图3.9 </center>

看上去所有的 Node.js 镜像占用了同样的 75MB 的空间，它们包括： diamol/node, 从 Docker Hub 拉取的原始示例镜像 diamol/ch03-web-ping，以及你自己构建的镜像 web-ping。它们共享了基础镜像层，但是 docker images ls 的输显示每个镜像都是差不多 75M 大小，所以它们总共是 75 * 3 = 225 MB ？

并不完全正确，你看到的 size 列只是镜像的逻辑大小——指的是如果没有其它镜像存在的情况下该镜像实际占用的磁盘空间大小。如果你还有其它镜像共享层，Docker 镜像占用的磁盘空间相对会小一些，所以你无法从 image 清单看到真实信息，但是 Docker system 命令可以告知你更多的信息。

<b>现在就试试</b>  我的镜像清单显示了总共 363.96 MB 镜像大小,但是那只是逻辑上的总数。 system df 命令实际会显示 Docker 占用的磁盘大小:

`docker system df`

![图3.10](./images/Figure3.10.png)
<center>图3.10 </center>

你可以通过图 3.10 看到我的镜像缓存实际占用 202.2MB，也就意味着有 163MB 的空间被镜像共享层所共享，大约节省 45% 的磁盘空间。当你拥有大量共享同样的基础层的运行时镜像时，将会节省很多的磁盘空间，那些基础层可以是 java\.Net\PHP 等，不管你使用什么技术栈，Docker 的行为是一模一样的。

最后再说明一下，如果镜像层被共享，则它们就不可以被编辑——否则一个镜像的更改将会影响所有其它共享层的镜像。Docker 通过将镜像层设置为只读，来控制这一点。一旦你通过构建镜像创建了一个层，那么该层就可以被其它镜像所共享，而且也不可变更，你可以利用这一点来使得你的镜像尽量小一点，通过优化 Dockerfile 文件来加快构建的速度。

## 3.5 优化 Dockerfile 使用镜像缓存层

在你的 web-ping 镜像中包含了一个 JavaScript 文件的层，如果你对那个文件做点变更然后重新构建你的镜像，你将会得到一个新的镜像层。Docker 假设镜像中的层遵循定义的序列，所以如果你在序列中间更改一个层，Docker 将不会认为可以重用后续的层。

<b>现在就试试</b>  对 ch03-web-ping 目录中的 app.js 文件做出一些修改，你不必修改代码，只需要新增一个空行即可，然后构建一个新版本的 Docker 镜像:

`docker image build -t web-ping:v2 .`

你将会看到与图 3.11 类似的输出。 步骤 2 到 5 使用了缓存中的层,然后步骤 6 和 7 生成了新的层。

![图3.11](./images/Figure3.11.png)
<center>图3.11 </center>

每个 Dockerfile 指令都会生成一个镜像层，但如果某指令在构建之间没有变化并且进入指令的内容是相同的，那么 Docker 会使用缓存中的前一层。这么做避免了再次执行 Dockerfile 指令生成的重复层。因为输入是相同的，所以输出也是相同的，所以 Docker 可以使用缓存中已经存在的内容。

Docker 通过生成的 hash 值去判断缓存中是否包含匹配层，类似数字指纹。这个 hash 值是通过 Dockerfile 指令以及它拷贝的文件内容来创建，如果在现有镜像层中找不到 hash 匹配项，那么 Docker 将执行该指令，并且这会中断缓存。一旦缓存被中断，docker 将会执行后续所有的指令，即使他们没有任何变化。 

即使在这个小示例镜像中，这也会产生影响。app.js文件自上次构建以来已更改，因此需要运行步骤6中的Copy指令。步骤7中的CMD指令与上次构建相同，但由于缓存在步骤6中被破坏，该指令也会运行。

您编写的任何 Dockerfile 都应该进行优化，以便根据变化的频率对指令进行排序, 相比较 DOckerfile 的开头，更有可能修改 Dockerfilel 结尾的内容。我们的目标是对于大多数构建，只需要执行最后一条指令，其他则使用缓存的一切。这在启动时基于共享的镜像节省了时间、磁盘空间和网络带宽。

web-ping Dockerfile 中只有七条指令，但它仍然可以优化。CMD 指令不需要在 Dockerfile的 末尾；它可以在 FROM 指令之后的任何位置，仍然具有相同的结果。因为它不太可能改变，这样你就可以把它移得更靠近顶部。一条 ENV 指令可用于设置多个环境变量，因此三个单独的ENV指令可以组合成一个固定的。优化的Dockerfile如清单3.2所示。

> 展示 3.2 优化的 web-ping Dockerfile

```
FROM diamol/node
CMD ["node", "/web-ping/app.js"]
ENV TARGET="blog.sixeyed.com" \
METHOD="HEAD" \
INTERVAL="3000"
WORKDIR /web-ping
COPY app.js .
```

<b>现在就试试</b>  The optimized Dockerfile is in the source code for this chapter
too. Switch to the web-ping-optimized folder and build the image from the
new Dockerfile:
cd ../web-ping-optimized
docker image build -t web-ping:v3 .
优化的 Dockerfile 在本章的源代码中可以被找到，切换到 web-ping-optimized 文件夹，并从新Dockerfile 构建： 

```
cd ../web-ping-optimized
docker image build -t web-ping:v3
```

您不会注意到与以前的版本有太大的差异。现在有五个步骤而不是七步，但最终结果是相同的，您可以用这个镜像运行容器，它的行为与其他版本一样。但现在如果你改变app.js 中的应用代码并重建，除了最后一步所有步骤都会来自缓存，这正是你想要的，因为这就是你所改变的。

这就是本章中构建镜像的全部内容。您已经看到 Dockerfile 语法以及你需要知道的关键指令，你已经学会了如何构建镜像以及使用 Docker CLI 运行镜像。

本章说到了两个更重要的内容，在您构建的每个镜像中都能为您提供良好的服务：优化 Dockerfile 确保您的镜像是可移植的，以便可以同时部署到不同的环境。这真的意味着你应该注意你的Dockerfile指令结构，并确保应用程序可以从容器读取配置值。这意味着您可以快速构建镜像，您生产所使用的镜像与测试中通过质量认证的镜像完全相同。

## 3.6 实验室

Okay, it’s lab time. The goal here is to answer this question: how do you produce a
Docker image without a Dockerfile? The Dockerfile is there to automate the deploy-
ment of your app, but you can’t always automate everything. Sometimes you need to run
the application and finish off some steps manually, and those steps can’t be scripted.

This lab is a much simpler version of that. You’re going to start with an image on
Docker Hub: diamol/ch03-lab. That image has a file at the path /diamol/ch03.txt.
You need to update that text file and add your name at the end. Then produce your
own image with your changed file. You’re not allowed to use a Dockerfile.

There’s a sample solution on the book’s GitHub repository if you need it. You’ll
find it here: https://github.com/sixeyed/diamol/tree/master/ch03/lab.
Here are some hints to get you going:
- Remember that the -it flags let you run to a container interactively.
- The filesystem for a container still exists when it is exited.
- There are lots of commands you haven’t used yet. docker container --help
  will show you two that could help you solve the lab.