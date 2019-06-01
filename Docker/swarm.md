# Docker **Swarm**

Swarm is a mode introduce in Docker during 2016 after years of seeing what it
takes to run *containers* in production.

Swarm is a clustering solution built inside Docker. It will orchestrate the
communications between multiple nodes(server/VPS/Droplets) all one location.
It makes the issue of dealing with multiple nodes and services for smaller
organization easier. Large companies have teams working to maintain their
deployment nodes; however smaller companies can now use Swarm Mode to handle the
administrative tasks.

Swarm **must** be enabled by the user. Not available be default.

SwarmKit was added in 2016
Stacks and Secrets added in 2017
New commands:
  - `docker swarm`
  - `docker node`
  - `docker service`
  - `docker stack`
  - `docker secret`

## Managers and Workers of a Swarm

Swarms consist of *Managers* and *Workers* nodes(servers/VPS/Droplet). 
Managers will have a local database known as the RAFT, which stores config info
which gives them the *authority* to be a **manager** in the Swarm. Almost sounds
like the Config Server Database/Replica Set when it comes to MongoDB and sharding.
Managers can also be workers. And there is the ablility to promote and demote
nodes to the positions. A **manager** should be thought of as a worker with the
permission to control the swarm.

### Managers

Handle:
  - API: accepts commands from the client and creates service objects
  - Orchestrator: Reconciliation loop for servie objects and creates tasks
  - Allocator: Allocates IP addresses to tasks
  - Scheduler: Assigns nodes to tasks(will evenly spread out tasks across nodes)
  - Dispatcher: Checks in on workers

### Workers

Handle:
  - Worker: Connects to dispatcher to check on assigned tasks
  - Executor: Executes the tasks assigned to worker node

## `docker run` vs `docker service`

With `docker run` it will only start one container, and that container will only
'live' on the node which the Docker cli is talking to; usually the local machine
or a cloud server where the app is hosted; there is no ability to *scale* the
application. `docker server` replaces the `docker run` command when it comes to
Swarms. And gives the ability to add replication and other features that are
useful when it comes to deployments.
The replications from `docker service` are knows as **tasks**, and each task will
launch a docker container. 

## Downed Nodes

When a node goes down in a Swarm replica set, then the *orchestrator* will create
a new node to replace the downed one. `docker service` command puts things into
a *job queue* and will execute them when it can. This can be demonstarted by
starting a new service, and shutting it down, and quickly checking the container
list: as follows.
```bash
docker service create --replica 3 apline ping 8.8.8.8
docker container ls
docker service rm <service name>
docker container ls // will show the 3 containers for a few moments after the above command.
``` 

## Docker Service `--detach`

Starting in version 17.10 of Docker the `--detach` flag on `docker service create` was changed to `false`; now the UI/bash prompt will wait synchronously while the service task are deployed/updated. 

When working with the CLI interactively leave the default of `false` for `--detach`; and with shell scripts use `--detach true`

## Network `overlay`

Swarm uses the `overlay` network driver, which I set with the following command:
```bash
docker network create --driver overlay <name of network>
```

This network allows the nodes of the swarm to talk to each other as if they were on the same machine/VPS/server.