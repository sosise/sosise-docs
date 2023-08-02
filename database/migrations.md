# Database Migrations
## Overview
Database migrations are akin to version control for your database schema, enabling your team to define and share the application's database structure. Migrations help avoid manual database modifications, making it more seamless for team members to adapt to schema changes.

Sosise provides an agnostic approach to creating and altering tables across all the database systems supported by `Knex`.

## Generating Migrations
You can use the `make:migration` command provided by Sosise to generate a new database migration. This migration will be stored in the `src/database/migrations` directory. Migration filenames include a timestamp to determine the sequence of migrations:

```sh
./artisan make:migration create_flights_table
```

> Note: As a migration is a typescript file, after modifying your migrations, you need to build your project using `npm run build`.

## Structure of Migrations
A migration class typically comprises two methods: `up` and `down`. The `up` method is utilized to introduce new tables, columns, or indexes to your database, whereas the `down` method is designed to reverse the operations carried out by the `up` method.

Both of these methods can utilize the Knex schema builder to expressively create and alter tables. For a comprehensive guide on the methods available on the Schema builder, please refer to its [documentation](http://knexjs.org/#Schema). The example below illustrates a migration creating a flights table:

```typescript
import BaseSchema from 'sosise-core/build/Database/BaseSchema';

/**
 * For more information, refer to: http://knexjs.org/#Schema
 */
export default class CreateFlightsTable extends BaseSchema {

    protected tableName = 'table_name_comes_here';

    /**
     * Run the migrations.
     */
    public async up(): Promise<void> {
        await this.dbConnection.schema.createTable(this.tableName, (table) => {
            table.increments('id');
            table.timestamps(true);
        });
    }

    /**
     * Reverse the migrations.
     */
    public async down(): Promise<void> {
        await this.dbConnection.schema.dropTable(this.tableName);
    }
}
```

## Executing Migrations
To execute all your pending migrations, use the migrate command:

```sh
./artisan migrate
```

## Reversing Migrations
To revert the most recent migration operation, use the rollback command. This command rolls back the last "batch" of migrations, which could comprise multiple migration files:

```sh
./artisan migrate:rollback
```

> Note: Some migration operations are destructive and may result in data loss. For protection, you will be asked to confirm before the commands are executed. To override the prompt, use the `-f` or `--force` flag.

> Additionally, `./artisan migrate:rollback` displays which migrations would be rolled back, and only proceeds with the operation after your confirmation.

## Resetting Database & Migration
The `migrate:fresh` command drops all tables from the database and then executes the migration:

```sh
./artisan migrate:fresh
```

This command drops all database tables, irrespective of their prefix. Exercise caution when using this command, especially when developing on a database shared with other applications.

## Creating a New Table Migration
To create a migration for a new table, use the following command:

```sh
./artisan make:migration create_flights_table
```

or

```sh
./artisan make:migration create_flights_table -c
```

> Note: The `-c` (create) flag is used by default.

## Updating an Existing Table Migration
To create a migration that alters an existing table, use the following command:

```sh
./artisan make:migration add_date_field_to_flights_table -u
```

> Note: Use the `-u` (update) flag to alter a table.