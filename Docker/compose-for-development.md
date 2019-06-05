# Docker-Compose for Development

## Version 2 vs 3

Too be clear version 3 is not a direct replacement to verison 2.

Version 2 even has a few features that are not in verison 3 since they do not work well with swarms quite yet(first half 2019).

If I need to work with swarms and stacks for deployment then verison **3** is what I _need_ to use.

## Database storage

**Best** to be done with a named volume. This will insue the data remains after a `docker-compose down`.

**Do not** bind mount the database

## Source Code - Bind Mounting

Source code is best to be bind mounted into the container. The bind mount **shall** always be a relavtive path to the `docker-compose.yml` file. [Bind-Mount Docs](https://docs.docker.com/storage/bind-mounts/)

```bash
// CLI version
--mount type=bind,source="$(pwd)"/target,target=/app
```

```yml
// docker-compose.yml file
volumes:
  - .:/app
```

## node_modules in Docker Images

Best not to include the `node_modules` directory in the docker image. Therefore **include** `node_modules` in the `.dockerignore` file.

### Bind-mount `node_modules`

The following solution will have the result of being able to use the `node_modules` directory on the local host for local development _outside_ the container; with the `node_modules` directory _inside_ the container for use by the container itself. Therefore the container will be able to run on any OS.

The image of an app will build and run successfully when the `node_modules` is in the same directory(not following step #1). However when trying to run the container with a `docker-compose.yml file`(for development) the source code bind mount will _include_ the local host's `node_modules`; this maybe undesired if not on a linux machine or do not have the `node_modules` installed on the local computer. Therefore the app may not run unless the following steps are taken.

**Note** with the following I do not even need to install the `node_modules` on my local machine _as long as_ I **do not** plan on developing the app outside of the container; meaning not runing the app with `docker-compose up` but locally as before Docker. However I can run `npm install` locally and have the advantage of both options!!

**Steps**

1. Move the `node_modules` folder up a directory in the `Dockerfile`. This folder will be used by the app in the container since part of the node standard is to repeatly look up directories for until a `node_modules` folder is found and use that location:

```bash
mkdir -p /node/app && chown -R node:node /node

# node_modules
WORKDIR /node

RUN npm install && npm cache clean --force

# Source Code
WORKDIR /node/app

COPY . .
```

2. Create an anonymous bind-mount in the `docker-compose.yml` file.

```yml
volumes:
  # if a node_modules folder exist here it will be used to run the app; usually undesired.
  - .:/node/app
  # creates anonymous volume to hide localhost node_modules folder
  - /node/app/node_modules
```

Therefore the `node_modules` inside the container in the `/node/app` directory will be empty since there is no bind-mount to the localhost(anonymous volume) and there has been no `npm install` run in the `docker-compose.yml` file. Remember the image has already installed its `node_modules` dir in the `/node` folder during the build process.

## .dockerignore File

Will be very similar to the `.gitignore` file

## How to Run `npm` commands?

Developing inside the container means I need to be able to run `npm` commands(ie, `npm install <package>`) inside that conatiner/service.

- `docker-compose run ...` will spin up a new container, with all the setup stuff in the file, and run the command given against that service(the app) [docs](https://docs.docker.com/compose/reference/run/). This could be used to instal `node_modules` inside the service to run a conatiner with `docker-compose up`.

- `docker-compose exec` is used with an already running container, which will create a new shell/connection to that conatiner which will not interrupt anything else. Therefore I would have already run a `docker-compose up` on the _service_

## File Monitoring

When using `nodemon` or `webpack-dev-server` or any other service that watches files and restarts a server it is best to have the `docker-compose.yml` file start that service with the `command:` configuration. This will override the `CDM` in the Dockerfile.

## Tini for Proper Shut Down

**Only available in [Version 2.x](https://docs.docker.com/compose/compose-file/compose-file-v2/#init) or [Version 3.7](https://docs.docker.com/compose/compose-file/#init)**

When running docker containers I use `docker container run ... --init <image>` to add [Tini](https://github.com/krallin/tini). In order to do this with `docker-compose.yml` I need to include:

```yml
servicename:
  image: abcd1234
  init: true
  ports:
    - 3000:3000
  # ... rest of docker-compose.yml file
```

## Environment Variables

For any `Express.js` app I need to ensure that the host is set to `0.0.0.0` instead of localhost that I am used to doing:

```js
const app = express();
const port = process.env.PORT || 3000;
const host = process.env.HOST || '0.0.0.0';
app.listen(port, host).then(...)
```

Therefore in the `docker.compose.yml` file:

```yml
version: "2.4"

services:
  app:
    image: <image name>
    build:
      context: .
      target: dev
    # Tini
    init: true 
    ports:
      - 3000:3000
    volumes:
      - .:/app
      - /node/app/node_modules
    # optionally use the values in the env file to populate the ${}s
    # env_file:
    #   - .env
    environment:
      - HOST=${HOST:-0.0.0.0}

```