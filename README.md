# Docker
Mainly from amazingly well put together App Academy Open [Docker Curriculum](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/)

## Basic Docker Commands

`docker image ls` - list your docker images

`docker container ls` - lists your docker containers that are running
`docker container ls -a` - also lists your containers that are NOT running

`docker build .` - this builds a docker image from the existing `Docker` file in the _current_ directory - `.`.  However, keep in mind that this will not tag your image, and hence `REPOSITORY` and `TAG` will both be empty when you do a `docker image ls` after building the image:

    z@Mac-Users-Apple-Computer phase1 % docker image ls
    REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
    <none>                   <none>              5590e09e8948        4 seconds ago       109MB

`docker build . -t day_2_project` - this actually builds the image of the `Dockerfile` with a tag using the `-t` flag.  Now when you run `docker image ls` after building the image,`REPOSITORY` and `TAG` will have `day_2_project` and `latest` - the default tag.

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

`docker container run -d -p 80:80 day_2_project` - this builds and runs your container using the `day_2_project` image, and also runs it in detached mode with the `-d` flag so that you get your terminal prompt back, and using `-p` to publish the port.  When we were doing this command, the first port is the `<local_port>`, and the second port is the container port, `<container_port>`, meaning  `docker container run -d -p <local_port>:<container_port>`.

## A Basic `Dockerfile`

    FROM nginx:1.15
    WORKDIR /usr/share/nginx/html
    COPY index.html /usr/share/nginx/html
