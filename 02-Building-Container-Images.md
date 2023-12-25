# Building container images

#### Source

> d:\Docker\dockerfile-example

## Introducing Dockerfiles

```bash
    FROM alpine:latest
    LABEL maintainer="Alan Robson"
    LABEL description="This example Dockerfile installs NGINX."

    RUN apk add --update nginx && \
    RUN apk add nano
    
    rm -rf /var/cache/apk/* && \
    mkdir -p /tmp/nginx/
    
    COPY files/nginx.conf /etc/nginx/nginx.conf
    COPY files/default.conf /etc/nginx/conf.d/default.conf
    ADD files/html.tar.gz /usr/share/nginx/
        
    EXPOSE 80/tcp
    
    ENTRYPOINT ["nginx"]
    CMD ["-g", "daemon off;"]
```

## Reviewing Dockerfiles in depth

Let's take a look at the instructions we used in the preceding Dockerfile example. We will look at them in the order they appeared in:

* FROM
* LABEL
* RUN
* COPY and ADD
* EXPOSE
* ENTRYPOINT and CMD
* Other Dockerfile instructions

### FROM

The **FROM** instruction tells Docker which base you would like to use for your image. We are using **Alpine Linux**, so we simply have to state the name of the image and the release tag we wish to use. In our case, to use the latest official Alpine Linux image, we simply need to add ``alpine:latest``.

### LABEL

The **LABEL** instruction can be used to add extra information to the image. This information can be anything from a version number to a description. It’s also recommended that you limit the number of labels you use. A good label structure will
help others who will use our image later on.

However, using too many labels can cause the image to become inefficient as well, so I would recommend using the label schema detailed at <http://label-schema.org> . You can view the containers’ labels with the following docker inspect command:

Check the labels with this command.

```bash
    docker image inspect dockerfile-example-docker-example
```

Or for just the labels.

```bash
    docker image inspect -f "{{.Config.Labels}}" dockerfile-example-docker-example
```

Returns:

> map[com.docker.compose.project:dockerfile-example com.docker.compose.service:docker-example com.docker.compose.version:2.23.3 description:This example Dockerfile installs NGINX and runs my HTML content. maintainer:Alan Robson]

Generally, it is better to define your labels when you create a container from your image, rather than at build time, so it is best to keep labels down to just metadata about the image and nothing else.

### RUN

The **RUN** instruction is where we interact with our image to install software and run scripts, commands, and other tasks. As you can see from the first RUN instruction, we are actually running three commands:

```bash
    RUN apk add --update nginx && \
    rm -rf /var/cache/apk/* && \
    mkdir -p /tmp/nginx/
    RUN apk add nano
```

The first of our three commands is the equivalent of running the following command if we had a shell on an Alpine Linux host:

```bash
    apk add --update nginx
```

This command installs NGINX using Alpine Linux’s package manager.

#### Tip

We are using the **&&** operator to move on to the next command if the previous command was successful. This makes it more obvious which commands we are running in the Dockerfile. We are also using **\\**, which allows us to split the command over multiple lines, making it even easier to read.

The following command in our chain removes any temporary files to keep the size of our image to a minimum:

```bash
    rm -rf /var/cache/apk/*
```

The final command in our chain creates a folder with a path of /tmp/nginx/ so that NGINX will start correctly when we run the container:

```bash
    mkdir -p /tmp/nginx/
```

I also added another RUN command. 

```bash
    RUN apk add nano
```

This allows me to use the **Nano** text editor in case I want to edit my HTML files in the container.

We could have also used the following in our Dockerfile to achieve the same results:

```bash
    RUN apk add --update nginx
    RUN rm -rf /var/cache/apk/*
    RUN mkdir -p /tmp/nginx/
    RUN apk add nano
```

However, much like adding multiple labels, this is considered inefficient as it can add to the overall size of the image, which we should try to avoid. There are some valid use cases for this as some commands do not work well when they are stringed together using &&. However, for the most part, this approach to running commands should be avoided when your image is being built.

### COPY and ADD

At first glance, **COPY** and **ADD** look like they are doing the same task in that they are both used to transfer files to the image. However, there are some important differences, which we will discuss here.

The COPY instruction is the more straightforward of the two:

```bash
    COPY files/nginx.conf /etc/nginx/nginx.conf
    COPY files/default.conf /etc/nginx/conf.d/default.conf
```

As you have probably guessed, we are copying two files from the files folder on the host we are building our image on. The first file is nginx.conf, which is a minimal NGINX configuration file:

```bash
    user  nginx;
    worker_processes  1;

    error_log  /var/log/nginx/error.log warn;
    pid        /var/run/nginx.pid;

    events {
        worker_connections  1024;
    }

    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';
        access_log  /var/log/nginx/access.log  main;
        sendfile        off;
        keepalive_timeout  65;
        include /etc/nginx/conf.d/*.conf;
    }
```

This will overwrite the NGINX configuration that was installed as part of the APK installation in the RUN instruction.

The next file, ``default.conf``, is the simplest virtual host that we can configure, and contains the following content:

```bash
    server {
      location / {
          root /usr/share/nginx/html;
      }
    }
```

Again, this will overwrite any existing files. So, why might we use the **ADD** instruction?

In our example Dockerfile, the ADD instruction looks as follows:

```bash
    ADD files/html.tar.gz /usr/share/nginx/
```

As you can see, we are adding a file called **html.tar.gz**, but we are not actually doing anything with the archive to uncompress it in our Dockerfile. This is because **ADD** automatically uploads, uncompresses, and adds the resulting folders and files to the path we request it to, which in our case is ``/usr/share/nginx/``. This gives us our web root of ``/usr/share/nginx/html/``, as we defined in the virtual host block in the **default.conf** file that we copied to the image.

The **ADD** instruction can also be used to add content from remote sources. For example, consider the following:

```bash
    ADD https://raw.githubusercontent.com/PacktPublishing/Mastering-Docker-Fourth-Edition/master/chapter02/dockerfile-   example/files/html.tar.gz /usr/share/nginx/
```

The preceding command line would download html.tar.gz from <https://raw.githubusercontent.com/PacktPublishing/Mastering-Docker-Fourth-Edition/master/chapter02/dockerfile-example/files/> and place the file in the /usr/share/nginx/ folder on the image.

Archive files from a remote source are treated as files and are not uncompressed, which you will have to take into account when using them. This means that the file will have to be added before the **RUN** instruction so that we can manually unarchive the folder and also remove the **html.tar.gz** file.

### EXPOSE

The **EXPOSE** instruction lets Docker know that when the image is executed, the port and protocol defined will be exposed at runtime. This instruction does not map the port to the host machine; instead, it opens the port to allow access to the service on the container network.

For example, in our Dockerfile, we are telling Docker to open port 80 every time the image runs:

```bash
    EXPOSE 80/tcp
```

### ENTRYPOINT

The benefit of using **ENTRYPOINT** over CMD is that you can use them in conjunction with each other. ENTRYPOINT can be used by itself but remember that you would only want to use ENTRYPOINT by itself if you wanted your container to be executable.

For reference, if you think of some of the CLI commands you might use, you must specify more than just the CLI command. You might have to add extra parameters that you want the command to interpret. This would be the use case for using ENTRYPOINT only.

For example, if you want to have a default command that you want to execute inside a container, you could do something similar to the following example. Be sure to use a command that keeps the container alive.

Here, we are using the following:

```BASH
    ENTRYPOINT ["nginx"]
    CMD ["-g", "daemon off;"]
```

What this means is that whenever we launch a container from our image, the NGINX binary is executed, which, as we have defined, is our entry point. Then, whatever we have as CMD is executed, giving us the equivalent of running the following command:

```bash
    nginx -g daemon off;
```

Another example of how ENTRYPOINT can be used is as follows:

```bash
    docker container run --name nginx-version dockerfile-example -v
```

This would be the equivalent of running the following command on our host:

```bash
    nginx -v
```

Notice that we didn’t have to tell Docker to use NGINX. Since we have the NGINX binary as our entry point, any command we pass overrides the CMD instruction that has been defined in the Dockerfile.

This would display the version of NGINX we have installed and our container would stop, as the NGINX binary would only be executed to display the version information. We will look at this once we have built and launched a container using our image. Before we move on, we should look at some of the instructions that are not included in our Dockerfile.

## Other Dockerfile instructions

There are some instructions that we have not included in our example Dockerfile. Let’s take a look at them here:

### USER

The **USER** instruction lets you specify the username to be used when a command is run. The **USER** instruction can be used on the **RUN** instruction, the **CMD** instruction, or the **ENTRYPOINT** instruction in the Dockerfile. Also, the
user defined in the **USER** instruction must exist, or your image will fail to build.

Using the **USER** instruction can also introduce permission issues, not only on the container itself, but also if you mount volumes.

### WORKDIR

The **WORKDIR** instruction sets the working directory for the same set of instructions that the **USER** instruction can use (**RUN**, **CMD**, and **ENTRYPOINT**). It will allow you to use the CMD and ADD instructions as well.

### ONBUILD

The **ONBUILD** instruction lets you stash a set of commands to be used when the image is used in the future, as a base image for another container image.

For example, if you want to give an image to developers and they all have a different code base that they want to test, you can use the **ONBUILD** instruction to lay the groundwork ahead of the fact of needing the actual code. Then, the developers will simply add their code to the directory you ask them to, and when they run a new Docker build command, it will add their code to the running image.

The **ONBUILD** instruction can be used in conjunction with the **ADD** and **RUN** instructions, such as in the following example:

```bash
    ONBUILD RUN apk update && apk upgrade && rm -rf /var/cache/apk/*
```

This would run an update and package upgrade every time our image is used as a base for another container image.

### ENV

The **ENV** instruction sets **ENV**s within the image both when it is built and when it is executed. These variables can be overridden when you launch your image.

## Dockerfiles – best practices

Now that we have covered Dockerfile instructions, let’s take a look at a few tips that are considered best practices for writing our own Dockerfiles. Following these will ensure that your images are lean, consistent, and easy for others to follow:

* You should try to get into the habit of using a ``.dockerignore`` file. We will cover the ``.dockerignore`` file in the Building Docker images section of this chapter; it will seem very familiar if you are used to using a ``.gitignore`` file. It will essentially ignore the items you specified in the file during the build process.
* Remember to only have one Dockerfile per folder to help you organize your containers.
* Use a version control system, such as Git, for your Dockerfile; just like any other text-based document, version control will help you move not only forward, but also backward, as necessary.
* Minimize the number of packages you need to install per image. One of the biggest goals you want to achieve while building your images is to keep them as small and secure as possible. Not installing unnecessary packages will greatly help in achieving this goal.
* Make sure there is only one application process per container. Every time you need a new application process, it is good practice to use a new container to run that application in.
* Keep things simple; over-complicating your Dockerfile will add bloat and potentially cause you issues down the line.
* Learn by example! Docker themselves have quite a detailed style guide for publishing the official images they host on Docker Hub.

## Building Docker images

It’s time for us to build the base upon which we will start building our future images. We will be looking at different ways to accomplish this goal. Consider this as a template that you may have created earlier with virtual machines. This will help save you time as this will complete the hard work for you; you will just have to create the application that needs to be added to the new images.

There are a lot of switches that you can use while using the docker build command. Let's look at the help.

```bash
    docker image build --help
```

There are a lot of different flags listed that you can pass when building your image. Now, it may seem like a lot to digest, but out of all these options, we only need to use --tag, or its shorthand, -t, to name our image.

You can use the other options to limit how much CPU and memory the build process will use. In some cases, you may not want the build command to take as much CPU or memory as it can have. The process may run a little slower, but if you are running it on your local machine or a production server and it’s a long build process, you may want to set a limit. There are also options that affect the network configuration of the container that was launched to build our image.

Typically, you don’t use the ``--file`` or ``-f`` switch since you run the docker build command from the same folder that the Dockerfile is in. Keeping the Dockerfile in separate folders helps sort the files and keeps the naming convention of the files the same.

It’s also worth mentioning that, while you are able to pass additional **ENV**s as arguments at build time, they are used at build time and your container image does not inherit them. This is useful for passing information such as proxy settings, which may only be applicable to your initial build/test environment.

The ``.dockerignore`` file, as we discussed earlier, is used to exclude those files or folders we don’t want to be included in the Docker build since, by default, all the files in the same folder as the Dockerfile will be uploaded. We also discussed placing the Dockerfile in a separate folder, and the same applies to ``.dockerignore``. It should go in the folder
where the Dockerfile was placed.

Keeping all the items you want to use in an image in the same folder will help you keep the number of items, if any, in the ``.dockerignore`` file to a minimum. Let’s start building images using the example file we have covered here.

## Using a Dockerfile

The first method that we are going to look at for building our base container images is creating a **Dockerfile**. In fact, we will be using the Dockerfile from the previous section and then executing a docker image build command against it to get ourselves an NGINX image.

Don’t forget that you will also need the ``default.conf``, ``html.tar.gz``, and ``nginx.conf`` files in the files folder. 

So, there are two ways we can go about building our image. The first way would be by specifying the --file switch when we use the docker image build command. We will also utilize the --tag switch to give the new image a unique name:

```bash
    docker image build --file <path_to_Dockerfile> --tag <REPOSITORY>:<TAG> .
```

Now, ``<REPOSITORY>`` is typically the username you sign up for on Docker Hub. For now, we will be using local. ``<TAG>`` is a unique value that allows you to identify a container. Typically, this will be a version number or an other descriptor.

As we have a file called **Dockerfile**, we can also skip using the ``--file`` switch. This is the second way of building an image. The following is the code for this:

```bash
    docker image build --tag local:dockerfile-example .
 ```

 **Note:** The most important thing to remember is the dot (or period) at the very end. This is to tell the docker image build command to build in the current folder.

 Once it’s built, you should be able to run the following command to check whether the image is available, as well as the size of your image:

```bash
    docker image ls d*
```

## Using an existing container

The easiest way to build a base image is to start off by using one of the official images from Docker Hub. Docker also keeps the Dockerfile for these official builds in their GitHub repositories. So, there are at least two choices you have for using existing images that others have already created. By using the Dockerfile, you can see exactly what is included in the build and add what you need. You can then version control that Dockerfile if you want to change or share it later.
