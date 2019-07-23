# User Defined Roles

You will be getting into User Defined Roles if you need that level of authorization validations. Specifically the resource field below

```
use admin
db.createRole(
   {
     role: "customRoleName",
     privileges: [
       { resource: { db: "test", collection: "one" }, actions: [ "find", "insert" ] }
     ],
     roles: []
   }
)
```

Therefore the follow role will give a user access to the collection called **one** on the **test** database, with the ability to only do the **find** or **insert** actions.

See the doc for more info.
[Create a User defined Role](https://docs.mongodb.com/manual/tutorial/manage-users-and-roles/#create-a-user-defined-role)
[Privilege Actions](https://docs.mongodb.com/manual/tutorial/manage-users-and-roles/#create-a-user-defined-role)
[Reasoure Document](https://docs.mongodb.com/manual/reference/resource-document/#resource-document)
