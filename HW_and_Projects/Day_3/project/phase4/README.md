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

## **Terminal Outputs/Solution**

Files solution is of course in the `docker-compose.yml`, `Dockerfile`, and `.dockerignore` files in this directory, ie, above.

When the files were ready and I ran `docker-compose up -d` command to start the application, I headed to http://localhost:8080 and saw this after following the instructions, note that I used our service name `db` for postgres, from the `docker-compose.yml` file, for `Host` in `ADVANCED OPTIONS`:

<img src="https://user-images.githubusercontent.com/17362519/111874516-18c57880-896c-11eb-84d7-313f10b99aff.png" width="650" />

Next, Next drupal built the site:

<img src="https://user-images.githubusercontent.com/17362519/111874527-2b3fb200-896c-11eb-854e-a0da5c77cbe9.png" width="650" />

Next I configured the site with "whatever" - name of site, email, etc - and was finally taken to this page:

<img src="https://user-images.githubusercontent.com/17362519/111874566-46122680-896c-11eb-9815-95ee0cc62edb.png" width="650" />

Then went to the website comes up, clicked on `Appearance` in the top bar, saw a theme called `Bootstrap` and that's expected, the theme we  were attempting to import! That's the one we added with our custom Dockerfile. It's just not installed yet:

<img src="https://user-images.githubusercontent.com/17362519/111874630-74900180-896c-11eb-98a1-b3e5fd444964.png" width="650" />

Clicked `Install and set as default`:

<img src="https://user-images.githubusercontent.com/17362519/111874646-82458700-896c-11eb-9a22-3fec1361c765.png" width="650" />

Then clicked `Back to site` (in the top left) and the website interface looked different/nicer:

<img src="https://user-images.githubusercontent.com/17362519/111874660-91c4d000-896c-11eb-891d-bb6cdafb3c3b.png" width="650" />

**From instructions(roughly)**:
> You've successfully installed and activated a new theme in your own custom image without installing anything on your host other than Docker! If you exit (`ctrl-c`) and then `docker-compose down` it will delete containers, but not the volumes, so on next `docker-compose up` everything will be as it was. 

But, we actually ran this in `-d` for detached mode, so my terminal is already there, and `ctrl-c` isn't necessary.  I ran `docker-compose down` and it stopped my containers, but woops, **it also deleted them  after stopping them**. I have to go through the same setup again. so it is better to run in regular `docker-compose up` for non-detached mode, then `ctrl-c` and then you can docker-compose up and see things still as is.  Note, I had to also do `docker-compose down -v` to remove the volume, because without that, some of the data stored in the volume gave me funky website behavior when bringing back the containers with `docker-compose up`, before that `-v` removal of the volume.

```bash
z@Mac-Users-Apple-Computer phase4 % docker-compose down
Stopping phase4_drupal_app_1 ... done
Stopping phase4_db_1         ... done
Removing phase4_drupal_app_1 ... done
Removing phase4_db_1         ... done
Removing network phase4_default
```

So I did remove the volume and the containers and then did the setup again with `docker-compose up` and so I have the log here, and then did `ctrl-c` and then it stopped the containers, **without removing them(!)**, and then here I am:

```bash
drupal_app_1  | 172.23.0.1 - - [20/Mar/2021:13:09:36 +0000] "GET /sites/default/files/js/js_UY8Qfm89Bf2Yx3GP05FSvqn9NCxdWTBt2BBH4r9BcAg.js HTTP/1.1" 200 61667 "http://localhost:8080/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
drupal_app_1  | 172.23.0.1 - - [20/Mar/2021:13:09:36 +0000] "POST /contextual/render HTTP/1.1" 200 1740 "http://localhost:8080/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 11_1_0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
^CGracefully stopping... (press Ctrl+C again to force)
Stopping phase4_db_1         ... done
Stopping phase4_drupal_app_1 ... done
```
I run `docker-compose up` and have my site back:

<img src="https://user-images.githubusercontent.com/17362519/111874737-ef591c80-896c-11eb-8098-8b23359e0bc0.png" width="650" />

Finally, I stopped with `ctrl-c` and then removed the container and the volume:

```bash
0) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.82 Safari/537.36"
^CGracefully stopping... (press Ctrl+C again to force)
Stopping phase4_db_1         ... done
Stopping phase4_drupal_app_1 ... done
z@Mac-Users-Apple-Computer phase4 % docker-compose down -v
Removing phase4_db_1         ... done
Removing phase4_drupal_app_1 ... done
Removing network phase4_default
Removing volume phase4_drupal-data
```

#### **Bonuses: Add Health Checks!**

Health checks are of course in the `docker-compose.yml` file of this directory. 

I added the postgres health check based on this simple example: https://github.com/peter-evans/docker-compose-healthcheck

