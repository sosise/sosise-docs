## 0.4 - 11 January, 2022
### Changelog
- `./artisan make:exception` is now automatically increments code property, so you do not have to do it manually
- Exceptions now have additional property `protected sendToSentry = xxx`, if you set it to false, exceptions of this type would not be sent to sentry.
  If the property is true or does not exists exception will be sent to sentry
- Exceptions now have additional property `protected loggingChannel = 'default';`, if you set it you can log the exception to some non default channels
- `src/Exceptions/Handler.ts` changed a lot, please take the new version from skeleton: https://github.com/sosise/sosise/blob/master/src/app/Exceptions/Handler.ts and integrate with your changes if there are any.
- Property `useColorizedOutputInLogFiles` is deprecated, remove it from `src/config/logging.ts` this did not make any sense
- Rename property `enableLoggingToFile` to `enableLoggingToFiles` in `src/config/logging.ts`

### How to upgrade to that version
- Run `npm install sosise-core@latest` or `npm run update-sosise`
- Manual code insertion is needed, see messages above

## 0.3.2 - 11 January, 2022
### Changelog
- Error handling documentation added
- Changed default framework exception codes (2000) now

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