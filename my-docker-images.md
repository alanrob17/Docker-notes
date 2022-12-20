# Docker images

These are notes on the Docker images that I have created. I am keeping this list because I often forget how my images run.

**Note:** I am running Docker from Powershell and WSL Ubuntu. I use a permanent alias for Docker named ``dc``.

## Basic images

### hello-world

This is the basic image showing that Docker is working correctly. It prints a simple message and then shuts down.

```bash
    docker run -it hello-world
```

### ubuntu

I can run a Bash shell in Powershell using this image.

```bash
    docker run -it ubuntu bash
```

Where ``-it`` is run a terminal interface. We want to run the ``ubuntu:latest`` image and we want to run the ``bash`` shell in the container.

### docker/getting-started

These are the Docker getting started notes, a great introduction on how to use Docker.

```bash
    docker run -d -p 8080:80 docker/getting-started
```

### alpine

Alpine is a minimal Linux distribution that take about 5 Mb of disk space. It contains Busybox, an application that contains the core Unix utilities in a single binary. These include applications such as ``vi``, a basic text editor and touch to allow you to create a text file.

```bash
    docker run -it alpine
```

You can install software into this container.

```bash
    apk add nano
```

or you can use a ``dockerfile`` to install software into your image (\Sandbox\docker-alpine-git).

```dockerfile
    FROM alpine

    RUN apk update

    RUN apk add git 

    RUN git config --global user.email "alanr@live.com.au"
    RUN git config --global user.name "Alan Robson"
    RUN git config --global init.defaultBranch main

    RUN apk add nano
```

Build

```bash
    docker build -t alpine-git .
```

Run 

```bash
     docker run -it alpine-git
```

### Create a web server

Folder: Sandbox\docker\alpine-website

```dockerfile
FROM httpd:alpine

COPY ./html/ /usr/local/apache2/htdocs
```

Build

```bash
     docker build -t alpine-website:1.0.0 .
```

Run

```bash
    docker run --name alpineweb -p 8000:80 alpine-website:1.0.0     
```

## SQL Server images

I have a basic SQL Server image that I will use for all other SQL Server images. This is just a bare bones SQL Server with no databases. You can add a database into the container but it will only be saved in that container, not the image.

```bash
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Pwd12345!" -p 11433:1433 -d mcr.microsoft.com/mssql/server:2019-latest
```

**Notes:**

* You have to accept the license agreement in the EULA as an environment variable, ``-e "ACCEPT_EULA=Y"`` .
* You have to add a password as another environment variable, ``-e "MSSQL_SA_PASSWORD=Pwd12345!"`` .
* You can't use the standard SQL Server port if you already have SQL Server working as a service on your system. In this case modify the **external port** to ``11433`` (or any other valid port number) and the **internal port** in the container has to be ``1433``.

Once you create the image and run the container you can access SQL Server with MSSQL.

**Server:** localhost, 11433
**Username:** sa
**Password:** Pwd12345!

**Note:** localhost uses a comma (not a colon) in this situation.

The following images are made from the original image above and doesn't have to reference the EULA or the password.

### RecordDB

```bash
    docker run -it -d -p 11433:1433 record-db
```

Is built from the dockerfile in ``Projects\docker-recorddb-mssql``. This folder contains the RecordDB database backup and the script to restore the database.

**Note:** If you add a new database backup into this folder you can delete the original image and create a new image with the updates.

### Adventureworks

Is a script built from a dockerfile in ``\Sandbox\docker-adventureworkslt-mssql``. It has a script that restores the backup and also sanitises sensitive customer data so that other users of the database don't see the real data.

```bash
docker run -it -d -p 11433:1433 adventureworks
```
