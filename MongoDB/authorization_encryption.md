# Authorization and Encryption

## Roles

Roles are defined as: groups of **privileges**, **actions** over resources, that are grant to **users** over a given _namespace_(**database**).

MongoDB uses role base authorization becuase it provides administrators a high level of responsibility isolation for users' operational tasks. Meaning, since there will be different types of users accessing the DB they will need only certain abilities; DBA - CRUD users, Developer, readWrite data

### Actions

All operation and commands that a user can run are known as **actions**.

### Resources

Are Databases, collections, clusters. Which **actions** are performed on.

### Privileges

Is the abilities to perform the above **action** against an above **resource**
