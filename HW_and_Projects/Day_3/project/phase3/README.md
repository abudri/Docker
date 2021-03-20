# **Phase 3: The Voting App**

We will be creating the `docker-compose.yml` file for a Voting Application for the world's toughest question: "cats or dogs?". Users can cast their votes, which will be saved, and admin users can see the votes cast. This is an application based on micro-services architecture, consisting of 5 individually simple services.

![https://assets.aaonline.io/Docker/voting.png](https://assets.aaonline.io/Docker/voting.png)

1. **Voting-App:** The frontend of the application written in Python, used by users to cast their votes.
2. **Redis:** The in-memory database, used as intermediate storage.
3. **Worker:** A .Net service, used to fetch votes from Redis and store in a Postgres database.
4. **DB:** A PostgreSQL database, used as the database.
5. **Result-App:** Frontend of the application written in Node.js which displays the voting results.

All of the images you need are on Docker Hub. We will creating the services including exposing ports, volumes, and connecting containers thorough our own networks. Let's get started!

## **Services**

Start off by defining your Compose version as '3'. You'll be creating 5 separate services and pulling their images from Docker Hub. The **names of these services do matter** for the images, so make sure you use the names for each service as described below. You'll be creating two [custom networks](https://docs.docker.com/compose/networking/#specify-custom-networks) "frontend" and "backend".

## **Breakdown of the Five Services**

**Service One: vote**

- The frontend of the application written in Python
- The image will be `dockersamples/examplevotingapp_vote:before`
- The locally exposed port should be `5000`.
    - The container's internally exposed port should be `80`
- On the frontend Network

**Service Two: redis**:

- The in-memory key/value storage for incoming votes
- The image will be `redis:3.2`
- No exposed ports
- On the Frontend Network

**Service Three: worker**

- Used to fetch votes from Redis and store in a Postgres database.
- The image is `dockersamples/examplevotingapp_worker`
- No public ports
- On both the frontend and backend networks

**Service Four: db**

- The database, enough said.
- The image will be `postgres:9.4`
- One [named volume](https://docs.docker.com/compose/compose-file/#short-syntax-3) will be needed, pointing to `/var/lib/postgresql/data` in order to persist data.
- On the backend network

**Service Five: result**

- The node web app that shows results to administrators
- The image is `dockersamples/examplevotingapp_result:before`
- Run this service with the local port of 5001, and the container listening on 80
- On the backend network

Now try it all together! Run `docker-compose up`. Notice that when you specify custom networks Docker Compose doesn't automatically make a network for you, it just makes the networks you specified:

```
Creating network "phase_frontend" with the default driver
Creating network "phase_backend" with the default driver
```

If you use `docker compose down`, then `docker compose up -d` you actually be able to look at the containers running with `docker container ps`. You can also see the networks that compose created for you using `docker network ls`. Go to `http://localhost:5000` and you should be able to vote for either "dogs" or "cats". Once you've cast your vote refresh and make sure your vote persisted.

Now checkout `http://localhost:5001` and you'll see the `result` service at work as your can look on the number of votes and who voted for what.

Awesome job! After you've debated about "cats vs. dogs" in your heart move onto the next phase.

## Terminal Outputs/Solution

File solutions is of course in the `docker-compose.yml` file in this directory, ie, above.

After the `docker-compose.yml` file was built, this is the result of running `docker-compose up`:

![image](https://user-images.githubusercontent.com/17362519/111874203-748f0200-896a-11eb-803a-e0d8d4497629.png)

Also, this is the terminal output from running `docker-compose up`:

```bash
z@Mac-Users-Apple-Computer phase3 % docker-compose up
Creating network "phase3_frontend" with the default driver
Creating network "phase3_backend" with the default driver
Creating volume "phase3_postgres-db" with default driver
Pulling voteapp (dockersamples/examplevotingapp_vote:before)...
before: Pulling from dockersamples/examplevotingapp_vote
9ca846b27f6e: Pull complete
59d32c641ca1: Pull complete
f48f6e9494e9: Pull complete
effaebf8be80: Pull complete
ca205e91b47b: Pull complete
83273eb88c36: Pull complete
1462ca768ad1: Pull complete
Digest: sha256:8e64b18b2c87de902f2b72321c89b4af4e2b942d76d0b772532ff27ec4c6ebf6
Status: Downloaded newer image for dockersamples/examplevotingapp_vote:before
Pulling redis_memory (redis:3.2)...
3.2: Pulling from library/redis
f17d81b4b692: Pull complete
b32474098757: Pull complete
8980cabe8bc2: Pull complete
58af19693e78: Pull complete
a977782cf22d: Pull complete
9c1e268980b7: Pull complete
Digest: sha256:7b0a40301bc1567205e6461c5bf94c38e1e1ad0169709e49132cafc47f6b51f3
Status: Downloaded newer image for redis:3.2
Pulling net_worker (dockersamples/examplevotingapp_worker:)...
latest: Pulling from dockersamples/examplevotingapp_worker
6d827a3ef358: Pull complete
2726297beaf1: Pull complete
7d27bd3d7fec: Pull complete
e468afdd059a: Pull complete
5f0ad24bb908: Pull complete
585a9e672fec: Pull complete
7bac6a95ca79: Pull complete
07bce396b431: Pull complete
c2f836b84924: Pull complete
Digest: sha256:55753a7b7872d3e2eb47f146c53899c41dcbe259d54e24b3da730b9acbff50a1
Status: Downloaded newer image for dockersamples/examplevotingapp_worker:latest
Pulling db (postgres:9.4)...
9.4: Pulling from library/postgres
619014d83c02: Pull complete
7ec0fe6664f6: Pull complete
9ca7ba8f7764: Pull complete
9e1155d037e2: Pull complete
febcfb7f8870: Pull complete
8c78c79412b5: Pull complete
5a35744405c5: Pull complete
27717922e067: Pull complete
36f0c5255550: Pull complete
dbf0a396f422: Pull complete
ec4c06ea33e5: Pull complete
e8dd33eba6d1: Pull complete
51c81b3b2c20: Pull complete
2a03dd76f5d7: Pull complete
Digest: sha256:42a7a6a647a602efa9592edd1f56359800d079b93fa52c5d92244c58ac4a2ab9
Status: Downloaded newer image for postgres:9.4
Pulling resultsapp (dockersamples/examplevotingapp_result:before)...
before: Pulling from dockersamples/examplevotingapp_result
ab01941cd0bf: Pull complete
a3ed95caeb02: Pull complete
2b1e5c52e9e5: Pull complete
59083d36382c: Pull complete
7fee8b489cf3: Pull complete
1715e729137b: Pull complete
20efc6598be1: Pull complete
6968e16b870e: Pull complete
a7b5f422f6db: Pull complete
7badaf018bf9: Pull complete
9be6a59b4be0: Pull complete
f04c9d519422: Pull complete
Digest: sha256:83b568996e930c292a6ae5187fda84dd6568a19d97cdb933720be15c757b7463
Status: Downloaded newer image for dockersamples/examplevotingapp_result:before
Creating phase3_voteapp_1      ... done
Creating phase3_resultsapp_1   ... done
Creating phase3_redis_memory_1 ... done
Creating phase3_db_1           ... done
Creating phase3_net_worker_1   ... done
Attaching to phase3_redis_memory_1, phase3_net_worker_1, phase3_voteapp_1, phase3_db_1, phase3_resultsapp_1
redis_memory_1  | 1:C 20 Mar 12:19:31.620 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis_memory_1  |                 _._                                                  
redis_memory_1  |            _.-``__ ''-._                                             
redis_memory_1  |       _.-``    `.  `_.  ''-._           Redis 3.2.12 (00000000/0) 64 bit
redis_memory_1  |   .-`` .-```.  ```\/    _.,_ ''-._                                   
redis_memory_1  |  (    '      ,       .-`  | `,    )     Running in standalone mode
redis_memory_1  |  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
redis_memory_1  |  |    `-._   `._    /     _.-'    |     PID: 1
redis_memory_1  |   `-._    `-._  `-./  _.-'    _.-'                                   
redis_memory_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
redis_memory_1  |  |    `-._`-._        _.-'_.-'    |           http://redis.io        
redis_memory_1  |   `-._    `-._`-.__.-'_.-'    _.-'                                   
redis_memory_1  |  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
redis_memory_1  |  |    `-._`-._        _.-'_.-'    |                                  
redis_memory_1  |   `-._    `-._`-.__.-'_.-'    _.-'                                   
redis_memory_1  |       `-._    `-.__.-'    _.-'                                       
redis_memory_1  |           `-._        _.-'                                           
redis_memory_1  |               `-.__.-'                                               
redis_memory_1  | 
redis_memory_1  | 1:M 20 Mar 12:19:31.626 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis_memory_1  | 1:M 20 Mar 12:19:31.626 # Server started, Redis version 3.2.12
redis_memory_1  | 1:M 20 Mar 12:19:31.626 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
redis_memory_1  | 1:M 20 Mar 12:19:31.626 * The server is now ready to accept connections on port 6379
db_1            | Error: Database is uninitialized and superuser password is not specified.
db_1            |        You must specify POSTGRES_PASSWORD for the superuser. Use
db_1            |        "-e POSTGRES_PASSWORD=password" to set it in "docker run".
db_1            | 
db_1            |        You may also use POSTGRES_HOST_AUTH_METHOD=trust to allow all connections
db_1            |        without a password. This is *not* recommended. See PostgreSQL
db_1            |        documentation about "trust":
db_1            |        https://www.postgresql.org/docs/current/auth-trust.html
phase3_db_1 exited with code 1
resultsapp_1    | Sat, 20 Mar 2021 12:19:31 GMT body-parser deprecated bodyParser: use individual json/urlencoded middlewares at server.js:67:9
resultsapp_1    | Sat, 20 Mar 2021 12:19:31 GMT body-parser deprecated undefined extended: provide extended option at ../node_modules/body-parser/index.js:105:29
resultsapp_1    | App running on port 80
voteapp_1       | [2021-03-20 12:19:32 +0000] [1] [INFO] Starting gunicorn 19.6.0
voteapp_1       | [2021-03-20 12:19:32 +0000] [1] [INFO] Listening at: http://0.0.0.0:80 (1)
voteapp_1       | [2021-03-20 12:19:32 +0000] [1] [INFO] Using worker: sync
voteapp_1       | [2021-03-20 12:19:32 +0000] [10] [INFO] Booting worker with pid: 10
voteapp_1       | [2021-03-20 12:19:32 +0000] [11] [INFO] Booting worker with pid: 11
voteapp_1       | [2021-03-20 12:19:32 +0000] [20] [INFO] Booting worker with pid: 20
voteapp_1       | [2021-03-20 12:19:32 +0000] [25] [INFO] Booting worker with pid: 25
net_worker_1    | System.TimeoutException: The operation has timed out.
net_worker_1    |    at Npgsql.NpgsqlConnector.Connect(NpgsqlTimeout timeout)
net_worker_1    |    at Npgsql.NpgsqlConnector.RawOpen(NpgsqlTimeout timeout)
net_worker_1    |    at Npgsql.NpgsqlConnector.Open(NpgsqlTimeout timeout)
net_worker_1    |    at Npgsql.ConnectorPool.Allocate(NpgsqlConnection conn, NpgsqlTimeout timeout)
net_worker_1    |    at Npgsql.NpgsqlConnection.OpenInternal()
net_worker_1    |    at Worker.Program.OpenDbConnection(String connectionString) in /code/src/Worker/Program.cs:line 74
net_worker_1    |    at Worker.Program.Main(String[] args) in /code/src/Worker/Program.cs:line 19
phase3_net_worker_1 exited with code 1
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:47 +0000] "GET / HTTP/1.1" 200 1285 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:47 +0000] "GET /static/stylesheets/style.css HTTP/1.1" 200 - "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:47 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:48 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:49 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:52 +0000] "GET / HTTP/1.1" 200 1285 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:52 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:53 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:53 +0000] "GET / HTTP/1.1" 200 1285 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:53 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:19:54 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | [2021-03-20 12:20:00,040] ERROR in app: Exception on / [POST]
voteapp_1       | Traceback (most recent call last):
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1982, in wsgi_app
voteapp_1       |     response = self.full_dispatch_request()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1614, in full_dispatch_request
voteapp_1       |     rv = self.handle_user_exception(e)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1517, in handle_user_exception
voteapp_1       |     reraise(exc_type, exc_value, tb)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1612, in full_dispatch_request
voteapp_1       |     rv = self.dispatch_request()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1598, in dispatch_request
voteapp_1       |     return self.view_functions[rule.endpoint](**req.view_args)
voteapp_1       |   File "/app/app.py", line 31, in hello
voteapp_1       |     redis.rpush('votes', data)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/client.py", line 1282, in rpush
voteapp_1       |     return self.execute_command('RPUSH', name, *values)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/client.py", line 578, in execute_command
voteapp_1       |     connection.send_command(*args)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 563, in send_command
voteapp_1       |     self.send_packed_command(self.pack_command(*args))
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 538, in send_packed_command
voteapp_1       |     self.connect()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 442, in connect
voteapp_1       |     raise ConnectionError(self._error_message(e))
voteapp_1       | ConnectionError: Error connecting to redis:6379. timed out.
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:20:00 +0000] "POST / HTTP/1.1" 500 291 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:20:01 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | [2021-03-20 12:20:03,296] ERROR in app: Exception on / [POST]
voteapp_1       | Traceback (most recent call last):
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1982, in wsgi_app
voteapp_1       |     response = self.full_dispatch_request()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1614, in full_dispatch_request
voteapp_1       |     rv = self.handle_user_exception(e)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1517, in handle_user_exception
voteapp_1       |     reraise(exc_type, exc_value, tb)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1612, in full_dispatch_request
voteapp_1       |     rv = self.dispatch_request()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1598, in dispatch_request
voteapp_1       |     return self.view_functions[rule.endpoint](**req.view_args)
voteapp_1       |   File "/app/app.py", line 31, in hello
voteapp_1       |     redis.rpush('votes', data)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/client.py", line 1282, in rpush
voteapp_1       |     return self.execute_command('RPUSH', name, *values)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/client.py", line 578, in execute_command
voteapp_1       |     connection.send_command(*args)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 563, in send_command
voteapp_1       |     self.send_packed_command(self.pack_command(*args))
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 538, in send_packed_command
voteapp_1       |     self.connect()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 442, in connect
voteapp_1       |     raise ConnectionError(self._error_message(e))
voteapp_1       | ConnectionError: Error connecting to redis:6379. timed out.
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:20:03 +0000] "POST / HTTP/1.1" 500 291 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | [2021-03-20 12:20:04,627] ERROR in app: Exception on / [POST]
voteapp_1       | Traceback (most recent call last):
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1982, in wsgi_app
voteapp_1       |     response = self.full_dispatch_request()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1614, in full_dispatch_request
voteapp_1       |     rv = self.handle_user_exception(e)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1517, in handle_user_exception
voteapp_1       |     reraise(exc_type, exc_value, tb)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1612, in full_dispatch_request
voteapp_1       |     rv = self.dispatch_request()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1598, in dispatch_request
voteapp_1       |     return self.view_functions[rule.endpoint](**req.view_args)
voteapp_1       |   File "/app/app.py", line 31, in hello
voteapp_1       |     redis.rpush('votes', data)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/client.py", line 1282, in rpush
voteapp_1       |     return self.execute_command('RPUSH', name, *values)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/client.py", line 578, in execute_command
voteapp_1       |     connection.send_command(*args)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 563, in send_command
voteapp_1       |     self.send_packed_command(self.pack_command(*args))
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 538, in send_packed_command
voteapp_1       |     self.connect()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 442, in connect
voteapp_1       |     raise ConnectionError(self._error_message(e))
voteapp_1       | ConnectionError: Error connecting to redis:6379. timed out.
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:20:04 +0000] "POST / HTTP/1.1" 500 291 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:20:07 +0000] "GET /favicon.ico HTTP/1.1" 404 233 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | [2021-03-20 12:20:11,688] ERROR in app: Exception on / [POST]
voteapp_1       | Traceback (most recent call last):
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1982, in wsgi_app
voteapp_1       |     response = self.full_dispatch_request()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1614, in full_dispatch_request
voteapp_1       |     rv = self.handle_user_exception(e)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1517, in handle_user_exception
voteapp_1       |     reraise(exc_type, exc_value, tb)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1612, in full_dispatch_request
voteapp_1       |     rv = self.dispatch_request()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/flask/app.py", line 1598, in dispatch_request
voteapp_1       |     return self.view_functions[rule.endpoint](**req.view_args)
voteapp_1       |   File "/app/app.py", line 31, in hello
voteapp_1       |     redis.rpush('votes', data)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/client.py", line 1282, in rpush
voteapp_1       |     return self.execute_command('RPUSH', name, *values)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/client.py", line 578, in execute_command
voteapp_1       |     connection.send_command(*args)
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 563, in send_command
voteapp_1       |     self.send_packed_command(self.pack_command(*args))
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 538, in send_packed_command
voteapp_1       |     self.connect()
voteapp_1       |   File "/usr/local/lib/python2.7/site-packages/redis/connection.py", line 442, in connect
voteapp_1       |     raise ConnectionError(self._error_message(e))
voteapp_1       | ConnectionError: Error connecting to redis:6379. timed out.
voteapp_1       | 172.19.0.1 - - [20/Mar/2021:12:20:11 +0000] "POST / HTTP/1.1" 500 291 "http://localhost:5000/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
voteapp_1       | [2021-03-20 12:20:36 +0000] [1] [CRITICAL] WORKER TIMEOUT (pid:25)
voteapp_1       | [2021-03-20 12:20:36 +0000] [25] [INFO] Worker exiting (pid: 25)
voteapp_1       | [2021-03-20 12:20:36 +0000] [30] [INFO] Booting worker with pid: 30
resultsapp_1    | Waiting for db
```

Look at the resulting running containers:

```bash
z@Mac-Users-Apple-Computer phase3 % docker container ls
CONTAINER ID        IMAGE                                          COMMAND                  CREATED             STATUS              PORTS                  NAMES
453f7eff9db2        dockersamples/examplevotingapp_vote:before     "gunicorn app:app -b…"   2 minutes ago       Up 2 minutes        0.0.0.0:5000->80/tcp   phase3_voteapp_1
fbf03428710a        redis:3.2                                      "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        6379/tcp               phase3_redis_memory_1
ed85379369d8        dockersamples/examplevotingapp_result:before   "node server.js"         2 minutes ago       Up 2 minutes        0.0.0.0:5001->80/tcp   phase3_resultsapp_1
```

Look at the resulting networks created:

```bash
z@Mac-Users-Apple-Computer phase3 % docker network ls
NETWORK ID          NAME                    DRIVER              SCOPE
[SNIP]
f00ef848d634        phase3_backend          bridge              local
ba59d77e1d55        phase3_frontend         bridge              local
```

I did `ctrl+c` to stop running containers in the log:

```bash
ç^CGracefully stopping... (press Ctrl+C again to force)
Stopping phase3_voteapp_1      ... done
Stopping phase3_redis_memory_1 ... done
Stopping phase3_resultsapp_1   ... done
```

Do `docker compose up -d` you actually be able to look at the containers running with `docker container ps` (results of `docker container ps` not shown below, but it would show the running containers):

```bash
z@Mac-Users-Apple-Computer phase3 % docker-compose up -d
Starting phase3_voteapp_1      ... done
Starting phase3_db_1           ... done
Starting phase3_redis_memory_1 ... done
Starting phase3_resultsapp_1   ... done
Starting phase3_net_worker_1   ... done
```

Note: You can also see the networks that compose created for you using `docker network ls`. Go to `http://localhost:5000` and you should be able to vote for either "dogs" or "cats". Once you've cast your vote refresh and make sure your vote persisted.

I had voted before on the front end and did `docker-compose down` (or actually, `ctrl+c`)and then `docker-compose up` and yes, my votes persisted, due to the docker volume still available from the last time it was built and used by me for the voting app:

![image](https://user-images.githubusercontent.com/17362519/111874327-1f072500-896b-11eb-9787-183805cea3ee.png)

The above of course was at http://localhost:5001 where you see the result service at work as you can look on the number of votes and who voted for what.

Note: a `Dockerfile` wasn't even needed, since we were pulling all our images - already existing images - from Docker hub registry which contained all we needed for our apps.

