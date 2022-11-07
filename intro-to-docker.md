# Tim Corey - Introduction to Docker

## Create an image

We want to create a web server to run some HTML code. A very small image can be created using ``httpd`` which will use a cutdown version of Alpine as the operating system. This will only be around 55 Mb in size.

Command to pull the image: **docker pull httpd:alpine** .

We won't use this but will use a build command to create the image.

We now create a folder to hold our web source code which consists of a simple HTML page named ``Index.html``.

Root folder ``IntroToDocker``, contains ``Dockerfile`` that is used to build our image.

Sub folder ``html``, contains an ``Index.html`` page.

### Dockerfile contains

```dockerfile
	FROM httpd:alpine
	COPY ./html/ /usr/local/apache2/htdocs/
```

Where **FROM** is the image to pull.
And we **COPY** our files from the ``./html/`` folder and load them into the Alpine image folder, ``/usr/local/apache2/htdocs/`` .

We are now able to user our **Dockerfile** to build an image.

```powershell
docker build -t hello-docker:1.0.0 .
```

Do a search for the image you created.

```powershell
	docker images h*
```

> REPOSITORY     TAG       IMAGE ID       CREATED         SIZE
> hello-docker   1.0.0     113447870bc6   3 minutes ago   55MB
> hellonode      latest    fc0b418d4dbd   11 months ago   113MB
> hello-world    latest    feb5d9fea6a5   13 months ago   13.3kB

We can do a history on the image using the **Image Id**.

```powershell
	docker image history 113
```

Is the complete setup of our image and includes the byte size of each part.

```text
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
113447870bc6   3 minutes ago   COPY ./html/ /usr/local/apache2/htdocs/ # bu…   349B      buildkit.dockerfile.v0
<missing>      4 weeks ago     /bin/sh -c #(nop)  CMD ["httpd-foreground"]     0B
<missing>      4 weeks ago     /bin/sh -c #(nop)  EXPOSE 80                    0B
<missing>      4 weeks ago     /bin/sh -c #(nop) COPY file:c432ff61c4993ecd…   138B
<missing>      4 weeks ago     /bin/sh -c #(nop)  STOPSIGNAL SIGWINCH          0B
<missing>      4 weeks ago     /bin/sh -c set -eux;   apk add --no-cache --…   12.7MB
<missing>      4 weeks ago     /bin/sh -c #(nop)  ENV HTTPD_PATCHES=           0B
<missing>      4 weeks ago     /bin/sh -c #(nop)  ENV HTTPD_SHA256=eb397fee…   0B
<missing>      4 weeks ago     /bin/sh -c #(nop)  ENV HTTPD_VERSION=2.4.54     0B
<missing>      4 weeks ago     /bin/sh -c set -eux;  apk add --no-cache   a…   36.7MB
<missing>      4 weeks ago     /bin/sh -c #(nop) WORKDIR /usr/local/apache2    0B
<missing>      4 weeks ago     /bin/sh -c mkdir -p "$HTTPD_PREFIX"  && chow…   0B
<missing>      4 weeks ago     /bin/sh -c #(nop)  ENV PATH=/usr/local/apach…   0B
<missing>      4 weeks ago     /bin/sh -c #(nop)  ENV HTTPD_PREFIX=/usr/loc…   0B
<missing>      4 weeks ago     /bin/sh -c set -x  && adduser -u 82 -D -S -G…   4.68kB
<missing>      2 months ago    /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>      2 months ago    /bin/sh -c #(nop) ADD file:2a949686d9886ac7c…   5.54MB
```

The bottom line is the operating system and this is the first thing that was created.

This line is the web server.

> /bin/sh -c set -eux;  apk add --no-cache   a…   36.7MB

This line (the latest line) is the code we copied into the image.

> COPY ./html/ /usr/local/apache2/htdocs/ # bu…   349B

## Create a container

We now create a container to run our image. The container is **read/write** so we can add content to it. If you stop running the container you will lose your changes because the container will be created from your existing image which is **read only**.

If you destroy the container it is gone but the image stays until you remove it.

```powershell
	docker run --name first-container -p 8080:80 hello-docker:1.0.0
```

We call our container ``first-container``.

We need a port to run our web server, ``-p 8080:80`` .

*8080* is a port on our system and *80* is used to connect to the image, ``hello-docker:1.0.0`` .

We usually only need the first 3 characters of the **Container Id** to identify it.

If we look at our Powershell output we see the line.

> 172.17.0.1 - - [07/Nov/2022:07:13:04 +0000] "GET / HTTP/1.1" 200 185

This is returning the ``Index.html`` page (return status 200) and it returned 185 bytes.

This is the container we are running with its image name and tag.

```text
CONTAINER ID   IMAGE ...
a014eaedaf47   hello-docker:1.0.0 
```

We can stop the container from running by.

```powershell
	docker stop a01
```

If we want to remove the container.

```powershell
	docker rm a01
```

Add a new file to our *html* folder named ``help.html``.

The original hello-docker:1.0.0 image won't have our changes because images never change so we will create a new image with our changes in it.

```powershell
	docker build -t hello-docker:1.0.1 .
```

To check our images

```powershell
	docker images hello-docker*
```
 
> REPOSITORY     TAG       IMAGE ID       CREATED              SIZE
> hello-docker   1.0.1     71a874cb898c   About a minute ago   55MB
> hello-docker   1.0.0     113447870bc6   About an hour ago    55MB

We increment the tag and we can run it with.

```powershell
	docker run --name second-container -p 8080:80 hello-docker:1.0.1
```

We now have our new website running.

Note: we can't name two containers with the same name. This would cause an error.

We now have two images with both containing different code.

```powershell
	docker images hello-docker*
```

> REPOSITORY     TAG       IMAGE ID       CREATED       SIZE
> hello-docker   1.0.1     71a874cb898c   2 hours ago   55MB
> hello-docker   1.0.0     113447870bc6   3 hours ago   55MB

Shows that I have two images. I know know that ``hello-docker:1.0.1`` has my full web source so I can delete ``hello-docker:1.0.0``.

```powershell
	docker rmi 113
```

Where ``rmi`` removes an image.

## Benefits of Docker

You can build and tear down images and containers very quickly and easily.

Once you finish testing you can deploy your code and remove your images.

Docker saves you from having to install a lot of software on your system. You can create an image and use that.

A Docker image will never change so once you get your code working it will work forever on any system or cloud that runs Docker images.

### Remove all containers

```powershell
	docker container prune
```
