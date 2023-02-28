# Helpers
## Introduction
Sosise includes a variety of "helper" functions. You are free to use them in your own applications if you find them convenient.

## Available Methods
### Helper.dd
The `Helper.dd` method dumps a given argument and dies, please note that this method show full depth of the variable if it is a object.

``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.dd({ Hello: 'world' });
```

> Can accept multiple arguments

### Helper.dump
The `Helper.dump` method dumps a given argument, please note that this method show full depth of the variable if it is a object.

``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.dump({ Hello: 'world' });
```

> Can accept multiple arguments

### Helper.sleep
The `Helper.sleep` method returns a promise which resolves after given amount of milliseconds.
It's more convenient than writing over and over again:

Without helper:
```typescript
await new Promise((resolve) => setTimeout(resolve, 1000));
```

With helper:
``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
await Helper.sleep(1000);
```

### Helper.parseDate
The `Helper.parseDate` method tries to parse a date, returns in format `"YYYY-MM-DD"`, if no date could be parsed null is returned;

``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.parseDate('25.12.2000');
```

### Helper.parseDateTime
The `Helper.parseDateTime` method tries to parse a date and time, returns in format `"YYYY-MM-DD HH:mm:ss"`, if no date time could be parsed null is returned;

``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.parseDateTime('25.12.2000 13:41:28');
```

### Helper.getCurrentDateTime
The `Helper.getCurrentDateTime` returns current date and time in format `"YYYY-MM-DD HH:mm:ss"`

``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
const currentDateTime = Helper.getCurrentDateTime();
```

### Helper.projectPath
The `Helper.projectPath` returns current project path with ending slash
``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
const projectPath = Helper.projectPath();
```

Returns:
```
/path/to/your/project/your-project-name/
```

### Helper.storagePath
The `Helper.storagePath` returns current storage path with ending slash
``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
const storagePath = Helper.storagePath();
```

Returns:
```
/path/to/your/project/your-project-name/storage/
```

### Helper.pluckMany
The `Helper.storagePath` returns object or array with needed fields
```typescript
import Helper from 'sosise-core/build/Helper/Helper';
const arrayOfObjectsWithNeededFields = Helper.pluckMany([{name: 'alex', age: 10}, {name: 'sharon', age: 11}], ['age']);
const objectWithNeededFields = Helper.pluckMany({name: 'igor', age: 33, birthday: 'foobar'}, ['age', 'birthday']);
```

Returns:
```
[ { age: 10 }, { age: 11 } ]

{ age: 33, birthday: 'foobar' }
```

### Helper.startProfiling
The `Helper.startProfiling` method remembers the time when it was called
```typescript
import Helper from 'sosise-core/build/Helper/Helper';

Helper.startProfiling();
await Helper.sleep(1000);
Helper.stopProfiling();

Helper.startProfiling();
await Helper.sleep(2000);
Helper.stopProfiling();
```

### Helper.stopProfiling
The `Helper.stopProfiling` method displays past time after last `Helper.starProfiling()` method was called
```typescript
import Helper from 'sosise-core/build/Helper/Helper';

Helper.startProfiling();
await Helper.sleep(1000);
Helper.stopProfiling();

Helper.startProfiling();
await Helper.sleep(2000);
Helper.stopProfiling();
```

### Helper.paginateArray
The `Helper.paginateArray` returns part of array on specific page and size
```typescript
import Helper from 'sosise-core/build/Helper/Helper';
const data = [
    { age: 1 },
    { age: 2 },
    { age: 3 },
    { age: 4 },
    { age: 5 },
];
const dataOnPage = Helper.paginateArray(data, 1, 2);
```

Returns:
```
[ { age: 1 }, { age: 2 } ]
```

### Helper.assemblePagination
The `Helper.assemblePagination` returns pagination object
```typescript
import Helper from 'sosise-core/build/Helper/Helper';
const data = [
    { age: 1 },
    { age: 2 },
    { age: 3 },
    { age: 4 },
    { age: 5 },
];
const pagination = Helper.assemblePagination(data, 1, 2);
```

Returns:
```
{ page: 1, pageSize: 2, totalPages: 3, totalElements: 5 }
```