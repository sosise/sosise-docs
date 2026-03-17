### Changelog for Project Configuration

## 2.0.0 - 17 March 2026
### Accompanying Sosise-Core Version
`2.0.0`

> ⚠️ **Major release with breaking changes.** Follow the upgrade guide below step by step. Each step includes before/after code examples so you know exactly what to change in your project.

### New Features
- **Log file rotation with gzip compression** — automatic management of log files: size-based rotation of active files (when exceeding `maxFileSizeMb`), gzip compression of old files, age-based deletion, and total directory size limit. Configure via `rotation` block in `src/config/logging.ts`
- **PostgreSQL `migrate:fresh` support** — previously `migrate:fresh` only worked with MySQL and CockroachDB, now PostgreSQL is fully supported
- **Server startup info for all DB drivers** — mysql2, PostgreSQL, CockroachDB now show host/port/database at startup
- **Beautiful artisan help grouping** — commands are organized into visual groups (User-defined, Make, Multiagent, Migrate, Seed, Queue) in `./artisan --help`

### Documentation Updates
- Added install notes for packages that are no longer bundled with the skeleton:
  - `database/getting-started.md` — database driver install instructions (`pg`, `mysql2`, `tedious`)
  - `database/seeding.md` — `@faker-js/faker` install instruction
  - `testing/getting-started.md` — `@faker-js/faker` install instruction
  - `documentation/controllers.md` — `bcrypt` import and install instruction
  - `documentation/artisan.md` — updated all command examples to Commander v14 API (`cli.opts()`, import `Command` from `BaseCommand`)
  - `getting-started/directory-structure.md` — updated Unifier example from old `validatorjs` syntax to new `Validator` API

### Updated packages
| Package | Before | After |
|---|---|---|
| TypeScript | `4.x` | `5.x` |
| Express | `4.x` | `5.x` |
| BullMQ | `1.x` | `5.x` |
| Commander | `6.x` | `14.x` |
| Redis | `4.x` | `5.x` |
| connect-redis | `5.x` | `9.x` |
| Nodemailer | `6.x` | `8.x` |
| ESLint | `9.x` | `10.x` |
| @types/node | `14.x` | `22.x` |
| @types/express | `4.x` | `5.x` |
| @types/nodemailer | `6.x` | `7.x` |
| @types/jest | `29.x` | `30.x` |
| dotenv | `16.x` | `17.x` |
| `faker` | `5.x` | replaced with `@faker-js/faker` |
| `mysql` | `2.x` | replaced with `mysql2` |
| `@sentry/node` | `5.x` | **removed** |
| `sqlite3` | `5.x` | **removed** |
| `body-parser` | `1.x` | **removed** (Express 5 built-in) |

---

### Upgrade guide

> Go through **every** step. Check **every** file type listed — you may have multiple files of each kind in your project.

---

#### Step 1 — Update dependencies

```bash
npm install sosise-core@latest
npm install dotenv@^17.3.1
npm install typescript@^5.9.3 @types/express@^5.0.2 @types/node@^22.0.0 @types/jest@^30.0.0 --save-dev
```

**Update `dotenv.config()` calls** — dotenv v17 prints an advertising message by default. Add `quiet: true` to suppress it:

In `src/server.ts`, `src/artisan.ts`, and `src/config/jest.ts`:

Before:
```typescript
dotenv.config();
```

After:
```typescript
dotenv.config({ quiet: true });
```

For jest config with a custom path:
```typescript
dotenv.config({ path: './.env.testing', quiet: true });
```

---

#### Step 2 — Remove Sentry

Sentry has been completely removed from the framework.

**2a. Delete the config file:**
```bash
rm -f src/config/sentry.ts
```

**2b. Update `src/app/Exceptions/Handler.ts`:**

Remove these imports:
```typescript
// DELETE these lines:
import * as Sentry from '@sentry/node';
import sentryConfig from '../../config/sentry';
```

In `reportCommandException()`, remove the Sentry block:
```typescript
// DELETE this entire block:
if (exception.sendToSentry !== undefined && exception.sendToSentry === true) {
    Sentry.init(sentryConfig);
    Sentry.captureException(exception);
    await Sentry.flush();
}
```

**2c. Update `.env`, `.env.example`, `.env.testing`:**
```bash
# DELETE this line from all env files:
SENTRY_DSN=...
```

---

#### Step 3 — Update every custom exception

Check **every** file in `src/app/Exceptions/`. Remove the `sendToSentry` property:

Before:
```typescript
export default class MyException extends Exception {
    protected httpCode = 500;
    protected code = 3000;
    // If set to false no exception will be sent to sentry
    protected sendToSentry = true;
```

After:
```typescript
export default class MyException extends Exception {
    protected httpCode = 500;
    protected code = 3000;
```

> Run: `grep -rn "sendToSentry" src/app/Exceptions/` — fix every file that shows up.

---

#### Step 4 — Update database config

**If you use MySQL/MariaDB**, change the client name in `src/config/database.ts`:

Before:
```typescript
client: 'mysql',
```

After:
```typescript
client: 'mysql2',
```

**If you had SQLite config** (even commented out), remove it — SQLite is no longer supported.

---

#### Step 5 — Update every custom command

Check **every** file in `src/app/Console/Commands/`. Three changes per file:

**5a. Fix import** — `Command` is now re-exported from `BaseCommand`, no separate `commander` install needed:

Before:
```typescript
import commander from 'commander';
import BaseCommand, { OptionType } from 'sosise-core/build/Command/BaseCommand';
```

After:
```typescript
import BaseCommand, { OptionType, Command } from 'sosise-core/build/Command/BaseCommand';
```

**5b. Fix `handle()` signature:**

Before:
```typescript
public async handle(cli: commander.Command): Promise<void> {
```

After:
```typescript
public async handle(cli: Command): Promise<void> {
```

**5c. Fix option access — this is the most important change:**

Before (Commander v6 — options were direct properties):
```typescript
if (cli.debug) { ... }
console.log(cli.since);
console.log(cli.limit);
```

After (Commander v14 — options are accessed via `cli.opts()`):
```typescript
const options = cli.opts();
if (options.debug) { ... }
console.log(options.since);
console.log(options.limit);
```

> Run: `grep -rn "cli\." src/app/Console/Commands/` — every `cli.something` (except `cli.opts()`) needs to change to `options.something`.

---

#### Step 6 — Update every queue worker

Check **every** file in `src/app/Console/QueueWorkers/`. Two changes per file:

**6a. Fix import — remove `QueueScheduler`:**

Before:
```typescript
import { Worker, QueueScheduler, Job } from 'bullmq';
```

After:
```typescript
import { Worker, Job } from 'bullmq';
```

**6b. Remove `QueueScheduler` instantiation in `listen()` method:**

Before:
```typescript
private async listen(): Promise<void> {
    // Instantiate queue scheduler
    const scheduler = new QueueScheduler(this.queueName, { connection: this.redisConnection });

    // Instantiate worker
    const myWorker = new Worker(this.queueName, ...);
```

After:
```typescript
private async listen(): Promise<void> {
    // Instantiate worker
    const myWorker = new Worker(this.queueName, ...);
```

> `QueueScheduler` was removed in BullMQ v3. Its functionality (delayed job scheduling) is now built into the `Worker` class automatically.

> Run: `grep -rn "QueueScheduler" src/` — fix every file that shows up.

---

#### Step 7 — Update every seed file

Check **every** file in `src/database/seeds/`. If the file uses faker:

**7a. Fix import:**

Before:
```typescript
import * as faker from 'faker';
```

After:
```typescript
import { faker } from '@faker-js/faker';
```

**7b. Fix renamed API calls:**

| Before (faker v5) | After (@faker-js/faker v9+) |
|---|---|
| `faker.name.firstName()` | `faker.person.firstName()` |
| `faker.name.lastName()` | `faker.person.lastName()` |
| `faker.name.findName()` | `faker.person.fullName()` |
| `faker.address.city()` | `faker.location.city()` |
| `faker.address.country()` | `faker.location.country()` |
| `faker.address.streetAddress()` | `faker.location.streetAddress()` |
| `faker.address.zipCode()` | `faker.location.zipCode()` |
| `faker.datatype.number()` | `faker.number.int()` |
| `faker.datatype.float()` | `faker.number.float()` |
| `faker.datatype.uuid()` | `faker.string.uuid()` |
| `faker.datatype.boolean()` | `faker.datatype.boolean()` (same) |
| `faker.phone.phoneNumber()` | `faker.phone.number()` |
| `faker.date.between('2020-01-01', '2024-12-31')` | `faker.date.between({ from: '2020-01-01', to: '2024-12-31' })` |
| `faker.date.recent()` | `faker.date.recent()` (same) |
| `faker.internet.email()` | `faker.internet.email()` (same) |
| `faker.internet.url()` | `faker.internet.url()` (same) |
| `faker.commerce.price()` | `faker.commerce.price()` (same) |
| `faker.random.word()` | `faker.word.sample()` |
| `faker.random.words()` | `faker.word.words()` |
| `faker.image.imageUrl()` | `faker.image.url()` |

Full migration guide: https://fakerjs.dev/guide/upgrading.html

**7c. Install the package:**
```bash
npm install @faker-js/faker --save-dev
```

---

#### Step 8 — Clean up removed packages

```bash
# Remove packages that no longer exist in the framework:
npm uninstall @sentry/node @types/faker @types/bcrypt @types/uuid tslint 2>/dev/null

# If you import any of these directly in YOUR code, install them back:
npm install axios          # if you have: import axios from 'axios'
npm install bcrypt         # if you have: import bcrypt from 'bcrypt'
npm install uuid           # if you have: import { v4 } from 'uuid'
npm install lodash         # if you have: import _ from 'lodash'
npm install dayjs          # if you have: import dayjs from 'dayjs'
npm install pg             # if you have: import pg from 'pg'
```

> These packages were removed from the skeleton's `package.json` because the skeleton code did not import them. They are still available as transitive dependencies through sosise-core, but if you import them in your own code, add them to your `package.json` explicitly.

---

#### Step 9 — Add log rotation config

Add the `rotation` block to your `src/config/logging.ts`:

```typescript
const loggingConfig = {
    // ...your existing fields (enableLoggingToConsole, enableLoggingToFiles, etc.)...

    /**
     * Log rotation settings
     * Rotation runs on application startup and periodically
     */
    rotation: {
        enabled: true,
        maxFileSizeMb: 50,       // Rotate active file when it exceeds 50MB
        compressAfterDays: 1,    // Gzip compress files older than 1 day
        maxAgeDays: 14,          // Delete files older than 14 days
        maxTotalSizeMb: 500,     // Keep total log directory under 500MB
        checkIntervalHours: 1,   // Check every hour (also runs on startup)
    },

    // ...your existing channels...
};
```

> Without this config, log files will accumulate indefinitely. This is a new required section in `logging.ts`.

---

#### Step 10 — Build and fix any remaining TypeScript errors

```bash
npm run build
```

TypeScript 5 may flag new errors that TypeScript 4 did not catch. Common issues:

- **`@types/express` v5** may have slightly different types for `Request`/`Response`. If you get type errors in controllers, check Express 5 type changes.
- **Stricter `null` checks** — TS5 may catch nullable values that TS4 allowed silently.

---

### Checklist — verify you haven't missed anything

Run these commands in your project root to find files that need updating:

```bash
# Find remaining Sentry references:
grep -rn "sentry\|Sentry\|sendToSentry" src/ --include="*.ts"

# Find old commander import (should now import from BaseCommand):
grep -rn "from 'commander'" src/ --include="*.ts"

# Find old option access pattern in commands (cli.someOption without .opts()):
grep -rn "cli\.\(debug\|force\|since\|limit\|name\|verbose\|output\)" src/app/Console/ --include="*.ts"

# Find QueueScheduler usage:
grep -rn "QueueScheduler" src/ --include="*.ts"

# Find old faker import:
grep -rn "from 'faker'" src/ --include="*.ts"

# Find old mysql client config:
grep -rn "client: 'mysql'" src/config/ --include="*.ts"

# Find dotenv.config() without quiet option:
grep -rn "dotenv.config()" src/ --include="*.ts"

# Find sqlite references:
grep -rn "sqlite" src/ --include="*.ts"
```

If any of these return results — you have files that still need updating.

---

### Summary of all changes

| What changed | Who is affected | Action required | Files to check |
|---|---|---|---|
| Sentry removed | Everyone (was in skeleton) | Delete config, clean Handler.ts, remove sendToSentry | `config/sentry.ts`, `Exceptions/Handler.ts`, `Exceptions/*.ts`, `.env` |
| mysql → mysql2 | Anyone using MySQL | Change `client: 'mysql'` → `'mysql2'` | `config/database.ts` |
| SQLite3 removed | Anyone using SQLite | Switch to PostgreSQL/MySQL/MSSQL | `config/database.ts` |
| Commander 6 → 14 | Anyone with custom commands | Import `Command` from `BaseCommand`, use `cli.opts()` | `Console/Commands/*Command.ts` |
| BullMQ 1 → 5 | Anyone with queue workers | Remove QueueScheduler | `Console/QueueWorkers/*.ts` |
| faker → @faker-js/faker | Anyone with seed files using faker | Update import + API methods | `database/seeds/*.ts` |
| Express 4 → 5 | No action needed | Internal change — body parsing, session init handled by framework | — |
| Redis 4 → 5 | No action needed | Internal change — scan cursor types handled by framework | — |
| connect-redis 5 → 9 | No action needed | Internal change — session Redis init handled by framework | — |
| TypeScript 4 → 5 | Everyone | `npm install typescript@^5.9.3 --save-dev` | `package.json` |
| @types/express 4 → 5 | Everyone | `npm install @types/express@^5.0.2 --save-dev` | `package.json` |
| dotenv 16 → 17 | Everyone | `npm install dotenv@^17.3.1` + add `quiet: true` | `server.ts`, `artisan.ts`, `config/jest.ts` |
| Log rotation added | Everyone | Add `rotation` block to `logging.ts` | `config/logging.ts` |
| Skeleton packages removed | Anyone who imported them directly | Install explicitly if needed | `package.json` |

### Migration Notes
- This is a major version — test thoroughly after migration
- Run `npm run build` after each step to catch errors incrementally
- If you need Sentry, integrate the latest `@sentry/node` SDK directly in your `Handler.ts`
- If you need SQLite, install `sqlite3` yourself — note that `migrate:fresh` will not support it

---

## 1.1.4 - 13 January 2026
### Accompanying Sosise-Core Version
`1.1.4`

### Updates
- **Multi-Agent Graph System**: New architecture for building LLM-powered multi-agent workflows
  - `AgentGraphService` - orchestrator for agent execution with visual logging
  - Worker-Validator pattern template for common validation workflows
  - Three-tier memory system: agent-private, shared (thread-level), and global
  - Two storage drivers: InMemory (development) and File (production)
  - Automatic iteration tracking and protection against infinite loops
  - Support for OpenAI and Groq LLM providers out of the box
  - New Artisan commands: `ma:init` and `ma:create`

### Upgrade Steps
1. **Update sosise-core**:
   ```bash
   npm install sosise-core@latest
   # or
   npm run update-sosise
   ```

2. **Initialize multi-agent system** (optional):
   ```bash
   ./artisan ma:init [ServiceName]
   ```
   This creates all necessary files:
   - `src/app/Services/MA/` - Service and agents
   - `src/app/Repositories/LLM/` - LLM repositories (OpenAI, Groq)
   - `src/app/Repositories/Prompt/` - Prompt repository
   - `storage/ma/prompts/` - Prompt files
   - `src/app/Console/Commands/MACommand.ts` - Run command

3. **Add environment variables** to `.env`, `.env.example`, `.env.testing`:
   ```env
   # OpenAI
   OPENAI_API_KEY=your-openai-key

   # Groq (optional)
   GROQ_API_KEY=your-groq-key
   ```

4. **Run the example**:
   ```bash
   ./artisan ma:run
   ```

### Migration Notes
- This is a new feature and does not affect existing functionality
- No breaking changes in this release
- The multi-agent system is optional and only initialized when you run `ma:init`

## 1.1.3 - 01 October 2025
### Accompanying Sosise-Core Version
`1.1.2`

### Updates
- `nodemon` configuration has been optimized
- `build-docker-image.sh` has been refactored
- `package.json` scripts has been refactored

### Upgrade Steps
1. Replace `nodemon.json` with following content:
```
{
  "watch": [
    "src/**/*",
    "docs/**/*",
    ".env"
  ],
  "ignore": [
    "storage/**/*",
    "build/**/*",
    "docs/src/**/*",
    "node_modules/**/*",
    "*.log"
  ],
  "ext": "ts,js,json,html,md",
  "exec": "npm run build && npm run start"
}
```

2. Replace `build-docker-image.sh` with following content:
  ```
  #!/usr/bin/env sh

  # Build app
  npm run build

  # Build Docker image
  IMAGE="sosise:$(date +%Y-%m-%d)"

  if [ "$(uname -m)" = "arm64" ]; then
      echo "Building for AMD64 on ARM..."
      docker buildx build --platform linux/amd64 -t "$IMAGE" -f docker/Dockerfile .
  else
      echo "Building native image..."
      docker build -t "$IMAGE" -f docker/Dockerfile .
  fi

  # Show run command
  echo "Run: docker run --rm --name sosise -p 10000:10000 $IMAGE"
  ```

3. Replace `package.json` scripts or integrate with yours:
```
"scripts": {
  "dev": "nodemon",
  "watch": "nodemon",
  "build": "npm run clean:docs && npm run clean:migrations && npm run copy-docs && tsc",
  "start": "node build/server.js",
  "clean:docs": "rm -rf docs/src/Types docs/src/Unifiers docs/src/Enums",
  "clean:migrations": "rm -rf build/database/migrations",
  "copy-docs": "cp -R src/app/Types src/app/Unifiers src/app/Enums docs/src/.",
  "test": "jest -c ./build/config/jest.js --coverage",
  "lint": "eslint .",
  "lint:fix": "eslint . --fix",
  "update-sosise": "npm install sosise-core@latest"
},
```

## 1.1.2 - 17 September 2025
### Accompanying Sosise-Core Version
`1.1.1`

### Updates
- **Development Experience**: Replaced `tsc-watch` with `nodemon` for improved development workflow
  - Automatic recompilation on file changes
  - Better process management and restart behavior
  - Consistent with other Sosise projects

### Upgrade Steps
1. **Update sosise-core**:
   ```bash
   npm install sosise-core@latest
   # or
   npm run update-sosise
   ```

2. **Install nodemon and remove tsc-watch**:
   ```bash
   npm install --save-dev nodemon@^3.1.10
   npm remove tsc-watch
   ```

3. **Create `nodemon.json` configuration file**:
   ```json
   {
     "watch": ["src/**/*"],
     "ignore": [
       "storage/**/*",
       "build/**/*", 
       "node_modules/**/*",
       "*.log"
     ],
     "ext": "ts,js,json",
     "exec": "npm run build && node build/server.js"
   }
   ```

4. **Update `package.json` scripts**:
   ```json
   "scripts": {
     // ... other scripts
     "watch": "nodemon",
     "dev": "nodemon",
     // ... other scripts
   }
   ```

### Migration Notes
- No breaking changes in this release
- The `watch` and `dev` commands now use nodemon instead of tsc-watch
- Development workflow remains the same: use `npm run watch` or `npm run dev` for development

## 1.1.0 - 13 September 2025
### Accompanying Sosise-Core Version
`1.1.0`

### Updates
- **Event Bus System**: New event-driven architecture for building reactive applications
  - Support for two drivers: `memory` (in-process) and `redis` (distributed)
  - Pattern-based event subscriptions with wildcard support (e.g., `user.*`, `order.*.completed`)
  - Durable subscriptions for guaranteed message delivery (Redis driver only)
  - TTL support for automatic event expiration
  - Automatic reconnection and error recovery for Redis driver
  - Service-specific position tracking for distributed systems

### Upgrade Steps
1. **Update sosise-core**:
   ```bash
   npm install sosise-core@latest
   # or
   npm run update-sosise
   ```

2. **Create Event Bus configuration file** at `src/config/eventbus.ts`:
   ```typescript
   /**
    * Event Bus configuration
    */
   const eventBusConfig = {
       /**
        * Event bus driver
        * Available drivers: 'memory', 'redis'
        */
       driver: process.env.EVENTBUS_DRIVER || 'memory',

       /**
        * Driver-specific configuration
        */
       driverConfiguration: {
           // Redis configuration (for 'redis' driver)
           redis: {
               host: process.env.EVENTBUS_REDIS_HOST || 'localhost',
               port: Number(process.env.EVENTBUS_REDIS_PORT) || 6379,
               db: Number(process.env.EVENTBUS_REDIS_DB) || 0,
               password: process.env.EVENTBUS_REDIS_PASSWORD || undefined,
               // Service name for distributed position tracking
               serviceName: process.env.SERVICE_NAME || 'default-service',
           },
       },
   };

   export default eventBusConfig;
   ```

3. **Add logging channel to** `src/config/logging.ts`
```typescript
    /**
     * Cache channel for cache-related logs
     */
    cache: {
        logFileNamePrefix: 'cache',
    },
    
    /**
     * EventBus channel for event bus-related logs
     */
    eventbus: {
        logFileNamePrefix: 'eventbus',
    },
```

4. Add to your `.env` and `.env.example` and `.env.testing` new ENVs, you can take a look at `https://raw.githubusercontent.com/sosise/sosise/1.1.0/.env.example`

5. **Optional: Use Event Bus in your application**:
   - For in-process events, use the `memory` driver (default)
   - For distributed events across multiple services, use the `redis` driver
   - Consider using durable subscriptions for critical events that must not be lost

### Migration Notes
- The Event Bus is a new feature and does not affect existing functionality
- No breaking changes in this release
- The `memory` driver is suitable for single-instance applications
- The `redis` driver is recommended for multi-instance or microservices architectures

## 1.0.0 - 03 June 2025
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