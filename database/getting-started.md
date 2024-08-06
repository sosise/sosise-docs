# Getting Started

## Introduction

Nearly all contemporary web applications engage with a database. Sosise streamlines this interaction across various supported databases using raw SQL and a fluent query builder. The Express Framework Builder employs the [Knex.js](http://knexjs.org) library for database operations. Knex.js, pronounced /kəˈnɛks/, is a comprehensive SQL query builder supporting the following databases:

- `Postgres`
- `MSSQL`
- `MySQL`
- `MariaDB`
- `SQLite3`
- `Oracle`
- `Amazon Redshift`

For complete documentation, kindly visit the [Knex.js website](http://knexjs.org).

## Configuration

You can find the configuration for Sosise database services in your application's `src/config/database.ts` configuration file. This file allows you to define all your database connections and specify the default connection. The configuration options primarily depend on your application's environment variables. Examples for most of the Sosise-supported database systems are included in this file.

## Usage

We will illustrate database interaction using a simple repository example:

```typescript
import Database from 'sosise-core/build/Database/Database';
import { Knex } from 'knex';

export default class MyLocalDatabaseRepository {

    private dbClient: Knex;

    /**
     * Constructor
     */
    constructor() {
        this.dbClient = Database.getConnection(process.env.DB_PROJECT_CONNECTION as string).client;
    }

    /**
     * Get last 100 rows
     */
    public async selectHundredLastRows(): Promise<any> {
        // Get 100 rows
        const rows = await this.dbClient
            .table('sometable')
            .orderBy('id', 'desc')
            .limit(100);

        return rows;
    }
}
```

Please view this repository as an example, serving as a foundation for further database query explanations.

## Select

### Single Row

```typescript
const row = await this.dbClient
    .table('sometable')
    .first();
```

### Multiple Rows

```typescript
const rows = await this.dbClient
    .table('sometable')
    .limit(100);
```

### Specific Fields

```typescript
const rows = await this.dbClient
    .table('sometable')
    .select([
        'id',
        'name as author',
        this.dbClient.raw('SUM(total) as sum')
    ]);
```

### Ordering

```typescript
const rows = await this.dbClient
    .table('sometable')
    .orderBy('id', 'desc');
```

### Grouping

#### By a Single Field

```typescript
const rows = await this.dbClient
    .table('sometable')
    .groupBy('customer_id');
```

#### By Multiple Fields

```typescript
const rows = await this.dbClient
    .table('sometable')
    .groupBy(['name', 'customer_id']);
```

### Where Clauses

#### Equals

```typescript
const rows = await this.dbClient
    .table('sometable')
    .where('customer_id', 13);
```

#### Between

```typescript
const rows = await this.dbClient
    .table('sometable')
    .whereBetween('created_at', ['2020-01-01', '2020-01-10']);
```

#### Conditions

```typescript
const rows = await this.dbClient
    .table('sometable')
    .where(function () {
        this.orWhere('customer_id', 1);
        this.orWhere('customer_id', 2);
    });
```

### Joins

#### Inner Join

```typescript
const rows = await this.dbClient
    .table('sometable')
    .innerJoin('customer', 'customer.id', 'sometable.customer_id');
```

#### Left Join

```typescript
const rows = await this.dbClient
    .table('sometable')
    .leftJoin('customer', 'customer.id', 'sometable.customer_id');
```

### Debug

#### Retrieve Resulting Query

```typescript
const query = this.dbClient
    .table('sometable')
    .where(function () {
        this.orWhere('customer_id', 1);
        this.orWhere('customer_id', 2);
    })
    .toQuery();
console.log(query);
```

#### Retrieve Resulting SQL (without params)

```typescript
const sql = this.dbClient
    .table('sometable')
    .where(function () {
        this.orWhere('customer_id', 1);
        this.orWhere('customer_id', 2);
    })
    .toSQL();
console.log(sql);
```

## Update

### Update Specific Row

```typescript
await this.dbClient
    .table('sometable')
    .where('id', 2)
    .update({ customer_id: 'field value comes here' });
```

### Update All Rows

```typescript
await this.dbClient
    .table('sometable')
    .update({ customer_id: 'field value comes here' });
```

## Insert

### Insert a Single Row

```typescript
const insertedRow = await this.dbClient
    .table('sometable')
    .insert({ customer_id: 1 })
    .returning('id');

console.log(insertedRow[0]);
```

### Insert Multiple Rows

```typescript
await this.dbClient
    .table('sometable')
    .insert([
        { customer_id: 1 },
        { customer_id: 2 },
        { customer_id: 3 },
    ]);
```

## Delete

### Delete Specific Rows

```typescript
await this.dbClient
    .table('sometable')
    .whereIn('customer_id', [1, 2, 3])
    .delete();
```

### Delete All Rows

```typescript
await this.dbClient
    .table('sometable')
    .delete();
```

### Using Raw Queries

```typescript
const rawQuery = await this.dbClient.raw('SELECT * FROM sometable WHERE customer_id = ?', [1]);
console.log(rawQuery.rows);
```