
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

## Terminal Outputs/Solution

File solution is of course in the `docker-compose.yml` file in this directory, ie, above. The `Dockerfile`, `app.py`, and `requirements.txt` files were all given.

My output after running `docker-compose up`:

```bash
z@Mac-Users-Apple-Computer phase1 % docker-compose up
Creating network "phase1_default" with the default driver
Building web
Step 1/7 : FROM python:3.7.0-alpine3.8
3.7.0-alpine3.8: Pulling from library/python
4fe2ade4980c: Pull complete
7cf6a1d62200: Pull complete
b66090d9998c: Pull complete
e0dc374a57ff: Pull complete
eb580e903ff2: Pull complete
Digest: sha256:e12594db7297ebf9d9478ba60373e0181974f373016e7495926a7da81bda323b
Status: Downloaded newer image for python:3.7.0-alpine3.8
 ---> cf41883b24b8
Step 2/7 : WORKDIR /usr/src/app
 ---> Running in 53bdc2938865
Removing intermediate container 53bdc2938865
 ---> 7252273a24fe
Step 3/7 : COPY requirements.txt ./
 ---> d8d82440ee4d
Step 4/7 : RUN pip install --no-cache-dir -r requirements.txt
 ---> Running in ba253a025c5a
Collecting flask (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/f2/28/2a03252dfb9ebf377f40fba6a7841b47083260bf8bd8e737b0c6952df83f/Flask-1.1.2-py2.py3-none-any.whl (94kB)
Collecting redis<3.0.0 (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/3b/f6/7a76333cf0b9251ecf49efff635015171843d9b977e4ffcf59f9c4428052/redis-2.10.6-py2.py3-none-any.whl (64kB)
Collecting click>=5.1 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/d2/3d/fa76db83bf75c4f8d338c2fd15c8d33fdd7ad23a9b5e57eb6c5de26b430e/click-7.1.2-py2.py3-none-any.whl (82kB)
Collecting itsdangerous>=0.24 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/76/ae/44b03b253d6fade317f32c24d100b3b35c2239807046a4c953c7b89fa49e/itsdangerous-1.1.0-py2.py3-none-any.whl
Collecting Werkzeug>=0.15 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/cc/94/5f7079a0e00bd6863ef8f1da638721e9da21e5bacee597595b318f71d62e/Werkzeug-1.0.1-py2.py3-none-any.whl (298kB)
Collecting Jinja2>=2.10.1 (from flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7e/c2/1eece8c95ddbc9b1aeb64f5783a9e07a286de42191b7204d67b7496ddf35/Jinja2-2.11.3-py2.py3-none-any.whl (125kB)
Collecting MarkupSafe>=0.23 (from Jinja2>=2.10.1->flask->-r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/b9/2e/64db92e53b86efccfaea71321f597fa2e1b2bd3853d8ce658568f7a13094/MarkupSafe-1.1.1.tar.gz
Installing collected packages: click, itsdangerous, Werkzeug, MarkupSafe, Jinja2, flask, redis
  Running setup.py install for MarkupSafe: started
    Running setup.py install for MarkupSafe: finished with status 'done'
Successfully installed Jinja2-2.11.3 MarkupSafe-1.1.1 Werkzeug-1.0.1 click-7.1.2 flask-1.1.2 itsdangerous-1.1.0 redis-2.10.6
You are using pip version 18.1, however version 21.0.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Removing intermediate container ba253a025c5a
 ---> d73a0fc52328
Step 5/7 : COPY . .
 ---> 4c330203dcc8
Step 6/7 : ENV FLASK_APP=app.py
 ---> Running in 065dfe747700
Removing intermediate container 065dfe747700
 ---> 2cebda254e26
Step 7/7 : CMD flask run --host=0.0.0.0
 ---> Running in 5fa547880688
Removing intermediate container 5fa547880688
 ---> cf61291a0661

Successfully built cf61291a0661
Successfully tagged flask_server:latest
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Pulling redis (redis:4.0.11-alpine)...
4.0.11-alpine: Pulling from library/redis
4fe2ade4980c: Already exists
fb758dc2e038: Pull complete
989f7b0c858b: Pull complete
d5318f13abaa: Pull complete
3521559474dd: Pull complete
add04b113886: Pull complete
Digest: sha256:2953e537b8eaa5120855285497d4f936d9f02a16480a9d76e8ba014dc3998704
Status: Downloaded newer image for redis:4.0.11-alpine
Creating phase1_web_1   ... done
Creating phase1_redis_1 ... done
Attaching to phase1_redis_1, phase1_web_1
redis_1  | 1:C 19 Mar 18:42:34.055 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis_1  | 1:C 19 Mar 18:42:34.055 # Redis version=4.0.11, bits=64, commit=00000000, modified=0, pid=1, just started
redis_1  | 1:C 19 Mar 18:42:34.055 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_1  | 1:M 19 Mar 18:42:34.059 * Running mode=standalone, port=6379.
redis_1  | 1:M 19 Mar 18:42:34.059 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_1  | 1:M 19 Mar 18:42:34.059 # Server initialized
redis_1  | 1:M 19 Mar 18:42:34.059 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
redis_1  | 1:M 19 Mar 18:42:34.059 * Ready to accept connections
web_1    |  * Serving Flask app "app.py" (lazy loading)
web_1    |  * Environment: development
web_1    |  * Debug mode: on
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
web_1    |  * Restarting with stat
web_1    |  * Debugger is active!
web_1    |  * Debugger PIN: 137-669-480
web_1    | 172.20.0.1 - - [19/Mar/2021 18:42:44] "GET /favicon.ico HTTP/1.1" 404 -
web_1    | 172.20.0.1 - - [19/Mar/2021 18:42:45] "GET /favicon.ico HTTP/1.1" 404 -
web_1    | 172.20.0.1 - - [19/Mar/2021 18:43:20] "GET / HTTP/1.1" 200 -
web_1    | 172.20.0.1 - - [19/Mar/2021 18:43:20] "GET /favicon.ico HTTP/1.1" 404 -
web_1    | 172.20.0.1 - - [19/Mar/2021 18:43:22] "GET /favicon.ico HTTP/1.1" 404 -
web_1    | 172.20.0.1 - - [19/Mar/2021 18:43:26] "GET / HTTP/1.1" 200 -
web_1    | 172.20.0.1 - - [19/Mar/2021 18:43:26] "GET /favicon.ico HTTP/1.1" 404 -
web_1    | 172.20.0.1 - - [19/Mar/2021 18:43:27] "GET /favicon.ico HTTP/1.1" 404 -
^CGracefully stopping... (press Ctrl+C again to force)
```

Instructions:
> A quick `docker container ls` will confirm that your container are no longer running, but if you do a `docker container ls -a`? Then you'll see your containers there - just stopped. If you try `docker network ls` you'll also see the network that had been automatically created for you is still there too. Let's try that again shall we? Use `docker-compose down` and Docker Compose will take care of removing not only the containers but the network. So kind! 

```bash
z@Mac-Users-Apple-Computer phase1 % docker-compose down
Removing phase1_redis_1 ... done
Removing phase1_web_1   ... done
Removing network phase1_default
```

Running `docker compose up -d` to run compose in detached mode.

```bash
z@Mac-Users-Apple-Computer phase1 % docker-compose up -d
Creating network "phase1_default" with the default driver
Creating phase1_web_1   ... done
Creating phase1_redis_1 ... done
```

Instructions:
> **Reminder:** If you did have to change something in your `Dockerfile` and rebuild your image you'd need to make use `docker-compose up --build` or the `docker compose build` command to rebuild the image. Otherwise Compose won't know about it!
> Let's try out the Flask app! Head to `http://localhost:5000` and you should see and empty array. Boooring. You could use Postman to test your app or use the below command to send a 'POST' request (me: just enter this in the terminal):

```
curl --header "Content-Type: application/json" \
--request POST \
--data '{"name":"Jake"}' \
localhost:5000
```

> You should get a response with the name input. Now head to `http://localhost:5000`. If you see Jake then you've done it!

I did get that POST to take and show the data:

<img src="https://user-images.githubusercontent.com/17362519/111876484-5d094680-8975-11eb-83a4-15834ef3e294.png" width="650" />
