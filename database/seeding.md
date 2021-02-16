# Database: Seeding
## Introduction
Sosise includes the ability to seed your database with test data using seed classes.

## Generating Seeds
You may use the `make:seed` Artisan command to generate a database seed. The new seed will be placed in your `src/database/seeds` directory. Each seed filename contains a timestamp that allows Sosise to determine the order of seeding database:

```sh
./artisan make:seed fill_customers_table
```

> Since a seed is a typescript file, you will need to build your project `npm run build` after manipulating your seeds.

## Seed Structure
A seed class contains one method: `run`. This method is used to add new rows to the table. You can specify anything in this method to fill up your table or tables. When you have generated a seed there is already a example code to help you.


```typescript
import BaseSchema from 'sosise-core/build/Database/BaseSchema';
import * as faker from 'faker';

/**
 * If you need more information, see: http://knexjs.org/#Schema ; https://www.npmjs.com/package/faker
 */
export default class FillCustomersTable extends BaseSchema {

    protected tableName = 'customers';

    /**
     * Run seed
     */
    public async run(): Promise<void> {
        // Prepare data to seed
        const data: any = [];

        // Generate 100 rows
        for (let row = 0; row < 100; row++) {
            data.push({
                customer_id: faker.random.number()
            });
        }

        // Insert to table
        await this.dbConnection.table(this.tableName).insert(data);
    }
}
```

## Running Seeds
To run all of your seeds, execute the migrate Artisan command:

```sh
./artisan seed
```