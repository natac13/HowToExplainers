# Indexes

## Overview

- Indexes are used to increase the speed of our queries.
- The \_id field is automatically indexed on all collections.
- Indexes reduce the number of documents MongoDB needs to examine to satisfy a query.
- Indexes can decrease write, update, and delete performance.

## Important

When building/creating indexes, think **Equality, Sort, Range**. Therefore any query that has the above steps the index field are best place in the bolded order.
