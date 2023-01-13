# 第六章 使用 Docker Volumes 作为持久化存储

Containers are a perfect runtime for stateless applications. You can meet increased demand by running multiple containers on your cluster, knowing that every con-tainer will handle requests in the same way. You can release updates with an auto-mated rolling upgrade, which keeps your app online the whole time.

But not all parts of your app will be stateless. There will be components that use disks to improve performance or for permanent data storage. And you can run those components in Docker containers too.

Storage does add complications, so you need to understand how to Dockerize stateful apps. This chapter takes you through Docker volumes and mounts, and shows you how the container filesystem works.

## 6.1 为什么容器中的数据不是永久的

A Docker container has a filesystem with a single disk drive, and the contents of that drive are populated with the files from the image. You’ve seen that already:
when you use the COPY instruction in a Dockerfile, the files and directories you copy into the image are there when you run a container from the image. And you know Docker images are stored as multiple layers, so the container’s disk is actually a virtual filesystem that Docker builds up by merging all the image layers together.

Each container has its own filesystem, independent of other containers. You can run multiple containers from the same Docker image, and they will all start with the same disk contents. The application can alter files in one container, and that won’t affect the files in other containers—or in the image. That’s straightforward to see by running a couple of containers that write data, and then looking at their output.

TRY IT NOW Open a terminal session and run two containers from the same image. The application in the image writes a random number to a file in the container:
```
docker container run --name rn1 diamol/ch06-random-number
docker container run --name rn2 diamol/ch06-random-number
```

That container runs a script when it starts, and the script writes some random data to a text file and then ends, so those containers are in the exited state. The two containers started from the same image, but they will have different file contents. You learned in chapter 2 that Docker doesn’t delete the container’s filesystem when it exits—it’s retained so you can still access files and folders.

The Docker CLI has the docker container cp command to copy files between containers and the local machine. You specify the name of the container and the file path, and you can use that to copy the generated random number files from these containers onto your host computer, so you can read the contents.

TRY IT NOW
Use docker container cp to copy the random number file from each of the containers, and then check the contents:
```
docker container cp rn1:/random/number.txt number1.txt
docker container cp rn2:/random/number.txt number2.txt
cat number1.txt
cat number2.txt
```

Your output will be similar to mine in figure 6.1. Each container has written a file at the same path, /random/number.txt, but when the files are copied onto the local machine, you can see that the contents are different. This is a simple way of showing that every container has an independent filesystem. In this case it’s a single file that’s different, but these could be database containers that start with the same SQL engine running, but store completely different data.

![图6.1](./images/Figure6.1.png)
<center>图6.1 </center>
 
The filesystem inside a container appears to be a single disk: /dev/sda1 on Linux containers and C:\ on Windows containers. But that disk is a virtual filesystem that Docker builds from several sources and presents to the container as a single unit. The basic sources for that filesystem are the image layers, which can be shared between containers, and the container’s writeable layer, which is unique to each container.

Figure 6.2 shows how that looks for the random number image and the two containers. You should take away two important things from figure 6.2: image layers are shared so they have to be read-only, and there is one writeable layer per container,which has the same life cycle as the container. Image layers have their own life cycle—any images you pull will stay in your local cache until you remove them. But the container writeable layer is created by Docker when the container is started, and it’s deleted by Docker when the container is removed. (Stopping a container doesn’t
automatically remove it, so a stopped container’s filesystem does still exist.)

![图6.2](./images/Figure6.2.png)
<center>图6.2 </center>

Of course, the writeable layer isn’t just for creating new files. A container can edit existing files from the image layers. But image layers are read-only, so Docker does some special magic to make that happen. It uses a copy-on-write process to allow edits to files that come from read-only layers. When the container tries to edit a file in an image layer, Docker actually makes a copy of that file into the writable layer, and the edits happen there. It’s all seamless for the container and the application, but it’s the cornerstone of Docker’s super-efficient use of storage.

Let’s work through that with one more simple example, before we move on to running some more useful stateful containers. In this exercise you’ll run a container that prints out the contents of a file from an image layer. Then you’ll update the file contents and run the container again to see what’s changed.

TRY IT NOW
Run these commands to start a container that prints out its file contents, then change the file, and start the container again to print out the new file contents:
```
docker container run --name f1 diamol/ch06-file-display
echo "http://eltonstoneman.com" > url.txt
docker container cp url.txt f1:/input.txt
docker container start --attach f1
```

This time you’re using Docker to copy a file from your host computer into the container, and the target path is the file that the container displays. When you start the container again, the same script runs, but now it prints out different contents—you can see my output in figure 6.3.

![图6.3](./images/Figure6.3.png)
<center>图6.3 </center>

Modifying the file in the container affects how that container runs, but it doesn’t affect the image or any other containers from that image. The changed file only lives in the writeable layer for that one container—a new container will use the original contents from the image, and when container f1 is removed, the updated file is gone.

TRY IT NOW
Start a new container to check that the file in the image is unchanged. Then remove the original container and confirm that the data is gone:
```
docker container run --name f2 diamol/ch06-file-display
docker container rm -f f1
docker container cp f1:/input.txt 
```

You’ll see the same output as mine in figure 6.4. The new container uses the original file from the image, and when you remove the original container, its filesystem is removed and the changed file is gone forever.

![图6.4](./images/Figure6.4.png)
<center>图6.4</center>

The container filesystem has the same life cycle as the container, so when the container is removed, the writeable layer is removed, and any changed data in the container is lost.Removing containers is something you will do a lot. In production, you upgrade your app by building a new image, removing the old containers, and replacing them with new ones from the updated image. Any data that was written in your original app containers is lost, and the replacement containers start with the static data from the image.

There are some scenarios where that’s fine, because your application only writes transient data—maybe to keep a local cache of data that is expensive to calculate or retrieve—and it’s fine for replacement containers to start with an empty cache. In other cases, it would be a disaster. You can run a database in a container, but you wouldn’t expect to lose all your data when you roll out an updated database version.

Docker has you covered for those scenarios too. The virtual filesystem for the container is always built from image layers and the writeable layer, but there can be additional sources as well. Those are Docker volumes and mounts. They have a separate life cycle from containers, so they can be used to store data that persists between container replacements.

## 6.2 运行容器使用 Docker volumes

A Docker volume is a unit of storage—you can think of it as a USB stick for containers.Volumes exist independently of containers and have their own life cycles, but they can be attached to containers. Volumes are how you manage storage for stateful applications when the data needs to be persistent. You create a volume and attach it to your application container; it appears as a directory in the container’s filesystem. The container writes data to the directory, which is actually stored in the volume. When you update your app with a new version, you attach the same volume to the new container,and all the original data is available.

There are two ways to use volumes with containers: you can manually create volumes and attach them to a container, or you can use a VOLUME instruction in the Dockerfile. That builds an image that will create a volume when you start a container. The syntax is simply VOLUME <target-directory>. Listing 6.1 shows part of the multi-stage Dockerfile for the image diamol/ch06-todo-list, which is a stateful app that uses a volume.

> 清单 6.1 Part of a multi-stage Dockerfile using a volume

```
FROM diamol/dotnet-aspnet
WORKDIR /app
ENTRYPOINT ["dotnet", "ToDoList.dll"]

VOLUME /data
COPY --from=builder /out/ .
```

When you run a container from this image, Docker will automatically create a volume and attach it to the container. The container will have a directory at /data (or C:\data on Windows containers), which it can read from and write to as normal. But the data is actually being stored in a volume, which will live on after the container is removed. You can see that if you run a container from the image and then check the volumes.

TRY IT NOW
Run a container for the to-do list app, and have a look at the volume Docker created:
```
docker container run --name todo1 -d -p 8010:80 diamol/ch06-todo-list
docker container inspect --format '{{.Mounts}}' todo1
docker volume ls
```

You’ll see output like mine in figure 6.5. Docker creates a volume for this containe and attaches it when the container runs. I’ve filtered the volume list to show just the volume for my container.

![图6.5](./images/Figure6.5.png)
<center>图6.5 </center>

Docker volumes are completely transparent to the app running in the container.Browse to http:/ /localhost:8010 and you’ll see the to-do app. The app stores data in a file at the /data directory, so when you add items through the web page, they are being stored in the Docker volume. Figure 6.6 shows the app in action—it’s a special to-do list that works very well for people with workloads like mine; you can add items but you can’t ever remove them.

![图6.6](./images/Figure6.6.png)
<center>图6.6 </center>

Volumes declared in Docker images are created as a separate volume for each container, but you can also share volumes between containers. If you start a new container running the to-do app, it will have its own volume, and the to-do list will start off being empty. But you can run a container with the volumes-from flag, which attaches another container’s volumes. In this example you could have two to-do app containers sharing the same data.

TRY IT NOW
Run a second to-do list container and check the contents of the data directory. Then compare that to another new container that shares the volumes from the first container (the exec commands are slightly different for Windows and Linux):

```
# this new container will have its own volume
docker container run --name todo2 -d diamol/ch06-todo-list

# on Linux:
docker container exec todo2 ls /data

# on Windows:
docker container exec todo2 cmd /C "dir C:\data"

# this container will share the volume from todo1
docker container run -d --name t3 --volumes-from todo1 diamol/ch06-
todo-list

# on Linux:
docker container exec t3 ls /data

# on Windows:
docker container exec t3 cmd /C "dir C:\data"
```

The output will look like figure 6.7 (I’m running on Linux for this example). The second container starts with a new volume, so the /data directory is empty. The third container uses the volumes from the first, so it can see the data from the original application container.

![图6.7](./images/Figure6.7.png)
<center>图6.7 </center>

Sharing volumes between containers is straightforward, but it’s probably not what you want to do. Apps that write data typically expect exclusive access to the files, and they may not work correctly (or at all) if another container is reading and writing to the same file at the same time. Volumes are better used to preserve state between application upgrades, and then it’s better to explicitly manage the volumes. You can create a named volume and attach that to the different versions of your application container.

TRY IT NOW
Create a volume and use it in a container for version 1 of the todo app. Then add some data in the UI and upgrade the app to version 2. The filesystem paths for the container need to match the operating system, so I’m using variables to make copy and pasting easier:

```
# save the target file path in a variable:
target='/data' # for Linux containers
$target='c:\data' # for Windows containers

# create a volume to store the data:
docker volume create todo-list

# run the v1 app, using the volume for app storage:
docker container run -d -p 8011:80 -v todo-list:$target --name todo-v1
diamol/ch06-todo-list

# add some data through the web app at http://localhost:8011

# remove the v1 app container:
docker container rm -f todo-v1

# and run a v2 container using the same volume for storage:
docker container run -d -p 8011:80 -v todo-list:$target --name todo-v2
diamol/ch06-todo-list:v2
```

The output in figure 6.8 shows that the volume has its own life cycle. It exists before any containers are created, and it remains when containers that use it are removed.The application preserves data between upgrades because the new container uses the same volume as the old container.

![图6.8](./images/Figure6.8.png)
<center>图6.8 </center>

Now when you browse to http:/ /localhost:8011 you’ll see version 2 of the to-do application, which has had a UI makeover from an expensive creative agency. Figure 6.9 shows this is ready for production now.

![图6.9](./images/Figure6.9.png)
<center>图6.9 </center>

There’s one thing to make clear about Docker volumes before we move on. The VOLUME instruction in the Dockerfile and the volume (or v) flag for running containers are separate features. Images built with a VOLUME instruction will always create a volume for a container if there is no volume specified in the run command. The volume will have a random ID, so you can use it after the container is gone, but only if you can work out which volume has your data.

The volume flag mounts a volume into a container whether the image has a volume specified or not. If the image does have a volume, the volume flag can override it for the container by using an existing volume for the same target path—so a new volume won’t be created. That’s what happened with the to-do list containers.

You can use the exact same syntax and get the same results for containers where no volume is specified in the image. As an image author, you should use the VOLUME instruction as a fail-safe option for stateful applications. That way containers will always write data to a persistent volume even if the user doesn’t specify the volume flag. But as an image user, it’s better not to rely on the defaults and to work with named volumes.

## 6.3 运行容器使用文件系统挂载

Volumes are great for separating out the life cycle of storage and still have Docker manage all the resources for you. Volumes live on the host, so they are decoupled from containers. Docker also provides a more direct way of sharing storage between containers and hosts using bind mounts. A bind mount makes a directory on the host available as a path on a container. The bind mount is transparent to the container—it’s just a directory that is part of the container’s filesystem. But it means you can access host files from a container and vice versa, which unlocks some interesting patterns.

Bind mounts let you explicitly use the filesystems on your host machine for container data. That could be a fast solid-state disk, a highly available array of disks, or even a distributed storage system that’s accessible across your network. If you can access that filesystem on your host, you can use it for containers. I could have a server with a RAID array and use that as reliable storage for my to-do list application database.

TRY IT NOW
I really do have a server with a RAID array, but you may not, so we’ll just create a local directory on your host computer and bind mount it into a container. Again, the filesystem paths need to match the host operating system, so I’ve declared variables for the source path on your machine and the target path for the container. Note the different lines for Windows and Linux:

```
$source="$(pwd)\databases".ToLower(); $target="c:\data" # Windows
source="$(pwd)/databases" && target='/data' # Linux

mkdir ./databases

docker container run --mount type=bind,source=$source,target=$target
-d -p 8012:80 diamol/ch06-todo-list

curl http://localhost:8012

ls ./databases
```

This exercise uses the curl command (which is on Linux, Mac, and Windows systems) to make an HTTP request to the to-do app. That causes the app to start up, which creates the database file. The final command lists the contents of the local databases directory on your host, and that will show that the application’s database file is actually there on your host computer, as you see in figure 6.10.

![图6.10](./images/Figure6.10.png)
<center>图6.10 </center>

The bind mount is bidirectional. You can create files in the container and edit them on the host, or create files on the host and edit them in the container. There’s a security aspect here, because containers should usually run as a least-privilege account, to minimize the risk of an attacker exploiting your system. But a container needs elevated permissions to read and write files on the host, so this image is built with the USER instruction in the Dockerfile to give containers administrative rights—it uses the built-in root user in Linux and the ContainerAdministrator user in Windows.

If you don’t need to write files, you can bind mount the host directory as read-only inside the container. This is one option for surfacing configuration settings from the host into the application container. The to-do application image is packaged with a default configuration file that sets the logging level for the app to a minimum amount. You can run a container from the same image but mount a local configuration directory into the container, and override the app’s configuration without changing the image.

TRY IT NOW
The to-do application will load an extra configuration file from the /app/config path if it exists. Run a container that bind-mounts a local directory to that location, and the app will use the host’s configuration file.Start by navigating to your local copy of the DIAMOL source code, and then run these commands:

```
cd ./ch06/exercises/todo-list

# save the source path as a variable:
$source="$(pwd)\config".ToLower(); $target="c:\app\config" # Windows
source="$(pwd)/config" && target='/app/config' # Linux

# run the container using the mount:
docker container run --name todo-configured -d -p 8013:80 --mount
type=bind,source=$source,target=$target,readonly diamol/ch06-
todo-list

# check the application:
curl http://localhost:8013

# and the container logs:
docker container logs todo-configured
```


The config file in the directory on the host is set to use much more detailed logging.When the container starts, it maps that directory, and the application sees the config file and loads the logging configuration. In the final output shown in figure 6.11,there are lots of debug log lines, which the app wouldn’t write with the standard configuration.

![图6.11](./images/Figure6.11.png)
<center>图6.11 </center>

You can bind-mount any source that your host computer has access to. You could use a shared network drive mounted to /mnt/nfs on a Linux host, or mapped to the X: drive on a Windows host. Either of those could be the source for a bind mount and be surfaced into a container in the same way. It’s a very useful way to get reliable and even distributed storage for stateful apps running in containers, but there are some limitations you need to understand.

## 6.4 文件系统挂载的局限性

To use bind mounts and volumes effectively, you need to understand some key scenarios and limitations, some of which are subtle and will only appear in unusual combinations of containers and filesystems.

The first scenario is straightforward: what happens when you run a container with a mount, and the mount target directory already exists and has files from the image layers? You might think that Docker would merge the source into the target. Inside the container you’d expect to see that the directory has all the existing files from the image, and all the new files from the mount. But that isn’t the case. When you mount a target that already has data, the source directory replaces the target directory—so the original files from the image are not available.

You can see this with a simple exercise, using an image that lists directory contents when it runs. The behavior is the same for Linux and Windows containers, but the filesystem paths in the commands need to match the operating system.

TRY IT NOW
Run the container without a mount, and it will list the directory contents from the image. Run it again with a mount, and it will list the contents of the source directory (there are variables again here to support Windows and Linux):

```
cd ./ch06/exercises/bind-mount

$source="$(pwd)\new".ToLower(); $target="c:\init" # Windows
source="$(pwd)/new" && target='/init' # Linux

docker container run diamol/ch06-bind-mount

docker container run --mount type=bind,source=$source,target=$target
diamol/ch06-bind-mount
```

You’ll see that in the first run the container lists two files: abc.txt and def.txt. These are loaded into the container from the image layers. The second container replaces the target directory with the source from the mount, so those files are not listed. Only the files 123.txt and 456.txt are shown, and these are from the source directory on the host. Figure 6.12 shows my output.

![图6.12](./images/Figure6.12.png)
<center>图6.12 </center>

The second scenario is a variation on that: what happens if you mount a single file from the host to a target directory that exists in the container filesystem? This time the directory contents are merged, so you’ll see the original files from the image and the new file from the host—unless you’re running Windows containers, where this feature isn’t supported at all.

The container filesystem is one of the few areas where Windows containers are not the same as Linux containers. Some things do work in the same way. You can use standard Linux-style paths inside Dockerfiles, so /data works for Windows containers and becomes an alias of C:\data. But that doesn’t work for volume mounts and bind mounts, which is why the exercises in this chapter use variables to give Linux users /data and Windows C:\data.

The limitation on single-file mounts is more explicit. You can try this yourself if you have Windows and Linux machines available, or if you’re running Docker Desktop on Windows, which supports both Linux and Windows containers.

TRY IT NOW
The behavior of single-file mounts is different on Linux and Windows. If you have Linux and Windows containers available, you can see that in action:

```
cd ./ch06/exercises/bind-mount

# on Linux:
docker container run --mount
type=bind,source="$(pwd)/new/123.txt",target=/init/123.txt
diamol/ch06-bind-mount

# on Windows:
docker container run --mount
type=bind,source="$(pwd)/new/123.txt",target=C:\init\123.txt
diamol/ch06-bind-mount

docker container run diamol/ch06-bind-mount

docker container run --mount
type=bind,source="$(pwd)/new/123.txt",target=/init/123.txt
diamol/ch06-bind-mount
```


The Docker image is the same, and the commands are the same—apart from the OS-specific filesystem path for the target. But you’ll see when you run this that the Linux example works as expected but you get an error from Docker on Windows, as in figure 6.13.

![图6.13](./images/Figure6.13.png)
<center>图6.13 </center>

The third scenario is less common. It’s very difficult to reproduce without setting up a lot of moving pieces, so there won’t be an exercise to cover this—you’ll have to take my word for it. The scenario is, what happens if you bind-mount a distributed filesystem into a container? Will the app in the container still work correctly? See,even the question is complicated.

Distributed filesystems let you access data from any machine on the network, and they usually use different storage mechanisms from your operating system’s local filesystem. It could be a technology like SMB file shares on your local network, Azure Files, or AWS S3 in the cloud. You can mount locations from distributed storage systems like these into a container. The mount will look like a normal part of the filesystem, but if it doesn’t support the same operations, your app could fail.

There’s a concrete example in figure 6.14 of trying to run the Postgres database system in a container on the cloud, using Azure Files for container storage. Azure Files supports normal filesystem operations like read and write, but it doesn’t support some of the more unusual operations that apps might use. In this case the Postgres container tries to create a file link, but Azure Files doesn’t support that feature, so the app crashes.

![图6.14](./images/Figure6.14.png)
<center>图6.14 </center>

This scenario is an outlier, but you need to be aware of it because if it happens there’s really no way around it. The source for your bind mount may not support all the filesystem features that the app in your container expects. This is something you can’t plan for—you won’t know until you try your app with your storage system. If you want to use distributed storage for containers, you should be aware of this risk, and you also need to understand that distributed storage will have very different performance characteristics from local storage. An application that uses a lot of disk may grind to a halt if you run it in a container with distributed storage, where every file write goes over the network.

## 6.5 理解容器文件系统如何构建

We’ve covered a lot in this chapter. Storage is an important topic because the options for containers are very different from storage on physical computers or virtual machines. I’m going to finish up with a consolidated look at everything we’ve covered,with some best-practice guidelines for using the container filesystem.

Every container has a single disk, which is a virtual disk that Docker pieces together from several sources. Docker calls this the union filesystem. I’m not going to look at how Docker implements the union filesystem, because there are different technologies for different operating systems. When you install Docker, it makes the right choice for your OS, so you don’t need to worry about the details.

The union filesystem lets the container see a single disk drive and work with files and directories in the same way, wherever they may be on the disk. But the locations on the disk can be physically stored in different storage units, as figure 6.15 shows.

![图6.15](./images/Figure6.15.png)
<center>图6.15 </center>

Applications inside a container see a single disk, but as the image author or container user, you choose the sources for that disk. There can be multiple image layers,multiple volume mounts, and multiple bind mounts in a container, but they will always have a single writeable layer. Here are some general guidelines for how you should use the storage options:

- Writeable layer—Perfect for short-term storage, like caching data to disk to save on network calls or computations. These are unique to each container but are gone forever when the container is removed.

- Local bind mounts—Used to share data between the host and the container.Developers can use bind mounts to load the source code on their computer into the container, so when they make local edits to HTML or JavaScript files,the changes are immediately in the container without having to build a new image.

- Distributed bind mounts—Used to share data between network storage and containers. These are useful, but you need to be aware that network storage will not have the same performance as local disk and may not offer full filesystem features. They can be used as read-only sources for configuration data or a shared cache, or as read-write to store data that can be used by any container on any
machine on the same network.

- Volume mounts—Used to share data between the container and a storage object that is managed by Docker. These are useful for persistent storage, where the application writes data to the volume. When you upgrade your app with a new container, it will retain the data written to the volume by the previous version.
- Image layers—These present the initial filesystem for the container. Layers are stacked, with the latest layer overriding earlier layers, so a file written in a layer at the beginning of the Dockerfile can be overridden by a subsequent layer that writes to the same path. Layers are read-only, and they can be shared between containers.

## 6.6 实验室
We’ll put those pieces together in this lab. It’s back to the good old to-do list app, but this time with a twist. The app will run in a container and start with a set of tasks already created. Your job is to run the app using the same image but with different storage options, so that the to-do list starts off empty, and when you save items they get stored to a Docker volume. The exercises from this chapter should get you there, but here are some hints:

- Remember it’s docker rm -f $(docker ps -aq) to remove all your existing containers.
- Start by running the app from diamol/ch06-lab to check out the tasks.
- Then you’ll need to run a container from the same image with some mounts.
- The app uses a configuration file—there’s more in there than settings for the log.

My sample solution is on the book’s GitHub repository if you need it, but you should try to work through this one because container storage can trip you up if you haven’t had much experience. There are a few ways to solve this. My solution is here: https://github.com/sixeyed/diamol/blob/master/ch06/lab/README.md.