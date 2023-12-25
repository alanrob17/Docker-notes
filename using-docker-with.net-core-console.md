# Using Docker to run .Net Core console programs

[Containerize a .NET app](https://learn.microsoft.com/en-us/dotnet/core/docker/build-container?tabs=linux&pivots=dotnet-8-0)

Source: **Docker\console**

We will containerize a .NET application with Docker. Containers have many features and benefits, such as being an immutable infrastructure, providing a portable architecture, and enabling scalability. The image can be used to create containers for your local development environment, private cloud, or public cloud.

Features

* Create and publish a simple .NET app
* Create and configure a Dockerfile for .NET
* Build a Docker image
* Create and run a Docker container

You'll understand the Docker container build and deploy tasks for a .NET application. The Docker platform uses the Docker engine to quickly build and package apps as Docker images. These images are written in the Dockerfile format to be deployed and run in a layered container.

Build a new .Net console application.

```bash
    dotnet new console -o App -n DotNet.Docker
```

Structure

```bash
    console
        └──📂 App
            ├──DotNet.Docker.csproj
            ├──Program.cs
            └──📂 obj
                ├── DotNet.Docker.csproj.nuget.dgspec.json
                ├── DotNet.Docker.csproj.nuget.g.props
                ├── DotNet.Docker.csproj.nuget.g.targets
                ├── project.assets.json
                └── project.nuget.cache
```

The default template creates an app that prints to the terminal and then immediately terminates. I am using an app that loops indefinitely.

```bash
    var counter = 0;
    var max = args.Length is not 0 ? Convert.ToInt32(args[0]) : -1;
    while (max is -1 || counter < max)
    {
        Console.WriteLine($"Counter: {++counter}");
        await Task.Delay(TimeSpan.FromMilliseconds(1_000));
    }
```

This loops indefinitely.

```bash
    dotnet run -- 5
```

An argument of 5 will loop 5 times.

## Publish the .NET app

Before adding the .NET app to the Docker image, first it must be published. It's best to have the container run the published version of the app. To publish the app, run the following command:

```bash
    dotnet publish -c Release
```

**Note:** I had an issue with this. In the folder there is a ``.csproj`` file and a ``.sln`` file. The publish command told me I had two projects so couldn't run. I renamed the ``.sln`` to ``.sln.old`` and ran the command again and it worked.

Use the **ls** command to check that this has worked.

```bash
    ls bin/Release/net8.0/publish
```

You should see.

> DotNet.Docker.deps.json  DotNet.Docker.dll*DotNet.Docker.exe*  DotNet.Docker.pdb  DotNet.Docker.runtimeconfig.json

## Create the Dockerfile

The Dockerfile file is used by the docker **build** command to create a container image. This file is a text file named Dockerfile that doesn't have an extension.

Create a file named **Dockerfile** in the directory containing the ``.csproj`` and open it in a text editor. This tutorial uses the ASP.NET Core runtime image (which contains the .NET runtime image) and corresponds with the .NET console application.

```bash
    FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build-env
    WORKDIR /App

    # Copy everything
    COPY . ./
    # Restore as distinct layers
    RUN dotnet restore
    # Build and publish a release
    RUN dotnet publish -c Release -o out

    # Build runtime image
    FROM mcr.microsoft.com/dotnet/aspnet:8.0
    WORKDIR /App
    COPY --from=build-env /App/out .
    ENV DOTNET_EnableDiagnostics=0
    ENTRYPOINT ["dotnet", "DotNet.Docker.dll"]
```

**Note:** The ASP.NET Core runtime image is used intentionally here, although the ``mcr.microsoft.com/dotnet/runtime:8.0`` image could have been used.

**Tip:** This Dockerfile uses multi-stage builds, which optimizes the final size of the image by layering the build and leaving only required artifacts. For more information, see Docker Docs: [multi-stage builds](https://docs.docker.com/build/building/multi-stage/).

The **FROM** keyword requires a fully qualified Docker container image name. The Microsoft Container Registry (MCR, mcr.microsoft.com) is a syndicate of Docker Hub, which hosts publicly accessible containers. The dotnet segment is the container repository, whereas the sdk or aspnet segment is the container image name. The image is tagged with 8.0, which is used for versioning. Thus, ``mcr.microsoft.com/dotnet/aspnet:8.0`` is the .NET 8.0 runtime. Make sure that you pull the runtime version that matches the runtime targeted by your SDK. For example, the app created in the previous section used the .NET 8.0 SDK, and the base image referred to in the Dockerfile is tagged with 8.0.

Save the Dockerfile file. The directory structure of the working folder should look like the following. Some of the deeper-level files and folders have been omitted to save space:

```bash
    console
        └──📂 App
            ├── Dockerfile
            ├── DotNet.Docker.csproj
            ├── Program.cs
            ├──📂 bin
            │   └──📂 Release
            │       └──📂 net8.0
            │           └──📂 publish
            │               ├── DotNet.Docker.deps.json
            │               ├── DotNet.Docker.exe
            │               ├── DotNet.Docker.dll
            │               ├── DotNet.Docker.pdb
            │               └── DotNet.Docker.runtimeconfig.json
            └──📁 obj
                └──...
```

Run the following command:

```bash
    docker build -t counter-image -f Dockerfile .
```

Docker will process each line in the Dockerfile. The . in the docker build command sets the build context of the image. The ``-f`` switch is the path to the Dockerfile. This command builds the image and creates a local repository named **counter-image** that points to that image. After this command finishes, run ``docker images`` to see a list of images installed:

> REPOSITORY          TAG       IMAGE ID       CREATED         SIZE
> counter-image       latest    3e95851708f9   4 hours ago     218MB

The final steps of the *Dockerfile* are to create a container from the image and run the app, copy the published app to the container, and define the entry point.

```bash
    FROM mcr.microsoft.com/dotnet/aspnet:8.0
    WORKDIR /App
    COPY --from=build-env /App/out .
    ENV DOTNET_EnableDiagnostics=0
    ENTRYPOINT ["dotnet", "DotNet.Docker.dll"]
```

The **COPY** command tells Docker to copy the specified folder on your computer to a folder in the container. In this example, the publish folder is copied to a folder named ``App/out`` in the container.

The **WORKDIR** command changes the current directory inside of the container to App.

The next command, **ENTRYPOINT**, tells Docker to configure the container to run as an executable. When the container starts, the **ENTRYPOINT** command runs. When this command ends, the container will automatically stop.

**Tip:** Before .NET 8, containers configured to run as read-only may fail with Failed to create CoreCLR, HRESULT: 0x8007000E. To address this issue, specify a **DOTNET_EnableDiagnostics** environment variable as 0 (just before the ENTRYPOINT step):

```bash
    ENV DOTNET_EnableDiagnostics=0
```

## Create a container

Now that you have an image that contains your app, you can create a container. You can create a container in two ways. First, create a new container that is stopped.

```bash
    docker create --name core-counter counter-image
```

This docker create command creates a container based on the **counter-image** image. The output of that command shows you the **CONTAINER ID** of the created container:

> fba5e27135cc937b7ff8a60da354c69830950d2328d7dea76b1f883e9034b013

```bash
    docker ps -a
```

> CONTAINER ID   IMAGE           COMMAND                  CREATED        STATUS                    PORTS    NAMES
> fba5e27135cc   counter-image   "dotnet DotNet.Docke…"   12 hours ago   Created

## Manage the container

The container was created with a specific name **core-counter**, this name is used to manage the container. The following example uses the docker start command to start the container, and then uses the ``docker ps`` command to only show containers that are running:

```bash
    docker start core-counter
```

Similarly, the ``docker stop`` command stops the container. The following example uses the docker stop command to stop the container, and then uses the ``docker ps`` command to show that no containers are running:

```bash
    docker stop core-counter
```

## Connect to a container

After a container is running, you can connect to it to see the output. Use the ``docker start`` and ``docker attach`` commands to start the container and peek at the output stream. In this example, the **Ctrl+C** keystroke is used to detach from the running container. This keystroke ends the process in the container unless otherwise specified, which would stop the container. The --sig-proxy=false parameter ensures that **Ctrl+C** won't stop the process in the container.

```bash
    docker attach core-counter
```

Output:

> ...
> Counter: 207
> Counter: 208
> ...

After you detach (**Ctrl+C**) from the container, reattach to verify that it's still running and counting.

## Delete a container

You don't want containers hanging around that don't do anything. Delete the container you previously created. If the container is running, stop it.

```bash
    docker stop core-counter
```

Then delete it.

```bash
    docker rm core-counter
```

## Single run

Docker provides the ``docker run`` command to create and run the container as a single command. This command eliminates the need to run ``docker create`` and then ``docker start``. You can also set this command to automatically delete the container when the container stops. For example, use ``docker run -it --rm`` to do two things, first, automatically use the current terminal to connect to the container, and then when the container finishes, remove it:

```bash
    docker run -it --rm counter-image
```

Output:

> Counter: 1
> Counter: 2
> Counter: 3
> ...

**Note:** if we didn't have the infinite loop the container would end after it completed running.

The container also passes parameters into the execution of the .NET app. To instruct the .NET app to count only to three, pass in 3.

```bash
    docker run -it --rm counter-image 3
```

Output:

> Counter: 1
> Counter: 2
> Counter: 3

Since the ``--rm`` parameter was provided, the container is automatically deleted when the process is stopped.

## Change the ENTRYPOINT

The docker run command also lets you modify the **ENTRYPOINT** command from the Dockerfile and run something else, but only for that container. For example, use the following command to run **bash** or *cmd.exe*. Edit the command as necessary.

```bash
    docker run -it --rm --entrypoint "bash" counter-image
```

Then, inside the container.

```bash
    dotnet DotNet.Docker.dll 7
```

Output:

> root@10b7ef2809ed:/App# dotnet DotNet.Docker.dll 7
> Counter: 1
> Counter: 2
> Counter: 3
> Counter: 4
> Counter: 5
> Counter: 6
> Counter: 7
> root@10b7ef2809ed:/App#

You can also do this to get into the container.

> PS D:\Docker\console> dc ps
> CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS     NAMES
> 20bdc043e938   counter-image   "dotnet DotNet.Docke…"   43 seconds ago   Up 43 seconds             eloquent_mcclintock

```bash
    docker exec -it 20bdc043e938 "bash"
```

> root@20bdc043e938:/App# ls
> DotNet.Docker  DotNet.Docker.deps.json  DotNet.Docker.dll  DotNet.Docker.pdb  DotNet.Docker.runtimeconfig.json
>
> root@20bdc043e938:/App# dotnet DotNet.Docker.dll 5
> Counter: 1
> Counter: 2
> Counter: 3
> Counter: 4
> Counter: 5

## Clean up resources

You have created containers and images. If you want, delete these resources. Use the following commands to

List all containers.

```bash
    docker ps -a
```

Stop containers that are running by their name.

```bash
    docker rm core-counter
```

Next, delete any images that you no longer want on your machine. Delete the image created by your Dockerfile and then delete the .NET image the Dockerfile was based on. You can use the **IMAGE ID** or the **REPOSITORY:TAG** formatted string.

```bash
    docker rmi counter-image:latest
    docker rmi mcr.microsoft.com/dotnet/aspnet:8.0
```

**Note:** I'll delete the **counter-image** but I won't delete **aspnet:8.0** as I am constantly using it.

## Next steps

[Containerize a .NET app with dotnet publish](https://learn.microsoft.com/en-us/dotnet/core/docker/publish-as-container)
[.NET container images](https://learn.microsoft.com/en-us/dotnet/core/docker/container-images)
[Containerize an ASP.NET Core application](https://learn.microsoft.com/en-us/aspnet/core/host-and-deploy/docker/building-net-docker-images)
[Azure services that support containers](https://azure.microsoft.com/overview/containers/)
[Dockerfile commands](https://docs.docker.com/engine/reference/builder/)
[Container Tools for Visual Studio](https://learn.microsoft.com/en-us/visualstudio/containers/overview)
