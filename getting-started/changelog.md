## 0.6.8 - 13 April, 2023
### Sosise-Core version
`0.8.15`

### Changelog
- When a command starts a start file will now generated `COMMANDNAME-start`
- When a command ends (without exceptions) a end file will now generated `COMMANDNAME-end`

> You can use this functionality to monitor command execution, e.g. monitor the file age of `COMMANDNAME-end`

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`

## 0.6.7 - 20 March, 2023
### Sosise-Core version
`0.8.13`

### Changelog
- Command registration bug fixed in sosise-core

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`

## 0.6.6 - 23 February, 2023
### Sosise-Core version
`0.8.12`

### Changelog
- When you create a database migration now, it slightly changed. Timestamps like `created_at` and `updated_at` now tells to DB to set the date automatically.
- Sosise/Sosise project: removed unused dependencies ; Spell errors fixed

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- After that run `npm remove colors mysql @types/mysql` (unneeded dependencies)

## 0.6.5 - 17 November, 2022
### Sosise-Core version
`0.8.11`

### Changelog
- Any `sosise-core` method now displays comment. Example: `Helper.storagePath()` - Displays: `Path to the storage with ending slash @return e.g. /tmp/myproject/storage/`
- Method `Helper.pluckMany()` added, read more here: [documentation](documentation/helpers.md?id=helperpluckmany)
- Method `Helper.startProfiling()` added, read more here: [documentation](documentation/helpers.md?id=helperstartprofiling)
- Method `Helper.stopProfiling()` added, read more here: [documentation](documentation/helpers.md?id=helperstopprofiling)
- Method `Helper.paginateArray()` added, read more here: [documentation](documentation/helpers.md?id=helperpaginatearray)
- Method `Helper.assemblePagination()` added, read more here: [documentation](documentation/helpers.md?id=helperassemblepagination)

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`

## 0.6.4 - 17 October, 2022
### Sosise-Core version
`0.8.10`

### Changelog
- Any command execution now creates a file in `storage/framework/%COMMANDNAME%` directory, it can be used for monitoring purposes. It helps you to be sure that a command has not frozen or does not takes too long to execute.

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- After that run any `command`, it will create a directory `storage/framework` with `.gitignore` file in it. Just push it to your repository

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