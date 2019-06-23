# Aggregation - M121

[Quick Reference Guide](https://docs.mongodb.com/manual/meta/aggregation-quick-reference/)

## Pipeline

The **aggregation pipeline** can be thought of line a conveyor belt in a factory. Each section of the pipeline is known as a stage; and there can be any number of stages in the pipeline.

## \$match

[\$match Docs](https://docs.mongodb.com/manual/reference/operator/aggregation/match/)

The `$match` operator can be thought of as similar to the `db.collection.find()` method. Therefore I can use the same query operators to 'filter' the collection to a subset of what I specify in the \$match stage of the pipeline.

`$match` on a collection from the database is equivalent to calling `filter()` on an array.

## \$project

[\$project Docs](https://docs.mongodb.com/manual/reference/operator/aggregation/project/)

Although `$project` is similar to the `projection` object with normal queries, in the sense that it can filter out fields or include fields of the returned document(s), it can also do much more than that; it can reassign fields to new keys or create entirely new fields. Can be used as many times as required in the aggregate pipeline. And _should_ be used aggressively to trim and shape the returned value(s).

`$project` on a collection from the database is equivalent to calling `map()` on an array

### Field Path Expressions

Can be used to reassign values to existing fields or to create an entirely new field.

Syntax:

```sh
db.collection.aggregate([
  {
    $project: { _id: 0, 'temperature.value': 1}
  }
]);

db.collection.aggregate([
  {
    $project: { _id: 0, temperature: "$temperature.value"} // Field path expression to reassign the temperature field
  }
])

db.collection.aggregate([
  {
    $project: { _id: 0, surfaceTemp: "$temperature.value" } // Create a new field with the value from temperature.value
  }
])
```
