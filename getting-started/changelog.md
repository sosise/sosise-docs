### Changelog for Project Configuration

## 0.8.20 - 03 June 2025
### Accompanying Sosise-Core Version
`1.0.0`

### Updates
- Middlewares were refactored.  
- Throttling is now supported. See for more information:  
  https://sosise.github.io/sosise-docs/#/documentation/throttling

### Upgrade Steps
1. Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.
2. Delete the old middleware file:
   `src/app/Http/Middlewares/DocumentationBasicAuthMiddleware.ts`
3. In `src/routes/api.ts`, update the import:
   ```diff
   - import DocumentationBasicAuthMiddleware
   -   from '../app/Http/Middlewares/DocumentationBasicAuthMiddleware';
   + import DocumentationBasicAuthMiddleware
   +   from 'sosise-core/build/Middlewares/DocumentationBasicAuthMiddleware';
   ```
4. In `src/app/Http/Middlewares/Kernel.ts`, prepend  
   `'ThrottlingMiddleware'`  
   to the beginning of the middleware array.
5. Create a new config file at `src/config/throttling.ts` with the following example (tune values as needed):
   ```typescript
   /**
    * Configuration for request throttling
    */
   const throttlingConfig = {
       /**
        * Enables or disables request throttling
        */
       isEnabled: true,
       /**
        * The HTTP header used to determine the client's IP address.
        * Must be set by a trusted proxy or load balancer.
        */
       clientIpHeader: 'X-Forwarded-For',
       /**
        * CIDR subnets to skip (not throttled)
        */
       skipSubnets: ['10.0.0.0/24'],
       /**
        * Route-specific throttling rules
        */
       routeRules: [
           {
               // HTTP method for this rule
               httpMethod: 'GET',
               // URL path for this rule
               path: '/customer/get-all-customers',
               // Max requests allowed per rolling minute
               maxRequestsPerMinute: 5,
           },
           {
               httpMethod: 'GET',
               path: '/customer/:id',
               maxRequestsPerMinute: 10,
           },
       ],
   };

   export default throttlingConfig;
   ```

## 0.8.19 - 02 June 2025

### Accompanying Sosise-Core Version
`0.11.10`

### Updates
- The Unifier now includes support for the `enumOrNull` method.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.18 - 04 April 2025

### Accompanying Sosise-Core Version
`0.11.9`

### Updates
- The Unifier now includes support for the `shouldBeStringOrNull` method.
- The Unifier now includes support for the `shouldBeNumberOrNull` method.
- The Unifier now includes support for the `shouldBeBooleanOrNull` method.
- The Unifier now includes support for the `emailOrNull` method.
- Adjusted ESLint configuration: `.prettierrc`, `eslint.config.mjs`.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.17 - 12 December 2024

### Accompanying Sosise-Core Version
`0.11.8`

### Updates
#### Standardized linting and formatting by replacing TSLint with ESLint and Prettier, updating dependencies, and cleaning up obsolete files and scripts
- Added `eslint.config.mjs` for ESLint configuration.
- Added `.prettierrc` for Prettier configuration.
- Removed `tslint.json` and `.editorconfig` files.
- Installed ESLint and Prettier dependencies.
- Removed `editorconfig` and `editorconfig-checker` dependencies.
- Updated `package.json`:
  - Removed `cscheck` and `eccheck` scripts.
  - Added `lint` and `lint:fix` scripts.
- Recommended installing the VSCode ESLint plugin.

### Upgrade Steps

1. **Add Configuration Files**:
   - Create `eslint.config.mjs`:
     ```javascript
    import globals from "globals";
    import pluginJs from "@eslint/js";
    import tseslint from "typescript-eslint";
    import eslintPluginPrettierRecommended from "eslint-plugin-prettier/recommended";
    import unusedImports from "eslint-plugin-unused-imports";

    export default [
        { files: ["**/*.{js,mjs,cjs,ts}"] },
        { languageOptions: { globals: globals.node } },
        pluginJs.configs.recommended,
        ...tseslint.configs.recommended,
        eslintPluginPrettierRecommended,
        {
            "plugins": {
                'unused-imports': unusedImports
            },
            "rules": {
                "@typescript-eslint/no-explicit-any": "off",
                "@typescript-eslint/no-unused-vars": "off",
                "unused-imports/no-unused-imports": "warn",
                "unused-imports/no-unused-vars": "off",
                "@typescript-eslint/no-require-imports": "off",
                "@typescript-eslint/ban-ts-comment": "off",
                "no-case-declarations": "off",
            }
        },
        {
            ignores: [
                "build/**",
                "node_modules/**",
                "doc/**",
                ".history/**",
                ".vscode/**",
                "eslint.config.mjs",
                "package-lock.json",
                "tsconfig.json",
                "package.json"
            ],
        }
    ];
     ```

   - Create `.prettierrc`:
     ```json
    {
        "printWidth": 120,
        "useTabs": false,
        "tabWidth": 4,
        "endOfLine": "lf",
        "semi": true,
        "singleQuote": true,
        "bracketSpacing": true,
        "insertPragma": false,
        "requirePragma": false,
        "json": {
            "insertFinalNewline": false
        }
    }
     ```

2. **Install Dependencies**:
   ```bash
   npm install --save-dev typescript-eslint@^8.18.0 eslint-plugin-prettier@^5.2.1 eslint-config-prettier@^9.1.0 eslint@^9.16.0 eslint-plugin-unused-imports@^4.1.4
   ```

3. **Remove Obsolete Files**:
   ```bash
   rm tslint.json .editorconfig
   ```

4. **Remove Unused Dependencies**:
   ```bash
   npm remove editorconfig editorconfig-checker
   ```

5. **Update `package.json` Scripts**:
   - Remove:
     ```json
     "cscheck": "./node_modules/.bin/tslint --project tsconfig.json",
     "eccheck": "./node_modules/.bin/editorconfig-checker -exclude \"(node_modules|build|docs)\"",
     ```
   - Add:
     ```json
     "lint": "eslint .",
     "lint:fix": "eslint . --fix"
     ```

6. **Install VSCode Plugin**:
   - Install **ESLint** by **Microsoft**. 


7. **Upgrade sosise-core**
   - Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

8. **Execute linter**
   - Execute `npm run lint:fix`

## 0.8.16 - 30 October, 2024

### Accompanying Sosise-Core version
`0.11.7`

### Updates
- **Redis Client Reconnection Support:** Enhanced Redis client with reconnection support, ensuring stability during unexpected disconnections.
- **Docker Build Support for Apple Silicon:** Updated build-docker-image.sh script to support building Docker images on Apple Silicon processors (M1/2/3/...).

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.15 - 24 September, 2024
### Accompanying Sosise-Core version
`0.11.5`

### Updates
- **HttpClient Fixes**: The `requestWithRetry` method now works correctly when a timeout occurs.
- **HttpClient Config Enhancements**: Added the following configurations to `HttpClientConfig`:
    - `debug` option allows you to enable or disable debug information.
    - `loggingChannel` can be set to specify the channel for debug information.
- **Docker Compose Update**: Updated `docker-compose.yml` for improved compatibility and configuration.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.14 - 24 September, 2024
### Accompanying Sosise-Core version
`0.11.4`

### Updates
- **Cache Improvements**: Added `putMany` method for Redis, allowing the storing of multiple cache items by their keys. This method is only applicable to the Redis cache driver.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.13 - 24 September, 2024
### Accompanying Sosise-Core version
`0.11.3`

### Updates
- **HttpClient**: Added a new `requestDoNotRetryForHttpCodes` option to `HttpClientRetryConfig`. This allows the configuration of HTTP status codes that should not trigger retries, providing more granular control over the retry logic in requests.
- **HttpClient**: The exception message now includes the HTTP method and URL where the exception occurred.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.12 - 11 September, 2024
### Accompanying Sosise-Core version
`0.11.2`

### Updates
- **Cache Improvements**: Added `getMany` method for Redis, allowing the retrieval of multiple cache items by their keys. This method is only applicable to the Redis cache driver.
- **Redis Client Updated**: Upgraded Redis client to version `4.7.0`.
- **Package Cleanup**: Removed unused packages for optimization.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.11 - 12 August, 2024
### Accompanying Sosise-Core version
`0.11.1`

### Updates
- CockroachDB `./artisan migrate:fresh` fixed

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.10 - 08 August, 2024 (Breaking change)
### Accompanying Sosise-Core version
`0.11.0`

### Breaking changes
- Knex was updated, please take a look at your requests to database and how you get the `inserted id`. You now have to use `.returning('...')`. Read more at: https://knexjs.org/guide/query-builder.html#returning . This change only touches only PostgreSQL, MSSQL, SQLite, and Oracle databases. 

### Updates
- Reverted `IOC` to it's previous codes, since it was not finished yet, will refactor it later.
- Sosise-Core knex, mssql, sqlite3 drivers updated
- Sosise-Core pg driver installed
- Sosise now supports PostgreSQL and CockroachDB out of the box

### Upgrade Steps
- Execute `npm install knex@^3.1.0`
- Execute `npm install pg@^8.12.0`
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.9 - 06 August, 2024 (Broken version)
### Accompanying Sosise-Core version
`0.10.1`

### Updates
- Removed unused parameters from `.env.example` and `.env.testing`
- Updated `Dockerfile` to use the latest Alpine version and removed unused packages
- Refactored `src/config/database.ts` (uncommented important parameters)
- Updated `HttpRepository` template to use Sosise's `HttpClient`. Example: `./artisan make:repository FooRepository -h`
- Refactored `IOC` to allow usage like:
```typescript
const service = IOC.make(CustomerService)
```
instead of:
```typescript
const service = IOC.make(CustomerService) as CustomerService
```
- Updated documentation. See more at https://sosise.github.io/sosise-docs/#/database/getting-started

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.8 - 20 March, 2024 (Breaking change)
### Accompanying Sosise-Core version
`0.10.0`

### Updates
- Validatorjs removed
- Implemented self written `validator` in sosise-core.
- See more at https://sosise.github.io/sosise-docs/#/documentation/unifiers

### Upgrade Steps
- Execute `npm remove validatorjs`.
- Execute `npm remove @types/validatorjs`.
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.7 - 5 March, 2024
### Accompanying Sosise-Core version
`0.9.9`

### Updates
- HttpClient added `keepAlive` and `keepAliveMsecs` config options when instantiating Http client
  This is necessary if you are making multiple HTTP requests and intend to reuse the connections.

### Upgrade Steps
- Execute `npm install sosise-core@latest` or `npm run update-sosise` to upgrade.

## 0.8.6 - 6 November, 2023
### Accompanying Sosise-Core version
`0.9.8`

### Updates
- HttpClient added configurable behavior on specific HTTP Response Status Codes
- HttpClient added configurable default exception, to avoid try / catch every time in the repositories
- See more at https://sosise.github.io/sosise-docs/#/documentation/http-client?id=httpclientrequestconfig

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