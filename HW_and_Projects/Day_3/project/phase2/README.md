

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
