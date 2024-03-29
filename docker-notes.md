# Learning Docker 2021 Step by Step

## What is Docker

Docker carves up a Linux computer system into sealed containers that run your code. Each container is sealed from the world and runs by itself.

These containers are designed to be portable so that they can be shipped from one place to another. Docker does the work of getting these containers to and from your systems.

Docker also builds these containers for you and is a social platform designed to help you to find and share containers. Other people may have built containers that you can use to build on top of your won containers.

Docker isn't a virtual machine it is just an operating system that has been carved up into little pieces.

### What is a container

A container is a sealed unit of software. It has everything in it to run that piece of code including the operating system.

A container includes:

* All of the **code**.
* All of the **configs**.
* contains all of the **processes** within that container
* It has all of the **networking** to allow these containers to talk to other containers that they need to talk to and nothing else.
* It has all the **dependencies** that your system needs bundled up in that container and it also includes just enough of the operating system to run your code.

So Docker takes up all of the services to run a Linux server

![Docker process](assets/images/docker/linux-server.jpg "Docker process")

The way Docker works is that it takes up all the services to run a Linux server including networking, storage, code, interprocess communication and it makes a copy of that in the Linux kernel for each container. Each container has its own little world that it can't see out of and other containers can't see in. You might have one container that runs a database in Red Hat Linux and another container that runs a web server in Ubuntu Linux and that web server might be talking to a Redis caching server container in Open Suse Ubuntu. It doesn't matter which Linux is running in each container and Docker is program that manages all of this.

Docker sets it up, monitors and tears it down when it is no longer needed.

Docker is a client program that you run in a terminal. It is also a server program that listens to messages from that command and manages a running Linux system.

Docker has a program that builds containers from code. It takes your code, along with its dependencies and bundles it up and seals it into a container and it is a service that distributes these containers across the internet and it makes it so you can find other peoples work and they can find your work. Docker is also a company that makes all of these containers.

## Setting up Docker

Dockers primary job is to manage a Linux server and start and stop your containers as required.

Most of us don't work on laptops that run Linux all the time so many people use a virtual machine on their laptop to run the Linux server to run Docker on the server side. Docker helps run this virtual machine.

Docker has tools that make this nearly transparent.

To start with you have your computer and in your terminal it interacts with a program named Docker. This version of Docker is the client and that is connecting to a program named Docker that is a server controlling a Linux virtual machine. This virtual machine is being managed by Docker for Windows. Once you have it install you can click on the whale icon in your status bar. You usually don't have to interact with this program as it is fairly well automated.

## Installing Docker Desktop on Windows

**Note:** You should install Hyper-V and containers through Windows features before you install Docker Desktop.

Setup a Docker account. Once you have done this you can download Docker Desktop for Windows.

In the configuration option make sure you **don't** select - *Use Windows containers instead of Linux containers*. We will only want to use Linux containers.

**Note:** I have WSL for Linux on my PC so I didn't get this option. It is automatically using Linux containers.

Once installed pull up a terminal.

```bash
    docker info
```

Returns:

> Client:       
>  Context:    default      
>  Debug Mode: false        
>  Plugins:     
>   buildx: Docker Buildx (Docker Inc., v0.7.1)     
>   compose: Docker Compose (Docker Inc., v2.2.1)       
>   scan: Docker Scan (Docker Inc., v0.14.0)        
>       
> Server:       
>  Containers: 10       
>   Running: 0      
>   Paused: 0       
>   Stopped: 10     
>  Images: 28       
>  Server Version: 20.10.11     
>  Storage Driver: overlay2     
>   Backing Filesystem: extfs

This prints out information about Docker client and server. If you see a similar message you know your system is up and running.

Let's run a Docker container.

```bash
    docker run hello-world
```

This will run the ``hello-world`` container. If you don't have this container on your system Docker will download it from its repository. This won't take long because it is the smallest container you can run.

The Docker container will run and print out a massage and then tear itself down.

> Hello from Docker!

It also prints out some other information that describes the process that has just completed.

## The Docker flow - Images to containers

The Docker flow is a fundamental concept. It all begins with an image. An image is every file that is just enough of the operating system to allow you to do what you need to do. Traditionally you would install a whole operating system to do what you need to do. With Docker you pair it way down so you have a little container with just enough of the operating system to do what you need to do and you can have lots of these containers on your computer.

Let's look at our Docker images.

```bash
    docker images
```

Returns.

```bash
    REPOSITORY                    TAG       IMAGE ID       CREATED        SIZE        
    ubuntu                        latest    d13c942271d6   12 days ago    72.8MB      
    busybox                       latest    beae173ccac6   2 weeks ago    1.24MB      
    mysql                         8.0       3218b38490ce   4 weeks ago    516MB       
    alanrob17/nginx-website       latest    ae6252556552   5 weeks ago    148MB       
    ...
```

**Note:** The ``TAG`` is the version number of the image and ``IMAGE ID`` is the internal representation of the image in Docker.

Now that you have an image what do you do with it? You can run the Docker image and turn it into a running container with a process in it that is doing something. Let's look at running an image to make a container.

```bash
    docker run -it ubuntu:latest bash
```

Where ``-it`` is run a terminal interface. We want to run the ``ubuntu:latest`` image and we want to run the ``bash`` shell in the container.

We end up with the bash shell running a root.

> PS C:\Temp\docker> docker run -it ubuntu:latest bash      
> root@35801cbffbc1:/# ls       
> bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt ...      
> root@35801cbffbc1:/#

You see the bash prompt and you can run any linux command like ``ls``. When we run this we can see that we have a full command line version of Ubuntu linux with a large directory structure.

```bash
    cat /etc/lsb-release
```

Will show us the version of Ubuntu we are using.

> root@35801cbffbc1:/home/my-folder# cat /etc/lsb-release       
> DISTRIB_ID=Ubuntu     
> DISTRIB_RELEASE=20.04     
> DISTRIB_CODENAME=focal        
> DISTRIB_DESCRIPTION="Ubuntu 20.04.3 LTS"      
> root@35801cbffbc1:/home/my-folder#

To exit your container type ``exit`` or press ``CTRL-D``.

Run the Docker image again and this time pop open another terminal window at the same time.

In the new terminal window type.

```bash
    docker ps
```

This will show you a list of containers that are running at the moment.

```bash
    PS C:\Temp\docker> docker ps
    CONTAINER ID   IMAGE           COMMAND   CREATED         STATUS         PORTS     NAMES
    39e097b6bf4e   ubuntu:latest   "bash"    2 minutes ago   Up 2 minutes             nifty_lichterman
    PS C:\Temp\docker>
```

This time you can see the container that is running and because you didn't specify a name Docker will create one for you.

**Note:** Now that you are running the container you are seeing the ``container id`` not the ``image id``. These are two different Id's. You can also see the image that it came from, ``ubuntu:latest`` and see how long it has been running.

**Note:** If you are using a bash or Ubuntu terminal you can create a format string and add it to your ``.bashrc``. This will allow you to see vertical output.

In ~/.bashrc add.

```bash
    # vertical output for - docker ps --format=$FORMAT
    export FORMAT="ID\t{{.ID}}\nNAME\t{{.Names}}\nIMAGE\t{{.Image}}\nPORTS\t{{.Ports}}\nCOMMAND\t{{.Command}}\nCREATED\t{{.CreatedAt}}\nSTATUS\t{{.Status}}\n"
```

Now run.

```bash
    docker ps --format=$FORMAT
```

Returns.

> alanr@TIGER:/mnt/c/Temp/docker$ docker ps --format=$FORMAT        
> ID      39e097b6bf4e      
> NAME    nifty_lichterman      
> IMAGE   ubuntu:latest     
> PORTS     
> COMMAND "bash"        
> CREATED 2022-01-19 17:27:00 +1100 AEDT        
> STATUS  Up 23 minutes

This is an easier format to look at.

Now we have a running container. When we run a container we don't change the image. Any changes that we make will happen to the container and once you tear that container down you will lose the containers changes.

Now we are going to look at something that is different from running a virtual machine or a Linux server. Inside our container we can create a directory and a file.

> root@35801cbffbc1:/# mkdir home/my-folder     
> root@35801cbffbc1:/# cd /home/my-folder/      
> root@35801cbffbc1:/home/my-folder# touch list.txt     
> root@35801cbffbc1:/home/my-folder# ls     
> list.txt      
> root@35801cbffbc1:/home/my-folder#

**Note:** If you spin up another container you won't see the directory or file you created in the previous container. This because the image doesn't contain that directory or file.

If you exit the original container and start it up again the directory and file will have disappeared. They aren't part of the original image.

![Docker Image to Container](assets/images/docker/image-to-container.jpg "Docker Image to Container")

## The Docker flow - Containers to images

When you have a running container you can put files in it but it is only temporary and lasts until the container shuts down.

What if you want to save those files?

The next step in the Docker flow is the stopped container. When the process exits the container is still there so the file we created is still in the container. I can go back and find that container, it didn't get deleted. It is just that it is still in a stopped container.

I can look at the most recently stopped container with the ``ps`` command to show my containers.

```bash
    docker ps -a
```

``-a`` is show all stopped containers.

Shows all the containers that have run. At the top is the container we just closed.

```bash
PS C:\Temp\docker> docker ps -a
CONTAINER ID   IMAGE           COMMAND    CREATED           STATUS                      PORTS     NAMES
39e097b6bf4e   ubuntu:latest   "bash"     49 minutes ago    Exited (0) 7 seconds ago              nifty_lichterman
...
```

```bash
    docker ps -l
```

Will just show you the last container.

> alanr@TIGER:/mnt/c/Temp/docker$ docker ps -l --format=$FORMAT     
> ID      39e097b6bf4e      
> NAME    nifty_lichterman      
> IMAGE   ubuntu:latest     
> PORTS     
> COMMAND "bash"        
> CREATED 2022-01-19 17:27:00 +1100 AEDT        
> STATUS  Exited (0) 2 hours ago

![Docker Container flow](assets/images/docker/stopped-container.jpg "Docker Container flow")

At the moment we have started from an image ran a container and installed our software on it and then stopped the container.

The next step is the Docker ``commit`` command. That takes containers and makes a new image out of them. It doesn't delete the container but now we have a new image with the same content that was in the container.

![Docker Container commit flow](assets/images/docker/container-commit.jpg "Docker Container commit flow")

So Docker run and Docker commit are complimentary to each other.

Now we can make a new image.

```bash
    docker commit 39e097b6bf4e
```

This returns the new image id.

> docker commit 39e097b6bf4e        
> sha256:97f6058c77a32bcafcc53631fb73121bcf6aba6a7e16fbc4cf20a1f32f7cb33b

We have made a new image with a large sha value which isn't convenient. Let's check the images.

```bash
    docker images
```

Returns.

``` 
    REPOSITORY                    TAG       IMAGE ID       CREATED         SIZE
    <none>                        <none>    97f6058c77a3   2 minutes ago   72.8MB
    ubuntu                        latest    d13c942271d6   12 days ago     72.8MB
    busybox                       latest    beae173ccac6   2 weeks ago     1.24MB
    mysql                         8.0       3218b38490ce   4 weeks ago     516MB
    ...
```

Notice that the image that we created is at the top. The first 12 characters of the sha value identifies that this the image we created.

Also note that there is no Repository or Tag.

We can give this image a tag name.

```bash
    docker tag 97f6058c77a32bcafcc53631fb73121bcf6aba6a7e16fbc4cf20a1f32f7cb33b my-ubuntu
```

Check the images again and you see.

```
    REPOSITORY                    TAG       IMAGE ID       CREATED         SIZE
    my-ubuntu                     latest    97f6058c77a3   9 minutes ago   72.8MB
    ubuntu                        latest    d13c942271d6   12 days ago     72.8MB
    ...
```

The repository is now named ``my-ubuntu``.

Run this image and see if your directory and file still exist.

```bash
    PS C:\Temp\docker> docker run -it my-ubuntu
    root@2718af6622db:/# ls
    bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt ...
    root@2718af6622db:/# cd home
    root@2718af6622db:/home# ls
    alan
    root@2718af6622db:/home# cd alan
    root@2718af6622db:/home/alan# ls
    list.txt
```

You can do both of these steps in the one command with ``docker commit magical_blackwell my-newer-ubuntu``.

```bash
    PS C:\Temp\docker> docker ps -l
    CONTAINER ID   IMAGE       COMMAND   CREATED         STATUS                       PORTS     NAMES
    2718af6622db   my-ubuntu   "bash"    5 minutes ago   Exited (127) 3 minutes ago             magical_blackwell
    PS C:\Temp\docker> docker commit magical_blackwell my-newer-ubuntu
    sha256:3c3dd820dc428075845a2ee816611ddb608a77473a686d36b39a232210b530d4
    PS C:\Temp\docker> docker images
    REPOSITORY                    TAG       IMAGE ID       CREATED          SIZE
    my-newer-ubuntu               latest    3c3dd820dc42   13 seconds ago   72.8MB
    myubuntu                      latest    edd05ad27157   13 minutes ago   72.8MB
    ...
```

This is a simpler process. It is much easier than doing a commit and then doing a tag but just remember that underneath the hood that it is doing both commands.

## Using Path Variables in Volume Mapping in containers

We want to use an Nginx image to serve web pages. We need to run Nginx with an exposed port.

Nginx works on http or https and can be used as a load balancer spreading content across multiple web servers.

**Note:** I usually run Docker to download an image if it isn't on my PC. Another way to do this is.

```bash
    docker pull nginx
```

Once we ``pull`` an image we are able to use it as a container.

The basic command is

```bash
    docker run nginx
```

This leaves the container running but you don't see any output. This means that we are connected to the STDIN and STDOUT of the **nginx process** of the container.

Open another terminal and check that the container is running.

```bash
    docker ps -l
```

Where ``-l`` show the last container opened.

```bash
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS     NAMES
7310c22962df   nginx     "/docker-entrypoint.…"   14 seconds ago   Up 13 seconds   80/tcp    kind_mirzakhani
```

**Note:** that this container is running on Port 80. If you try to run port 80 in your browser you will get a 404 error. You are not able to connect to this web server.

If you want to connect to the web server you have to use **port mapping**.

Stop this container with a ``CTRL-C``.

### Stopping a container

If, for some reason you want to kill a container you can do this with the container Id or the name of the container.

In the case above I could kill this with.

```bash
    docker stop 7310c22962df
```

**Note:** you could have done ``docker stop 73`` and the container would have stopped.

Or

```bash
    docker stop kind_mirzakhani
```

In this case it would be better to use.

```bash
    docker kill 7310c22962df
```

We do this because the ``sh`` process doesn't react to the SIGTERM that is sent by Docker to the Docker container after ``docker stop``. In 10 seconds ``docker stop`` reverts to the ``docker kill`` process.

Do the following command to see if all containers have stopped.

```bash
    docker ps
```

### Port mapping

We need to open up an external port for the container so that we can run the web server.

```bash
    docker run -p 8080:80 nginx
```

This opens us up in the container leaving a cursor running in the terminal.

Crack open a browser with ``localhost:8080``.

![Docker container running a web server](assets/images/docker/localhost-web-server.jpg "Docker container running a web server")

We have launched this container with option ``-p``. Nginx is running on port 80 inside the container. On windows with Docker Desktop IP addresses of the containers are "not reachable". ``-p 8080:80`` means open up the port 8080 on my computer and forward it to port 80 inside the container. The external port 8080 is mapped to the internal port 80 inside your container. You can use any external port but the internal port must be 80.

If you look at the terminal you will see logs displayed through STDOUT from the container to the screen. 

The web page shows the default page that is coming from the container. We can create our own content to run with the nginx container.

### Nginx container with custom content

Once we stop the nginx container the port 8080 is no longer exposed to the browser.

To run our own content within the nginx container we have to create a new web project.

Navigate into the **home/alanr/Documents* folder and create a new folder named **website**.

Create a new index.html file and add a favicon.ico file to the folder.

To show this content we will have to do port mapping and volume mapping.

```bash
    docker run -p 8080:80 -v /home/alanr/Documents/website:/usr/share/nginx/html nginx
```

In this command we are exposing the external port ``-p 8080:80`` and mapping a local volume to the default content folder in nginx.

``-v /home/alanr/Documents/website:/usr/share/nginx/html`` means map the local content (``/home/alanr/Documents/website``) to the default content folder in nginx (``/usr/share/nginx/html``).

In the web browser.

![Docker container running my content](assets/images/docker/my-content.jpg "Docker container running my content")

**Note:** The favicon has been installed correctly. You can see it in the tab.

## Volume Mapping in the Docker Containers

We have a hard coded local path in our volume. This can cause problems in containers so we are going to replace that with the current patch as a variable.

```bash
    docker run -p 8080:80 -v $PWD:/usr/share/nginx/html nginx
```

**$PWD** is the current working folder where our website resides.

**Note:** We can use the PC's IP to server our web pages. On Windows use **ipconfig** to get your IP address. In my case, *<http://192.168.1.105:8080/>*.

## Docker Container Management (Ubuntu, Nginx)

### Running containers in the background

Usually containers with long running processes like **nginx**, **mongodb** or **redis** are started and are running in the background.

If we run our **hello-world** image again it prints out a message and once its process has finished it shuts down the container.

When we run **alpine**.

```bash
    docker run alpine
```

The process runs and once again when the process terminates it shuts down the container.

Check the history for the alpine image.

```bash
    docker history alpine
```

Returns.

```bash
IMAGE          CREATED       CREATED BY                                      SIZE      COMMENT
c059bfaa849c   8 weeks ago   /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      8 weeks ago   /bin/sh -c #(nop) ADD file:9233f6f2237d79659…   5.59MB
```

This shows the command that has run. In one of those process's it ran the shell, ``/bin/sh``. This process opened up a shell and waited for STDIN and STDOUT messages to run.

When we ran ``docker run alpine`` we didn't allow it to run with STDIN and STDOUT. We can do this with the flags ``-it``.

```bash
    docker run -ti alpine
```

In this case you will be presented with a cursor in the running process that is waiting for STDIN or STDOUT. If you do a ``docker ps`` in another terminal window the command will show you that you are running in a shell (/bin/sh)

## Deleting containers

Show all containers.

```bash
    docker ps -a
```

Show running containers.

```bash
    docker ps
```

Show id's of all containers

```bash
    docker ps -a -q
```

> 5f079a4efe6a
> fc580faebd11
> bec60e52a172
> ...

Remove a container.

```bash
    docker rm 5f0
```

I have approximately 20 containers at present and to remove each one would be tedious. Powershell can easily delete all of my containers in one command.

```powershell
    docker rm $(docker ps -a -q)
```

Will bring back a list of all the container id's that have been removed.

**Note:** be careful with this command as it deletes everything.

## Deleting images

Remove an image.

```bash
    docker rmi f3d
```

Will delete the image with that Image Id.

Once again there is a Powershell command to delete all images.

```powershell
    docker rmi $(docker images -q)
```

**Note:** we could easily delete all images and recreate them as required. This would require downloading the images from Docker Hub again.

## Dockerfile

Docker can build images automatically by reading the instructions from a ``Dockerfile``. A ``Dockerfile`` is a text document that contains all the commands a user could call on the command line to assemble an image. 

### Format

Here is the format of the Dockerfile:

```bash
    # Comment
    INSTRUCTION arguments
```

### FROM

Docker runs instructions in a Dockerfile in order. A Dockerfile must begin with a ``FROM`` instruction. This may be after parser directives, comments, and globally scoped ARGs. The ``FROM`` instruction specifies the Parent Image from which you are building. FROM may only be preceded by one or more ARG instructions, which declare arguments that are used in FROM lines in the Dockerfile.

E.g.

```dockerfile
    FROM httpd:alpine
```

```dockerfile
    FROM httpd:alpine
```

Where ``httpd`` is the image. We want a specific **tagged** version named ``alpine``.

Alpine is the base Alpine image but there are a number of other versions of Alpine we could use depending on what we want to do.

You could also name this image.

E.g.

```dockerfile
    FROM httpd:alpine AS base
```

You could have multiple ``FROM`` statements in your Dockerfile.

In most Dockerfile's FROM is the first command but you can add ``ARG``'s  before a FROM.

E.g.

```dockerfile
    ARG VERSION=latest
    FROM httpd:alpine AS base
```

Note that the ``ARG`` command can only be used before a FROM command. It can't be used after the FROM command has been run.

There is an exception to this.

```dockerfile
    ARG VERSION=latest
    FROM busybox:$VERSION

    ARG VERSION
    RUN Echo $VERSION > image_version
```

### COPY

Format

> COPY chown source destination

Where you can set permissions for the folder with ``chown``.

``source`` is the source destination. This is the local directory. If you have files in a HTML folder that you want to copy to the image destination use ``./html/``. Where ``.`` is the current directory with the Dockerfile and ``html`` is the directory in the current directory.

``destination`` is where you want the files loaded to which is usually ``/usr/local/apache2/htdocs`` in a linux web server.

We can extend this by using wildcards in the ``source``.

E.g.

```dockerfile
    # use a httpd web server with a minimal alpine Linux system
    FROM httpd:alpine

    COPY hom* /usr/local/apache2/htdocs
```

This adds all files starting with **hom**.

The ``destination`` is a relative or absolute path to ``WORKDIR``, where the ``source`` files will be copied into the ``destination`` container.

**Note:** lines with a ``#`` is a comment line.

#### from flag

Optionally ``COPY`` accepts a flag ``--from``=<name> that can be used to set the source location to a previous build stage (created with FROM .. AS <name>) that will be used instead of a build context sent by the user. In case a build stage with a specified name can't be found an image with the same name is attempted to be used instead.

```dockerfile
    FROM httpd:alpine AS base
    COPY ./html/ /usr/local/apache2/htdocs

    FROM httpd AS final
    COPY --from=base /usr/local/apache2/htdocs/ /usr/local/apache2/htdocs/app/
```

This is creating an image from two different images (httpd:alpine and httpd). The files from ``base`` in **htdocs** are going to be copied to ``/usr/local/apache2/htdocs/app/``.

Build

```bash
    docker build -t alpineweb .
```

Run.

```bash
    docker run --name alpine -p 8000:80 alpineweb
```

Now when we can open the browser to *<http://127.0.0.1:800/app>* we will see the index.html file we created.

Why would you do this? This would come in handy for an ASP.Net core application. The first build we do has the ``.csproj`` and ``bin`` folder and the next build could remove these files and the source code from our final release build.

An important point to note is that there are certain commands that create layers. ``COPY`` is one of these. In our first copy we create an images and our second copy creates another image. The ``RUN`` and ``ADD`` commands also build images.

### WORKDIR

Format.

> WORKDIR /path/to/workdir

The ``WORKDIR`` instruction sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile. If the ``WORKDIR`` doesn't exist, it will be created even if it's not used in any subsequent Dockerfile instruction.

We can use it like this.

```dockerfile
    FROM httpd:alpine
    WORKDIR /usr/local/apache2/htdocs/
    COPY ./html/ .
```

``WORKDIR`` will use this path in the container.

### EXPOSE

Format.

> EXPOSE <port> [<port>/<protocol>...]

The ``EXPOSE`` instruction informs Docker that the container listens on the specified network ports at runtime. You can specify whether the port listens on TCP or UDP, and the default is TCP if the protocol is not specified.

The ``EXPOSE`` instruction does not actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container, about which ports are intended to be published. To actually publish the port when running the container, use the -p flag on docker run to publish and map one or more ports, or the -P flag to publish all exposed ports and map them to high-order ports.

By default, EXPOSE assumes TCP. You can also specify UDP:

We can use it like this.

```dockerfile
    FROM httpd:alpine
    WORKDIR /usr/local/apache2/htdocs/
    EXPOSE 80
    EXPOSE 443
    COPY ./html/ .
```

These ports are just for documentation. You need to add the ports when using the RUN command to run a container.

### RUN

``RUN`` has 2 forms:

``RUN`` <command> (shell form, the command is run in a shell, which by default is /bin/sh -c on Linux or cmd /S /C on Windows)

``RUN`` ["executable", "param1", "param2"] (exec form)

The RUN instruction will execute any commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile.

Layering ``RUN`` instructions and generating commits conforms to the core concepts of Docker where commits are cheap and containers can be created from any point in an image's history, much like source control.

The exec form makes it possible to avoid shell string munging, and to RUN commands using a base image that does not contain the specified shell executable.

The default shell for the shell form can be changed using the SHELL command.

In the shell form you can use a \ (backslash) to continue a single RUN instruction onto the next line. For example, consider these two lines:

```bash
    RUN /bin/bash -c 'source $HOME/.bashrc; \
    echo $HOME'
```

Together they are equivalent to this single line:

```bash
    RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
```

To use a different shell, other than '/bin/sh', use the exec form passing in the desired shell. For example:

```bash
    RUN ["/bin/bash", "-c", "echo hello"]
```

**Note:**

The exec form is parsed as a JSON array, which means that you must use double-quotes (") around words not single-quotes (').

### Full working example

The following dockerfile is a complete working example that creates a database as one layer and then adds a database as another layer in the final release build of the container.

```dockerfile
FROM mcr.microsoft.com/mssql/server:2019-latest AS build
ENV MSSQL_PID=Developer
ENV ACCEPT_EULA=Y
ENV SA_PASSWORD=Pwd12345!

WORKDIR /tmp
COPY RecordDB_BU.BAK .
COPY restore-backup.sql .

RUN /opt/mssql/bin/sqlservr --accept-eula & sleep 30 \
    && /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "Pwd12345!" -i /tmp/restore-backup.sql \
    && pkill sqlservr

FROM mcr.microsoft.com/mssql/server:2019-latest AS release

ENV ACCEPT_EULA=Y

COPY --from=build /var/opt/mssql/data /var/opt/mssql/data
```

Sql code that is executed.

#### restore-backup.sql

```sql
    RESTORE DATABASE [RecordDB] FROM DISK = '/tmp/RecordDB_BU.BAK'
    WITH FILE = 1,
    MOVE 'RecordDB' TO '/var/opt/mssql/data/RecordDB.mdf',
    MOVE 'RecordDB_log' TO '/var/opt/mssql/data/RecordDB.ldf',
    NOUNLOAD, REPLACE, STATS = 5
    GO
``` 

Build

```bash
    docker build -t record-db .
```

Run

```bash
    docker run -p 11433:1433 -d record-db
```

I can run the container in MSSQL by connecting to server with a name of ``localhost,11433``, SQL Server Authentication, sa, Pwd12345!

For full notes on this process see ``docker-mssql-f.html``.
