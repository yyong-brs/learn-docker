# 第五章 通过 Docker Hub 及其它仓库共享镜像

在前面几章中，您已经很好地理解了Docker工作流的构建和运行部分——现在是分享的时候了。共享就是将您在本地机器上构建的镜像提供给其他人使用。我真的认为这是Docker等式中最重要的部分。打包软件及其所有依赖项意味着任何人都可以在任何机器上轻松使用它——没有环境之间的差距，因此不需要再浪费时间设置软件或跟踪实际上是部署问题的错误。

## 5.1 使用 registry、repository 以及镜像 Tag

软件分发内置在 Docker 平台中。您已经看到可以从镜像运行容器，如果本地没有该镜像，Docker将下载它。集中存储镜像的服务器称为Docker 仓库（registry）。Docker Hub是最受欢迎的镜像仓库，拥有数十万个镜像，每个月被下载数十亿次。它也是Docker引擎的默认仓库，这意味着它是Docker寻找本地不可用镜像的第一个地方。

Docker images need a name, and that name contains enough information for Docker to find the exact image you’re looking for. So far we’ve used very simple names with one or two parts, like image-gallery or diamol/golang. There are actually four parts to a full image name (which is properly called the image refer-ence). Figure 5.1 shows all those parts in the full reference for diamol/golang:
Docker 镜像需要一个名称，该名称包含足够的信息，以便Docker找到您正在寻找的确切镜像。到目前为止，我们使用的名称都非常简单，只有一到两个部分，比如image-gallery或diamol/golang。一个完整的镜像名称实际上有四个部分(正确地称为镜像引用)。图5.1显示了diamol/golang完整参考中的所有部分:

![图5.1](./images/Figure5.1.png)
<center>图5.1 解剖 Docker 镜像引用</center>

当您开始管理自己的应用程序镜像时，您将使用镜像引用的所有部分。在您的本地机器上，您可以为镜像命名任何您喜欢的名称，但是当您希望在仓库上共享它们时，您将需要添加更多细节，因为镜像引用是仓库上一个特定镜像的唯一标识符。

如果你不为镜像引用的某些部分提供值，Docker会使用一些默认值。默认的仓库是Docker Hub，默认的 tag 是 latest。Docker Hub的域是Docker.io。因此，我的镜像，diamol/golang，是docker.io/diamol/golang:latest的简短版本。你可以使用这些参考中的任何一个。diamol帐户是Docker Hub上的一个组织，golang是该组织中的一个存储库。它是一个公共存储库，因此任何人都可以提取镜像，但您必须是diamol组织的成员才能推送镜像。

大公司通常在自己的云环境或本地网络中有自己的Docker 仓库。通过在引用的第一部分包含域名来针对自己的仓库，因此Docker知道不要使用Docker Hub。如果我在r.sixeyed.com上托管自己的仓库，我的镜像就可以存储在r.sixeyed.com/diamol/golang上。这非常简单，但镜像引用中最重要的部分是 tag。

到目前为止，您还没有使用镜像 Tag，因为开始时不使用它们会更简单，但是当您开始构建自己的应用程序镜像时，您应该始终 tag 它们。tag 用于标识同一应用程序的不同版本。官方Docker OpenJDK镜像有数百个 tag——OpenJDK:13是最新版本，OpenJDK:8u212-jdk是Java 8的特定版本，还有更多针对不同Linux发行版和Windows版本的 tag。如果在创建镜像时没有指定 tag，Docker将使用默认tag latest。这是一个误导性的名称，因为标记为“最新”的镜像实际上可能不是最新的镜像版本。当你推送自己的镜像时，你应该总是用明确的版本来标记它们。

## 5.2 推送你自己的镜像到 Docker Hub

我们将从将您在第4章中构建的一个镜像推到 Docker Hub 开始。为此，你需要一个Docker Hub账户——如果你没有，请浏览https://hub.docker.com并按照链接注册一个账户(它是免费的，而且它不会让你的收件箱里有大量垃圾邮件)。

要将镜像推送到仓库，需要做两件事。首先，您需要使用Docker命令行登录仓库，这样Docker就可以检查您的用户帐户是否被授权推送镜像。然后，您需要为您的镜像提供一个引用，其中包括您有权推送的帐户的名称。

每个读者都有自己的Docker Hub用户名，为了便于练习，让我们先在终端会话的变量中捕获自己的Docker ID。在此之后，您将能够复制和粘贴本章的其余命令。

<b>现在就试试</b> 打开一个终端会话并将Docker Hub ID保存在一个变量中。Docker ID是您的用户名，而不是您的电子邮件地址。这是一个在Windows和Linux上不同的命令，所以你需要选择正确的选项:

```
# 使用 Windows 上的 PowerShell:
$dockerId="<输入你的 Docker id>"
# 使用 linux or mac 上的 bash:
export dockerId="<输入你的 docker id>"
```

我现在正在运行Windows，我的Docker Hub用户名是sixeyed，所以我运行的命令是$dockerId="sixeyed";在Linux上，我会运行dockerId="sixeyed"。在任何系统上，您都可以运行echo $dockerId，您应该会看到显示您的用户名。从现在开始，你可以复制练习中的命令，它们将使用你的Docker ID。

首先登录到 Docker Hub。实际上是Docker引擎推送和提取镜像，但是您使用Docker命令行进行身份验证——当您运行登录命令时，它将询问您的密码，请输入您的Docker Hub密码。

<b>现在就试试</b> 登录 Docker Hub。Hub是默认仓库，所以你不需要指定域名:

`docker login --username $dockerId`

您将看到如图5.2所示的输出——很明显，当您输入密码时，Docker不会显示密码。

![图5.2](./images/Figure5.2.png)
<center>图5.2 登录 Docker Hub</center>

现在您已经登录，您可以将镜像推送到您自己的帐户或您可以访问的任何组织。我不认识你，但如果我想让你帮忙照看这本书的镜像，我可以把你的账号添加到diamol组织，你就可以推送以diamol/开头的镜像。如果您不是任何组织的成员，则只能在自己的帐户中将镜像推送到存储库。

你在第4章中构建了一个名为image-gallery的Docker 镜像。该镜像引用没有帐户名称，因此不能将其推送到任何仓库。您不需要重新构建镜像来给它一个新的引用-一个镜像可以有几个引用。

<b>现在就试试</b> 为现有镜像创建一个新的引用，将其标记为v1:

`docker image tag image-gallery $dockerId/image-gallery:v1`

现在你有了两个引用;其中一个有帐户和版本号，但两个引用都指向相同的镜像。镜像也有唯一的ID，如果一个镜像ID有多个引用，可以在列出它们时看到。

<b>现在就试试</b> 列出镜像 image-gallery 的引用：

`docker image ls --filter reference=image-gallery --filter reference='*/image-gallery'`

在图5.3中，您将看到与我的类似的输出，只不过您的 tag 镜像将显示您的Docker Hub用户名而不是sixeyed。

![图5.3](./images/Figure5.3.png)
<center>图5.3 一个镜像两个引用</center>

现在，您的帐户名中有一个Docker ID的镜像引用，并且您已经登录到Docker Hub，因此您已经准备好共享您的镜像了!docker image push命令相当于pull命令;它将您的本地镜像层上传到仓库。

<b>现在就试试</b> 列出镜像 image-gallery 的引用：

`docker image push $dockerId/image-gallery:v1`

Docker 仓库在镜像层级别上的工作方式与本地Docker引擎相同。你推送一个镜像，但Docker实际上上传了镜像层。在输出中，你会看到一个层id列表和它们的上传进度。在我的(缩写)输出中，你可以看到层被推送:

```
The push refers to repository [docker.io/sixeyed/image-gallery]
c8c60e5dbe37: Pushed
2caab880bb11: Pushed
3fcd399f2c98: Pushed
...
v1: digest: sha256:127d0ed6f7a8d1... size: 2296
```

仓库与镜像层一起工作的事实是您需要花时间优化dockerfile的另一个原因。层只有在没有与该层的散列匹配的情况下才会物理上传到注册表。它就像你的本地Docker引擎缓存，但应用于仓库上的所有镜像。如果你优化到构建时90%的层来自缓存，那么当你推送时，90%的层已经在仓库中了。优化后的dockerfile减少了构建时间、磁盘空间和网络带宽。

您现在可以访问 Docker Hub并检查您的镜像。Docker Hub UI使用与镜像引用相同的存储库名称格式，因此您可以从帐户名计算出镜像的URL。

<b>现在就试试</b> 这个小脚本输出 Docker Hub上的镜像页面URL

`echo "https://hub.docker.com/r/$dockerId/image-gallery/tags"`

当您访问该URL时，您将看到如图5.4所示的内容，其中显示了镜像的Tag和最后一次更新时间。

![图5.4](./images/Figure5.4.png)
<center>图5.4 查看 Docker Hub 中镜像</center>

这就是推送镜像的全部内容。如果镜像还不存在，Docker Hub会为镜像创建一个新的存储库，默认情况下，该存储库具有公共读权限。现在任何人都可以找到、提取和使用您的镜像库应用程序。他们需要自己弄清楚如何使用它，但是你也可以把文档放在Docker Hub上。

Docker Hub是最容易上手的仓库，它为你提供了大量零成本的功能——尽管你可以每月付费订阅额外的功能，比如私有存储库。还有很多可供选择的仓库。

仓库是一个开放的API规范，core registry server 是来自Docker的开源产品。所有的云都有自己的仓库服务，你可以在数据中心中使用Docker Trusted registry等商业产品管理自己的仓库，也可以在容器中运行一个简单的仓库。

## 5.3 运行并使用你自己的 Docker 仓库

在本地网络上运行自己的仓库非常有用。它减少了带宽的使用和传输时间，并允许您拥有环境中的数据。即使您不担心这一点，知道您可以快速启动本地仓库仍然是件好事，如果您的主仓库离线，您可以将其用作备份选项。

Docker在源代码存储库 Docker /distribution中维护GitHub上的核心注册表服务器。它为你提供了推送和拉取镜像的基本功能，它使用了与Docker Hub相同的层缓存系统，但它没有为你提供Hub所提供的web UI。它是一个超轻量级的服务器，我将它打包到一个diamol 镜像中，因此您可以在容器中运行它。

<b>现在就试试</b> 使用我的镜像在容器中允许 Docker 仓库：
```
# 使用一个restart标志运行仓库，这样当你重启Docker时容器就会重新启动:
docker container run -d -p 5000:5000 --restart always diamol/registry
```

您现在在本地机器上有一个仓库服务了。服务器的默认端口是5000，该命令发布该端口。您可以用域localhost:5000 tag 映像并将它们推送到这个仓库，但这并不是真正有用的—您只能在本地机器上使用仓库。相反，最好给你的机器一个别名，这样你就可以为你的仓库使用一个合适的域名。

下一个命令创建别名。它将为您的计算机提供名称 registry.local。它通过写入计算机的hosts文件来实现这一点，该文件是一个简单的文本文件，将网络名称链接到IP地址。

<b>现在就试试</b> Windows、Linux和Mac机器的hosts文件格式相同，但路径不同。Windows上的命令也不同，因此需要选择正确的命令：

```
# 在 Windows 使用 powershell
Add-Content -Value "127.0.0.1 registry.local" -Path /windows/system32/drivers/etc/hosts
# 使用 Linux or Mac 上的 bash
echo $'\n127.0.0.1 registry.local' | sudo tee -a /etc/hosts
```

如果您从该命令得到权限错误，您将需要在Windows上的提升PowerShell会话中以管理员权限登录，或者在Linux或Mac上使用sudo。当您成功运行该命令时，您应该能够运行ping registry.local,然后看到来自计算机的主IP地址127.0.0.1的响应，如图5.5所示。

![图5.5](./images/Figure5.5.png)
<center>图5.5 为你的计算机添加新的网络别名</center>

现在你可以使用域名registry.local:5000 在镜像中引用来使用仓库。将域名添加到镜像中涉及到与Docker Hub中所做的tag 相同的过程。这一次，您只需在新的镜像引用中包含仓库域名。

<b>现在就试试</b> 使用你的仓库域名来为镜像 image-gallery 打tag:

`docker image tag image-gallery registry.local:5000/gallery/ui:v1`

您的本地仓库没有设置任何身份验证或授权。这显然不是产品质量，但它可能适用于小型团队，而且它确实允许您使用自己的镜像命名方案。在第4章中，有三个容器组成了NASA每日图片应用程序——你可以用gallery作为项目名称来标记所有镜像并将它们组合在一起:

- registry.local:5000/gallery/ui:v1—The Go web UI
- registry.local:5000/gallery/api:v1—The Java API
- registry.local:5000/gallery/logs:v1—The Node.js API

在将该镜像推送到本地仓库之前，还需要做一件事。仓库容器使用明文HTTP而不是加密的HTTPS来推送和提取镜像。Docker默认情况下不会与未加密的仓库通信，因为它不安全。在Docker允许您使用仓库域名之前，您需要显式地将仓库域名添加到允许的不安全仓库列表中。

这就把我们带到了配置Docker。Docker引擎使用JSON配置文件进行各种设置，包括Docker在磁盘上存储镜像层的位置，Docker API监听连接的位置，以及允许哪些不安全仓库。这个文件叫做daemon.json。它通常在Windows Server上的C:\ProgramData\docker\config文件夹中，在Linux Server上的/etc/docker文件夹中。你可以直接编辑这个文件，但如果你在Macor Windows上使用Docker Desktop，你需要使用UI，在那里你可以更改主要的配置设置。

<b>现在就试试</b> 右键单击任务栏中的Docker鲸鱼图标，并选择设置(或Mac上的首选项)。然后打开Daemon选项卡并输入insecure registries 信息registry.Local:5000 —您可以在图5.6中看到我的设置。

![图5.6](./images/Figure5.6.png)
<center>图5.6 允许在 Docker Desktop 中使用不安全协议的仓库</center>

Docker引擎需要重新启动来加载任何新的配置设置，当你应用更改时，Docker Desktop会为你做这些。

如果您没有运行Docker Desktop，则需要手动执行此操作。首先打开守护进程。在文本编辑器中使用json文件(如果不存在则创建它)并以json格式添加不安全的仓库详细信息。配置设置看起来像这样，但如果你编辑一个现有的文件，一定要保留原始的设置:
```
{
  "insecure-registries": [
    "registry.local:5000"
  ]
}
```

然后在Windows Server上使用restart - service Docker重启Docker，或者在Linux上使用service Docker restart重启Docker。你可以使用info命令检查Docker引擎允许哪些不安全仓库，以及其他信息。

<b>现在就试试</b> 列出关于Docker引擎的信息，并在insecure registries 列表中检查您的仓库：

`docker info`

在输出的最后，您将看到仓库配置，其中应该包括您的不安全仓库—您可以在图5.7中看到我的仓库。

![图5.7](./images/Figure5.7.png)
<center>图5.7 非安全的仓库允许 Docker 使用</center>

在Docker配置中添加不安全仓库时，你应该小心。您的连接可能会受到损害，攻击者可以在您推送镜像时读取图层。或者更糟的是，当您提取镜像时，它们可能会注入自己的数据。所有的商业仓库都运行在HTTPS上，你也可以配置Docker的开源仓库来使用HTTPS。然而，对于使用本地服务器进行演示，这是可以接受的风险。

您现在可以将标记的镜像推到自己的仓库中。仓库域名是镜像引用的一部分，因此Docker知道使用Docker Hub以外的东西，并且在容器中运行的HTTP 仓库在不安全仓库列表中被清除。

<b>现在就试试</b> 推送你已标记好的镜像：

`docker image push registry.local:5000/gallery/ui:v1`

当你运行第一次推送时，仓库是完全空的，所以你会看到所有的层都被上传了。如果你重复push命令，你会看到所有的层都已经存在，没有上传任何东西。这就是在容器中运行自己的Docker 仓库所需要做的全部工作。您可以使用机器的IP地址或真实域名在网络上共享它。

## 5.4 有效运用镜像 Tag

您可以将任何字符串放入 Docker 镜像 tag 中，正如您已经看到的，您可以为同一镜像设置多个 tag。您将使用它来为您的镜像中的软件设置版本，并让用户对他们想要使用的内容做出明智的选择，并在使用其他人的镜像时做出明智的选择。

许多软件项目使用带有小数点的数字版本控制方案来表示版本之间的变化有多大，您可以使用镜像标记来实现这一点。基本概念是[major (大调)].[minor(小调)].[patch (补丁)]，其中有一些隐含的信息。一个只增加补丁号的版本可能有bug修复，但它应该具有与上一个版本相同的功能;增加小版本的版本号可能会增加功能，但不应该删除任何功能;一个主要版本可能会有完全不同的功能。

如果您对镜像标记使用相同的方法，您可以让用户选择是坚持使用主要版本还是次要版本，或者始终使用最新版本。

<b>现在就试试</b> 为镜像中打包的Go应用程序创建几个新 tag ，以指示主要、次要和补丁发布版本:

```
docker image tag image-gallery registry.local:5000/gallery/ui:latest
docker image tag image-gallery registry.local:5000/gallery/ui:2
docker image tag image-gallery registry.local:5000/gallery/ui:2.1
docker image tag image-gallery registry.local:5000/gallery/ui:2.1.106
```

现在假设一个应用程序每月发布一次，它会增加版本号。图5.8显示了从7月到10月的各个版本中镜像 tag 可能如何演变。

![图5.8](./images/Figure5.8.png)
<center>图5.8 软件发布期间镜像tag的演变</center>

您可以看到其中一些镜像 tag 是一个移动的目标。gallery/ui:2.1是7月份2.1.106发行版的别名，但在8月份，同样的2.1标记是2.1.114发行版的别名。gallery/ui:2在7月也是2.1.106的别名，但到了9月，2标记是2.2.11发行版的别名。最新的 tag 变化最大——7月份gallery/ui是2.1.106的别名，但10月份它是3.0.42的别名。

这是Docker 镜像的典型版本控制方案。这是你自己应该采用的，因为它让你的目标用户选择他们想要的最新程度。他们可以在镜像提取命令或dockerfile中的FROM指令中固定到特定的补丁版本，并确保他们使用的镜像始终是相同的。本例中的2.1.106标记是7月到10月的相同镜像。如果他们想要获得补丁更新，他们可以使用2.1 tag，如果他们想要获得小版本，他们可以使用2 tag。

这些选择都可以;这只是一个平衡风险的例子——使用特定的补丁版本意味着无论何时使用它，应用程序都是一样的，但你不会得到安全补丁。使用主版本意味着您将获得所有最新的修复，但可能会有意想不到的功能更改。

在您自己的dockerfile中为基础镜像使用特定的镜像 tag 尤其重要。使用产品团队的构建工具来构建应用程序，使用它们的运行时镜像来打包应用程序是很好的，但如果你没有在标记中指定版本，你就会在未来给自己带来麻烦。构建镜像的新版本可能会破坏Docker构建。或者更糟的是，运行时的新版本可能会破坏应用程序。

## 5.5 转换官方镜像为黄金镜像

当您查看 Docker Hub 和其他仓库时，还有最后一件事需要了解:您能相信在那里找到的镜像吗?任何人都可以将镜像推送到Docker Hub并使其公开可用。对于黑客来说，这是一种传播恶意软件的好方法;你只需要给你的镜像一个无辜的名字和一个虚假的描述，然后等待人们开始使用它。Docker Hub通过经过验证的发布者和官方镜像解决了这个问题。

经过验证的发布者是像 Microsoft、Oracle和IBM这样在Docker Hub上发布镜像的公司。他们的镜像要经过一个审批流程，包括安全扫描漏洞;他们也可能是经过认证的，这意味着他们有Docker和出版商的支持。如果您想在容器中运行现成的软件，那么来自经过验证的发布者的认证镜像是最好的选择。

官方镜像某些方面有所不同——它们通常是开源项目，由项目团队和Docker共同维护。它们经过安全扫描并定期更新，并且符合Dockerfile最佳实践。官方镜像的所有内容都是开源的，所以你可以在GitHub上看到Dockerfiles。大多数人开始使用官方镜像作为自己的基础镜像，但在某些时候，他们发现自己需要更多的控制。然后他们介绍自己首选的基础镜像，称为黄金镜像——图5.9向您展示了它是如何工作的。

![图5.9](./images/Figure5.9.png)
<center>图5.9 使用一个黄金镜像来封装官方镜像</center>

黄金镜像使用官方镜像作为基础，然后添加所需的任何自定义设置，例如安装安全证书或配置默认环境设置。黄金镜像存在于Docker Hubor上的公司仓库中，在他们自己的仓库中，所有应用程序镜像都基于黄金镜像。这种方法提供了官方镜像的好处(具有项目团队的最佳实践设置)，但需要额外的配置。

<b>现在就试试</b> 在本章的源代码中有两个dockerfile，它们可以被构建为.Net Core应用程序的黄金镜像。访问每个文件夹并构建镜像：
```
cd ch05/exercises/dotnet-sdk
docker image build -t golden/dotnetcore-sdk:3.0 .

cd ../aspnet-runtime
docker image build -t golden/aspnet-core:3.0 .
```

黄金镜像并没有什么特别之处。它们从Dockerfile开始，然后用您自己的引用和命名方案构建一个镜像。如果您查看您构建的dockerfile，您将看到它们使用LABEL指令向镜像添加了一些元数据，并设置了一些公共配置。现在可以在.net Core应用程序的多阶段Dockerfile中使用这些镜像，类似于清单5.1。

> 清单 5.1 一个多阶段 Dockerfile 使用 .NET Core 黄金镜像

```
FROM golden/dotnetcore-sdk:3.0 AS builder
COPY . .
RUN dotnet publish -o /out/app app.csproj
FROM golden/aspnet-core:3.0
COPY --from=builder /out /app
CMD ["dotnet", "/app/app.dll"]
```

应用程序 Dockerfile 具有与任何多阶段构建相同的格式，但现在您拥有基础镜像。官方镜像可能每个月都有一个新的版本，但你可以选择限制你的黄金镜像每季度更新一次。而且黄金镜像提供了另一种可能性——您可以在持续集成(CI)管道中使用工具强制使用它们:dockerfile可以被扫描，如果有人试图在不使用黄金镜像的情况下构建应用程序，那么构建就会失败。这是锁定团队可以使用的源镜像的好方法。

## 5.6 实验室

这个实验室需要一些侦查工作，但最终是值得的。你需要深入挖掘Docker Registry API v2规范(https://docs.docker.com/registry/spec/api/)，因为REST API是你与本地Docker Registry交互的唯一方式——你不能使用Docker CLI搜索或删除镜像(目前)。

本实验的目标是将 gallery/ui 镜像的所有 tag 推到本地仓库，检查它们是否都在那里，然后将它们全部删除并检查它们是否已经消失。我们将不包括 gallery/api或gallery/logs图像，因为这个实验室关注的是带有多个 tag 的镜像，我们为gallery/ui提供了这些 tag。这里有一些提示:
- 你可以使用一个镜像推送命令来推送所有这些 tag。
- 本地注册表API的URL是 http://registry.local:5000/v2。
- 首先列出存储库的镜像 tag。
- 然后你需要拿到镜像清单。
- 可以通过API删除镜像，但需要使用manifest。
- 阅读文档-在HEAD请求中需要使用特定的请求头。

解决方案在这本书的 GitHub 存储库中，这是一个很少的情况。最初的几个步骤对您来说应该很简单，但接下来就会有点尴尬，所以如果您最终访问了这里:https://github.com/yyong-brs/learn-docker/tree/master/diamol/ch05/lab，请不要感到太糟糕。

祝你好运，记得阅读文档。