# Indexes

## Overview

- Indexes are used to increase the speed of our queries.
- The \_id field is automatically indexed on all collections.
- Indexes reduce the number of documents MongoDB needs to examine to satisfy a query.
- Indexes can decrease write, update, and delete performance.

## Important

When building/creating indexes, think **Equality, Sort, Range**. Therefore any query that has the above steps the index field are best place in the bolded order.

## Covered Queries

_Very_ fast queries. Need to include in the query and the project all and only the fields that make up the index. This is because the covered queries data is coming from the index itself; there is not FETCH stage in the winning plan since the projected data in the query was 'covered' with the index.

## Regex & Indexes

With a regular index(non-text) on a username field I can use a regex to query; however to ensure that I utlilize the index on username make sure to inculde a hardcoded beginning as to help the index filter out the list.

## Performance

An index wil reduce write speed by about 6%, **however** will increase read speed by a factor of 10!
