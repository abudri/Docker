
# **Phase 1: Flask and Redis**
Time to take on your destiny and write some fantastic Docker Compose files! To start off today you'll be taking a very simple Flask and Redis application and creating a compose file to run it locally. Start off by taking a look at the [skeleton](https://assets.aaonline.io/Docker/projects/compose_pros/skeleton.zip).

### Docker-Compose File
Our Flask app will contain the following components:

1. The Flask server that accepts user requests and stores the data in Redis.
2. Redis which will be acting as our database.

We've provided you with the `Dockerfile` that will set up the Flask server for you. Now, what we don't want is one container running both our Flask Server and Redis database. Ideally, we want two containers - one for our Flask server, and one for our database.

**Quick Reminder - indentation is how the YAML file formats group information**, so indentation is important. Start by creating a `docker-compose.yml` file. For the rest of this phase this is where you'll be working. As we previous learned `docker-compose` has many versions, for this project you will use version `'3'`. Take a look at the [Compose documentation](https://docs.docker.com/compose/compose-file/) if you need a syntax reference.

Since we have two main parts in the current architecture for our application (server and database) we'll want to run two `services`: one for the Flask server, and one for the redis database. Create two services, one called `web` and the other called `redis`. For the future, a simple rule of thumb is to create one service for each image in your application.

For our Flask app we will need to build the image in the Dockerfile in order for it to run properly. If you need a reminder on how that is done check out the [Docker documentatio](https://docs.docker.com/compose/compose-file/#build)n on the subject. Indicate to Docker that you want to `build` and make sure to name your image using the image command in your `docker-compose.yml`.

Similarly to how we usually run containers - we can set our exposed ports and environment variables for this image. Set the `FLASK_ENV=development` for your environment. For this service you'll also want to expose ports on localhost and within the container on port `5000`.

We won't need to customize the `redis` image - we are just going to use the image straight from Docker Hub. Use the `image` command to use the `redis:4.0.11-alpine` version.

Now try running it! Use the `docker-compose up` command! It'll take a while to build because it has to pull the necessary images and build from the Dockerfile.

You'll see the first thing it does is create a new network for you, before beginning to build your image.

```bash
Creating network "phase1_default" with the default driver
Building web
Step 1/7 : FROM python:3.7.0-alpine3.8
 ---> cf41883b24b8
```

After it has built your image you'll see a message that both of your containers have been started and attached to the network. Then you'll get some colored logs for each container!

```bash
Creating phase1_web_1   ... done
Creating phase1_redis_1 ... done
Attaching to phase1_web_1, phase1_redis_1
```

You can use CTRL+C to exit the logs and you'll see this message as you exit:

```bash
Stopping phase1_redis_1 ... done
Stopping phase1_web_1   ... done
```

A quick `docker container ls` will confirm that your container are no longer running, but if you do a `docker container ls -a`? Then you'll see your containers there - just stopped. If you try `docker network ls` you'll also see the network that had been automatically created for you is still there too. Let's try that again shall we? Use `docker-compose down` and Docker Compose will take care of removing not only the containers but the network. So kind! Now try running `docker compose up -d` to run compose in detached mode.

Reminder: If you did have to change something in your Dockerfile and rebuild your image you'd need to make use `docker-compose up --build` or the `docker compose build` command to rebuild the image. Otherwise Compose won't know about it!

Let's try out the Flask app! Head to http://localhost:5000 and you should see and empty array. Boooring. You could use Postman to test your app or use the below command to send a 'POST' request:

```bash
curl --header "Content-Type: application/json" \
--request POST \
--data '{"name":"Jake"}' \
localhost:80
```
Note: I bound my local port 80 to the container port 5000.  Hence I changed `localhost:5000` from the project page to `localhost:80` in the line above.

You should get a response with the name input. Now head to http://localhost:5000. If you see Jake then you've done it!

I'd like to point out something very cool that is going on in the `app.py` file where Redis is being set up. Namely, containers on the same docker-compose network can refer to each other by service name. If you look below you'll see where we set up access from the web container to the redis container. We can use the service name redis as our host name to access the running container.

```redis
# host="redis" is referring to the name of your service!!
redis = Redis(host="redis", db=0, socket_timeout=5, charset="utf-8", decode_responses=True)
```

As you can see, the value of the host argument is set to `redis`, because the name of the Redis service is `redis` in our Compose file. In this way you can easily connect containers to each other with Docker Compose. You can also specify more complex network setups beyond the default network. Make sure you use `docker-compose down` to clean up you containers and networks and don't forget to use Git from now on to commit your Compose files.

Head to the next phase!
