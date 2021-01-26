# Database: Migrations
# Introduction
Migrations are like version control for your database, allowing your team to define and share the application's database schema definition. If you have ever had to tell a teammate to manually add a column to their local database schema after pulling in your changes from source control, you've faced the problem that database migrations solve.

Sosise provides database agnostic support for creating and manipulating tables across all of `Knex's` supported database systems.

# Generating Migrations
You may use the `make:migration` Artisan command to generate a database migration. The new migration will be placed in your `src/database/migrations` directory. Each migration filename contains a timestamp that allows Sosise to determine the order of the migrations:

```sh
./artisan make:migration create_flights_table
```

> Since a migration is a typescript file, you will need to build your project `npm run build` after manipulating your migrations.

# Migration Structure
A migration class contains two methods: `up` and `down`. The `up` method is used to add new tables, columns, or indexes to your database, while the `down` method should reverse the operations performed by the `up` method.

Within both of these methods you may use the Knex schema builder to expressively create and modify tables. To learn about all of the methods available on the Schema builder, check out its [documentation](http://knexjs.org/#Schema). For example, the following migration creates a flights table:

```typescript
import BaseSchema from 'sosise-core/build/Database/BaseSchema';

/**
 * If you need more information, see: http://knexjs.org/#Schema
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

# Running Migrations
To run all of your outstanding migrations, execute the migrate Artisan command:

```sh
./artisan migrate
```

# Rolling Back Migrations
To roll back the latest migration operation, you may use the rollback Artisan command. This command rolls back the last "batch" of migrations, which may include multiple migration files:

```sh
./artisan migrate:rollback
```

> Some migration operations are destructive, which means they may cause you to lose data. In order to protect you from running these commands against your production database, you will be prompted for confirmation before the commands are executed. To force the commands to run without a prompt, use the `-f` or `--force` flag

# Drop All Tables & Migrate
The `migrate:fresh` command will drop all tables from the database and then execute the migrate command:

```sh
./artisan migrate:fresh
```

The `migrate:fresh` command will drop all database tables regardless of their prefix. This command should be used with caution when developing on a database that is shared with other applications.

# Create new table migration
In order to create a migration that will create a new table use following command:

```sh
./artisan make:migration create_flights_table
```

or

```sh
./artisan make:migration create_flights_table -c
```

> The flag -c (create) will be used by default

# Update table migration
In order to create a migration that will update existing table use following command:

```sh
./artisan make:migration add_date_field_to_flights_table -u
```

> The -u (update) flag should be used to alter table