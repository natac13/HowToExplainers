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

```
-v .:/app

--mount type=bind,source="$(pwd)"/target,target=/app

volumes:
  - .:/app
```

## node_modules in Docker Images

Best not to include the `node_modules` directory in the docker image. Therefore **include** `node_modules` in the `.dockerignore` file.

### Bind-mount `node_modules`

The following solution will have the result of being able to use the `node_modules` directory on the local host for local development _outside_ the container; with the `node_modules` directory _inside_ the container for use by the container itself.

The image of an app will build and run successfully when the `node_modules` is in the same directory(not following step #1). However when trying to run the container with a `docker-compose.yml` file the source code bind mount will _include_ the local host's `node_modules`; this maybe undesired if not on a linux machine or do not have the `node_modules` installed on the local computer. Therefore the app may not run unless the following steps are taken.

**Steps**

1. Move the `node_modules` folder up a directory in the `Dockerfile`. This folder will be used by the app in the container since part of the node standard is to repeatly look up directories for until a `node_modules` folder is found and use that location:

```
mkdir -p /node/app && chown -R node:node /node

# node_modules
WORKDIR /node

RUN npm install && npm cache clean --force

# Source Code
WORKDIR /node/app

COPY . .
```

2. Create an anonymous bind-mount in the `docker-compose.yml` file.

```
volumes:
  # if a node_modules folder exist here it will be used to run the app; usually undesired.
  - .:/node/app
  # creates anonymous volume to hide localhost node_modules folder
  - /node/app/node_modules
```

Therefore the `node_modules` inside the container in the `/node/app` directory will be empty since there is no bind-mount to the localhost(anonymous volume) and there has been no `npm install` run in the `docker-compose.yml` file. Remember the image has already installed its `node_modules` dir in the `/node` folder during the build process.

## .dockerignore File

Will be very similar to the `.gitignore` file
