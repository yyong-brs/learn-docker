# 第七章 通过 Docker Compose 运行多容器应用

Most applications don’t run in one single component. Even large old apps are typi-
cally built as frontend and backend components, which are separate logical layers
running in physically distributed components. Docker is ideally suited to running
distributed applications—from n-tier monoliths to modern microservices. Each
component runs in its own lightweight container, and Docker plugs them together
using standard network protocols. You define and manage multi-container apps
like this using Docker Compose. 
 Compose is a file format for describing distributed Docker apps, and it’s a tool
for managing them. In this chapter we’ll revisit some apps from earlier in the book
and see how Docker Compose makes it easier to use them.

## 7.1 剖析 Docker Compose 文件

You’ve worked with lots of Dockerfiles, and you know that the Dockerfile is a script
for packaging an application. But for distributed apps, the Dockerfile is really just
for packaging one part of the application. For an app with a frontend website, a
backend API, and a database, you could have three Dockerfiles—one for each com-
ponent. How would you run that app in containers? 
 You could use the Docker CLI to start each container in turn, specifying all the
options for the app to run correctly. That’s a manual process that is likely to become
a point of failure, because if you get any of the options wrong, the applications might
not work correctly, or the containers might not be able to communicate. Instead you
can describe the application’s structure with a Docker Compose file.
 The Docker Compose file describes the desired state of your app—what it should
look like when everything’s running. It’s a simple file format where you place all
the options you would put in your docker container run commands into the Com-
pose file. Then you use the Docker Compose tool to run the app. It works out what
Docker resources it needs, which could be containers, networks, or volumes—and it
sends requests to the Docker API to create them. 
 Listing 7.1 shows a full Docker Compose file—you’ll find this in the exercises
folder for this chapter in the book’s source code.

> Listing 7.1 A Docker Compose file to run the to-do app from chapter 6

```
version: '3.7'
services:
  todo-web:
    image: diamol/ch06-todo-list
    ports:
      - "8020:80"
    networks:
      - app-net
networks:
  app-net:
    external:
      name: nat
```

This file describes a simple application with one Docker container plugging into one
Docker network. Docker Compose uses YAML, which is a human-readable text format
that’s widely used because it translates easily to JSON (which is the standard language
for APIs). Spaces are important in YAML—indentation is used to identify objects and
the child properties of objects.
 In this example there are three top-level statements:
- version is the version of the Docker Compose format used in this file. The fea-
ture set has evolved over many releases, so the version here identifies which
releases this definition works with.
- services lists all the components that make up the application. Docker Com-
pose uses the idea of services instead of actual containers, because a service
could be run at scale with several containers from the same image.
- networks lists all the Docker networks that the service containers can plug into.

You could run this app with Compose, and it would start a single container to get to
the desired state. Figure 7.1 shows the architecture diagram of the app’s resources.
 There are a couple of things to look at more closely before we actually run this
app. The service called todo-web will run a single container from the diamol/ch06-
todo-list image. It will publish port 8020 on the host to port 80 on the container,
and it will connect the container to a Docker network referred to as app-net inside
the Compose file. The end result will be the same as running docker container run -
p 8020:80 --name todo-web --network nat diamol/ch06-todo-list.

Under the service name are the properties, which are a fairly close map to the options
in the docker container run command: image is the image to run, ports are the
ports to publish, and networks are the networks to connect to. The service name
becomes the container name and the DNS name of the container, which other con-
tainers can use to connect on the Docker network. The network name in the service is
app-net, but under the networks section that network is specified as mapping to an
external network called nat. The external option means Compose expects the nat
network to already exist, and it won’t try to create it.
 You manage apps with Docker Compose using the docker-compose command line,
which is separate from the Docker CLI. The docker-compose command uses different
terminology, so you start an app with the up command, which tells Docker Compose
to inspect the Compose file and create anything that’s needed to bring the app up to
the desired state.

TRY IT NOW Open a terminal and create the Docker network. Then browse to
the folder with the Compose file from listing 7.1, and then run the app using
the docker-compose command line:

```
docker network create nat
cd ./ch07/exercises/todo-list
docker-compose up
```

You don’t always need to create a Docker network for Compose apps, and you may
already have that nat network from running the exercises in chapter 4, in which case
you’ll get an error that you can ignore. If you use Linux containers, Compose can
manage networks for you, but if you use Windows containers, you’ll need to use the
default network called nat that Docker creates when you install it on Windows. I’m
using the nat network, so the same Compose file will work for you whether you’re run-
ning Linux or Windows containers.
 The Compose command line expects to find a file called docker-compose.yml in
the current directory, so in this case it loads the to-do list application definition. You
won’t have any containers matching the desired state for the todo-web service, so
Compose will start one container. When Compose runs containers, it collects all the
application logs and shows them grouped by containers, which is very useful for devel-
opment and testing.
 My output from running the previous command is in figure 7.2—when you run it
yourself you’ll also see the images being pulled from Docker Hub, but I’d already
pulled them before running the command.

Now you can browse to http://localhost:8020 and see the to-do list application. It
works in exactly the same way as in chapter 6, but Docker Compose gives you a much
more robust way to start the app. The Docker Compose file will live in source control
alongside the code for the app and the Dockerfiles, and it becomes the single place to
describe all the runtime properties of the app. You don’t need to document the image
name or the published port in a README file, because it’s all in the Compose file.

The Docker Compose format records all the properties you need to configure
your app, and it can also record other top-level Docker resources like volumes and
secrets. This app just has a single service, and even in this case it’s good to have a Com-
pose file that you can use to run the app and to document its setup. But Compose
really makes sense when you’re running multi-container apps.

# 7.2 通过 Compose 运行多容器应用

## 7.3 [Docker 如何将容器连接起来

## 7.4 Docker Compose 中的应用配置

## 7.5 理解 Docker Compose 可以解决的问题

## 7.6 实验室 