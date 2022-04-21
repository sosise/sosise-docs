## 0.6.0 - 21 April, 2022 (Breaking change)
### Changelog
- Logging config changed, this is done to give the possibility to disable console and/or file logging

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- Replace your `src/config/logging.ts` with the one from `https://raw.githubusercontent.com/sosise/sosise/0.6.0/src/config/logging.ts`
- Add to your `.env` and `.env.example` and `.env.testing` new ENVs, you can take a look at `https://raw.githubusercontent.com/sosise/sosise/0.6.0/.env.example`

## 0.5.0 - 19 April, 2022 (Breaking change)
### Changelog
- Sentry config changed, this is done primarily to disable `tracing`, to prevent high load on sentry instance
- ./artisan ascii update
- ./artisan now renders the documentation link
- Removed unneeded sentry packages

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- Replace your `src/config/sentry.ts` with the one from `https://raw.githubusercontent.com/sosise/sosise/0.5.0/src/config/sentry.ts`


## 0.4.1 - 16 February, 2022
### Changelog
- `sosise-core` bullmq package updated
- bullmq package updated

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`

## 0.4 - 11 January, 2022
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
### Changelog
- `sosise-core` packages updated
- `sosise` packages updated

### How to upgrade
1. `npm install sosise-core@latest`
2. `npm update`
3. `npm audit fix`

## 0.3 - 19 November, 2021
### Changelog
- Helper methods `dd` and `dump` bug fixes

## 0.2 - 29 September, 2021
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
### Changelog
- Added environment param to enable or disable documentation `basic auth`
- `docker-compose.yml` file updated to mount logs
- /docs folder excluded from `editorconfig-checker`
- Database default configuration refactored
- Some other small non breaking changes

### How to upgrade to that version
Just run `npm install sosise-core@latest` or `npm run update-sosise`