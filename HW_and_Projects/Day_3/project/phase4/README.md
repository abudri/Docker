# **Phase 4: Build Your Own Dockerfile and Compose File**

One of the best things about Docker is that you can work with unfamiliar technologies super easily because you don't have to spend hours setting up a development environment. For this next phase we'll be using a service called [Drupal](https://hub.docker.com/_/drupal/) a free and open-source content-management framework written in PHP. So even though you've probably never worked with Drupal you can have it up and running quickly.

We'll write a custom `Dockerfile` and start your app with Docker Compose. When configured properly, this will let you build a custom image and start everything with `docker compose up` including storing important `db` and `config` data in volumes so the site will remember your changes across Compose restarts.

## **Part A: Dockerfile**

For this first part, we will be: 

1. Creating a custom `Dockerfile` for the `drupal` image 

2. Downloading and then using `Git` to install a custom Drupal HTML theme([Bootstrap](https://git.drupalcode.org/project/bootstrap)).

Start by creating a `.dockerignore` and ignoring the usual things. Then create a `Dockerfile` that will be built from `drupal:8.6`. Now we know we'll need to install Git for the next part but the `drupal:8.6` image doesn't currently have it - meaning we'll need to download it!

Start off by using `RUN` to run the `apt` package manager command to install git: `apt-get update && apt-get install -y git`. Whenever you download anything inside a docker container the installation will almost always leave a lot of extra files which we won't want in our image. Clean up after your installation by adding the command `rm -rf /var/lib/apt/lists/*`. Make sure to use `\` and `&&` properly! Take a look back at the Dockerfile docs if you need a reminder on syntax.

Next step is to change your working directory(`WORKDIR`) within the container to access where Drupal keeps the html templates - `/var/www/html/themes`. Then use git to clone in our chosen theme using the command:

```
git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupal.org/project/bootstrap.git
```

**Note:** The reason we are telling git `--single-branch --depth 1` is because we only want the most recent version of this one branch. This saves you a **ton** of time over downloading all the branches so it's a handy way to avoid extra bloat in your image.

Now we just need to solve one last problem. Something you might encounter while working with Docker is having to sometimes [change file permissions](https://www.hostingadvice.com/how-to/change-file-ownershipgroups-linux/). The files we just used Git to download have been put in the directory under the ownership of `root`. However the `drupal` image is expecting all the files it will be running to be under the ownership of the `www-data` user. Meaning we will need to **change the permissions** of these files.

We will use the `[chown](https://linux.die.net/man/1/chown)` command to change the file ownership of these permissions. Chain the following command to the last `RUN` statement in your Dockerfile - `chown -R www-data:www-data bootstrap`. When you use `chown -R` you are saying you want to change the owner for all files (including directories) - which will allow `drupal` to access all the files in the `bootstrap` directory properly.

Nicely done! Now let's build it up using Compose.

## **Part B: Compose File**

So now we will build our custom `drupal` image in a `docker-compose.yml`. This Compose file will be pretty basic so use version '2'. We'll be using one custom Drupal service and one PostgreSQL service.

Build the custom Drupal image using the `Dockerfile` you previously created, and name it `<yourusername>/custom-drupal`. Expose your local port `8080` pointing at port `80` in the container.

For the PostgreSQL service, you'll be using both an environment variable and a volume. Use the image for `postgres:9.6` and set your environment variable for the database password using `POSTGRES_PASSWORD`. Add a named volume for `drupal-data:/var/lib/postgresql/data` so the database will persist across Compose restarts.

## **Part C: Putting it Together**

Use the `docker-compose up -d` command to start your application. Head to `http://localhost:8080` and you'll see this nice UI to configure `drupal`. At this point all we want to know is if the `HTML` Bootstrap theme we downloaded in our custom `Dockerfile` is available. Click the obvious options, and use the "simple" profile, until you get to the `database` setup page.

Here select `PostgreSQL` because that is obviously what you are using. Now the following list corresponds directly to what you used in your `docker-compose.yml`.

1. `database name` - since we didn't specify the default name is 'postgres'.
2. `Database password` - will be what you set the postgres password environment variable to
3. `Database username` - defaults to `postgres`
4. Click `Advanced Options` - here the **host** name will be the **name of your postgres service**.

Next drupal will build your site, which will take a moment. On the next page you'll encounter a 'configure site' page which you can just fill in with whatever you please since you won't be checking this site in the future (the email boxes will need an `@` sign). After that you should have access to the main Drupal service.

When the website comes up, click on `Appearance` in the top bar, if you see a theme called `Bootstrap` then we are successful! That was the theme you were attempting to import! That's the one we added with our custom Dockerfile.

Click `Install and set as default`. Then click `Back to site` (in the top left) and the website interface should be different. You've successfully installed and activated a new theme in your own custom image without installing anything on your host other then Docker! If you exit (`ctrl-c`) and then `docker-compose down` it will delete containers, but not the volumes, so on next `docker-compose up` everything will be as it was. To totally clean up volumes, add `-v` to the `down` command.

Now you have all the tools you need to host your own projects locally! Make sure to add the images you built to Docker Hub, and commit the Compose files you made to Github.

## **Bonuses: Add Health Checks!**

Complete deploying your projects to Heroku using Docker before attempting this phase.

Show how devoted you are to good testing by adding [health checks](https://docs.docker.com/compose/compose-file/#healthcheck) to all the `docker_compose.yml` files you've written today. Make sure your health checks work and everything still runs properly.
