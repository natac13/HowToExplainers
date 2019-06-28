# Views

## Create

`db.createView(<nameOfView>, <collection>, <aggregation-pipeline>)`

Views are created to get a _slice_ of a colleciton. This is acchomplished by **vertically** or **horizontally** slicing the data.

### Vertical Slice

Is when the documents themselves are changed. Done with a `$project` stage.

### Horizontal Slice

Is when the number of documents returned is changed. Done with a `$match` stage.

## Usage

`db.view.find()`

Any query that can be run against a collection can be run against a view; like above.

## Restrictions

- no write operations
  - these are reflections of the data and therfore readonly.
- no indexes
  - are used at the creation of the view but that is it.
- no renaming
  - can drop and recreate without issues
- collation restrictions...
- no `$text`
  - must be the first stage in a pipeline; and since the view executes its pipeline first this does not work.
- no `mapReduce`
- no `geoNear` or `$geoNear`
  - same as with `$text`
- `find()` operation with the following projection operation are not permitted.
  - `$`
  - `$elemMatch`
  - `$slice`
  - `$meta`

## Other

- views are public
- they contain no data themselves
- readonly
