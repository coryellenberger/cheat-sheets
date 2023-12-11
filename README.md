# MongoDB
### Rules to Live by

- Rule #1: Embed unless there is reason not to (see other rules) EG: If you have a Teacher with 20 classes; embed the classes as part of the Teacher Document. If the classes are boundless (never ending) then use reverse referenced document. (Class Document with TeacherID referencing the Teacher Document)
- Rule #2: Avoid JOINs whenever possible. But don't be afraid of JOINs that can give a better Schema design.
- Rule #3: Arrays cannot be unbounded. EG: Growing without end. (MongoDB Document size is limited to [16 MB](https://www.mongodb.com/docs/manual/core/document/#document-size-limit) when writing this)
- Rule #4: Needing to access an object on it's own is a compelling reason to not embed it.
- Rule #5: (Obvious but not) Design your Schema based on the unique needs of your Application.
