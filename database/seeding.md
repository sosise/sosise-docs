# Database: Seeding
## Overview
Sosise provides a feature to populate your database with test data using seed classes. This can be immensely helpful in generating dummy data for testing or demonstration purposes.

## Generating Seeders
The `make:seed` Artisan command allows you to generate a new database seeder. This newly created seeder will be stored in the `src/database/seeds` directory. The seeder filename includes a timestamp, facilitating the sequencing of database seeding:

```sh
./artisan make:seed fill_customers_table
```

> Note: A seeder is a TypeScript file. Therefore, it's necessary to build your project using `npm run build` after altering your seeders.

## Seeder Structure
Each seeder class contains a `run` method. This method is responsible for adding new rows to your tables. You have the flexibility to define any operation in this method to populate your tables. Each generated seeder provides example code to guide you.

```typescript
import BaseSchema from 'sosise-core/build/Database/BaseSchema';
import * as faker from 'faker';

/**
 * For additional information, refer to: http://knexjs.org/#Schema ; https://www.npmjs.com/package/faker
 */
export default class FillCustomersTable extends BaseSchema {

    /**
     * Allows running the seeder only in a local environment (APP_ENV=local)
     */
    protected onlyInLocalEnvironment = false;

    /**
     * Specifies the table where the data should be inserted
     */
    protected tableName = 'customers';

    /**
     * Executes the seeder
     */
    public async run(): Promise<void> {
        // Prepare data for seeding
        const data: any = [];

        // Generate 100 rows
        for (let row = 0; row < 100; row++) {
            data.push({
                customer_id: faker.random.number()
            });
        }

        // Insert data into the table
        await this.dbConnection.table(this.tableName).insert(data);
    }
}
```

## Executing Seeders
To execute all the available seeders, use the following command:

```sh
./artisan seed
```
This command will initiate the process of data seeding according to the methods defined in your seeder classes.