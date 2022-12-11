# 第四章 将应用程序源码打包到镜像中

构建 Docker 镜像非常简单，在第三章你已经学会了仅仅需要在 Dockerfile 中定义一些信息你就可以打包应用程序并在容器中运行。关于打包你自己的应用程序还有另外一件事情你需要知道：你同样可以在 Dockerfiles 中运行命令。

构建过程中执行的命令以及基于这些命令产生的任何文件系统的变更都被保存在了镜像层。这使得 Dockerfile 成为最具扩展性的打包格式；你可以解压 zip 文件，运行 windows 安装程序以及任何其他诸如此类的事情。在本章，你将使用此扩展性来基于你的源代码打包应用程序。

## 4.1 有了 Dockerfile 谁还需要构建服务器

在你的笔记本构建软件是你为本地开发环境做的事情，但是当你在一个团队中工作，会有一个更严格的交付过程。将会有一个像 GitHub 的源代码控制系统，要求大家推送他们的代码变更，通常当构建的软件包含一些变更的推送，将有一个单独的服务器(或在线服务)来执行构建。

这个过程的存在是为了及早发现问题。如果开发人员忘记添加文件。当他们推送代码时，构建服务器上的构建将失败，团队将收到警报。它使项目保持健康，但成本是必须维护构建服务器。大多数编程语言需要很多工具来构建项目--如图4.1所示的一些例子。

![图4.1](./images/Figure4.1.png)
<center>图4.1 </center>

这里有很大的维护开销。团队的新成员会花费一整体来安装工具。如果开发人员更新了他们的本地工具，而构建服务器正在运行不同的版本，构建可能会失败。你即使使用的是托管服务器来构建服务，也会出现同样的问题。

如果一次性打包构建工具集并将其共享，就会干净得多，你可以用 Docker 做什么。您可以编写 Dockerfile 脚本部署所有的工具，并将其构建到一个镜像中，然后你就可以用它了，在你的应用程序Dockerfiles中编译源代码，并最终输出的是打包的应用程序。

让我们从一个非常简单的例子开始，因为有一些新的东西用来了解这个过程。

> 清单 4.1 显示了具有基本工作流的 Dockerfile

```
FROM diamol/base AS build-stage
RUN echo 'Building...' > /build.txt
FROM diamol/base AS test-stage
COPY --from=build-stage /build.txt /build.txt
RUN echo 'Testing...' >> /build.txt
FROM diamol/base
COPY --from=test-stage /build.txt /build.txt
CMD cat /build.txt
```

这被称作多阶段 Dockerfile，因为针对构建存在几个阶段。每个阶段都以 FROM 指令开始，然后你可以可选的通过 AS 参数给予阶段一个名字。清单 4.1 有三个阶段：build-state、test-stage 以及最后一个未命名的 stage。尽管有多个阶段，但是输出将会是包含最后一个阶段内容的镜像。

每个阶段都独立运行，但您可以从以前的阶段复制文件和目录。我将COPY指令与--from参数一起使用，它告诉 Docker 从Dockerfile的早期阶段复制文件，而不是从
主机。在本例中，我在 build-stage 生成一个文件，并将其复制到 test-stage，然后将文件从 test-stage 复制到最终阶段。

这里有一个新的指令，RUN，我正在用它来写文件。RUN 指令在构建期间在容器内执行命令，该命令的任何输出都保存在镜像层中。您可以在 RUN指令中执行任何动作，但要运行的命令需要存在于FROM指令 使用的镜像中。在本例中，我使用了diamol/base作为基础图像，它包含echo命令，所以我知道 RUN 指令会起作用。

图 4.2 显示了当我们构建这个 Dockerfile时会发生什么： Docker将依次运行阶段。

![图4.2](./images/Figure4.2.png)
<center>图4.2 </center>

重要的是要了解各个阶段是孤立的。您可以在安装了不同工具集的情况下使用不同的基础镜像，并运行您喜欢的任何命令。最后阶段的输出将只包含从早期阶段显式复制的内容。如果命令在任何阶段失败，整个构建都会失败。

<b>现在就试试</b> 打开会话终端进入到本书的源代码目录, 构建多阶段 Dockerfile:

```
cd ch04/exercises/multi-stage
docker image build -t multi-stage .
```

你将会看到构建按照 Dockerfile 的顺序步骤执行，你可以看到图 4.3 类似的输出。

![图4.3](./images/Figure4.3.png)
<center>图4.3 </center>

这是一个简单的例子，但是和复杂应用的模式是一样的，都是通过单个 Dockerfile 构建。图 4.4 是构建Java 应用程序的例子。

![图4.4](./images/Figure4.4.png)
<center>图4.4 </center>

在 build 阶段，你使用了一个基础镜像，该镜像安装了你应用构建时所需的工具。你拷贝你主机上的源代码并允许构建命令。你可以添加一个 test 阶段以运行单元测试，该单元测试使用了一个包含已安装的测试框架的基础镜像，拷贝 build 阶段编译产生的二进制文件并运行测试。而最终的 final 阶段仅仅从包含了应用运行时的基础镜像开始，然后它拷贝 build 阶段产生的经过 test 阶段成功测试的二进制文件。

这种方法使您的应用程序真正可移植。你可以在任何地方的容器中运行应用，但你也可以在任何地方构建应用——docker是唯一的先决条件。你的构建服务器只需要安装Docker;新的团队成员开始工作在几分钟内，构建工具都集中在 Docker 镜像中，所以没有不同步的几率。

所有主流的应用程序框架都已经在 Docker Hub 上有了安装了构建工具的公共镜像，相关应用程序运行时有单独的镜像。您可以直接使用这些镜像，也可以将它们包装在自己的镜像中。你会得到使用由项目团队维护的镜像的所有最新更新的好处。

## 4.2 应用演练：Java 源代码

We’ll move on to a real example now, with a simple Java Spring Boot application that
we’ll build and run using Docker. You don’t need to be a Java developer or have any
Java tools installed on your machine to use this app; everything you need will come
in Docker images. If you don’t work with Java, you should still read through this
section—it describes a pattern that works for other compiled languages like .NET
Core and Erlang.

The source code is in the repository for the book, at the folder path ch04/
exercises/image-of-the-day. The application uses a fairly standard set of tools for
Java: Maven, which is used to define the build process and fetch dependencies, and
OpenJDK, which is a freely distributable Java runtime and developer kit. Maven uses
an XML format to describe the build, and the Maven command line is called mvn.
That should be enough information to make sense of the application Dockerfile in
listing 4.2.

> Listing 4.2 Dockerfile for building a Java app with Maven

```
FROM diamol/maven AS builder
WORKDIR /usr/src/iotd
COPY pom.xml .
RUN mvn -B dependency:go-offline
COPY . .
RUN mvn package
# app
FROM diamol/openjdk
WORKDIR /app
COPY --from=builder /usr/src/iotd/target/iotd-service-0.1.0.jar .
EXPOSE 80
ENTRYPOINT ["java", "-jar", "/app/iotd-service-0.1.0.jar"]
```

Almost all the Dockerfile instructions here are ones you’ve seen before, and the patterns are familiar from examples that you’ve built. It’s a multi-stage Dockerfile, which
you can tell because there’s more than one FROM instruction, and the steps are laid out
to get maximum benefit from Docker’s image layer cache.

The first stage is called builder. Here’s what happens in the builder stage:
- It uses the diamol/maven image as the base. That image has the OpenJDK Java
development kit installed, as well as the Maven build tool.
- The builder stage starts by creating a working directory in the image and then
copying in the pom.xml file, which is the Maven definition of the Java build.
- The first RUN statement executes a Maven command, fetching all the application dependencies. This is an expensive operation, so it has its own step to
make use of Docker layer caching. If there are new dependencies, the XML file
will change and the step will run. If the dependencies haven’t changed, the
layer cache is used.
- Next the rest of the source code is copied in—COPY . . means “copy all files and
directories from the location where the Docker build is running, into the working directory in the image.”
- The last step of the builder is to run mvn package, which compiles and packages
the application. The input is a set of Java source code files, and the output is a
Java application package called a JAR file.

When this stage completes, the compiled application will exist in the builder stage
filesystem. If there are any problems with the Maven build—if the network is offline
and fetching dependencies fails, or if there is a coding error in the source—the RUN
instruction will fail, and the whole build fails.

If the builder stage completes successfully, Docker goes on to execute the final
stage, which produces the application image:
- It starts from diamol/openjdk, which is packaged with the Java 11 runtime, but
none of the Maven build tools.
- This stage creates a working directory and copies in the compiled JAR file from
the builder stage. Maven packages the application and all its Java dependencies
in this single JAR file, so this is all that’s needed from the builder.
- The application is a web server that listens on port 80, so that port is explicitly listed in the EXPOSE instruction, which tells Docker that this port can be
published.
- The ENTRYPOINT instruction is an alternative to the CMD instruction—it tells
Docker what to do when a container is started from the image, in this case running Java with the path to the application JAR.

TRY IT NOW Browse to the Java application source code and build the image:

```
cd ch04/exercises/image-of-the-day
docker image build -t image-of-the-day
```

There’s a lot of output from this build because you’ll see all the logs from Maven,
fetching dependencies, and running through the Java build. Figure 4.5 shows an
abbreviated section of my build.

![图4.5](./images/Figure4.5.png)
<center>图4.5 </center>

So what have you just built? It’s a simple REST API that wraps access to NASA’s Astronomy Picture of the Day service (https://apod.nasa.gov). The Java app fetches the
details of today’s picture from NASA and caches it, so you can make repeated calls to
this application without repeatedly hitting NASA’s service.

The Java API is just one part of the full application you’ll be running in this
chapter—it will actually use multiple containers, and they need to communicate with
each other. Containers access each other across a virtual network, using the virtual IP
address that Docker allocates when it creates the container. You can create and manage virtual Docker networks through the command line.

TRY IT NOW Create a Docker network for containers to communicate with each other:

`docker network create nat`

If you see an error from that command, it’s because your setup already has a Docker
network called nat, and you can ignore the message. Now when you run containers
you can explicitly connect them to that Docker network using the --network flag, and
any containers on that network can reach each other using the container names.

TRY IT NOW Run a container from the image, publishing port 80 to the host
computer, and connecting to the nat network:

`docker container run --name iotd -d -p 800:80 --network nat image-of-the-day`

Now you can browse to http:/ /localhost:800/image and you’ll see some JSON details
about NASA’s image of the day. On the day I ran the container, the image was from a
solar eclipse—figure 4.6 shows the details from my API.

![图4.6](./images/Figure4.6.png)
<center>图4.6 </center>

The actual application in this container isn’t important (but don’t remove it yet—
we’ll be using it later in the chapter). What’s important is that you can build this on
any machine with Docker installed, just by having a copy of the source code with the
Dockerfile. You don’t need any build tools installed, you don’t need a specific version
of Java—you just clone the code repo, and you’re a couple of Docker commands away
from running the app.

One other thing to be really clear on here: the build tools are not part of the final
application image. You can run an interactive container from your new image-of-the-day Docker image, and you’ll find there’s no mvn command in there. Only the
contents of the final stage in the Dockerfile get made into the application image; any-
thing you want from previous stages needs to be explicitly copied in that final stage.

## 4.3 应用演练：Node.js 源代码

We’re going to go through another multi-stage Dockerfile, this time for a Node.js
application. Organizations are increasingly using diverse technology stacks, so it’s
good to have an understanding of how different builds look in Docker. Node.js is a
great option because of its popularity, and also because it’s an example of a different
type of build—this pattern also works with other scripted languages like Python,
PHP, and Ruby. The source code for this app is at the folder path ch04/exercises/access-log.

Java applications are compiled, so the source code gets copied into the build stage,
and that generates a JAR file. The JAR file is the compiled app, and it gets copied
into the final application image, but the source code is not. It’s the same with .NET
Core, where the compiled artifacts are DLLs (Dynamic Link Libraries). Node.js is
different—it uses JavaScript, which is an interpreted language, so there’s no compilation step. Dockerized Node.js apps need the Node.js runtime and the source code in
the application image.

There’s still a need for a multi-stage Dockerfile though: it optimizes dependency
loading. Node.js uses a tool called npm (the Node package manager) to manage depen-
dencies. Listing 4.3 shows the full Dockerfile for this chapter’s Node.js application.

> Listing 4.3 Dockerfile for building a Node.js app with npm

```
FROM diamol/node AS builder
WORKDIR /src
COPY src/package.json .
RUN npm install
# app
FROM diamol/node
EXPOSE 80
CMD ["node", "server.js"]
WORKDIR /app
COPY --from=builder /src/node_modules/ /app/node_modules/
COPY src/ .
```

The goal here is the same as for the Java application—to package and run the app
with only Docker installed, without having to install any other tools. The base image
for both stages is diamol/node, which has the Node.js runtime and npm installed. The
builder stage in the Dockerfile copies in the package.json files, which describe all the
application’s dependencies. Then it runs npm install to download the dependencies.
There’s no compilation, so that’s all it needs to do.

This application is another REST API. In the final application stage, the steps
expose the HTTP port and specify the node command line as the startup command.
The last thing is to create a working directory and copy in the application artifacts.
The downloaded dependencies are copied from the builder stage, and the source
code is copied from the host computer. The src folder contains the JavaScript files,
including server.js, which is the entry point started by the Node.js process.

We have a different technology stack here, with a different pattern for packaging
the application. The base images, tools, and commands for a Node.js app are all dif-
ferent from a Java app, but those differences are captured in the Dockerfile. The pro-
cess for building and running the app is exactly the same.

TRY IT NOW Browse to the Node.js application source code and build the image:

```
cd ch04/exercises/access-log
docker image build -t access-log .
```

You’ll see a whole lot of output from npm (which may show some error and warning
messages too, but you can ignore those). Figure 4.7 shows part of the output from my
build. The packages that are downloaded get saved in the Docker image layer cache,
so if you work on the app and just make code changes, the next build you run will be
super fast.

![图4.7](./images/Figure4.7.png)
<center>图4.7 </center>

The Node.js app you’ve just built is not at all interesting, but you should still run it
to check that it’s packaged correctly. It’s a REST API that other services can call to
write logs. There’s an HTTP POST endpoint for recording a new log, and a GET endpoint that shows how many logs have been recorded.

TRY IT NOW Run a container from the log API image, publishing port 80 to
host and connecting it to the same nat network:

`docker container run --name accesslog -d -p 801:80 --network nat access-log`

Now browse to http:/ /localhost:801/stats and you’ll see how many logs the service has
recorded. Figure 4.8 shows I have zero logs so far—Firefox nicely formats the API
response, but you may see the raw JSON in other browsers.

![图4.8](./images/Figure4.8.png)
<center>图4.8 </center>

The log API is running in Node.js version 10.16, but just like with the Java example,
you don’t need any versions of Node.js or any other tools installed to build and run
this app. The workflow in this Dockerfile downloads dependencies and then copies
the script files into the final image. You can use the exact same approach with Python,
using Pip for dependencies, or Ruby using Gems.

## 4.4 应用演练：Go 源代码

We’ve got one last example of a multi-stage Dockerfile—for a web application written
in Go. Go is a modern, cross-platform language that compiles to native binaries. That
means you can compile your apps to run on any platform (Windows, Linux, Intel, or
Arm), and the compiled output is the complete application. You don’t need a sepa-
rate runtime installed like you do with Java, .NET Core, Node.js, or Python, and that
makes for extremely small Docker images.

There are a few other languages that also compile to native binaries—Rust and Swift
are popular—but Go has the widest platform support, and it’s also a very popular lan-
guage for cloud-native apps (Docker itself is written in Go). Building Go apps in Docker
means using a multi-stage Dockerfile approach similar to the one you used for the Java
app, but there are some important differences. Listing 4.4 shows the full Dockerfile.

```
FROM diamol/golang AS builder

COPY main.go .
RUN go build -o /server

# app
FROM diamol/base

ENV IMAGE_API_URL="http://iotd/image" \
    ACCESS_API_URL="http://accesslog/access-log"
CMD ["/web/server"]

WORKDIR web
COPY index.html .
COPY --from=builder /server .
RUN chmod +x server
```

Go compiles to native binaries, so each stage in the Dockerfile uses a different base
image. The builder stage uses diamol/golang, which has all the Go tools installed. Go
applications don’t usually fetch dependencies, so this stage goes straight to building
the application (which is just one code file, main.go). The final application stage uses
a minimal image, which just has the smallest layer of operating system tools, called
diamol/base.

The Dockerfile captures some configuration settings as environment variables and
specifies the startup command as the compiled binary. The application stage ends by
copying in the HTML file the application serves from the host and the web server
binary from the builder stage. Binaries need to be explicitly marked as executable in
Linux, which is what the final chmod command does (this has no effect on Windows).

TRY IT NOW Browse to the Go application source code and build the image:

```
cd ch04/exercises/image-gallery
docker image build -t image-gallery .
```
This time there won’t be a lot of compiler output, because Go is quiet and only writes
logs when there are failures. You can see my abbreviated output in figure 4.9.

![图4.9](./images/Figure4.9.png)
<center>图4.9 </center>

This Go application does do something useful, but before you run it, it’s worth taking
a look at the size of the images that go in and come out.

TRY IT NOW Compare the Go application image size with the Go toolset image:

`docker image ls -f reference=diamol/golang -f reference=image-gallery`

Many Docker commands let you filter the output. This command lists all images and
filters the output to only include images with a reference of diamol/golang or image-
gallery—the reference is really just the image name. When you run this, you’ll see
how important it is to choose the right base images for your Dockerfile stages:
```
REPOSITORY TAG IMAGE ID CREATED SIZE
image-gallery latest b41869f5d153 20 minutes ago 25.3MB
diamol/golang latest ad57f5c226fc 2 hours ago 774MB
```

On Linux, the image with all the Go tools installed comes in at over 770 MB; the
actual Go application image is only 25 MB. Remember, that’s the virtual image size, so
a lot of those layers can be shared between different images. The important saving
isn’t so much the disk space, but all the software that isn’t in the final image. The
application doesn’t need any of the Go tools at runtime. By using a minimal base
image for the application, we’re saving nearly 750 MB of software, which is a huge
reduction in the surface area for potential attacks.

Now you can run the app. This ties together your work in this chapter, because the
Go application actually uses the APIs from the other applications you’ve built. You
should make sure you have those containers running, with the correct names from
the earlier try-it-now exercises. If you run docker container ls, you should see two
containers from this chapter—the Node.js container called accesslog and the Java
container called iotd. When you run the Go container, it will use the APIs from the
other containers.

TRY IT NOW Run the Go application image, publishing the host port and connecting to the nat network:

`docker container run -d -p 802:80 --network nat image-gallery`

You can browse to http:/ /localhost:802 and you’ll see NASA’s Astronomy Picture of
the Day. Figure 4.10 shows the image when I ran my containers.

![图4.10](./images/Figure4.10.png)
<center>图4.10 </center>

Right now you’re running a distributed application across three containers. The Go
web application calls the Java API to get details of the image to show, and then it calls
the Node.js API to log that the site has been accessed. You didn’t need to install any
tools for any of those languages to build and run all the apps; you just needed the
source code and Docker.

Multi-stage Dockerfiles make your project entirely portable. You might use Jenkins
to build your apps right now, but you could try AppVeyor’s managed CI service or
Azure DevOps without having to write any new pipeline code—they all support
Docker, so your pipeline is just docker image build.


## 4.5 理解 Dockerfile 的多阶段构建

We’ve covered a lot of ground in this chapter, and I’m going to end with some key
points so you’re really clear on how multi-stage Dockerfiles work, and why it’s incredi-
bly useful to build your apps inside containers.

The first point is about standardization. I know when you run the exercises for this
chapter that your builds will succeed and your apps will work because you’re using the
exact same set of tools that I’m using. It doesn’t matter what operating system you
have or what’s installed on your machine—all the builds run in Docker containers,
and the container images have all the correct versions of the tools. In your real projects you’ll find that this hugely simplifies on-boarding for new developers, eliminates
the maintenance burden for build servers, and removes the potential for breakages
where users have different versions of tools.

The second point is performance. Each stage in a multi-stage build has its own
cache. Docker looks for a match in the image layer cache for each instruction; if it
doesn’t find one, the cache is broken and all the rest of the instructions are executed—
but only for that stage. The next stage starts again from the cache. You’ll be spending
time structuring your Dockerfiles carefully, and when you get the optimization done,
you’ll find 90% of your build steps use the cache.

The final point is that multi-stage Dockerfiles let you fine-tune your build so the
final application image is as lean as possible. This is not just for compilers—any tool-
ing you need can be isolated in earlier stages, so the tool itself isn’t present in the final
image. A good example is curl—a popular command-line tool you can use for down-
loading content from the internet. You might need that to download files your app
needs, but you can do that in an early stage in your Dockerfile so curl itself isn’t
installed in your application image. This keeps image size down, which means faster
startup times, but it also means you have less software available in your application
image, which means fewer potential exploits for attackers.

## 4.6 实验室   
Lab time! You’re going to put into practice what you’ve learned about multi-stage
builds and optimizing Dockerfiles. In the source code for the book, you’ll find a
folder at ch04/lab which is your starting point. It’s a simple Go web server application,
which already has a Dockerfile, so you can build and run it in Docker. But the Docker-
file is in dire need of optimizing, and that is your job.

There are specific goals for this lab:
- Start by building an image using the existing Dockerfile, and then optimize the
Dockerfile to produce a new image.
- The current image is 800 MB on Linux and 5.2 GB on Windows. Your opti-
mized image should be around 15 MB on Linux or 260 MB on Windows.
- If you change the HTML content with the current Dockerfile, the build exe-
cutes seven steps.
- Your optimized Dockerfile should only execute a single step when you change
the HTML.

As always, there’s a sample solution on the book’s GitHub repository. But this is one
lab you should really try and find time to do, because optimizing Dockerfiles is a valu-
able skill you’ll use in every project. If you need it, though, my solution is here:
https://github.com/sixeyed/diamol/blob/master/ch04/lab/Dockerfile.optimized.

No hints this time, although I would say this sample app looks very similar to one
you’ve already built in this chapter.