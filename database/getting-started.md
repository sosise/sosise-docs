# Getting Started
## Introduction
Almost every modern web application interacts with a database. Sosise makes interacting with databases extremely simple across a variety of supported databases using raw SQL, a fluent query builder. Express Framework Builder uses [knex](http://knexjs.org) as a library to work with databases. Knex.js (pronounced /kəˈnɛks/) is a "batteries included" SQL query builder for:

- `Postgres`
- `MSSQL`
- `MySQL`
- `MariaDB`
- `SQLite3`
- `Oracle`
- `Amazon Redshift`

> Please visit [knex website](http://knexjs.org) for full documentation.

## Configuration
The configuration for Sosise database services is located in your application's `src/config/database.ts` configuration file. In this file, you may define all of your database connections, as well as specify which connection should be used by default. Most of the configuration options within this file are driven by the values of your application's environment variables. Examples for most of Sosise supported database systems are provided in this file.

## Usage
As an example we will take a look at a simple repository that is responsible for database interaction:

```typescript
import Database from 'sosise-core/build/Database/Database';

export default class MyLocalDatabaseRepository {

    private dbConnection: Database;

    /**
     * Constructor
     */
    constructor() {
        this.dbConnection = Database.getConnection('project');
    }

    /**
     * Get last 100 rows
     */
    public async selectHundredLastRows(): Promise<any> {
        // Get 100 rows
        const rows = await this.dbConnection.client
            .table('sometable')
            .orderBy('id', 'desc')
            .limit(100);

        return rows;
    }
}
```

> We will use this repository to explain more database queries, please consider this repository as an example, not more.

## Select
### First row
```typescript
const row = await this.dbConnection.client
    .table('sometable')
    .first();
```

### Multiple rows
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .limit(100);
```

### Specific fields
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .select([
        'id',
        'name as author',
        this.dbConnection.client.raw('SUM(total) as sum')
    ]);
```

### OrderBy
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .orderBy('id', 'desc');
```

### GroupBy
#### Single field
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .groupBy('customer_id');
```

#### Multiple fields
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .groupBy('name')
    .groupBy('customer_id');
```

### Where cases (only some are showed, please see knex website to see full documentation)
#### Equals
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .where('customer_id', '=', 13);
```

#### Between
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .whereBetween('created_at', [
        '2020-01-01',
        '2020-01-10'
    ]);
```

#### Braces
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .where(function () {
        this.orWhere('customer_id', '=', 1);
        this.orWhere('customer_id', '=', 2);
    });
```

### Joins (only some are showed, please see knex website to see full documentation)
#### Inner join
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .innerJoin('customer', 'customer.id', '=', 'sometable.customer_id');
```

#### Left join
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .leftJoin('customer', 'customer.id', '=', 'sometable.customer_id');
```

### Debug
#### Get resulting query
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .where(function () {
        this.orWhere('customer_id', '=', 1);
        this.orWhere('customer_id', '=', 2);
    })
    .toQuery();
```

#### Get resulting SQL (without params)
```typescript
const rows = await this.dbConnection.client
    .table('sometable')
    .where(function () {
        this.orWhere('customer_id', '=', 1);
        this.orWhere('customer_id', '=', 2);
    })
    .toSQL();
```

## Update
### Update row in table
```typescript
await this.dbConnection.client
    .table('sometable')
    .where('id', 2)
    .update({ customer_id: 'field value comes here' });
```

### Update all rows in table
```typescript
await this.dbConnection.client
    .table('sometable')
    .update({ customer_id: 'field value comes here' });
```

## Insert
### Insert single row
```typescript
await this.dbConnection.client
    .table('sometable')
    .insert([
        { customer_id: 1 }
    ]);
```

### Insert multiple rows
```typescript
await this.dbConnection.client
    .table('sometable')
    .insert([
        { customer_id: 1 },
        { customer_id: 2 },
        { customer_id: 3 },
    ]);
```

## Delete
### Delete specific rows in table
```typescript
await this.dbConnection.client
    .table('sometable')
    .whereIn('customer_id', [1, 2, 3])
    .delete();
```

### Delete all rows in table
```typescript
await this.dbConnection.client
    .table('sometable')
    .delete();
```