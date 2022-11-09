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
