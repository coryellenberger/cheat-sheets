# MongoDB

###### Cheat Sheet is meant as a quick reference guide.

##### The MongoDB Cheat Sheet came from a collation of information from the MongoDB Youtube Channel.

- [MongoDB Schema Design Best Practices](https://www.youtube.com/watch?v=QAqK-R9HUhc)
- [Schema Design Anti-Patterns - Part 1](https://www.youtube.com/watch?v=8CZs-0it9r4)
- [Schema Design Anti-Patterns - Part 2](https://www.youtube.com/watch?v=mHeP5IbozDU)
- [Schema Design Anti-Patterns - Part 3](https://www.youtube.com/watch?v=dAN76_47WtA)



## Rules to Live by

- Rule #1: Embed unless there is reason not to (see other rules). Data that is commonly accessed/updated together should be stored together.
- Rule #2: Avoid JOINs whenever possible. But don't be afraid of JOINs that can give a better Schema design.
- Rule #3: Arrays cannot be unbounded. EG: Growing without end. (MongoDB Document size is limited to [16 MB](https://www.mongodb.com/docs/manual/core/document/#document-size-limit) when writing this)
- Rule #4: Needing to access an object on it's own is a compelling reason to not embed it.
- Rule #5: (Obvious but not) Design your Schema based on the unique needs of your Application.

<br />

## Anti-Patterns (solutioning Patterns included)

- Anti-Pattern: [Massive Arrays or unbounded arrays](#massive-array-anti-pattern)
  - Solution: [Embedded Approach](#massive-array-solution--embedded-pattern)
  - Solution: [Reference Approach](#massive-array-solution--reference-pattern)
  - Solution: [Reverse Reference Approach](#massive-array-solution--reverse-reference-pattern)
  - Solution: [Extended Reference Approach](#massive-array-solution--extended-reference-pattern)
- Anti-Pattern: [Massive Collections](#massive-collections-anti-pattern)
- Anti-Pattern: [Unnecessary Indexes](#unnecessary-indexes-anti-pattern) 
  - Caveat: [Indexes are good](#indexes-are-good)
- Anti-Pattern: [Bloated Documents](#bloated-documents-anti-pattern)
- Anti-Pattern: [Case-insensitive queries without case-insensitive indexes](#case-insensitive-queries-without-indexes-anti-pattern)

<br />

### Massive Array Anti-Pattern [Source](https://www.youtube.com/watch?v=8CZs-0it9r4)

The below example Schema could result in Unbounded "employees" array. To the point where the Building document for City Hall grows beyond the 16 MB max size and no more employees can be added.

```micronaut-mongodb-json
// Building Document
{
  "_id": Object("abc123"),
  "name": "City Hall",
  "city": "Pawnee",
  "state": "IN",
  "employees": [{
    "_id": Object("zbc321"),
    "first": "John",
    "last": "Doe",
    "phone": "1234567890"
  }, {
    "_id": Object("232fff"),
    "first": "Jane",
    "last": "Doe",
    "phone": "0987654321"
  }, ...]
}
```

<br />

### Massive Array Solution: Embedded Pattern

Data Duplication isn't an issue in terms of storage cost. BUT can be an issue if the Building data is changing regularly. You will end up having to update multiple documents every time which comes at a cost.

```micronaut-mongodb-json
// Employee Collection
[{
  // Employee Document
  "_id": Object("zbc321"),
  "first": "John",
  "last": "Doe",
  "phone": "1234567890",
  "building": {
    "_id": Object("abc123"),
    "name": "City Hall",
    "city": "Pawnee",
    "state": "IN"
  }
}, {
  // Employee Document
  "_id": Object("232fff"),
  "first": "Jane",
  "last": "Doe",
  "phone": "0987654321",
  "building": {
    "_id": Object("abc123"),
    "name": "City Hall",
    "city": "Pawnee",
    "state": "IN"
  }
}, ...]
```

<br />

### Massive Array Solution: Reference Pattern

Similar Drawbacks to the reference pattern above as well as possible issue of unbounded array of Employees.

```micronaut-mongodb-json
// Employees Collection
[{
  // Employee Document
  "_id": Object("zbc321"),
  "first": "John",
  "last": "Doe",
  "phone": "1234567890"
}, {
  // Employee Document
  "_id": Object("232fff"),
  "first": "Jane",
  "last": "Doe",
  "phone": "0987654321"
}, ...]

// Buildings Collection
[{
  "_id": Object("abc123"),
  "name": "City Hall",
  "city": "Pawnee",
  "state": "IN",
  "employees": [Object("zbc321"), Object("232fff")]
}, ...]
```

Example Lookup (JOIN) for above scenario

```javascript
db.buildings.aggregate(
  [
    {
      $lookup: {
        from: 'buildings',
        localField: 'employees',
        foreignField: '_id',
        as: 'employee_info'
      }
    }
  ]
)
```

Results of the Lookup above

```micronaut-mongodb-json
{
  "_id": "abc123",
  "name": "City Hall",
  "city": "Pawnee",
  "state": "IN",
  "employees": ["zbc321", "232fff"],
  "employee_info": [{
    "_id": "zbc321",
    "first": "John",
    "last": "Doe",
    "phone": "1234567890"
  }, {
    "_id": "232fff",
    "first": "Jane",
    "last": "Doe",
    "phone": "0987654321"
  }]
}
```

<br />

### Massive Array Solution: Reverse Reference Pattern

Only Drawback to this approach is the need to aggregate data together using $lookup (JOIN) which if done too often can also be costly.

```micronaut-mongodb-json
// Employees Collection
[{
  // Employee Document
  "_id": Object("zbc321"),
  "first": "John",
  "last": "Doe",
  "phone": "1234567890",
  "building_id": Object("abc123")
}, {
  // Employee Document
  "_id": Object("232fff"),
  "first": "Jane",
  "last": "Doe",
  "phone": "0987654321",
  "building_id": Object("abc123")
}, ...]

// Buildings Collection
[{
  "_id": "abc123",
  "name": "City Hall",
  "city": "Pawnee",
  "state": "IN"
}, ...]
```

Example Lookup (JOIN) for above scenario

```javascript
db.buildings.aggregate(
  [
    {
      $lookup: {
        from: 'employees',
        localField: '_id',
        foreignField: 'building_id',
        as: 'employees'
      }
    }
  ]
)
```

Results of the Lookup above

```jsonpath
{
  "_id": "abc123",
  "name": "City Hall",
  "city": "Pawnee",
  "state": "IN",
  "employees": [{
    "_id": "zbc321",
    "first": "John",
    "last": "Doe",
    "phone": "1234567890",
    "building_id": "abc123"
  }, {
    "_id": "232fff",
    "first": "Jane",
    "last": "Doe",
    "phone": "0987654321",
    "building_id": "abc123"
  }]
}
```

<br />

### Massive Array Solution: Extended Reference Pattern

Duplicate some but not all of the data in the 2 collections. We only duplicate the data that is frequently accessed together. Knowing that the Name and State for the building won't change often.

```json5
// Employees Collection
[{
  // Employee Document
  "_id": Object("zbc321"),
  "first": "John",
  "last": "Doe",
  "phone": "1234567890",
  "building": {
    "name": "City Hall",
    "state": "IN"
  }
}, {
  // Employee Document
  "_id": Object("232fff"),
  "first": "Jane",
  "last": "Doe",
  "phone": "0987654321",
  "building": {
    "name": "City Hall",
    "state": "IN"
  }
}, ...]

// Buildings Collection
[{
  "_id": Object("abc123"),
  "name": "City Hall",
  "city": "Pawnee",
  "state": "IN"
}, ...]
```

<br />

### Massive Collections Anti-Pattern

Below is a screenshot of the same data separated into 365 collections or in 1 collection and the difference in size of the Database and Index.

This is a direct result of the way that the underlying service "WiredTiger" which is used by MongoDB. Creating a separate file for each Collection. (WiredTiger opens all files on startup) Anything beyond 10,000 collections results in performance degradation.

![results](mongodb-anti-pattern-massive-collections.png)

<br />

### Indexes are Good

Indexes allow MongoDB to efficiently query data. If a query does not have an index to support it; MongoDB performs a Collection scan (Scanning every document in a collection). Collection Scans can be very slow. Frequently executed queries should have indexes.

Should we just index every single data point? NO!

<br />

### Unnecessary Indexes Anti-Pattern

Indexes cost storage to exist; and more storage for every document referenced by an index. As the Collections grow the indexes will drain resources.

This is a direct result of the way that the underlying service "WiredTiger" which is used by MongoDB. Creating a separate file for each Index. (WiredTiger opens all files on startup) It is recommended to have no more than 50 indexes per collection.

<br />

### Bloated Documents Anti-Pattern

For commonly accessed documents; you want to target for smaller document sizes. It's ok to have large document sizes for less active collections. [Source](https://youtu.be/mHeP5IbozDU?t=502)

In the example on the video. They have Summary data for a list of Women that will be loaded on the main page of the website and used very often. Then the Detailed view when clicking on one of the listed Women. By moving the Detailed data to it's own collection and referencing between the two; you reduce on the amount of data being cached as commonly accessed documents (EG: Summary view). Then the In Memory cache has more space to also store some of the Detailed documents that are more frequently visited.

This is a direct result of how "WiredTiger" stores and caches documents which is used by MongoDB. Storing all indexes and files on Disk. Then Storing all the indexes and files that are most commonly used in Memory. If the commonly used documents are larger; they will take up more memory.

```json
// InfluentialWomen Collection
[{
  "_id": Object("abc123"),
  "firstname": "Harriet",
  "lastname": "Tulman",
  "birthday": "1999-06-05",
  "occupation": "Astronaut",
  "quote": "lorem ipsum ...",
  "hobbies": ["Tennis", "Writing"]
}, ...]
```

Updated to below to improve the performance of the Preview Data vs Detailed Data

```javascript
// InfluentialWomenSummary Collection
[{
  "_id": Object("abc123"),
  "firstname": "Harriet",
  "lastname": "Tulman",
  "detailed_id": Object("abc124")
}, ...]

// InfluentialWomenDetailed Collection
[{
  "_id": Object("abc124"),
  "firstname": "Harriet",
  "lastname": "Tulman",
  "birthday": "1999-06-05",
  "occupation": "Astronaut",
  "quote": "lorem ipsum ...",
  "hobbies": ["Tennis", "Writing"]
}, ...]
```

In the above case; although there is a duplication of the Firstname/Lastname data. It's not likely to change and if so you can just update both documents. If you had data that was changing frequently; you might not want to duplicate the data...

<br />

### Case-Insensitive Queries without Indexes Anti-Pattern

Suggested to watch the Video covering this topic: [Source](https://youtu.be/mHeP5IbozDU?t=949)

But to sum up: If you build Regex/Non-Regex for Case-Insensitive Queries without having Indexes you will have terrible performance of the query.

Suggested to build Collation Indexes for Case Insensitive queries as it will greatly improve the speed of your Case-Insensitive Queries 
