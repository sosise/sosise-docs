# Logging

## Introduction

Sosise provides powerful logging services to help you monitor and understand what's happening within your application. You can log messages to files or the console in different formats, giving you valuable insights into your application's behavior.

## Configuration

The logging behavior for your application is controlled by the `src/config/logging.ts` configuration file. By default, Sosise logs messages to both the console and log files.

## Logging to Files

Sosise creates separate log files for each day, named like `sosise-2021-01-01.log`.

> Note: Log files are not compressed.

## How to Use Logging

You can access the logging service by importing and using the `LoggerService` provided by the Inversion of Control (IOC) container.

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';

const logger = IOC.make(LoggerService);
logger.info('Hello world!');
logger.critical('Foo', { bar: 'You can also pass optional arrays or objects as the second parameter' });
```

## Log Output Depends on Your `APP_ENV`

The log format varies based on the `APP_ENV` environment variable:

- `APP_ENV = local`: Logs are pretty printed.
- `APP_ENV = staging`: Logs are printed in JSON format.
- `APP_ENV = production`: Logs are printed in JSON format.

## Logging to Different Channels

Sosise allows you to log information to different files (channels). You can define additional channels in the `src/config/logging.ts` configuration file.

### How to Use Channels

You can specify the channel when logging a message.

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';

const logger = IOC.make(LoggerService);
logger.info('Hello world!', null, 'specialchannel');
```

> Your exceptions also have the option `loggingChannel`, which can be used to log different types of exceptions to different channels. This provides a way to categorize and manage logs more effectively.