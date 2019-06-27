# Views

## Create

`db.createView(<nameOfView>, <collection>, <aggregation-pipeline>)`

Views are created to get a _slice_ of a colleciton. This is acchomplished by **vertically** or **horizontally** slicing the data.

### Vertical Slice

Is when the documents themselves are changed. Done with a `$project` stage.

### Horizontal Slice

Is when the number of documents returned is changed. Done with a `$match` stage.
