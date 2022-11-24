# 第一章 开始之前

Docker 是一个在称为容器的轻量级单元中运行应用程序的平台。
容器已经在软件中无处不在，从云上的 serverless functions 直至企业的战略规划。在相关的技术栈中，Docker 已然成为整个行业运营商和开发人员的核心竞争力。

Docker 是一门很容易学习的技术。你可以作为一个完全的初学者来阅读这本书，你将在第二章中运行容器，在第三章中封装在Docker中运行的应用程序。每一章都着重于实际操作，
在 Linx Mac Windows 等操作系统上都可以进行实操。

基于多年的磨练，你将跟随我一起学习 Docker。除了本章，都可以动手实操。在开始之前，了解容器是如何在现实世界被使用以及它可以解决的问题很有必要。同时将会说明我将如何教大家 docker，你可以弄清楚是否适合你。

现在让我们看看人们使用容器做了什么——我将介绍五个主要的容器
组织在Docker上取得巨大成功的场景。你会看到
使用容器可以解决很多问题，其中一些肯定会符合你的工作场景。读完本章你就会明白了
为什么 Docker 是一项你需要了解的技术，你会看到这本书是如何带你入门的。

## 1.1 为什么容器将会接管世界

我自己的Docker之旅始于2016年，当时我正在做一个基于 Docker 的 Paas 平台项目。我们开始使用 Docker 作为开发构建工具。然后我们获得了信心并开始运行 api用于测试环境的容器。在项目结束时，每个环境都是由 Docker驱动，包括生产，我们对可用性和规模有严格的要求。

当我离开这个项目时，向新团队的交接是单一的一个 GitLab 仓库的README文件。构建，部署和
管理应用程序(在任何环境中)的是Docker。新的开发人员获取源代码通过运行一个命令来构建和运行本地的一切。管理员使用完全相同的工具在生产集群中部署和管理容器。
 
通常这种规模的项目，移交需要两周时间。需要新的开发人员要安装半打特定版本的工具，管理员同样需要安装类似完全不同的工具。Docker集中了工具链，让所有事情都变得更容易，我认为有一天每个项目都必须这样做来使用容器。

近几年的发展，Docker 正在接近无所不在，部分原因是它使交付变得简单很多，部分原因是它非常灵活——你可以把它带进你所有的项目，包括旧的和新的，Windows 或者 Linux。

### 1.1.1 应用上云

将应用程序迁移上云是许多组织的首要任务。这个选择很吸引人，可以让微软、亚马逊或谷歌担心服务器、磁盘、网络或者影响力。在全球数据中心托管您的应用程序，具有几乎无限的扩展潜力。在几分钟内部署到新环境，并只收取您使用的资源的费用。但是，如何将应用程序放到云端？


 过去，将应用程序迁移到云有两种选择：基础设施即服务（IaaS）和平台即服务（PaaS）。两种选择都不好。您的选择基本上是一种妥协，选择PaaS并运行一个项目，将应用程序的所有部分从云迁移到相关的托管服务。这是一个艰难的选择，它将您锁定在一个云上，但它确实可以降低运行成本。另一种选择是IaaS，即为应用程序的每个组件启动虚拟机。您可以跨云移植，但运行成本要高得多。图1.1显示了使用IaaS和PaaS进行云迁移的典型分布式应用程序的形式。

![图1.1](./images/Figure1.1.png)
<center>图1.1 </center>

Docker 提供了没有妥协的第三种选择。您可以迁移你的应用中的任何一部分到容器中，然后您可以使用 Azure Kubernetes Service或Amazon的Elastic container Service在容器中运行整个应用程序，或者在你自己的数据中心的Docker集群上运行整个应用。您将在第7章中学习如何在容器中打包和运行这样的分布式应用程序，在第13章和第14章中，您将了解如何在生产中大规模运行。图1.2显示了选择 Docker，它为您提供了一个可移植的应用程序，您可以在任何云中、数据中心或笔记本电脑上以低成本运行。

![图1.2](./images/Figure1.2.png)
<center>图1.2 </center>

迁移到容器需要一些投资：您需要将现有的安装步骤转换为名为 Dockerfile 的脚本文件，并且使用类似 Docker Compose 或者 Kubernetes 格式的应用声明式清单文件来转换部署文档。你不需要更改任何的代码，最终从你的个人电脑到云端，以同样的方式使用同样的技术栈在所有环境运行应用程序。

### 1.1.2 更新旧版应用

You can run pretty much any app in the cloud in a container, but you won’t get the
full value of Docker or the cloud platform if it uses an older, monolithic design.
Monoliths work just fine in containers, but they limit your agility. You can do an
automated staged rollout of a new feature to production in 30 seconds with containers. But if the feature is part of a monolith built from two million lines of code,you’ve probably had to sit through a two-week regression test cycle before you get to the release.

Moving your app to Docker is a great first step to modernizing the architecture, adopting new patterns without needing a full rewrite of the app. The approach is simple—you start by moving your app to a single container with the Dockerfile and Docker Compose syntax you’ll learn in this book. Now you have a monolith in a container.

Containers run in their own virtual network, so they can communicate with each
other without being exposed to the outside world. That means you can start breaking your application up, moving features into their own containers, so gradually your monolith can evolve into a distributed application with the whole feature set being provided by multiple containers. Figure 1.3 shows how that looks with a sample application architecture.

![图1.4](./images/Figure1.4.png)
<center>图1.4 </center>

This gives you a lot of the benefits of a microservice architecture. Your key features are in small, isolated units that you can manage independently. That means you can test changes quickly, because you’re not changing the monolith, only the containers that run your feature. You can scale features up and down, and you can use different technologies to suit requirements.

Modernizing older application architectures is easy with Docker—you’ll do it
yourself with practical examples in chapters 20 and 21. You can deliver a more agile,scalable, and resilient app, and you get to do it in stages, rather than stopping for an 18-month rewrite.

### 1.1.3 构建新的云原生应用

Docker helps you get your existing apps to the cloud, whether they’re distributed apps or monoliths. If you have monoliths, Docker helps you break them up into modern architectures, whether you’re running in the cloud or in the datacenter. And brand-new projects built on cloud-native principles are greatly accelerated with Docker.The Cloud Native Computing Foundation (CNCF) characterizes these new architectures as using “an open source software stack to deploy applications as microservices,packaging each part into its own container, and dynamically orchestrating those containers to optimize resource utilization.”

Figure 1.4 shows a typical architecture for a new microservices application—this is a demo application from the community, which you can find on GitHub at https://
github.com/microservices-demo.

![图1.3](./images/Figure1.3.png)
<center>图1.3 </center>

It’s a great sample application if you want to see how microservices are actually
implemented. Each component owns its own data and exposes it through an API. The
frontend is a web application that consumes all the API services. The demo application uses various programming languages and different database technologies, but every component has a Dockerfile to package it, and the whole application is defined in a Docker Compose file.

You’ll learn in chapter 4 how you can use Docker to compile code, as part of packaging your app. That means you don’t need any development tools installed to build and run apps like this. Developers can just install Docker, clone the source code, and build and run the whole application with a single command.

Docker also makes it easy to bring third-party software into your application, adding features without writing your own code. Docker Hub is a public service where teams share software that runs in containers. The CNCF publishes a map of opensource projects you can use for everything from monitoring to message queues, and they’re all available for free from Docker Hub.
### 1.1.4 技术创新：Serverless 等

### 1.1.5 利用 DevOps 进行数字化转型
## 1.2 这本书适合你吗?

## 1.3 创建你的实验环境


## 1.4 立即见效