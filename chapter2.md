# 第二章 了解 Docker 并运行 Hello World

是时候动手使用 Docker了。在这一章,通过在容器中运行应用程序，你会了解到 Docker的核心特性。我还会讲到
一些背景知识，将帮助您准确理解容器是什么，以及为什么容器是运行应用程序的一种轻量级方式。你将会跟上
尝试一些练习，运行简单的命令来感受这种新的方式来处理应用程序。

## 2.1 运行 Hello World 容器

让我们以处理任何新的计算机概念的相同方式开始使用Docker: 即运行Hello World。在第一章你已经安装了 Docker 并运行了它，所以打开你最喜欢的终端——可以是Mac上的终端，也可以是Bash shell Linux，我推荐Windows中的 PowerShell。

You’re going to send a command to Docker, telling it to run a container that
prints out some simple “Hello, World” text.

<b>TRY IT NOW</b> Enter this command, which will run the Hello World container:

`docker container run diamol/ch02-hello-diamol`

When we’re done with this chapter, you’ll understand exactly what’s happening
here. For now, just take a look at the output. It will be something like figure 2.1.

![图2.1](./images/Figure2.1.png)
<center>图2.1 </center>

 There’s a lot in that output. I’ll trim future code listings to keep them short, but
this is the very first one, and I wanted to show it in full so we can dissect it. 

First of all, what’s actually happened? The docker container run command tells
Docker to run an application in a container. This application has already been
packaged to run in Docker and has been published on a public site that anyone can
access. The container package (which Docker calls an “image”) is named diamol/
ch02-hello-diamol (I use the acronym diamol throughout this book—it stands for
Docker In A Month Of Lunches). The command you’ve just entered tells Docker to run a
container from that image.

 Docker needs to have a copy of the image locally before it can run a container
using the image. The very first time you run this command, you won’t have a copy of
the image, and you can see that in the first output line: unable to find image
locally. Then Docker downloads the image (which Docker calls “pulling”), and you
can see that the image has been downloaded.

Now Docker starts a container using that image. The image contains all the con-
tent for the application, along with instructions telling Docker how to start the appli-
cation. The application in this image is just a simple script, and you see the output
which starts Hello from Chapter 2! It writes out some details about the computer it’s
running on:
- The machine name, in this example e5943557213b
- The operating system, in this example Linux 4.9.125-linuxkit x86_64
- The network address, in this example 172.17.0.2

I said your output will be “something like this”—it won’t be exactly the same, because
some of the information the container fetches depends on your computer. I ran this
on a machine with a Linux operating system and a 64-bit Intel processor. If you run it
using Windows containers, the I'm running on line will show this instead:

```
--------------------- 
I'm running on:
Microsoft Windows [Version 10.0.17763.557]
---------------------
```

If you’re running on a Raspberry Pi, the output will show that it’s using a different
processor (armv7l is the codename for ARM’s 32-bit processing chip, and x86_64 is
the code for Intel’s 64-bit chip):

```
---------------------
I'm running on:
Linux 4.19.42-v7+ armv7l
---------------------
```

This is a very simple example application, but it shows the core Docker workflow.
Someone packages their application to run in a container (I did it for this app, but
you will do it yourself in the next chapter), and then publishes it so it’s available to
other users. Then anyone with access can run the app in a container. Docker calls this
build, share, run. 

It’s a hugely powerful concept, because the workflow is the same no matter how
complicated the application is. In this case it was a simple script, but it could be a Java
application with several components, configuration files, and libraries. The workflow
would be exactly the same. And Docker images can be packaged to run on any com-
puter that supports Docker, which makes the app completely portable—portability is
one of Docker’s key benefits.

What happens if you run another container using the same command?

<b>TRY IT NOW</b> Repeat the exact same Docker command:

`docker container run diamol/ch02-hello-diamol`

You’ll see similar output to the first run, but there will be differences. Docker already
has a copy of the image locally so it doesn’t need to download the image first; it gets
straight to running the container. The container output shows the same operating sys-
tem details, because you’re using the same computer, but the computer name and the
IP address of the container will be different:

```
Hello from Chapter 2!
---------------------
My name is:
858a26ee2741 
---------------------
Im running on:
Linux 4.9.125-linuxkit x86_64
---------------------
My address is:
inet addr:172.17.0.5 Bcast:172.17.255.255 Mask:255.255.0.0
---------------------
```

Now my app is running on a machine with the name 858a26ee2741 and the IP address
172.17.0.5. The machine name will change every time, and the IP address will often
change, but every container is running on the same computer, so where do these dif-
ferent machine names and network addresses come from? We’ll dig into a little theory
next to explain that, and then it’s back to the exercises.

## 2.2 什么是容器？

A Docker container is the same idea as a physical container—think of it like a box with
an application in it. Inside the box, the application seems to have a computer all to
itself: it has its own machine name and IP address, and it also has its own disk drive
(Windows containers have their own Windows Registry too). Figure 2.2 shows how the
app is boxed by the container.

![图2.2](./images/Figure2.2.png)
<center>图2.2 </center>

Those things are all virtual resources—the hostname, IP address, and filesystem are
created by Docker. They’re logical objects that are managed by Docker, and they’re all
joined together to create an environment where an application can run. That’s the
“box” of the container.

The application inside the box can’t see anything outside the box, but the box is
running on a computer, and that computer can also be running lots of other boxes.
The applications in those boxes have their own separate environments (managed by
Docker), but they all share the CPU and memory of the computer, and they all share
the computer’s operating system. You can see in figure 2.3 how containers on the
same computer are isolated.

![图2.3](./images/Figure2.3.png)
<center>图2.3 </center>

Why is this so important? It fixes two conflicting problems in computing: isolation and
density. Density means running as many applications on your computers as possible,
to utilize all the processor and memory that you have. But apps may not work nicely
with other apps—they might use different versions of Java or .NET, they may use
incompatible versions of tools or libraries, or one might have a heavy workload and
starve the others of processing power. Applications really need to be isolated from
each other, and that stops you running lots of them on a single computer, so you don’t
get density.

The original attempt to fix that problem was to use virtual machines (VMs). Virtual
machines are similar in concept to containers, in that they give you a box to run your
application in, but the box for a VM needs to contain its own operating system—it
doesn’t share the OS of the computer where the VM is running. Compare figure 2.3,
which shows multiple containers, with figure 2.4, which shows multiple VMs on one
computer.

![图2.4](./images/Figure2.4.png)
<center>图2.4 </center>

 That may look like a small difference in the diagrams, but it has huge implica-
tions. Every VM needs its own operating system, and that OS can use gigabytes of
memory and lots of CPU time—soaking up compute power that should be available
for your applications. There are other concerns too, like licensing costs for the OS
and the maintenance burden of installing OS updates. VMs provide isolation at the
cost of density.

Containers give you both. Each container shares the operating system of the com-
puter running the container, and that makes them extremely lightweight. Containers
start quickly and run lean, so you can run many more containers than VMs on the same
hardware—typically five to ten times as many. You get density, but each app is in its own
container, so you get isolation too. That’s another key feature of Docker: efficiency. 

Now you know how Docker does its magic. In the next exercise we’ll work more
closely with containers.

## 2.3 像远程连接计算机一样连接容器

The first container we ran just did one thing—the application printed out some text
and then it ended. There are plenty of situations where one thing is all you want to do.
Maybe you have a whole set of scripts that automate some process. Those scripts need
a specific set of tools to run, so you can’t just share the scripts with a colleague; you
also need to share a document that describes setting up all the tools, and your col-
league needs to spend hours installing them. Instead, you could package the tools and
the scripts in a Docker image, share the image, and then your colleague can run your
scripts in a container with no extra setup work.
 
 You can work with containers in other ways too. Next you’ll see how you can run a
container and connect to a terminal inside the container, just as if you were connecting
to a remote machine. You use the same docker container run command, but you pass
some additional flags to run an interactive container with a connected terminal session.

<b>TRY IT NOW</b> Run the following command in your terminal session:

`docker container run --interactive --tty diamol/base`

The --interactive flag tells Docker you want to set up a connection to the container,
and the --tty flag means you want to connect to a terminal session inside the con-
tainer. The output will show Docker pulling the image, and then you’ll be left with a
command prompt. That command prompt is for a terminal session inside the con-
tainer, as you can see in figure 2.5.

![图2.5](./images/Figure2.5.png)
<center>图2.5 </center>

The exact same Docker command works in the same way on Windows, but you’ll drop
into a Windows command-line session instead:

```
Microsoft Windows [Version 10.0.17763.557]
(c) 2018 Microsoft Corporation. All rights reserved.
C:\>
```

Either way, you’re now inside the container and you can run any commands that you
can normally run in the command line for the operating system. 

<b>TRY IT NOW</b> Run the commands hostname and date and you’ll see details of
the container’s environment:

```
/ # hostname 
f1695de1f2ec
/ # date 
Thu Jun 20 12:18:26 UTC 2019
```

You’ll need some familiarity with your command line if you want to explore further,
but what you have here is a local terminal session connected to a remote machine—
the machine just happens to be a container that is running on your computer. For
instance, if you use Secure Shell (SSH) to connect to a remote Linux machine, or
Remote Desktop Protocol (RDP) to connect to a remote Windows Server Core
machine, you’ll get exactly the same experience as you have here with Docker.
 
 Remember that the container is sharing your computer’s operating system, which
is why you see a Linux shell if you’re running Linux and a Windows command line if
you’re using Windows. Some commands are the same for both (try ping google.com),
but others have different syntax (you use ls to list directory contents in Linux, and
dir in Windows).
 
 Docker itself has the same behavior regardless of which operating system or pro-
cessor you’re using. It’s the application inside the container that sees it’s running on
an Intel-based Windows machine or an Arm-based Linux one. You manage containers
with Docker in the same way, whatever is running inside them.

<b>TRY IT NOW</b> Open up a new terminal session, and you can get details of all
the running containers with this command:

`docker container ls`

The output shows you information about each container, including the image it’s
using, the container ID, and the command Docker ran inside the container when it
started—this is some abbreviated output:

```
CONTAINER ID IMAGE COMMAND CREATED STATUS 
f1695de1f2ec diamol/base "/bin/sh" 16 minutes ago Up 16 minutes 
```

If you have a keen eye, you’ll notice that the container ID is the same as the hostname
inside the container. Docker assigns a random ID to each container it creates, and
part of that ID is used for the hostname. There are lots of docker container com-
mands that you can use to interact with a specific container, which you can identify
using the first few characters of the container ID you want.

<b>TRY IT NOW</b> docker container top lists the processes running in the con-
tainer. I’m using f1 as a short form of the container ID f1695de1f2ec:

```
> docker container top f1
PID USER TIME COMMAND 
69622 root 0:00 /bin/sh
```

If you have multiple processes running in the container, Docker will show them all.
That will be the case for Windows containers, which always have several background
processes running in addition to the container application.
TRY IT NOW docker container logs displays any log entries the container
has collected:

```
> docker container logs f1
/ # hostname
f1695de1f2ec
```

Docker collects log entries using the output from the application in the container. In
the case of this terminal session, I see the commands I ran and their results, but for a
real application you would see your code’s log entries. For example, a web application
may write a log entry for every HTTP request processed, and these will show in the
container logs.

<b>TRY IT NOW</b> docker container inspect shows you all the details of a
container:

```
> docker container inspect f1
[ 
 {
 "Id": 
"f1695de1f2ecd493d17849a709ffb78f5647a0bcd9d10f0d97ada0fcb7b05e98",
 "Created": "2019-06-20T12:13:52.8360567Z"
```

The full output shows lots of low-level information, including the paths of the con-
tainer’s virtual filesystem, the command running inside the container, and the virtual
Docker network the container is connected to—this can all be useful if you’re track-
ing down a problem with your application. It comes as a large chunk of JSON, which is
great for automating with scripts, but not so good for a code listing in a book, so I’ve
just shown the first few lines.
 
 These are the commands you’ll use all the time when you’re working with contain-
ers, when you need to troubleshoot application problems, when you want to check if
processes are using lots of CPU, or if you want to see the networking Docker has set up
for the container.
 
 There’s another point to these exercises, which is to help you realize that as far as
Docker is concerned, containers all look the same. Docker adds a consistent manage-
ment layer on top of every application. You could have a 10-year-old Java app running
in a Linux container, a 15-year-old .NET app running in a Windows container, and a
brand-new Go application running on a Raspberry Pi. You’ll use the exact same com-
mands to manage them—run to start the app, logs to read out the logs, top to see the
processes, and inspect to get the details.
 
 You’ve now seen a bit more of what you can do with Docker; we’ll finish with some
exercises for a more useful application. You can close the second terminal window you
opened (where you ran docker container logs), go back to the first terminal, which
is still connected to the container, and run exit to close the terminal session.

## 2.4 在容器中运行 Web 站点

So far we’ve run a few containers. The first couple ran a task that printed some text
and then exited. The next used interactive flags and connected us to a terminal session
in the container, which stayed running until we exited the session. docker container
ls will show that you have no containers, because the command only shows running
containers.

<b>TRY IT NOW</b> Run docker container ls --all, which shows all containers in
any status:

```
> docker container ls --all
CONTAINER ID IMAGE COMMAND 
CREATED STATUS 
f1695de1f2ec diamol/base "/bin/sh" 
About an hour ago Exited (0)
858a26ee2741 diamol/ch02-hello-diamol "/bin/sh -c ./cmd.sh" 3 hours 
ago Exited (0)
2cff9e95ce83 diamol/ch02-hello-diamol "/bin/sh -c ./cmd.sh" 4 hours 
ago Exited (0)
```

The containers have the status Exited. There are a couple of key things to understand
here. 

 First, containers are running only while the application inside the container is run-
ning. As soon as the application process ends, the container goes into the exited state.
Exited containers don’t use any CPU time or memory. The “Hello World” container
exited automatically as soon as the script completed. The interactive container we
were connected to exited as soon as we exited the terminal application.
 
 Second, containers don’t disappear when they exit. Containers in the exited state
still exist, which means you can start them again, check the logs, and copy files to and
from the container’s filesystem. You only see running containers with docker container
ls, but Docker doesn’t remove exited containers unless you explicitly tell it to do so.
Exited containers still take up space on disk because their filesystem is kept on the
computer’s disk.
 
 So what about starting containers that stay in the background and just keep run-
ning? That’s actually the main use case for Docker: running server applications like
websites, batch processes, and databases.

<b>TRY IT NOW</b> Here’s a simple example, running a website in a container:

```
docker container run --detach --publish 8088:80 diamol/ch02-hello-
diamol-web
```

This time the only output you’ll see is a long container ID, and you get returned to
your command line. The container is still running in the background.

<b>TRY IT NOW</b> Run docker container ls and you’ll see that the new container
has the status Up:

```
> docker container ls
CONTAINER ID IMAGE 
COMMAND CREATED STATUS 
PORTS NAMES
e53085ff0cc4 diamol/ch02-hello-diamol-web 
"bin\\httpd.exe -DFOR…" 52 seconds ago Up 50 seconds 
443/tcp, 0.0.0.0:8088->80/tcp reverent_dubinsky
```

The image you’ve just used is diamol/ch02-hello-diamol-web. That image includes
the Apache web server and a simple HTML page. When you run this container, you
have a full web server running, hosting a custom website. Containers that sit in the
background and listen for network traffic (HTTP requests in this case) need a couple
of extra flags in the container run command:

- --detach—Starts the container in the background and shows the container ID
- --publish—Publishes a port from the container to the computer

Running a detached container just puts the container in the background so it starts
up and stays hidden, like a Linux daemon or a Windows service. Publishing ports
needs a little more explanation. When you install Docker, it injects itself into your
computer’s networking layer. Traffic coming into your computer can be intercepted
by Docker, and then Docker can send that traffic into a container. 
 
 Containers aren’t exposed to the outside world by default. Each has its own IP
address, but that’s an IP address that Docker creates for a network that Docker
manages—the container is not attached to the physical network of the computer.
Publishing a container port means Docker listens for network traffic on the computer
port, and then sends it into the container. In the preceding example, traffic sent to
the computer on port 8088 will get sent into the container on port 80—you can see
the traffic flow in figure 2.6.

![图2.6](./images/Figure2.6.png)
<center>图2.6 </center>

In this example my computer is the machine running Docker, and it has the IP
address 192.168.2.150. That’s the IP address for my physical network, and it was
assigned by the router when my computer connected. Docker is running a single con-
tainer on that computer, and the container has the IP address 172.0.5.1. That
address is assigned by Docker for a virtual network managed by Docker. No other
computers in my network can connect to the container’s IP address, because it only
exists in Docker, but they can send traffic into the container, because the port has
been published. 

<b>TRY IT NOW</b> Browse to http:/ /localhost:8088 on a browser. That’s an HTTP
request to the local computer, but the response (see figure 2.7) comes from
the container. (One thing you definitely won’t learn from this book is effec-
tive website design.)

![图2.7](./images/Figure2.7.png)
<center>图2.7 </center>

This is a very simple website, but even so, this app still benefits from the portability
and efficiency that Docker brings. The web content is packaged with the web server, so
the Docker image has everything it needs. A web developer can run a single container
on their laptop, and the whole application—from the HTML to the web server stack—
will be exactly the same as if an operator ran the app on 100 containers across a server
cluster in production.
 
 The application in this container keeps running indefinitely, so the container will
keep running too. You can use the docker container commands we’ve already used
to manage it.

<b>TRY IT NOW</b> docker container stats is another useful one: it shows a live
view of how much CPU, memory, network, and disk the container is using.
The output is slightly different for Linux and Windows containers:

```
> docker container stats e53 
CONTAINER ID NAME CPU % PRIV WORKING SET NET I/O 
BLOCK I/O
e53085ff0cc4 reverent_dubinsky 0.36% 16.88MiB 250kB / 53.2kB
19.4MB / 6.21MB
```

When you’re done working with a container, you can remove it with `docker container
rm ` and the container ID, using the `--force` flag to force removal if the container is
still running. 

We’ll end this exercise with one last command that you’ll get used to running
regularly.

<b>TRY IT NOW</b> Run this command to remove all your containers:

`docker container rm --force $(docker container ls --all --quiet)`

The $() syntax sends the output from one command into another command—it
works just as well on Linux and Mac terminals, and on Windows PowerShell. Combin-
ing these commands gets a list of all the container IDs on your computer, and removes
them all. This is a good way to tidy up your containers, but use it with caution, because
it won’t ask for confirmation.

## 2.5 理解 Docker 如何运行容器

We’ve done a lot of try-it-now exercises in this chapter, and you should be happy now
with the basics of working with containers. 
 
 In the first try-it-now for this chapter, I talked about the build, share, run workflow that
is at the core of Docker. That workflow makes it very easy to distribute software—I’ve built
all the sample container images and shared them, knowing you can run them in Docker
and they will work the same for you as they do for me. A huge number of projects now use
Docker as the preferred way to release software. You can try a new piece of software—say,
Elasticsearch, or the latest version of SQL Server, or the Ghost blogging engine—with
the same type of docker container run commands you’ve been using here.
 
 We’re going to end with a little more background, so you have a solid understand-
ing of what’s actually happening when you run applications with Docker. Installing
Docker and running containers is deceptively simple—there are actually a few differ-
ent components involved, which you can see in figure 2.8.

![图2.8](./images/Figure2.8.png)
<center>图2.8 </center>

- The Docker Engine is the management component of Docker. It looks after the
local image cache, downloading images when you need them, and reusing
them if they’re already downloaded. It also works with the operating system to
create containers, virtual networks, and all the other Docker resources. The
Engine is a background process that is always running (like a Linux daemon or
a Windows service).
- The Docker Engine makes all the features available through the Docker API,
which is just a standard HTTP-based REST API. You can configure the Engine
to make the API accessible only from the local computer (which is the default),
or make it available to other computers on your network.
- The Docker command-line interface (CLI) is a client of the Docker API. When you
run Docker commands, the CLI actually sends them to the Docker API, and the
Docker Engine does the work.

It’s good to understand the architecture of Docker. The only way to interact with the
Docker Engine is through the API, and there are different options for giving access to
the API and securing it. The CLI works by sending requests to the API. 
 
 So far we’ve used the CLI to manage containers on the same computer where Docker
is running, but you can point your CLI to the API on a remote computer running Docker
and control containers on that machine—that’s what you’ll do to manage containers in
different environments, like your build servers, test, and production. The Docker API is
the same on every operating system, so you can use the CLI on your Windows laptop to
manage containers on your Raspberry Pi, or on a Linux server in the cloud.
 
 The Docker API has a published specification, and the Docker CLI is not the only
client. There are several graphical user interfaces that connect to the Docker API and
give you a visual way to interact with your containers. The API exposes all the details
about containers, images, and the other resources Docker manages so it can power
rich dashboards like the one in figure 2.9.

![图2.9](./images/Figure2.9.png)
<center>图2.9 </center>

 This is Universal Control Plane (UCP), a commercial product from the company
behind Docker (https://docs.docker.com/ee/ucp/). Portainer is another option,
which is an open source project. Both UCP and Portainer run as containers them-
selves, so they’re easy to deploy and manage.
 
 We won’t be diving any deeper into the Docker architecture than this. The Docker
Engine uses a component called containerd to actually manage containers, and con-
tainerd in turn makes use of operating system features to create the virtual environ-
ment that is the container. 
 
 You don’t need to understand the low-level details of containers, but it is good to
know this: containerd is an open source component overseen by the Cloud Native
Computing Foundation, and the specification for running containers is open and
public; it’s called the Open Container Initiative (OCI). 
 
 Docker is by far the most popular and easy to use container platform, but it’s not
the only one. You can confidently invest in containers without being concerned that
you’re getting locked in to one vendor’s platform.

## 2.6 实验：索引容器文件系统

This is the first lab in the book, so here’s what it’s all about. The lab sets you a task to
achieve by yourself, which will really help you cement what you’ve learned in the chap-
ter. There will be some guidance and a few hints, but mostly this is about you going
further than the prescriptive try-it-now exercises and finding your own way to solve the
problem.
 
 Every lab has a sample solution on the book’s GitHub repository. It’s worth spend-
ing some time trying it out yourself, but if you want to check my solution you can find
it here: https://github.com/sixeyed/diamol/tree/master/ch02/lab.
 
 Here we go: your task is to run the website container from this chapter, but replace
the index.html file so when you browse to the container you see a different home-
page (you can use any content you like). Remember that the container has its own
filesystem, and in this application, the website is serving files that are on the con-
tainer’s filesystem. 
 Here are some hints to get you going:

- You can run docker container to get a list of all the actions you can perform on
a container.

- Add --help to any docker command, and you’ll see more detailed help text.

- In the diamol/ch02-hello-diamol-web Docker image, the content from the
website is served from the directory /usr/local/apache2/htdocs (that’s
C:\usr\local\apache2\htdocs on Windows). 

Good luck :)