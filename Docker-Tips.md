# Docker tips

## Running an image in interactive mode

You can inspect the contents of an image.

```bash
    docker run -it --entrypoint sh your_image_name
```

This will create a container and allow you to Docker Exec into it.

This is really handy to see if your container has the files that you are supposed to have in it. For example, in a node application I had to go into the ``node_modules`` folder to see if Express had been installed.

Another example

I have an image named, ``record-db-mssql-sc-recorddb``.

I want to inspect it to see if my database has been loaded into it.

```bash
    docker run -it --entrypoint sh record-db-mssql-sc-recorddb
```

This will exec into a container so that you can examine it.

```bash
    cd /var/opt/mssql/data
```

Now ``ls`` in this folder.

> Entropy.bin   mastlog.ldf         model_replicatedmaster.ldf  msdblog.ldf  tempdb4.ndf  tempdb8.ndf       
> RecordDB.ldf  model.mdf           model_replicatedmaster.mdf  tempdb.mdf   tempdb5.ndf  templog.ldf       
> RecordDB.mdf  model_msdbdata.mdf  modellog.ldf                tempdb2.ndf  tempdb6.ndf        
> master.mdf    model_msdblog.ldf   msdbdata.mdf                tempdb3.ndf  tempdb7.ndf

By doing this I can see that my RecordDB.mdf database has been installed.

I can see the running container with.

```bash
    docker ps
```

Lists.

> 32847a52c400   record-db-mssql-sc-recorddb   "sh"      15 minutes ago   Up 15 minutes   1433/tcp   tender_edison

## Checking the contents of an image from your Dockerfile

This is handy for debugging a Dockerfile.

I created a Node.js image from a Dockerfile and found that Express wouldn't run my web application.

In your Dockerfile you can list the contents of a folder with the ``RUN`` command.

```bash
    FROM node:alpine

    WORKDIR /usr/src/app

    # Copy package.json and package-lock.json first
    COPY package*.json ./

    # Install dependencies
    RUN npm ci --omit=dev

    # Check if express is installed correctly
    RUN ls node_modules && ls node_modules/express    
```

The last ``RUN`` **ls** command will list the ``node_modules/express`` folder to see if Express has been installed.
