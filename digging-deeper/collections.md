# Collections
## Introduction
Sosise uses `collect.js` library to work with collections.

> Please visit [collect.js website](https://collect.js.org) for full documentation.

## What are collections
Like in `Laravel` collections provide a convenient wrapper for working with arrays and objects. For example, check out the following code. We'll use the `collect.js` library to find all id's between specific range:

```typescript
const arrayOfObjects = [
    { id: 1 },
    { id: 2 },
    { id: 3 },
    { id: 4 },
    { id: 5 },
];
let collection = collect(arrayOfObjects);
collection = collection.whereBetween('id', [2, 4]);
```

As you can see, the `collection.js` library allows you to chain its methods to perform fluent database-like operations and not only. In general, collections are immutable, meaning every Collection method returns an entirely new Collection instance.