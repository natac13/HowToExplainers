# Best Practices For Node.js Apps

## Non-root User

On the official Node Docker images there is a `node` user that is *not* enabled by default as operations such as `npm i -g` and `apk/apt` required `root` access. 

Should switch to the `node` user after the above commands and **before** `npm install` of the app dependencies.

To active the `node` user in the Dockerfile use:
```
USER node
```
Then the `CDM`, `RUN` and `ENTRYPOINT` commands will be run with under the `node` user. **Every** other command is still executed by the `root` user. Therefore I will have to adjust permission on folder/files created by `WORKDIR` and other such commands.
`DIR` and `COPY` use:

```
RUN mkdir -p /path/to && chown -R node:node /path/to

// To get around WORKDIR creating the folder as root 

COPY --chown node:node ./ ./
```

## `node_modules` Executables

When I have a `node_modules` executable that I need to `RUN` or `CMD` in the `Dockerfile` I will have to prefix the package name with the relative path: `./node_modules/.bin/<package>` **UNLESS** I have the following `ENV` in the Dockerfile

```
ENV PATH /path/to/app/directory/node_modules/.bin:$PATH
```

which will add all the `node_modules` executables to the PATH environment variable of the shell.

## Build Kit

To enable the new buildkit prefix a build command with
```bash
DOCKER_BUILDKIT=1 docker build -t <name> --target <buildStage> .
```