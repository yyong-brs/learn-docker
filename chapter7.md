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

Back in chapter 4 we built a distributed app that shows an image from NASA’s astron-
omy picture of the day API. There was a Java frontend website, a REST API written in
Go, and a log collector written in Node.js. We ran the app by starting each container
in turn, and we had to plug the containers into the same network and use the correct
container names so the components could find each other. That’s exactly the sort of
brittle approach that Docker Compose fixes for us.
 In listing 7.2 you can see the services section for a Compose file that describes
the image gallery application. I’ve removed the network configuration to focus on the
service properties, but the services plug into the nat network just like in the to-do app
example.

> Listing 7.2 The Compose services for the multi-container image gallery app

```
accesslog:
  image: diamol/ch04-access-log
iotd:
  image: diamol/ch04-image-of-the-day
  ports:
    - "80"
image-gallery:
  image: diamol/ch04-image-gallery
  ports:
    - "8010:80" 
  depends_on:
    - accesslog
    - iotd

```

This is a good example of how to configure different types of services. The accesslog
service doesn’t publish any ports or use any other properties you would capture from
the docker container run command, so the only value recorded is the image name.
The iotd service is the REST API—the Compose file records the image name and also
publishes port 80 on the container to a random port on the host. The image-gallery
service has the image name and a specific published port: 8010 on the host maps to
port 80 in the container. It also has a depends_on section saying this service has a
dependency on the other two services, so Compose should make sure those are run-
ning before it starts this one.
 Figure 7.3 shows the architecture of this app. I’ve generated the diagrams in this
chapter from a tool that reads the Compose file and generates a PNG image of the
Listing 7.2 The Compose services for the multi-container image gallery app
architecture. That’s a great way to keep your documentation up to date—you can gen-
erate the diagram from the Compose file every time there’s a change. The diagram
tool runs in a Docker container of course—you can find it on GitHub at https://
github.com/pmsipilot/docker-compose-viz.
 We’ll use Docker Compose to run the app, but this time we’ll run in detached mode.
Compose will still collect the logs for us, but the containers will be running in the
background so we’ll have our terminal session back, and we can use some more fea-
tures of Compose.

TRY IT NOW Open a terminal session to the root of your DIAMOL source
code, and then navigate to the image gallery folder and run the app:

```
cd ./ch07/exercises/image-of-the-day
docker-compose up --detach
```

Your output will be like mine in figure 7.4. You can see that the accesslog and iotd
services are started before the image-gallery service, because of the dependencies
recorded in the Compose file.
 When the app is running, you can browse to http:/ /localhost:8010. It works just
like it did in chapter 4, but now you have a clear definition in the Docker Compose
file of how the containers need to be configured for them to work together. You can
also manage the application as a whole using the Compose file. The API service is
effectively stateless, so you can scale it up to run on multiple containers. When the
web container requests data from the API, Docker will share those requests across the
running API containers.

TRY IT NOW In the same terminal session, use Docker Compose to increase
the scale of the iotd service, and then refresh the web page a few times and
check the logs of the iotd containers:

```
docker-compose up -d --scale iotd=3
# browse to http://localhost:8010 and refresh
docker-compose logs --tail=1 iotd
```

You’ll see in the output that Compose creates two new containers to run the image
API service, so it now has a scale of three. When you refresh the web page showing the
photograph, the web app requests data from the API, and that request could be han-
dled by any of the API containers. The API writes a log entry when it handles requests,
which you can see in the container logs. Docker Compose can show you all log entries
for all containers, or you can use it to filter the output—the --tail=1 parameter just
fetches the last log entry from each of the iotd service containers. 
 My output is in figure 7.5—you can see that containers 1 and 3 have been used by
the web app, but container 2 hasn’t handled any requests so far.
 Docker Compose is now managing five containers for me. I can control the whole
app using Compose; I can stop all the containers to save compute resources, and start
them all again when I need the app running. But these are normal Docker containers
that I can also work with using the Docker CLI. Compose is a separate command-line
tool for managing containers, but it uses the Docker API in the same way that the
Docker CLI does. You can use Compose to manage your app, but still use the stan-
dard Docker CLI to work with containers that Compose created.

TRY IT NOW In the same terminal session, stop and start the app with Docker
Compose commands, and then list all running containers with the Docker CLI:
```
docker-compose stop
docker-compose start
docker container ls
```

Your output will be like mine in figure 7.6. You’ll see that Compose lists individual
containers when it stops the app, but it only lists the services when it starts the app
again, and the services are started in the correct dependency order. In the container
list you’ll see that Compose has restarted the existing containers, rather than creating
new ones. All my containers show they were created over 30 minutes ago, but they’ve
only been up for a few seconds.

There are many more features to Compose—run docker-compose without any options
to see the full list of commands—but there’s one really important consideration you
need to take in before you go much further. Docker Compose is a client-side tool. It’s
a command line that sends instructions to the Docker API based on the contents of
the Compose file. Docker itself just runs containers; it isn’t aware that many contain-
ers represent a single application. Only Compose knows that, and Compose only
knows the structure of your application by looking at the Docker Compose YAML file,
so you need to have that file available to manage your app.
 It’s possible to get your application out of sync with the Compose file, such as when
the Compose file changes or you update the running app. That can cause unexpected
behavior when you return to manage the app with Compose. We’ve already done this
ourselves—we scaled up the iotd service to three containers, but that configuration
isn’t captured in the Compose file. When you bring the application down and then
recreate it, Compose will return it to the original scale.

TRY IT NOW In the same terminal session—because Compose needs to use
the same YAML file—use Docker Compose to bring the application down and
back up again. Then check the scale by listing running containers:
```
docker-compose down
docker-compose up -d
docker container ls
```
The down command removes the application, so Compose stops and removes
containers—it would also remove networks and volumes if they were recorded in the
Compose file and not flagged as external. Then up starts the application, and because
there are no running containers, Compose creates all the services—but it uses the app
definition in the Compose file, which doesn’t record scale, so the API service starts
with one container instead of the three we previously had running. 
 You can see that in my output in figure 7.7. The goal here was to restart the app,
but we’ve accidentally scaled the API service down as well.

Docker Compose is simple to use and powerful, but you need to be mindful that it’s a
client-side tool, so it’s dependent on good management of the app definition YAML
files. When you deploy an app with Compose, it creates Docker resources, but the
Docker Engine doesn’t know those resources are related—they’re only an application
as long as you have the Compose file to manage them.

## 7.3 [Docker 如何将容器连接起来

All the components in a distributed application run in Docker containers with Com-
pose, but how do they communicate with each other? You know that a container is a
virtualized environment with its own network space. Each container has a virtual IP
address assigned by Docker, and containers plugged into the same Docker network
can reach each other using their IP addresses. But containers get replaced during the
application life cycle, and new containers will have new IP addresses, so Docker also
supports service discovery with DNS.
 DNS is the Domain Name System, which links names to IP addresses. It works on
the public internet and on private networks. When you point your browser to blog.six-
eyed.com, you’re using a domain name, which gets resolved to an IP address for one
of the Docker servers I have hosting my blog. Your machine actually fetches content
using the IP address, but you, as the user, work with the domain name, which is much
friendlier. 
 Docker has its own DNS service built in. Apps running in containers make domain
lookups when they try to access other components. The DNS service in Docker per-
forms that lookup—if the domain name is actually a container name, Docker returns
the container’s IP address, and the consumer can work directly across the Docker net-
work. If the domain name isn’t a container, Docker passes the request on to the server
where Docker is running, so it will make a standard DNS lookup to find an IP address
on your organization’s network or the public internet.
 You can see that in action with the image-gallery app. The response from
Docker’s DNS service will contain a single IP address for services running in a single
container, or multiple IP addresses if the service is running at scale across multiple
containers.

TRY IT NOW In the same terminal session, use Docker Compose to bring the
application up with the API running at a scale of three. Then connect to a ses-
sion in the web container—choose the Linux or Windows command to run—
and perform a DNS lookup:

```
docker-compose up -d --scale iotd=3
# for Linux containers:
docker container exec -it image-of-the-day_image-gallery_1 sh
# for Windows containers:
docker container exec -it image-of-the-day_image-gallery_1 cmd nslookup accesslog
exit
```
nslookup is a small utility that is part of the base image for the web application—it
performs a DNS lookup for the name you provide, and it prints out the IP address. My
output is in figure 7.8—you can see there’s an error message from nslookup, which
you can ignore (that’s to do with the DNS server itself), and then the IP address for
the container. My accesslog container has the IP address 172.24.0.2.

Containers plugged into the same Docker network will get IP addresses in the same
network range, and they connect over that network. Using DNS means that when your
containers get replaced and the IP address changes, your app still works because the
DNS service in Docker will always return the current container’s IP address from
the domain lookup. 
 You can verify that by manually removing the accesslog container using the
Docker CLI, and then bringing the application back up again using Docker Compose.
Compose will see there’s no accesslog container running, so it will start a new one.
That container may have a new IP address from the Docker network—depending on
other containers being created—so when you run a domain lookup, you may see a dif-
ferent response.

TRY IT NOW Still in the same terminal session, use the Docker CLI to remove
the accesslog container, and then use Docker Compose to bring the app
back to the desired state. Then connect to the web container again, using sh
in Linux or cmd in Windows, and run some more DNS lookups:
```
docker container rm -f image-of-the-day_accesslog_1
docker-compose up -d --scale iotd=3
# for Linux containers:
docker container exec -it image-of-the-day_image-gallery_1 sh
# for Windows containers:
docker container exec -it image-of-the-day_image-gallery_1 cmd
nslookup accesslog
nslookup iotd
exit
```
You can see my output in figure 7.9. In my case there were no other processes creating
or removing containers, so the same IP address 172.24.0.2 got used for the new
accesslog container. In the DNS lookup for the iotd API, you can see that three IP
addresses are returned, one for each of the three containers in the service.
 DNS servers can return multiple IP address for a domain name. Docker Compose
uses this mechanism for simple load-balancing, returning all the container IP
addresses for a service. It’s up to the application that makes the DNS lookup how it
processes multiple responses; some apps take a simplistic approach of using the first
address in the list. To try to provide load-balancing across all the containers, the
Docker DNS returns the list in a different order each time. You’ll see that if you repeat
the nslookup call for the iotd service—it’s a basic way of trying to spread traffic around
all the containers.
 Docker Compose records all the startup options for your containers, and it takes
care of communication between containers at runtime. You can also use it to set up
the configuration for your environments.

## 7.4 Docker Compose 中的应用配置

The to-do app from chapter 6 can be used in different ways. You can run it as a single
container, in which case it stores data in a SQLite database—which is just a file inside
the container. You saw in chapter 6 how to use volumes to manage that database file.
SQLite is fine for small projects, but larger apps will use a separate database, and the
to-do app can be configured to use a remote Postgres SQL database instead of local
SQLite.
 Postgres is a powerful and popular open source relational database. It works
nicely in Docker, so you can run a distributed application where the app is running
in one container and the database is in another container. The Docker image for
the to-do app has been built in line with the guidance in this book, so it packages a
default set of configuration for the dev environment, but config settings can be
applied so they work with other environments. We can apply those config settings
using Docker Compose.
 Take a look at the services for the Compose file in listing 7.3—these specify a Post-
gres database service and the to-do application service.

> Listing 7.3 The Compose services for the to-do app with a Postgres database

```
services:
  todo-db:
    image: diamol/postgres:11.5 
    ports:
      - "5433:5432"
    networks:
      - app-net
  todo-web:
    image: diamol/ch06-todo-list
    ports:
      - "8020:80"
    environment:
      - Database:Provider=Postgres
    depends_on:
      - todo-db
    networks:
      - app-net
    secrets:
      - source: postgres-connection
        target: /app/config/secrets.json
```
The specification for the database is straightforward—it uses the diamol/post-
gres:11.5 image, publishes the standard Postgres port 5342 in the container to port
5433 on the host, and uses the service name todo-db, which will be the DNS name for
the service. The web application has some new sections to set up configuration:
- environment sets up environment variables that are created inside the con-
tainer. When this app runs, there will be an environment variable called Data-
base:Provider set inside the container, with the value Postgres.
- secrets can be read from the runtime environment and populated as files
inside the container. This app will have a file at /app/config/secrets.json
with the contents of the secret called postgres-connection.

Secrets are usually provided by the container platform in a clustered environment—that
could be Docker Swarm or Kubernetes. They are stored in the cluster database and can
be encrypted, so they’re useful for sensitive configuration data like database connection
strings, certificates, or API keys. On a single machine running Docker, there is no cluster
database for secrets, so with Docker Compose you can load secrets from files. There’s a
secrets section at the end of this Compose file, shown in listing 7.4.

> Listing 7.4 Loading secrets from local files in Docker Compose
```
secrets:
  postgres-connection:
    file: ./config/secrets.json
```

This tells Docker Compose to load the secret called postgres-connection from the
file on the host called secrets.json. This scenario is like the bind mounts we covered
in chapter 6—in reality the file on the host gets surfaced into the container. But defin-
ing it as a secret gives you the option of migrating to a real, encrypted secret in a clus-
tered environment.
 Plugging app configuration into the Compose file lets you use the same Docker
images in different ways and be explicit about the settings for each environment. You
can have separate Compose files for your development and test environments, pub-
lishing different ports and triggering different features of the app. This Compose file
sets up environment variables and secrets to run the to-do app in Postgres mode and
provide it with the details to connect to the Postgres database.
 When you run the app, you’ll see it behaves in the same way, but now the data
is stored in a Postgres database container that you can manage separately from
the app. 

TRY IT NOW Open a terminal session at the root of the code for the book,
and switch to the directory for this exercise. In that directory you’ll see the
Docker Compose file and also the JSON file that contains the secret to load
into the application container. Start the app using docker-compose up in the
usual way:
```
cd ./ch07/exercises/todo-list-postgres
# for Linux containers:
docker-compose up -d
# OR for Windows containers (which use different file paths):
docker-compose -f docker-compose-windows.yml up -d
docker-compose ps
```
Figure 7.10 shows my output. There’s nothing new in there except the docker-
compose ps command, which lists all running containers that are part of this Compose
application.

You can browse to this version of the to-do app at http:/ /localhost:8030. The func-
tionality is the same, but now the data is being saved in the Postgres database con-
tainer. You can check that with a database client—I use Sqlectron, which is a fast, open
source, cross-platform UI for connecting to Postgres, MySQL, and SQL Server data-
bases. The address of the server is localhost:5433, which is the port published by the
container; the database is called todo, the username is postgres, and there is no
password. You can see in figure 7.11 that I’ve added some data to the web app, and I
can query it in Postgres.
 Separating the application package from the runtime configuration is a key bene-
fit of Docker. Your application image will be produced by your build pipeline, and
that same image will progress through the test environments until it is ready for pro-
duction. Each environment will apply its own config settings, using environment vari-
ables or bind mounts or secrets—that are easy to capture in Docker Compose files. In
every environment, you’re working with the same Docker images, so you can be confi-
dent you’re releasing the exact same binaries and dependencies into production that
have passed the tests in all other environments.

## 7.5 理解 Docker Compose 可以解决的问题
Docker Compose is a very neat way of describing the setup for complex distributed
apps in a small, clear file format. The Compose YAML file is effectively a deployment
guide for your application, but it’s miles ahead of a guide written as a Word docu-
ment. In the old days those Word docs described every step of the application release,
and they ran to dozens of pages filled with inaccurate descriptions and out-of-date
information. The Compose file is simple and it’s actionable—you use it to run your
app, so there’s no risk of it going out of date.
 Compose is a useful part of your toolkit when you start making more use of Docker
containers. But it’s important to understand exactly what Docker Compose is for, and
what its limitations are. Compose lets you define your application and apply the
definition to a single machine running Docker. It compares the live Docker resources
on that machine with the resources described in the Compose file, and it will send
requests to the Docker API to replace resources that have been updated and create
new resources where they are needed.
 You get the desired state of your application when you run docker-compose up, but
that’s where Docker Compose ends. It is not a full container platform like Docker
Swarm or Kubernetes—it does not continually run to make sure your application
keeps its desired state. If containers fail or if you remove them manually, Docker Com-
pose will not restart or replace them until you explicitly run docker-compose up again.
Figure 7.12 gives you a good idea of where Compose fits into the application life cycle.

That’s not to say Docker Compose isn’t suitable for production. If you’re just starting
with Docker and you’re migrating workloads from individual VMs to containers, it
might be fine as a starting point. You won’t get high availability, load balancing, or
failover on that Docker machine, but you didn’t get that on your individual app VMs
either. You will get a consistent set of artifacts for all your applications—everything has
Dockerfiles and Docker Compose files—and you’ll get consistent tools to deploy and
manage your apps. That might be enough to get you started before you look into run-
ning a container cluster.

## 7.6 实验室 

There are some useful features in Docker Compose that add reliability to running
your app. In this lab I’d like you to create a Compose definition to run the to-do web
app more reliably in a test environment:

- The application containers will restart if the machine reboots, or if the Docker
engine restarts.
- The database container will use a bind mount to store files, so you can bring the
app down and up again but retain your data.
- The web application should listen on standard port 80 for test. 
Just one hint for this one:
- You can find the Docker Compose file specification in Docker’s reference docu-
mentation at https://docs.docker.com/compose/compose-file. That defines all
the settings you can capture in Compose.

My sample solution is on the book’s GitHub repository as always. Hopefully this one
isn’t too complex, so you won’t need it: https://github.com/sixeyed/diamol/blob/
master/ch07/lab/README.md.