# Logging
## Introduction
To help you learn more about what's happening within your application, Sosise provides robust logging services that allow you to log messages to files or console in different formats.

## Configuration
All of the configuration options for your application's logging behavior is housed in the `src/config/logging.ts` configuration file. By default, Sosise will log to console and to files.

## Logging to files
Sosise creates separate log file per one day, e.g. `sosise-2021-01-01.log`

> Please note, that logfiles are not compressed

## How to use logging
```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';

const logger = IOC.make(LoggerService) as LoggerService;
logger.info('Hello world!');
logger.critical('Foo', { bar: 'You also may pass optional arrays or objects as second parameter' });
```