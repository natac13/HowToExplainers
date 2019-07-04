# Replicate Set Setup

## What is a Replicate Set?

- A replicate set is a set of nodes which are running Mongodb daemon processes, or `mongod`.
- Best to have an **odd** number of nodes; at least for voting purposes.
- Can have up to 50 Mongod nodes with at most 7 voting members.
- the voting members determine which node will be the primary.
- the primary node is the only mongod that handles write request with the other nodes jsut replicating the data.
- I can read from the `Secondary` node with running the command `rs.slaveOk()` before read queries; however can never write to SECONDARY.

## Setup

- Need to create the mongod.conf files for each node that will be in my replicate set.
- example below with all options found [here](https://docs.mongodb.com/manual/reference/configuration-options/#configuration-file-options)

```yaml
storage:
  dbPath: /var/mongodb/db/node1 // line 1
net:
  bindIp: 192.168.103.100,localhost
  port: 27011 // line 2
security:
  authorization: enabled
  keyFile: /var/mongodb/pki/m103-keyfile
systemLog:
  destination: file
  path: /var/mongodb/db/node1/mongod.log // line 3
  logAppend: true
processManagement:
  fork: true
replication:
  replSetName: m103-example
```

_I have indicated above the **3 line** that will need to be changed for each node in the repl set._

**Remember that mongod need permission to access the folders and files I use to start it**

## Setup cont

- create the db storage folders

```bash
mkdir -p /path/to/mongodb/db/<name of db>
```

- Once the `mongod` nodes are running connect to the first node; no auth needed as this is the localhost exception.
- Initiate the repl set `rs.initiate()` **done before I create the root user**
- **Create root User**

```shell
use admin
db.createUser({
  user: <username>,
  pwd: <pass>,
  roles: [
    {role: "root", db: "admin"}
  ]
})
```

- authenticate as the new root user I just created. `db.auth(username, password)`
- run `rs.status()` or `rs.isMaster()`
- Add the other node to the repl set

```shell
rs.add('<DNS/IP-host>:<port>')
```

**When possible use a DNS host name instead of an IP address for adding node to repl set**

### Key file

- a file used between the nodes of the repl set (copied to each server) to verify they are apart of the correct repl set.
- create one with the below command

```shell
sudo mkdir /path/to/pki
sudo chown user:user /path/to/pki
openssl rand -base64 741 > /path/to/pki/key-file
chmod 400 /path/to/pki/key-file
```

**In production** The tutorial says to use X509 Certificate would be what I would use over the generated key file from the tutorial. See [tutorial](https://hackernoon.com/create-a-mongodb-sharded-cluster-with-ssl-enabled-dace56bc7a17) && [article](https://medium.com/@rossbulat/deploy-a-3-node-mongodb-3-6-replica-set-with-x-509-authentication-self-signed-certificates-d539fda94db4) && [gist](https://gist.github.com/natac13/d12718a534bbc39428e8a974c740f323)

### Helpful links

[Key file setup](https://docs.mongodb.com/manual/tutorial/deploy-replica-set-with-keyfile-access-control/)

[Config file Options](https://docs.mongodb.com/manual/reference/configuration-options/#configuration-file-options)

## Restarting

- Restart the `mongod` processes.
- Login the replicate set
- Everything should be good.

## Re-configure a Repl Set

- Login to the mongo shell
- save the `rs.conf()` output to a variable in the shell

```shell
cfg = rs.conf()
// Change what I want
cfg.members[3].votes = 0 // cannot vote
cfg.members[3].hidden = true // hidden from application
cfg.members[3].priority = 0 // cannot become PRIMARY

rs.reconfig(cfg); // will cause an election
```

## Connection to the Repl set

- have to supply the repl set name before the `host`

```shel
mongo --host '<replsetName>/<DNS/IP-host>:<port>' -u username -p pass --authenticationDatabase admin
```

### Connect to a Secondary

```shell
mongo --host '<DNS/IP-host>:<port>' -u username -p pass --authenticationDatabase admin
```

**Notice** the repl set name has been removed. If not the shell will automatically connect me to the PRIMARY.

## oplog

- a collection found in the local `db` of a repl set `mongod` process
- is a capped collection ~5% of available disk space.
- is a colleciton of all the write operations that are performed against the repl set for writing to the db.
- Therefore when a client writes to the primary, this is recorded in the oplog of the primary and of the secondaries copying the data.
- If a secondary goes down it can _self_ recover **only** if it can find a **common** point in the `oplog` of the other nodes and then run all the commands since it has been down to catch up.
- there is a time window of opportunity for the _self recovery_ to happen depending on the amount of disk space and frequency of write operations.
- Use `rs.printReplicationInfo()` to see how long approximately it will take to fill the `oplog` - `log length start to end`

## Write Concerns

- A higher `writeConcern` mean more durability of the data.
- However the write will take longer.
- Levels
  - `0` - client does not wait for any response.
  - `1` (default) - client waits for acknowledgement of the write from only the PRIMARY
  - `>=2` - client wait for acknowledgement from primary and one or more(`n`) secondaries.
  - `majority` - Wait for simple `majority` of the replicate set voting members.

**Options**

- `wtimeout` : < int > does not mean a failure of the write but a failure of the acknowledgement of that write.
- `j` : < true|false > option require that each member receives that write and it is in the journal. With `majority` true this is `true` by default. If `false` the data only needs to be stored(written) in memory before reporting success; when `true` has to be in the journal as well.

## Read Concern

- Like write concern where a client will only be able to read data from the repl set if the `readConcer` has been met.
- 4 `readConcern` levels.
  - local - returns the most recent dat in the cluster
  - available - like local but acts differnt with sharded clusters
  - majority - where only docs written two a majority of nodes are returned.
  - linearizable - like majority but slower
- keep in mind that with SECONDARY reads `local` & `available` are only _fast_ not _latest_, as they are the secondary nodes.

#### What to use?

- Fast & Latest = `local` & `available` - No durability guarantee
- Fast & Safe = `majority` - May not be latest
- Safe & Latest = `linearizable` - Slower and single doc reads **only**

## Read preference

- driver side setting, therefore look into mongoose on how to set.
- 5 modes:
  - `primary`(default) - all reads to the primary node
  - `primaryPreferred` - goes to primary unless it is down for some reason
  - `secondary` - reads only from a secondary
  - `secondaryPreferred` - secondary first than to primary
  - `nearest` - lowest latency to the client. Supports Geo location
- with Secondary reads I may get stale data if the replication has not happened yet. Depends on the delay between the nodes.
- Nodes separated over the earth will have a higher delay between replication.
- **only** the `primary` setting will guarantee upto date data; all others have the potential of stale data.

## With Docker 

https://towardsdatascience.com/how-to-deploy-a-mongodb-replica-set-using-docker-6d0b9ac00e49
