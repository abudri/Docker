
# **Phase Four**

For this phase we'll have you create a `Dockerfile` for a React/Redux app. This app might look a little familiar. Check out the live version [here](https://aa-todos.herokuapp.com/). You'll be creating a `Dockerfile` for this application, and we are going to be getting fancy with this one.

Take a look through the files for this phase and you'll see a React/Redux application - strictly frontend. Meaning there is no server for rendering this application - so we'll use `nginx` in order to see this app on localhost. This presents us with a problem - we want to use `node` as a base image in order to bundle our React code, but we also want to use `nginx` as our server to show our code.

We have two main concerns but we only want to build one image. One way we can get around this is by using [multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/#name-your-build-stages). Once our code has been built using `node` we don't actually require any `node` functionality anymore. So we can use the files that `node` built and hand them to `nginx` to replace the default html that `nginx` renders.

Think about how you'd ordinarily run this kind of application. You would: 

1. Run webpack to bundle your code using `npm start`. 

2. Create a script tag in the `index.html` and then render that page in your browser.

That is basically our aim!

Start off by creating a `.dockerignore` and fill it with what you'll want to ignore in our built image. Then create a `Dockerfile`, for the `FROM` command use a recent node version like `node:8.15-alpine` and name this stage of the build for later reference.

```
FROM node:8.15-alpine as build-stage
```

Next, navigate to a folder named `/app`. Copy over all your files here. Then run the command to install dependencies, and the command to run webpack and create your "bundle.js".

Here’s the Docker multi-stage trick. Next you'll want to create a second `FROM` with the `nginx` image, version `1.15`. Expose port 80 from the image. Next is a normal COPY, but we'll add a `--from=build-stage`. That `build-stage` refers to the name we specified above in the `as build-stage`. Here, although we are in a `Nginx` image, starting from scratch, we can copy files from a previous stage. Copy over your `/app` folder into the place 'nginx' keeps its html information - `/usr/share/nginx/html`

For the last step we've provided you with a `nginx.conf` - this file will basically just take care of making sure all your files aren't overwritten by the `nginx` default configuration. Add this last line to your `Dockerfile`:

```
COPY --from=build-stage /app/nginx.conf /etc/nginx/conf.d/default.conf
```

That's it for your `Dockerfile`! Build and tag your image and try running it on port 80 on your localhost.

Congratulations on some amazing Dockerfiles! Make sure to push all to today's Dockerfiles up to Github and all of the images you built up to Docker Hub as proof of your victory!

## **Terminal Outputs/Solution**

Build the image:

```bash
$ docker build . -t day2_project_phase_4
```

Build and run the container:

```bash
$ docker container run -d -p 80:80 day2_project_phase_4
```

Go to http://localhost:80 and it's there!:

<img src="https://user-images.githubusercontent.com/17362519/112019724-172fb800-8b06-11eb-8bf0-9772a3a7154e.png" width="650;" />

