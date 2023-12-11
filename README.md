# MongoDB
### Rules to Live by

- Rule #1: Embed unless there is reason not to (see other rules). Data that is commonly accessed/updated together should be stored together.
- Rule #2: Avoid JOINs whenever possible. But don't be afraid of JOINs that can give a better Schema design.
- Rule #3: Arrays cannot be unbounded. EG: Growing without end. (MongoDB Document size is limited to [16 MB](https://www.mongodb.com/docs/manual/core/document/#document-size-limit) when writing this)
- Rule #4: Needing to access an object on it's own is a compelling reason to not embed it.
- Rule #5: (Obvious but not) Design your Schema based on the unique needs of your Application.

### Anti-Patterns (Patterns included)

- Anti-Pattern: [Massive Arrays](#massive-array-anti-pattern) or unbounded arrays
  - Pattern: 
- Anti-Pattern: 



#### Massive Array Anti Pattern

###### Employee and Building examples

```json
// Building Document
{
  "_id": "abc123",
  "name": "City Hall",
  "city": "Pawnee",
  "state": "IN",
  "employees": [{
    "_id": "zbc321",
    "first": "John",
    "last": "Doe",
    "phone": "1234567890"
  }, {
    "_id": "zbc321",
    "first": "Jane",
    "last": "Doe",
    "phone": "0987654321"
  }, ...]
}
```