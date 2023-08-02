# Helpers

## Introduction

Sosise provides a collection of convenient "helper" functions that you can use in your applications to streamline your development process.

## Available Methods

### Helper.dd

The `Helper.dd` method dumps the given argument and terminates the execution. It displays the full depth of the variable, especially useful for objects.

```typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.dd({ Hello: 'world' });
```

> You can pass multiple arguments to this method.

### Helper.dump

The `Helper.dump` method dumps the given argument, showing the full depth of the variable, especially useful for objects.

```typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.dump({ Hello: 'world' });
```

> You can pass multiple arguments to this method.

### Helper.sleep

The `Helper.sleep` method returns a promise that resolves after a given number of milliseconds. It is more convenient than manually using `setTimeout`.

Without helper:

```typescript
await new Promise((resolve) => setTimeout(resolve, 1000));
```

With helper:

```typescript
import Helper from 'sosise-core/build/Helper/Helper';
await Helper.sleep(1000);
```

### Helper.parseDate

The `Helper.parseDate` method attempts to parse a date and returns it in the format `"YYYY-MM-DD"`. If no date can be parsed, it returns `null`.

```typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.parseDate('25.12.2000');
```

### Helper.parseDateTime

The `Helper.parseDateTime` method attempts to parse a date and time and returns it in the format `"YYYY-MM-DD HH:mm:ss"`. If no date-time can be parsed, it returns `null`.

```typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.parseDateTime('25.12.2000 13:41:28');
```

### Helper.getCurrentDateTime

The `Helper.getCurrentDateTime` returns the current date and time in the format `"YYYY-MM-DD HH:mm:ss"`.

```typescript
import Helper from 'sosise-core/build/Helper/Helper';
const currentDateTime = Helper.getCurrentDateTime();
```

### Helper.projectPath

The `Helper.projectPath` returns the current project path with a trailing slash.

```typescript
import Helper from 'sosise-core/build/Helper/Helper';
const projectPath = Helper.projectPath();
```

Returns:

```
/path/to/your/project/your-project-name/
```

### Helper.storagePath

The `Helper.storagePath` returns the current storage path with a trailing slash.

```typescript
import Helper from 'sosise-core/build/Helper/Helper';
const storagePath = Helper.storagePath();
```

Returns:

```
/path/to/your/project/your-project-name/storage/
```

### Helper.pluckMany

The `Helper.pluckMany` returns an object or array with only the specified fields.

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

### Helper.startProfiling & Helper.stopProfiling

The `Helper.startProfiling` method remembers the time when it was called, and the `Helper.stopProfiling` method displays the time elapsed since the last call to `Helper.startProfiling`.

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

The `Helper.paginateArray` returns a part of an array based on the specified page and size.

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

The `Helper.assemblePagination` returns a pagination object based on the given data, current page, and page size.

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

These helper methods can greatly simplify your development process by providing useful functionalities for various common tasks.