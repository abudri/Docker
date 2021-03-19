# Docker

Notes mainly from the amazingly well put together App Academy Open [Docker Curriculum](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/)

## Table of Contents

* [Basic CLI Docker Commands](https://github.com/abudri/Docker/blob/main/README.md#basic-cli-docker-commands)
* [`Dockerfile` Basics](https://github.com/abudri/Docker/blob/main/README.md#dockerfile-basics)
* [`Dockerfile` Commands](https://github.com/abudri/Docker/blob/main/README.md#dockerfile-commands)
* [Nuances & Extra Notes](https://github.com/abudri/Docker/blob/main/README.md#nuances--extra-notes)
* [Best and Standard Practices](https://github.com/abudri/Docker/blob/main/README.md#best-and-standard-practices)
* [Multi-Stage Builds](https://github.com/abudri/Docker/blob/main/README.md#multi-stage-builds)
* [Anatomy of Docker Components](https://github.com/abudri/Docker/blob/main/README.md#anatomy-of-docker-components)
* [Docker Compose](https://github.com/abudri/Docker/blob/main/README.md#docker-compose)
* [Glossary](https://github.com/abudri/Docker/blob/main/README.md#glossary)

## Basic CLI Docker Commands

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

#### `network` Commands

`docker network create <some_name>` - creates a Docker network.

`docker network list` - lists available Docker networks.

`docker network inspect <network id or name>` - see the network's subnet and which containers are currently attached to it.  It also shows default getaway and which - if any - containers are attached to it.

`docker exec -it <docker conainer name or id> ifconfig` - Gets the IP of the container and other network information.

`docker exec -it <docker conainer name or id> ping <ip address of choice>` - After getting the IP address of a container from the previous command above, you can ping that container _from another container_ to check the connectivity between two containers.

[Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/) for commands in this subsection.


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

### Tags and the `latest` Tag

A tag serves to map a descriptive, user-given name to any single image ID. An image name suffix (the name that comes after the `:`) is often referred to as a tag as well, though it strictly refers to the full name of an image. (Example: `node:8.15-alpine` the `8.15-alpine` is the version, then the tag for that version). Tag names should be limited to the set of alphanumeric characters `[a-z, A-Z, 0-9]` and punctuation characters `[._-]`. Tag names also **MUST NOT** contain a `:` character.

The `latest` tag is the default tag applied when pulling or pushing an image. So if you pull down the `nginx` image (`docker pull nginx`) you are actually pulling down `nginx:latest` since you didn't specify a tag. The `latest` tag of an image is usually the most recent and stable version of an image. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/docker-hub---pushing-and-pulling-images)

The `latest` tag will always be the default if no tag is specified. Say if we have in the `Dockerfile` the `FROM ubuntu` command, that instruction is equal to `FROM ubuntu:latest`. There is a problem with this: what if ubuntu releases a new version that your build isn't compatible with? So, unless it is your intention to a generic `Dockerfile` that must stay up-to-date with the base image, provide a specific tag, such as `FROM ubuntu:16.03`. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfiles-galore)

### Using the Proper Base Image

Say we had a `Dockerfile` for a Node app, so at the top of the `Dockerfile` we had `FROM ubuntu:16.03`.  If this is a Node app, why is ubuntu our base image? Do we really need a general-purpose base image, when we just want to run a node application? A Better option is to use a specialized image with node _already installed_. And while we are it let's use the alpine tagged version of node to make sure our image is as small as possible . You can change the base image as such using `node:10-alpine`.

Note: If we were to install any packages using the alpine distribution we would need to remember to use the alpine package system `apk` instead of `apt-get`. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfiles-galore)

### Note on `alpine` versions of images

In general if you see an image has an `alpine` tag version available it means it is using the Linux distribution for `alpine` as it's base image. Alpine is a small security-focused distribution of Linux. Like super small, it's only about 5MB in size. The alpine version of images are popular for development because they use less memory. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/docker-hub---pushing-and-pulling-images)

## `Dockerfile` Commands

**Common `Dockerfile` Commands - [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfile-cheat-sheet) / [Also](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact)**

**`FROM` -** Almost every Dockerfile begins with the `FROM` instruction that will set the Base Image for all subsequent instructions. The only argument that can come before `FROM` is [`ARG`](https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact) which can be used to declare a variable outside of the build stage, that can be later be used inside the build stage.

**`ENV` -** This command is the preferred way to inject keys and values into image building. All the variables set using this command can be used in the subsequent instructions in the build stage.

**`RUN`-** The `RUN` instruction will execute each phrase of commands in a new layer on top of the current image and commit the results. The resulting committed image will be used for the next step in the Dockerfile. Additionally the `RUN` command can run shell scripts or whatever you could use within a container.

**`WORKDIR` -** The `WORKDIR` instruction sets the working directory for any `RUN`, `CMD`, `ENTRYPOINT`, `COPY` and `ADD` instructions that follow it in the Dockerfile. It is best practice to use `WORKDIR` to run commands that rely on being in a certain location in the file tree. The `WORKDIR` instruction can be also be used multiple times in a Dockerfile. If a relative path is provided, it will be relative to the path of the previous `WORKDIR` instruction.

**`COPY` -** Will make a copy of the files in the first given location to the second given location.

**`EXPOSE` -** Specify to the image which ports are going to be exposed within that image.

**`CMD` -** This is the final command that will run every time you launch a new container from this image or restart a stopped container of this image. Also from [documentation](https://docs.docker.com/engine/reference/builder/#cmd):
	The CMD instruction has three forms:
	
	CMD ["executable","param1","param2"] (exec form, this is the preferred form)
	CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
	CMD command param1 param2 (shell form)
	
There can only be one CMD instruction in a Dockerfile. If you list more than one CMD then only the last CMD will take effect.

Also: Docker containers are a bit different than usual Linux containers. Docker containers are made specifically to run a single process in an isolated environment of a Linux container. That’s why we have CMD in Dockerfile, which indicates which process is this going to be. The Docker container will be automatically terminated once there is no process running inside of it. [Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

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

Also from the [course](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfiles-galore):

Just like the title says - use `COPY` it's simpler. `ADD` has some logic for downloading remote files and extracting archives, which is most scenarios is more inefficient for what you need. Just stick with `COPY`.

## Best and Standard Practices

### A Container Should Do One Thing Only.
Technically, you CAN start multiple processes inside a Docker container. You CAN put your database, frontend, backend, ssh, and supervisor all into one docker image. But this is not good practice because:

1. Your build times will be long -remember every time something changes in a `Dockerfile` everything below that change won't be able to use the image cache.
2. You image will be large and take a while to download and upload.
3. Your container logs will be a mess with so much going on in one container
4. Doesn't scale well at all.

The list goes on! Docker's advice is to prepare separate Docker image for each component of your app. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfiles-galore)

## Multi-Stage Builds

Say we have a React/Redux application - strictly frontend. Meaning there is no server for rendering this application and we'll use nginx in order to see this app on localhost. So, we want to use node as a base image in order to bundle our React code, but we also want to use nginx as our server to show our code. We have two main concerns but we only want to build one image. One way we can get around this is by using [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/#name-your-build-stages). Once our code has been built using - for example, node - we don't actually require any node functionality anymore. So we can use the files that node built and hand them to nginx to replace the default html that nginx renders.

In the `Dockerfile` you can start with the `FROM` command use a recent node version like `node:8.15-alpine` and name this stage of the build using `as`, for later reference:

```docker
FROM node:8.15-alpine as build-stage
```

Once you put in the lines in the above `Dockerfile` for copying over any necessary files, installing dependencies, running webpack, etc, you can do the multi-stage trick.  Next you'll want to create a second `FROM` with the `nginx` image, version `1.15`.  So you can have:

```docker
FROM nginx:1.15
```

Next is a normal COPY, but we'll add a `--from=build-stage`. That `build-stage` refers to the name we specified above in the `as build-stage`. Here, although we are in a Nginx image, starting from scratch, we can copy files from a previous stage. Copy over your `/app` folder into the place 'nginx' keeps its html information - `/usr/share/nginx/html`.

For the last step we've provided you with a nginx.conf - this file will basically just take care of making sure all your files aren't overwritten by the nginx default configuration. Add this last line to your Dockerfile:

```docker
COPY --from=build-stage /app/nginx.conf /etc/nginx/conf.d/default.conf
```

[Reference](https://docs.docker.com/develop/develop-images/multistage-build/#name-your-build-stages) and from Stage 4 in this App Academy Day 2 [Project](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/dockerfiles-galore)

## Docker Compose

After publishing images to Docker Hub, put them to good use. If you are tired of setting up individual Docker Containers and images, imagine a world where you could boot up and tear down multiple containers with a single command. That is the magic of Docker Compose.

Docker Compose is a Python-based tool which according to Docker is, "a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration." With a single YAML file and a simple CLI you can use compose to set up all the networking, volumes, and containers you need to run an entire app.

Docker Compose is excellent in running in automated testing and development environments. As a developer you'll be interacting with Docker Compose in order to speed up your development cycle. Docker Compose could also suitable for production...if you’re deploying to a single host. There is a reason Docker says, "Compose is great for development, testing, and staging environments". Docker Compose wasn't designed to be a production-grade tool. You can however use Compose with Docker Swarm if you are interested in using Compose in a production environment. Feel free to read up on Docker Swarm [here](https://docs.docker.com/compose/swarm/) if you are interested.

If you installed Docker on either a Mac or Windows you should automatically have Docker Compose installed. If you are on a Linux machine check [here](https://github.com/docker/compose/releases) for installation instructions.

**Other Synopsis of Docker Compose:**

Docker compose is also an orchestration tool for docker. It allows you to easily manage multiple containers dependent on each other within one docker host via `docker-compose` CLI. You use a YAML file to configure all the containers. With one command you can start all containers in the correct order and set up networking between them. [Here](https://docs.docker.com/compose/reference/overview/) is the reference. [Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

Overview: In the words of Docker, "Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration". As you learned in the Compose readings you can define a complex stack in one file and have it running with a single command! This would mean you would no longer have to use separate terminal tabs open for running your server and your front end. No more defining containers by hand! The people at Docker choose to call this gift, Docker Compose. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/becoming-compose-pros)

#### Workflow

Using Compose is basically a three-step process.

1. Define your app's environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Lastly, run `docker-compose up` and Compose will start and run your entire app. [Reference](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/becoming-compose-pros)

### Compose Features

The most important features of Docker Compose are:

1. The Ability to Deploy Multiple Container Easily
2. Automatic Configuration of a single network for your app: Each container for a service joins the created network and is reachable by all the other containers on that network by their service name.
3. Preservation of volume data when containers are created. - When `docker-compose up` runs, if it finds any containers from previous runs, it copies the volumes from the old container to the new container it is creating.
4. Only recreate containers that have changed. - Compose caches the configuration used to create a container. When you restart a service and nothing has changed, Compose re-uses the existing containers.
5. Compose allows you to use [variables](https://docs.docker.com/compose/compose-file/#variable-substitution) in the Compose file.

### Docker Compose Composition

When we say "Docker Compose" we are really referring to two things:

1. A YAML formatted file - describing what is needed in terms of containers, networks, and volumes. This file is usually named `docker-compose.yml`.
2. A CLI tool (`docker-compose`) which is primarily used for local development and testing which relies on the YAML file.

The first line of a `docker-compose.yml` file is the version of Compose you will be using. Docker Compose has gone through several iterations and there are a many available versions for you to use. We generally recommend using at least version `2` for your files. Though for certain features you many need to upgrade to a higher version. See a list of Docker Compose versions and features [here](https://docs.docker.com/compose/compose-file/compose-versioning/).

Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
2. Define the `services` that make up your app in the `docker-compose.yml` so they can be run together in an isolated environment.
3. Run 	docker-compose up` and Compose will start and run your entire app.

When we are talking about Docker Compose the word `services` refers to running multiple containers. Below is a breakdown of the general way your file should be formatted:

```yaml
# If no version is specified then version 1.0 is assumed. 
# Recommend version 2 at the minimum
version: '3.1'  

services:  # Will start up containers. Is the same as using docker container run.
  servicename: # A Friendly name (postgres, node, etc.). This is also DNS name inside your network
    image: # the image this service will use
    command: # Optional, will replace the default CMD specified by the image
    environment: # Optional, same as -e in docker container run
    volumes: # Optional, same as -v in docker container run
  psql: # servicename2

volumes: # Optional, same as docker volume create

networks: # Optional, same as docker network create
```

Docker compose can setup containers, volumes and networks and environment variables all in one committable file that can be brought up with a single command.  You can build your entire full-stack application and bring it up with one command with docker compose. Some notes on each of the keys above:

`version` - every `docker-compose.yml` file starts with a version like `version: '2'` . If you do not specify a version it will automatically use Docker Compose version `1`, which has less functionality, so you definitely want to specify a version.

`services` - this is docker compose language for "these are the containers I will be running for you".  So think containers when you see the `services` key, meaning, "I will be making containers under this `services` key."

`servicename` - this is actually generic and can be whatever you want to call it, `my_app`, etc. 

`image` - Here you could have for example `image: redis` this container will be running the redis image. The redis service uses a public [Redis](https://registry.hub.docker.com/_/redis/) image pulled from the Docker Hub registry.  See this [reference](https://docs.docker.com/compose/gettingstarted/#step-3-define-services-in-a-compose-file) for the breakdown.

`ports` - when you run the container you tell it what ports you want to expose and be running on the image

`environment` - You can hand it the environment variables you want to be running with this container with the `environment` key.  

`volumes` - you can also set up a bind mount in this container with `volumes`.

Check out [this](https://yaml.org/refcard.html) resource if you need reminder about YAML formatting.  Note, YAML files can use both `.yml` and `.yaml` extensions.

Also here is a great video on [YAML in One Video](https://www.youtube.com/watch?v=cdLNKUoMc6c).

### Docker Compose CLI / Simple CLI Commands

Note: Main source for this material is [Day 3](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/docker-compose) of App Academy Open Course.

Before we start, if you haven't already, we recommend downloading the Docker Compose [Command-line completion](https://docs.docker.com/compose/completion/) especially while you are learning!

One of the coolest things about Docker Compose is the intuitive CLI commands. There are a lot of similar commands between the `docker-compose` CLI and the `docker` CLI. Docker Compose basically takes over the role of the Docker CLI by talking to the Docker server API in the background for you. The main `docker-compose` commands to know are:

1. `docker-compose --help` - Because who doesn't need a little help now and again?
2. `docker-compose up` - Which will setup your volumes, networks, and start the specified containers
3. `docker-compose down` - Which will stop and remove all containers and networks.
4. `docker-compose down -v` - Which will stop and remove all volumes, containers and networks.

No more giant multi-line container commands! No more removing containers by hand! Docker Compose cleans all that right up into simple single line commands. Imagine this: If all your projects had a Dockerfile and a `docker-compose.yml` all I'd have to do to run your project is:

```
git clone github.com/yourproject
docker-compose up
```

So nifty! 🙌 A complete list of Compose CLI flags can be found [here](https://docs.docker.com/compose/reference/overview/).

### Docker Compose using Custom Image

You can use Docker compose using a custom image from a `Dockerfile`.  This takes from the Day 3 [Homework](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/first-docker-compose-file) in the App Academy Open course.  Say you have in your project directory this `Dockerfile` to build a custom image:

```docker
# This shows how we can extend an existing official image from Docker Hub

FROM nginx:1.15-alpine
# Always pin a version. It'll make development so much easier. 

# Change our working directory to the root of nginx webhost
# using WORKDIR is preferred to using 'RUN cd /some/path'
WORKDIR /usr/share/nginx/html

# Copy in the file!
COPY index.html index.html
# Don't have to specify EXPOSE or CMD because they're in my FROM
```

From this same home directory in the project, you have the following `docker-compose.yml` file.  The comments have brief explanation:

```yaml
version: "2"  # needs to be version 2 or higher, version 1 has less functionality

services: # defined your service
  webapp:  # name of your service is `webapp`
    build:
      context: . # this takes the Dockerfile from your current directory
    image: nginx:latest_greatest #image name and tag
    ports:
      - '80:80'  # port 80 local host is binded to port 80 of container
```
From the terminal you can now build and run the container in one simple command, `docker-compose up`. Note, to free up the terminal you can run it in detached mode with a `-d` flag, so `docker-compose up -d`:

```bash
z@Mac-Users-Apple-Computer homework_day_3 % docker-compose up
Building webapp
Step 1/3 : FROM nginx:1.15-alpine
1.15-alpine: Pulling from library/nginx
e7c96db7181b: Pull complete
264026bbe255: Pull complete
a71634c55d29: Pull complete
5595887beb81: Pull complete
Digest: sha256:57a226fb6ab6823027c0704a9346a890ffb0cacde06bc19bbc234c8720673555
Status: Downloaded newer image for nginx:1.15-alpine
 ---> dd025cdfe837
Step 2/3 : WORKDIR /usr/share/nginx/html
 ---> Running in e734a7807195
Removing intermediate container e734a7807195
 ---> d1f46e4771db
Step 3/3 : COPY index.html index.html
 ---> 807fd3728909

Successfully built 807fd3728909
Successfully tagged nginx:latest_greatest
WARNING: Image for service webapp was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating homework_day_3_webapp_1 ... done
Attaching to homework_day_3_webapp_1
webapp_1  | 172.19.0.1 - - [19/Mar/2021:16:38:15 +0000] "GET / HTTP/1.1" 200 293 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36" "-"
webapp_1  | 2021/03/19 16:38:15 [error] 7#7: *2 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.19.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost", referrer: "http://localhost/"
```

Note you get get request logging above as well.  If you run a `docker container ls`, you see your running container listed:

```bash
z@Mac-Users-Apple-Computer homework_day_3 % docker container ls
CONTAINER ID        IMAGE                   COMMAND                  CREATED              STATUS              PORTS                NAMES
433d34494407        nginx:latest_greatest   "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   homework_day_3_webapp_1
```

### Docker Compose vs Docker Swarm

**Swarm**
Docker swarm is another orchestration tool aimed to manage a cluster of docker hosts. While docker-compose managers _multiple Docker containers within one docker host_, docker swarm manages _multiple docker hosts managing multiple Docker containers_.

Unlike docker-compose and docker-machine, docker swarm is not a standalone orchestration software. Swarm mode is built in docker engine and is managed through Docker client.

In order to create a swarm you need to ssh into a machine you intend to make into a swarm and `docker swarm init --advertise-addr <ip to publish>`. This command will make a machine accessible on `<ip to publish>`. Other docker hosts can now join the swarm on this IP. [Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

## Anatomy of Docker Components

![image](https://user-images.githubusercontent.com/17362519/111221493-efde6580-85b0-11eb-854b-09b61bdcdf0f.png)

[Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

## Glossary

**Docker** - What is it? Docker is not a standalone software, it’s a platform for managing Linux containers. Whenever someone mentions docker in the context of software, they are talking about docker CE or docker EE. Docker is developed by Docker, Inc to simplify the usage of Linux containers. The platform consists of multiple tools for running and managing Linux containers, which include:

**Docker daemon/engine**  - responsible for generation and running of Linux containers. [Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

**Docker client** - a separate application which controls docker daemon through the REST API. [Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

**Docker-compose, docker-machine, and docker swarm** -  orchestration tools, they are not necessary for running processes inside Linux containers, but they make container management very simple. To be frank, in real life scenarios they are pretty much a necessity, because managing all those containers, hosts and clusters of hosts manually is difficult. [Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

**Docker Hub** - is a service that provides a registry of docker images. We can store our images on the docker hub and pull images made by others for us to use. [Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

**Docker Registry** - allows us to host our own private registry in case we don’t want to use an existing one. [Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

**Docker for Mac and Docker for Windows** - separate tools that simplify developing with docker on Mac or Windows.[Reference](https://www.freecodecamp.org/news/comprehensive-introductory-guide-to-docker-vms-and-containers-4e42a13ee103/)

