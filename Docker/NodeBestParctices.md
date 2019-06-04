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