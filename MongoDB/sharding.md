# Sharding

## What is Sharding

Sharding solves the problem of scaling for a Mongo Database. Vertical scaling is when the server itself is upgraded, ie more RAM, or memory. However this is very expensive and is limited when it comes to cloud based servers; meaning the Host(AWS) will only sell up to a specific size and performance of server.

Therefore horizontal scaling (Sharding) is used which is the process of adding more servers (mongod nodes) and spreading the dataset over the nodes, instead of each node storing the **entire** copy of the dataset.

A group of sharded `mongod` servers is called a **Sharded Cluster**. And therefore each Shard in the cluster would have it's own replicate set. So 4 Shards = 12 mongod servers! This is to insure high availability.

Therefore query become a little more involved. So a type of _routing_ process is used between the client and the mongod servers to determine where the data is which is being queried. This is known as the `Mongos`. And the client connect to the `mongos` instead of each shard node individually.

- Can have `n` number of `mongos` processes.
- `mongos` knows nothing -> it reads meta data of each shard;

  - This meta data is stored on the **Config Server**; which again is a _replicate set_ of its own.
  - Say the data is split by lastNames; this data is stored on the `Config Server` which will inform `mongos` where to route the query.

**Overview**

- `Mongos` - routes queries to the shards
- `Shard Cluster` - contains the shard of replicate sets of `mongod`
- `Config Server` - stores the meta data about each shard
- There is a primary shard; not all collection will have to be sharded. This repl set holds these collections

## When to Shard

- Check if I can vertically scale first, _economically_!
- When we reach the most powerful servers available, maximizing our vertical scale options
- Data sovereignty laws require data to be located in a specific geography
- When holding more than 5TB per server and operational costs increase dramatically.
  - Server should be 2TB - 5TB

## Setting up a Sharded Cluster

**Need at least**

- One replica set of `mongod`
- `mongos`
- Config Server Repl-set

**In production** The tutorial says to use X509 Certificate would be what I would use over the generated key file from the tutorial. See [tutorial](https://hackernoon.com/create-a-mongodb-sharded-cluster-with-ssl-enabled-dace56bc7a17) && [article](https://medium.com/@rossbulat/deploy-a-3-node-mongodb-3-6-replica-set-with-x-509-authentication-self-signed-certificates-d539fda94db4) && [gist](https://gist.github.com/natac13/d12718a534bbc39428e8a974c740f323)

### ConfigServer Repl-set Setup

- Very similar to a normal replica set
- Create needed folders for the database and the config files. **Make sure** mongod can access them; _not_ sudo protected.
- add to the config file

```yaml
sharding:
  clusterRole: configsvr
```

- initiate replica set `rs.initiate()`
- Create super user
- authenticate as that user `db.auth(username, password)`
- add the other nodes in the replica set `rs.add()`

### Mongos

- create a `mongos` config file
- notice that it point to the **entire** `ConfigServer` replica set (`csrs`)
- also does _not_ have its own `dbPath` since it uses the `csrs`

```yaml
sharding:
  configDB: m103-csrs/192.168.103.100:26001,192.168.103.100:26002,192.168.103.100:26003
security:
  keyFile: /var/mongodb/pki/m103-keyfile
net:
  bindIp: localhost,192.168.103.100
  port: 26000
systemLog:
  destination: file
  path: /var/mongodb/db/mongos.log
  logAppend: true
processManagement:
  fork: true
```

### Replica Sets for Shards

- need to change the config file to enable sharding.

```yaml
sharding:
  clusterRole: shardsvr
```

- rolling upgrade the replica set; secondaries first then step down the primary to upgrade it
- `db.shutdownServer()` for each node and change the config file to enable sharding

> The cacheSizeGB: .1 section restricts the memory usage of each running mongod. Note that this is not good practice. However, in order to run a sharded cluster inside a virtual machine with only 2GB of memory, certain adjustments must be made

### Add the Shard To Mongos

```shell
sh.addShard('<replicaSetName>/<host>:<port>')
```

## Config Server Database

- should never write to the `configServerDB` unless instructed my MongoDB support.
- accessed through `mongos` -> `show dbs`
- collection include but not limited to:
  - databases: each doc is a database
  - collections: shows collections which have been sharded, and on what key.
  - chunks: returns a doc for each 'chunk' / split of data. Will have the inclusive min and exclusive max values that the chunk is split on(chunk ridge)
  - monogs - returns info about all the mongos processes connected to the cluster; includes the mongo version.

## Shard Key

- will determine how the data is distributed in a cluster
- indexed field on a document
- must be present on all documents in a collection
- should support the majority of queries
- **immutable** and cannot change them after post-sharding
- cannot unshard a collection
- cannot update any value in a document of the shard key.
- test in development before production
- ObjectID field is a monotonical change so it is **not** a good shard key.
- Best if the queries include the shard key
- can be on a compound index ie `{ _id: 1, otherField: 1 }`

**What makes a Good Shard Key**

1. High _Cardinality_ = **many** unique shard key values.
2. Frequency = **low repetition** of a given unique shard key value; inserting documents yield many different values on the shard key field.
3. (NOT) Monotonical change = a field where the input is based on a predictable change. Like a date or stopwatch timer is not a good shard key since the data will group in a periodical way. Therefore **not** monotonical

**Jumpbo Chunks**

A jumbo chunk is one where the upper bound value and the lower bound value are the same. ie `{ lastName: 'Campbell' } -> { lastName: 'Campbell }`. These jumbo chunks are undesirable since they cannot be split up across the shards in the cluster. To avoid these jumbo chunk I can use a more compound shard key. ie `{ lastName: 1, _id: 1 }`. Notice the use of the monotonical \_id field. This is alirght as long as the monotonically changing field is **not** the first field in the shard key.

**Hashed Shard Key**

- Provides a vastly more even distribution of data on a Monotonically changing shard key.
- Does not work on querys for:
  - fast sorts
  - targeted ranges of the shard key, since, once hashed the documents will be spread out in a non-linear order.
  - Geographically isolated workloads
- Hashed shard keys are on single fields, non-array!
- non-compound index

## How To Shard A Collection

1. Use `sh.enableSharding('<database>')` for the specific db.
2. `db.collection.createIndex({<key>: < 1|0 >}) // 1 = ascending`
3. `sh.shardCollection('<db>.<collection>', { <shardKey>: < 1|0 > })` to shard a collection

## Chunks

- lower bound number is inclusive
- upper bound number of a chunk is exclusive
- default size of a chunk is 64MB; can be 1MB - 1024MB, defined at run time.
- the shard key cardinality, input frequency and chunk size will determine the number of chunks in the cluster.

## Balancing

- Almost all automatic and will require very minimal input from me.
- The primary node of the Config Server is responsible for running the **Balancer**

## Mongos

It is a good idea to reduce latency to place the mongos on the same machine as the application.
