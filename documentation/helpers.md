# Helpers
## Introduction
Sosise includes a variety of "helper" functions. You are free to use them in your own applications if you find them convenient.

## Available Methods
### Debugging
- [Helper.dd](#Helper.dd)
- [Helper.dump](#Helper.dump)

### Working with dates
- [Helper.parseDate](#Helper.parseDate)
- [Helper.parseDateTime](#Helper.parseDateTime)
- [Helper.getCurrentDateTime](#Helper.getCurrentDateTime)

## Debugging
### Helper.dd
The `Helper.dd` method dumps a given argument and dies, please note that this method show full depth of the variable if it is a object.

``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.dd({ Hello: 'world' });
```

### Helper.dump
The `Helper.dump` method dumps a given argument, please note that this method show full depth of the variable if it is a object.

``` typescript
import Helper from 'sosise-core/build/Helper/Helper';
Helper.dump({ Hello: 'world' });
```

## Working with dates
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