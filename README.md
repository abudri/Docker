# Docker

Notes mainly from the amazingly well put together App Academy Open [Docker Curriculum](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/)

## Table of Contents

* [Basic Docker Commands](https://github.com/abudri/Docker/blob/main/README.md#basic-docker-commands)
* [`Dockerfile` Basics](https://github.com/abudri/Docker/blob/main/README.md#dockerfile-basics)
* [`Dockerfile` Commands](https://github.com/abudri/Docker/blob/main/README.md#dockerfile-commands)
* [Nuances & Exta Notes](https://github.com/abudri/Docker/blob/main/README.md#nuances--extra-notes)
* [Best and Standard Practices](https://github.com/abudri/Docker/blob/main/README.md#best-and-standard-practices)


## Basic Docker Commands

#### `image` Commands

`docker pull <IMAGENAME>` - You can pull down an image without running a container by using this command.
    
`docker image ls` - list your docker images

`docker image rm <image_name>` - removes a docker image.

`docker build .` - this builds a docker image from the existing `Docker` file in the _current_ directory - `.`.  However, keep in mind that this will not tag your image, and hence `REPOSITORY` and `TAG` will both be empty when you do a `docker image ls` after building the image:

    z@Mac-Users-Apple-Computer phase1 % docker image ls
    REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
    <none>                   <none>              5590e09e8948        4 seconds ago       109MB

`docker build -t <username/imagename>` - Now make sure you properly tag your image - you can do this easily using the `-t` flag in shown in this command. An example is: `docker build . -t day_2_project` - this actually builds the image of the `Dockerfile` with a tag using the `-t` flag.  Now when you run the command `docker image ls` after building the image,`REPOSITORY` and `TAG` will have `day_2_project` and `latest` - the default tag.

    z@Mac-Users-Apple-Computer phase1 % docker image ls                
    REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
    day_2_project            latest              5590e09e8948        54 seconds ago      109MB
    
`docker image inspect day_2_project` - this will give you all kinds of data related to your image, such as the default port the nginx image exposes. See the below:

    "Config": {
                "Hostname": "",
                "Domainname": "",
                "User": "",
                "AttachStdin": false,
                "AttachStdout": false,
                "AttachStderr": false,
                "ExposedPorts": {
                    "80/tcp": {}
                },

`docker image tag <originalImage> <USERNAME>/<originalImage` - command to re-tag pre existing images (even ones that you didn't make). Then you can use `docker image push <IMAGENAME>`. [See Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/docker-hub---pushing-and-pulling-images)
    
#### `container` Commands

Basic Example: 

`docker container run nginx` - When running a container you always have to pass an image like this command, which passes the image called `nginx`. This command will first search for that image in the local image cache and it doesn't find it, Docker Hub will be searched. You can additionally pull down an image without running a container by using the `docker pull <IMAGENAME>` command.

`docker container run -d -p 80:80 day_2_project` - this builds and runs your container using the `day_2_project` image, and also runs it in detached mode with the `-d` flag so that you get your terminal prompt back, and using `-p` to publish the port.  When we were doing this command, the first port is the `<local_port>`, and the second port is the container port, `<container_port>`, or more verbosely,  `docker container run -d -p <local_port>:<container_port>`.

After running the above command, you would be able to visit on local machine - such as your laptop - http://localhost:80 and see a web page if `index.html` was the file you chose to load in your app.

`docker container ls` - lists your docker containers that are running.

`docker container ls -a` - also lists your containers, including those that are NOT running.

`docker container stop <container_name>` - Stops your docker container.

`docker container start <container_name>` - Starts your docker container.

`docker container rm <container_name>` - will remove your container only if it is _not running_.  You will need to stop it first in order for this to take.

`docker container rm -f <container_name>` - will remove your container even it is currently running.

### `push`ing a Docker image to Docker Hub

`docker login` - this is your username and password for Docker Hub. Do this if necessary.

Tag your image first:

`docker build . -t abdullahb/bunk_node_image` - and so now we have the `REPOSITORY` of `abdullahb`, my username, and then we called the image `bunk_node_image` (wasn't working exactly at this time.

`docker push abdullahb/bunk_node_image` - push your image to docker hub. Now in Docker Hub you should have a repository called `bunk_node_image`, and under **Tags and Scans** you should see the `latest` tag. 

## `Dockerfile` Basics

### A Basic `Dockerfile`

One of the common best practices you'll see in Dockerfiles are chained commands using `\` and `&&`. Remember that each instruction in a Dockerfile will become a layer so chaining commands allows you to fit more into each layer saving on space. You'll see an example of this below:

```docker
    FROM node:argon
    # Create app directory
    RUN mkdir -p /usr/src/app \

    # These two stanzas are part of the same created RUN layer
    # you can chain commands like this using '\' and '&&'
        && echo "This is part of the same layer" \
        && echo "and so is this one!"

    # change our current working directory
    WORKDIR /usr/src/app

    # Install app dependencies
    COPY package.json /usr/src/app/
    RUN npm install

    # Bundle app source
    COPY . /usr/src/app

    # expose a port
    EXPOSE 8080

    #  This is the command that will be run
    # each time a container is built using this image
    CMD [ "npm", "start" ]
```

Good [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/images-and-the-dockerfile) for the above.

Shown below, when Docker builds the image from the above Dockerfile, each step corresponds to a command run in the Dockerfile. And each layer is made up of the files generated from running that command. Along with each step, the layer created is listed represented by its randomly generated ID. For example, the `layer ID` for step 1 is `530c750a346e`.

```docker
$ docker build -t expressapp .
Step 1 : FROM node:argon
argon: Pulling from library/node...
...
Status: Downloaded newer image for node:argon
 ---> 530c750a346e
Step 2 : RUN mkdir -p /usr/src/app     && echo "This is part of the same layer"     && echo "and so is this one!"
 ---> Running in 5090fde23e44
This is part of the same layer
and so is this one!
 ---> 7184cc184ef8
Removing intermediate container 5090fde23e44
Step 3 : WORKDIR /usr/src/app
 ---> Running in 2987746b5fba
 ---> 86c81d89b023
Removing intermediate container 2987746b5fba
Step 4 : COPY package.json /usr/src/app/
 ---> 334d93a151ee
Removing intermediate container a678c817e467
Step 5 : RUN npm install
 ---> Running in 31ee9721cccb
 ---> ecf7275feff3
Removing intermediate container 31ee9721cccb
Step 6 : COPY . /usr/src/app
 ---> 995a21532fce
Removing intermediate container a3b7591bf46d
Step 7 : EXPOSE 8080
 ---> Running in fddb8afb98d7
 ---> e9539311a23e
Removing intermediate container fddb8afb98d7
Step 8 : CMD npm start
 ---> Running in a262fd016da6
 ---> fdd93d9c2c60
Removing intermediate container a262fd016da6
Successfully built fdd93d9c2c60
```

### Rebuilding a Dockerfile

Whenever possible, Docker will re-use the intermediate images (cache), to accelerate the docker build process. So if nothing in the `Dockerfile` changed from the last time you built it, you will see the message `Using Cache` when building.

```docker
$ docker build -t expressapp .
Sending build context to Docker daemon 15.36 kB
Step 1 : FROM node:argon
 ---> 530c750a346e
Step 2 : RUN mkdir -p /usr/src/app
 ---> Using cache
 ---> 7184cc184ef8
```

Be wary though! If you make a change in a Dockerfile once it's been built, Docker will see that change and then will **no longer use the cache** of image layers it had. That's why it is always a good idea to put any layers that you intend to change as low in the Dockerfile as possible. That way the beginning of your Dockerfile will be able to use the cache until it hits your changes.

### Logging in Dockerfiles

Docker can handle all of our logging for us while building our images - we just need to tell it where to put those logs. You can use `stdout` and `stderr` by adding something like the following `RUN` command to your Dockerfile:

```
# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/IMAGENAME/access.log \
	&& ln -sf /dev/stderr /var/log/IMAGENNAME/error.log
```

### `.dockerignore` File

One of the first things you should do when you write a `Dockerfile` is to write a `.dockerignore`. Sounds familiar right? A .dockerignore file ignores the files you don't want to have in your Docker image. It's just like a `.gitignore` - and you can ignore the same sorts of things. For example your `.dockerignore` for this setup should be ignoring:

```docker
.git/
node_modules/
dist/
```

Nice! Looking more efficient already without all those node_modules taking up space in your image. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfiles-galore)

## `Dockerfile` Commands

**Common `Dockerfile` Commands - [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfile-cheat-sheet) / [Also](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact)**

**`FROM` -** Almost every Dockerfile begins with the `FROM` instruction that will set the Base Image for all subsequent instructions. The only argument that can come before `FROM` is [`ARG`](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact) which can be used to declare a variable outside of the build stage, that can be later be used inside the build stage.

**`ENV` -** This command is the preferred way to inject keys and values into image building. All the variables set using this command can be used in the subsequent instructions in the build stage.

**`RUN`-** The `RUN` instruction will execute each phrase of commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile. Additionally the `RUN` command can run shell scripts or whatever you could use within a container.

**`WORKDIR` -** The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile. It is best practice to use `WORKDIR` to run commands that rely on being in a certain location in the file tree. The `WORKDIR` instruction can be also be used multiple times in a Dockerfile. If a relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction.

**`COPY` -** Will make a copy of the files in the first given location to the second given location.

**`EXPOSE` -** Specify to the image which ports are going to be exposed within that image.

**`CMD` -** This is the final command that will run every time you launch a new container from this image or restart a stopped container of this image.

## Nuances & Extra Notes

#### `RUN` vs `CMD`

`RUN` and `CMD` are both Dockerfile instructions. `RUN` lets you execute commands inside of your Docker image. These commands get executed once at build time and get written into your Docker image as a new layer. ... `CMD` lets you define a default command to run when your container starts. [Reference](https://nickjanetakis.com/blog/docker-tip-7-the-difference-between-run-and-cmd#:~:text=RUN%20and%20CMD%20are%20both,image%20as%20a%20new%20layer.&text=CMD%20lets%20you%20define%20a,run%20when%20your%20container%20starts.)

#### `RUN cd` vs `WORKDIR`

Each `RUN` command runs in a new shell and a new environment (and technically a new container, though you won't usually notice this). The `ENV` and `WORKDIR` directives before it affect how it starts up. If you have a `RUN` step that just changes directories, that will get lost when the shell exits, and the next step will start in the most recent `WORKDIR` of the image. [Stack Overflow Reference](https://stackoverflow.com/questions/58847410/difference-between-run-cd-and-workdir-in-dockerfile#:~:text=1%20Answer&text=RUN%20cd%20%2F%20does%20absolutely%20nothing,'t%20usually%20notice%20this).

#### `ADD` vs `COPY`

**`ADD`** - It includes the source you want to copy (<src>) followed by the destination where you want to store it (<dest>). If the source is a directory, `ADD` copies everything inside of it (including file system metadata). `ADD` can also copy files from a URL. It can download an external file and copy it to the wanted destination. For example:

```docker
ADD http://source.file/url  /destination/path
```

An additional feature is that it copies compressed files, automatically extracting the content in the given destination. This feature only applies to locally stored compressed files/directories. Type in the source and where you want the command to extract the content as follows:

```docker
ADD source.file.tar.gz /temp
```

**`COPY`** - Due to some functionality issues, Docker had to introduce an additional command for duplicating content – `COPY`. Unlike its closely related `ADD` command, `COPY` only has only one assigned function. Its role is to duplicate files/directories in a specified location in their existing format. This means that it doesn’t deal with extracting a compressed file, but rather copies it as-is. The instruction can be used only for locally stored files. Therefore, you cannot use it with URLs to copy external files to your container. To use the `COPY` instruction, follow the basic command format: `COPY <src> … <dest>`.  For example:

```docker
COPY /source/file/path  /destination/path 
```
[Reference](https://phoenixnap.com/kb/docker-add-vs-copy).  Also see [Stack Overflow Answer](https://stackoverflow.com/questions/24958140/what-is-the-difference-between-the-copy-and-add-commands-in-a-dockerfile).

## Best and Standard Practices

### A Container Should Do One Thing Only.
Technically, you CAN start multiple processes inside a Docker container. You CAN put your database, frontend, backend, ssh, and supervisor all into one docker image. But this is not good practice because:

1. Your build times will be long -remember every time something changes in a `Dockerfile` everything below that change won't be able to use the image cache.
2. You image will be large and take a while to download and upload.
3. Your container logs will be a mess with so much going on in one container
4. Doesn't scale well at all.

The list goes on! Docker's advice is to prepare separate Docker image for each component of your app. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfiles-galore)

