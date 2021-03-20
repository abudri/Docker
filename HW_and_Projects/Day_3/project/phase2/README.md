

# **Phase 2: Mongo and Node with Compose**

For this next phase you'll be creating a `docker-compose.yml` file for a simple Node and Mongo app. Look inside the 'phase2' folder in your skeleton and you can see our application. For this app you’ll create two services: one for the NodeJS application, and one for the MongoDB database. We've provided you with the `Dockerfile` for the custom node image you'll be creating.

`Dockerfile`(given):

```docker
FROM node:8.4

COPY . /app

WORKDIR /app

RUN npm install --silent

EXPOSE 3000/tcp

CMD ["npm", "start"]
```

For this Compose file set your version to '3.3'. Create two services one called `app` for your Node application, and one called `db` for your Mongo backend. We'll start by filling out the Mongo `db` service.

Set your image for the `db` service to be `mongo:3.0.15`. We'll also want to set up a volume for our database so our data persists between `docker compose up`s. Name your volume `mongo-db` and have it pointing to where the Mongo image keeps it's volumes `/data/db`. Remember that if you have a named volume you have to name it under the `volumes` key both inside and outside the service.

Next we'll want to create a custom network we can reference by name to connect our database and Node app. Under and outside of the `services` instruction create a new line for `networks`. Name your custom network "nodemernapp" and use the default `bridge` driver. It'll look something like this:

```
networks:
  nodemernapp:
    driver: bridge
```

We'll come back to `db` in just a second, but let's start building the `app` service first. If you need a reminder on building images in compose check out the [Docker documentation](https://docs.docker.com/compose/compose-file/#build) on the subject. Build a new image based off the `Dockerfile` we provided you. Name the new image `nodeapp`. By default, NodeJS apps run on port 3000, so expose your local port `80` and use port `3000` in the container. Connect the `app` container to the `nodemernapp` network.

Now we'll do the work of connecting the `app` and `db` services. Your `app` service will need to be passed a connection string that it will get from an environment variable called [“MONGO_URI”](https://docs.mongodb.com/manual/reference/connection-string/). We can do this easily utilizing Docker's DNS networking abilities. In your `db` service for Mongo we can create a string alias for the network to be used in the `app` service for connection purposes. You `db` networks should look like this:

```
services:
  db:
    networks:
      nodemernapp:
        aliases:
          - "mongo_db"
```

Now whenever we refer to "mongo_db" we'll have access to our custom network. Now let's build out our Node service and connect them together!

In your `app` service you'll create an environment variable for `MONGO_URI` which will point to:

```
# mongodb://(the name of our alias)/nameofimage
mongodb://mongo_db/nodeapp
```

Let's test it out! Use `docker-compose up` to start up your containers.

Head to `http://localhost:80` and your should see a greeting. Tour around the App and create a couple of users! You should be able to refresh and your users stay right where they are.

To test that your named volume was installed properly use `docker-compose down` and then use `docker-compose up` again. Your users should still exist on your localhost.

Amazing Job! When you are done looking at your work use `docker-compose down -v` to remove your volume, containers and network. Move on to the next phase!

## **Terminal Outputs/Solution**

Final file solutions are of course in the `docker-compose.yml` and `Dockerfile` file in this directory, ie, above.

From our `services` in the `docker-compose.yml` file is `app`, which is built using `build` in the same `docker-compose.yml`file, and built from our `Dockerfile` in the same location, ie, `.`.

The following is the output after running `docker-compose up`:

```bash
z@Mac-Users-Apple-Computer mongo-node-app % docker-compose up
Creating network "mongo-node-app_nodemernapp" with driver "bridge"
Creating volume "mongo-node-app_mongo-db" with default driver
Building app
Step 1/6 : FROM node:8.4
8.4: Pulling from library/node
aa18ad1a0d33: Pull complete
90f6d19ae388: Pull complete
94273a890192: Pull complete
c9110c904324: Pull complete
788d73c0fb6b: Pull complete
f221bb562f24: Pull complete
14414a6a768d: Pull complete
af6d2b2ad991: Pull complete
Digest: sha256:080488acfe59bae32331ce28373b752f3f848be8b76c2c2d8fdce28205336504
Status: Downloaded newer image for node:8.4
 ---> 386940f92d24
Step 2/6 : COPY . /app
 ---> 7560bf45de0a
Step 3/6 : WORKDIR /app
 ---> Running in 5b75889a90e7
Removing intermediate container 5b75889a90e7
 ---> 3864a506a405
Step 4/6 : RUN npm install --silent
 ---> Running in 661180f3e245
added 358 packages in 7.366s
Removing intermediate container 661180f3e245
 ---> 8197682672d4
Step 5/6 : EXPOSE 3000/tcp
 ---> Running in 0717551fdf0c
Removing intermediate container 0717551fdf0c
 ---> 64d87969d88d
Step 6/6 : CMD ["npm", "start"]
 ---> Running in 93550e37da55
Removing intermediate container 93550e37da55
 ---> 2106fe1d13bf

Successfully built 2106fe1d13bf
Successfully tagged nodeapp:latest
WARNING: Image for service app was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Pulling db (mongo:3.0.15)...
3.0.15: Pulling from library/mongo
9e518726c72a: Pull complete
5bec5585393f: Pull complete
67b43c55e1d0: Pull complete
ac73d29bb1af: Pull complete
5f055f855756: Pull complete
5ff164565c7e: Pull complete
14cac0308fdb: Pull complete
d64ca7fda324: Pull complete
a785a68ac06d: Pull complete
749ebc08962c: Pull complete
6941183827c4: Pull complete
4fe0aacd6337: Pull complete
Digest: sha256:2eae1f99d5e49286d5e34a153c6ffaf25b038731e1cfbd786712ced672e8e575
Status: Downloaded newer image for mongo:3.0.15
Creating mongo-node-app_app_1 ... 
Creating mongo-node-app_app_1 ... error
WARNING: Host is already in use by another container
```

I did have some network conflict I had to fix, and then here was rest of regular output:

```bash
z@Mac-Users-Apple-Computer mongo-node-app % docker-compose up
Starting mongo-node-app_app_1 ... done
Starting mongo-node-app_db_1  ... done
Attaching to mongo-node-app_db_1, mongo-node-app_app_1
db_1   | 2021-03-19T19:49:30.741+0000 I CONTROL  [initandlisten] MongoDB starting : pid=1 port=27017 dbpath=/data/db 64-bit host=51ad1676f823
db_1   | 2021-03-19T19:49:30.742+0000 I CONTROL  [initandlisten] db version v3.0.15
db_1   | 2021-03-19T19:49:30.742+0000 I CONTROL  [initandlisten] git version: b8ff507269c382bc100fc52f75f48d54cd42ec3b
db_1   | 2021-03-19T19:49:30.742+0000 I CONTROL  [initandlisten] build info: Linux ip-10-166-66-3 3.2.0-4-amd64 #1 SMP Debian 3.2.46-1 x86_64 BOOST_LIB_VERSION=1_49
db_1   | 2021-03-19T19:49:30.743+0000 I CONTROL  [initandlisten] allocator: tcmalloc
db_1   | 2021-03-19T19:49:30.744+0000 I CONTROL  [initandlisten] options: {}
db_1   | 2021-03-19T19:49:30.760+0000 I JOURNAL  [initandlisten] journal dir=/data/db/journal
db_1   | 2021-03-19T19:49:30.760+0000 I JOURNAL  [initandlisten] recover : no journal files present, no recovery needed
db_1   | 2021-03-19T19:49:30.867+0000 I JOURNAL  [durability] Durability thread started
db_1   | 2021-03-19T19:49:30.867+0000 I JOURNAL  [journal writer] Journal writer thread started
db_1   | 2021-03-19T19:49:30.873+0000 I NETWORK  [initandlisten] waiting for connections on port 27017
app_1  | npm info it worked if it ends with ok
app_1  | npm info using npm@5.3.0
app_1  | npm info using node@v8.4.0
app_1  | npm info lifecycle docker-node-sample@0.0.0~prestart: docker-node-sample@0.0.0
app_1  | npm info lifecycle docker-node-sample@0.0.0~start: docker-node-sa
```
Instructions:
> Head to http://localhost:80 and your should see a greeting. Tour around the App and create a couple of users! You should be able to refresh and your users stay right where they are.

Did so and saw this:

![image](https://user-images.githubusercontent.com/17362519/111875699-7ad4ac80-8971-11eb-8983-b1dd8514de7c.png)

Created some users:

![image](https://user-images.githubusercontent.com/17362519/111875727-99d33e80-8971-11eb-9432-d147d3835a1e.png)

Instructions:
> To test that your named volume was installed properly use `docker-compose down` and then use `docker-compose up` again. Your users should still exist on your localhost.

I had to `ctrl+c` to get out of there and get the terminal prompt back, and then did the above commands:

```bash
^CGracefully stopping... (press Ctrl+C again to force)
Stopping mongo-node-app_db_1  ... done
Stopping mongo-node-app_app_1 ... done
z@Mac-Users-Apple-Computer mongo-node-app % docker-compose down
Removing mongo-node-app_db_1  ... done
Removing mongo-node-app_app_1 ... done
Removing network mongo-node-app_nodemernapp
z@Mac-Users-Apple-Computer mongo-node-app % docker-compose up  
Creating network "mongo-node-app_nodemernapp" with driver "bridge"
Creating mongo-node-app_db_1  ... done
Creating mongo-node-app_app_1 ... done
Attaching to mongo-node-app_db_1, mongo-node-app_app_1
```

And confirmed in the UI that yes, my users I created still exist, due to the use of the named volume created from the `docker-compose.yml` file, which persisted our data.

Finally, I ran the `docker-compose down -v` command to remove the containers as well as the network and volume:

```bash
z@Mac-Users-Apple-Computer mongo-node-app % docker-compose down -v
Removing mongo-node-app_app_1 ... done
Removing mongo-node-app_db_1  ... done
Removing network mongo-node-app_nodemernapp
Removing volume mongo-node-app_mongo-db
```
