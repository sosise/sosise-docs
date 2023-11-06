## 0.8.6 - 6 November, 2023
### Accompanying Sosise-Core version
`0.9.8`

### Updates
- HttpClient added configurable behavior on specific HTTP Response Status Codes
- HttpClient added configurable default exception, to avoid try / catch every time in the repositories
- See more at https://sosise.github.io/sosise-docs/#/documentation/http-client?id=HttpClientRequestConfig

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.5 - 27 September, 2023
### Accompanying Sosise-Core version
`0.9.7`

### Updates
- Helper.assemblePagination() method has been fixed, it now returns numbers instead of strings

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.4 - 11 September, 2023
### Accompanying Sosise-Core version
`0.9.6`

### Updates
- The ./artisan queue:accelerate command has been fixed

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.3 - 11 September, 2023
### Accompanying Sosise-Core version
`0.9.4`

### Updates
- The ./artisan queue:accelerate command has been added. It is necessary to expedite the retry of delayed jobs.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.2 - 6 September, 2023
### Accompanying Sosise-Core version
`0.9.3`

### Updates
- `Server.ts` has been modified to remove the `*/*` type acceptance in body-parser because it was causing issues with using Multer to upload files.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.1 - 17 August, 2023
### Accompanying Sosise-Core version
`0.9.2`

### Updates
- HttpClient timeout bug fix

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.0 - 16 August, 2023 (Breaking change)
### Accompanying Sosise-Core version
`0.9.0`

### Updates
- Upon project startup, configuration information is now displayed on the console. This includes details such as the database connections in use and their respective statuses.
- Refactoring has been performed for improved code structure and organization.

### Upgrade Steps
- If you are using sqlite3 database connection, please take a look at: `https://raw.githubusercontent.com/sosise/sosise/0.8.0/src/config/database.ts`. Default database file `./mydb.sqlite` changed to `process.cwd() + '/mydb.sqlite'`
- `server.ts` file was changed, replace the content of your `server.ts` with `https://raw.githubusercontent.com/sosise/sosise/0.8.0/src/server.ts`
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.7.1 - 11 August, 2023
### Accompanying Sosise-Core version
`0.8.19`

### Updates
- HttpClient requestWithRetry timeout bug fixed 

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.7.1 - 4 August, 2023
### Accompanying Sosise-Core version
`0.8.18`

### Updates
- HttpClient has been implemented [HttpClient Documentation](/documentation/http-client).

### Upgrade Steps
- Copy `src/config/cache.ts` from `https://raw.githubusercontent.com/sosise/sosise/0.7.1/src/config/cache.ts`
- Add to your `.env` and `.env.example` and `.env.testing` new ENVs, you can take a look at `https://raw.githubusercontent.com/sosise/sosise/0.7.1/.env.example`
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.7.0 - 2 August, 2023
### Accompanying Sosise-Core version
`0.8.17`

### Updates
- Introduced cache support with three different drivers: `fs`, `memory`, and `redis`. Detailed information can be found in our [Cache Documentation](/documentation/cache).
- Modified the Artisan ascii art. You can visualize the change by running `./artisan`.

### Upgrade Steps
- Copy `src/config/cache.ts` from `https://raw.githubusercontent.com/sosise/sosise/0.7.0/src/config/cache.ts`
- Add to your `.env` and `.env.example` and `.env.testing` new ENVs, you can take a look at `https://raw.githubusercontent.com/sosise/sosise/0.7.0/.env.example`
- Create following directory `storage/cache`
- Create `.gitignore` file in it with content from `https://raw.githubusercontent.com/sosise/sosise/0.7.0/storage/cache/.gitignore`
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.6.8 - 13 April, 2023
### Accompanying Sosise-Core version
`0.8.15`

### Updates
- Implemented new functionality that generates a start file, named `COMMANDNAME-start`, whenever a command begins.
- Upon the successful termination of a command (without exceptions), an end file, named `COMMANDNAME-end`, is generated.

> This functionality can be used to monitor command execution, for example, by tracking the age of the `COMMANDNAME-end` file.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.6.7 - 20 March, 2023
### Accompanying Sosise-Core version
`0.8.13`

### Updates
- Fixed a bug in sosise-core related to command registration.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.6.6 - 23 February, 2023
### Accompanying Sosise-Core version
`0.8.12`

### Updates
- Minor changes to the creation process of database migrations. Now, timestamps like `created_at` and `updated_at` will automatically be set by the database.
- Removed unused dependencies and fixed spelling errors in the Sosise/Sosise project.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.
- Run `npm remove colors mysql @types/mysql` to remove unneeded dependencies.

## 0.6.5 - 17 November, 2022
### Accompanying Sosise-Core version
`0.8.11`

### Updates
- Added comments for all `sosise-core` methods, providing better contextual understanding. For example, `Helper.storagePath()` now displays: `Path to the storage with ending slash @return e.g. /tmp/myproject/storage/`.
- Introduced several new methods to the `Helper` class. You can find more details in the [documentation](documentation/helpers.md?id=helperpluckmany).

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.6.4 - 17 October, 2022
### Accompanying Sosise-Core version
`0.8.10`

### Updates
- The execution of any command now creates a file in the `storage/framework/%COMMANDNAME%` directory. This functionality can be used for monitoring purposes, ensuring that a command has not frozen or is taking too long to execute.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.
- Run any `command`, which will create a `.gitignore` file within the `storage/framework` directory. You can then push this to your repository.

## 0.6.3 - 30 August, 2022
### Sosise-Core version
`0.8.9`

### Changelog
- Documentation in skeleton updated, it now supports syntax highlighting, copy to clipboard, etc...

### How to upgrade to that version
- Create new project

## 0.6.2 - 08 June, 2022
### Sosise-Core version
`0.8.9`

### Changelog
- `sosise-core` fixed bug with single execution

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`

## 0.6.1 - 22 April, 2022
### Sosise-Core version
`0.8.7`

### Changelog
- Logging config changed, log levels are now supported

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- Replace your `src/config/logging.ts` with the one from `https://raw.githubusercontent.com/sosise/sosise/0.6.1/src/config/logging.ts`
- Add to your `.env` and `.env.example` and `.env.testing` new ENVs, you can take a look at `https://raw.githubusercontent.com/sosise/sosise/0.6.1/.env.example`

## 0.6.0 - 21 April, 2022 (Breaking change)
### Sosise-Core version
`0.8.6`

### Changelog
- Logging config changed, this is done to give the possibility to disable console and/or file logging

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- Replace your `src/config/logging.ts` with the one from `https://raw.githubusercontent.com/sosise/sosise/0.6.0/src/config/logging.ts`
- Add to your `.env` and `.env.example` and `.env.testing` new ENVs, you can take a look at `https://raw.githubusercontent.com/sosise/sosise/0.6.0/.env.example`

## 0.5.0 - 19 April, 2022 (Breaking change)
### Sosise-Core version
`0.8.5`

### Changelog
- Sentry config changed, this is done primarily to disable `tracing`, to prevent high load on sentry instance
- ./artisan ascii update
- ./artisan now renders the documentation link
- Removed unneeded sentry packages

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- Replace your `src/config/sentry.ts` with the one from `https://raw.githubusercontent.com/sosise/sosise/0.5.0/src/config/sentry.ts`


## 0.4.1 - 16 February, 2022
### Sosise-Core version
`0.8.3`

### Changelog
- `sosise-core` bullmq package updated
- bullmq package updated

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`

## 0.4.0 - 11 January, 2022
### Sosise-Core version
`0.8.2`

### Changelog
- Artisan exceptions creation now automatically increments code property, so you do not have to do it manually
- Exceptions now have the property wether they should be sent to sentry or not
- Exceptions now have the property `loggingChannel`
- Logging can now be done to different channels

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- Run `npm update`
- Copy Handler.ts from https://raw.githubusercontent.com/sosise/sosise/0.4.0/src/app/Exceptions/Handler.ts
- Copy logging.ts from https://raw.githubusercontent.com/sosise/sosise/0.4.0/src/config/logging.ts
- Edit all your application exceptions and add:
  - `protected sendToSentry = true;`
  - `protected loggingChannel = 'default';`
- Additionally you can change your Dockerfile from alpine:3.13 to alpine:3.14 (optional)

## 0.3.2 - 11 January, 2022
### Sosise-Core version
`0.7.0`

### Changelog
- Error handling documentation added
- Changed default framework exception codes (2XXX)

### How to upgrade to that version
- Just run `npm install sosise-core@latest` or `npm run update-sosise`
- Please be aware that framework exception codes has been changed, please update your applications if they has used them:
    - DatabaseConfigurationException 3000 -> 2000
    - IOCMakeException 3001 -> 2001
    - FakeMailSendingException 3005 -> 2005
    - MailSendingException 3004 -> 2004
    - TestAccountCreationException 3003 -> 2003
    - ValidationException 3002 -> 2002


## 0.3.1 - 10 January, 2022
### Sosise-Core version
`0.6.8`

### Changelog
- `sosise-core` packages updated
- `sosise` packages updated

### How to upgrade
1. `npm install sosise-core@latest`
2. `npm update`
3. `npm audit fix`

## 0.3 - 19 November, 2021
### Sosise-Core version
`0.6.6`

### Changelog
- Helper methods `dd` and `dump` bug fixes

## 0.2 - 29 September, 2021
### Sosise-Core version
`0.6.5`

### Changelog
- Imports sorted in `sosise-core` (changes just for code beauty)
- `./artisan make:repository` by default creates a repository with an database client. Additionally you could use `-h or --http` flag to create repository prepared for HTTP Requests
- Method `Helper.dd()` improved, it can now accept multiple arguments `Helper.dd('a', 13, {a: 13});`
- Method `Helper.dump()` improved, it can now accept multiple arguments `Helper.dd('a', 13, {a: 13});`
- `./artisan migrate:rollback` Shows which migrations would be rolled back and prompts for a confirmation now, since this operation can be very dangerous.
- Default documentation icon added

### How to upgrade to that version
Just run `npm install sosise-core@latest` or `npm run update-sosise`

## 0.1 - 14 September, 2021
### Sosise-Core version
`0.6.4`

### Changelog
- Added environment param to enable or disable documentation `basic auth`
- `docker-compose.yml` file updated to mount logs
- /docs folder excluded from `editorconfig-checker`
- Database default configuration refactored
- Some other small non breaking changes

### How to upgrade to that version
Just run `npm install sosise-core@latest` or `npm run update-sosise`