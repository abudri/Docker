# Becoming a Compose Pro

Project page located [here](https://open.appacademy.io/learn/full-stack-online/docker-curriculum/becoming-compose-pros). 

## **Overview**
In the words of Docker, Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. As you learned in the Compose readings you can define a complex stack in one file and have it running with a single command! This would mean you would no longer have to use separate terminal tabs open for running your server and your front end. No more defining containers by hand! The people at Docker choose to call this gift, Docker Compose.

## Workflow:
Using Compose is basically a three-step process.

1. Define your app's environment with a Dockerfile so it can be reproduced anywhere.
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
3. Lastly, `run docker-compose up` and Compose will start and run your entire app.
