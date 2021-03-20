# **Phase 3: The Voting App**

We will be creating the `docker-compose.yml` file for a Voting Application for the world's toughest question: "cats or dogs?". Users can cast their votes, which will be saved, and admin users can see the votes cast. This is an application based on micro-services architecture, consisting of 5 individually simple services.

![https://assets.aaonline.io/Docker/voting.png](https://assets.aaonline.io/Docker/voting.png)

1. **Voting-App:** The frontend of the application written in Python, used by users to cast their votes.
2. **Redis:** The in-memory database, used as intermediate storage.
3. **Worker:** A .Net service, used to fetch votes from Redis and store in a Postgres database.
4. **DB:** A PostgreSQL database, used as the database.
5. **Result-App:** Frontend of the application written in Node.js which displays the voting results.

All of the images you need are on Docker Hub. We will creating the services including exposing ports, volumes, and connecting containers thorough our own networks. Let's get started!

# **Services**

Start off by defining your Compose version as '3'. You'll be creating 5 separate services and pulling their images from Docker Hub. The **names of these services do matter** for the images, so make sure you use the names for each service as described below. You'll be creating two [custom networks](https://docs.docker.com/compose/networking/#specify-custom-networks) "frontend" and "backend".

# **Breakdown of the Five Services**

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
