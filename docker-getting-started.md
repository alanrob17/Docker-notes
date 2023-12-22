# Docker getting started tutorial

<https://docs.docker.com/get-started/overview/>

## A simple To-Do List running in Docker

### Working folder

* D:\Docker\getting-started

Create a folder called *getting-started-app*

In this folder:

### Get the app

```bash
    git clone https://github.com/docker/getting-started-app.git
```

### Create a dockerfile

```bash
    FROM node:18-alpine
    WORKDIR /app
    COPY . .
    RUN yarn install --production
    CMD ["node", "src/index.js"]
    EXPOSE 3000
```

### Build the app's image

```bash
    docker build -t getting-started .
```

### Start an app container

```bash
    docker run -dp 127.0.0.1:3000:3000 getting-started
```

In your browser,

```bash
    https://localhost:3000
```

This will run the To-Do application.

### Update the application

You can update the application.

#### Update the source code

In the following steps, you'll change the "empty text" when you don't have any todo list items to "You have no todo items yet! Add one above!"

In the src/static/js/app.js file, update line 56 to change the text.

```js
- <p className="text-center">No items yet! Add one above!</p>
+ <p className="text-center">You have no todo items yet! Add one above!</p>
```

Build your updated version of the image, using the docker build command.

```bash
    docker build -t getting-started .
```

Start a new container using the updated code.

```bash
    docker run -dp 127.0.0.1:3000:3000 getting-started
```

You will get an error because your container is still running.

do the following command.

```bash
    docker ps
```

> CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                      NAMES
> 9458c0e26cd0   b6e9127c39ed   "docker-entrypoint.s…"   23 minutes ago   Up 23 minutes   127.0.0.1:3000->3000/tcp   funny_colden

You can stop this container.

```bash
    docker stop 9458c0e26cd0
```

Now, remove the container.

```bash
    docker rm 9458c0e26cd0
```

You could have stopped and removed the container with one command.

```bash
    docker rm -f 9458c0e26cd0
```

Now, start your updated app using the docker run command.

```bash
    docker run -dp 127.0.0.1:3000:3000 getting-started
```

When you run the web app you will notice the updated prompt.
