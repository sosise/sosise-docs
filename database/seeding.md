# Database Seeding

## Quick Start

Populate your database with test data using Sosise seeders - perfect for development, testing, and demos!

```bash
# Create a new seeder
./artisan make:seed UsersTableSeeder

# Run all seeders
./artisan seed

# Refresh database and then seed
./artisan migrate:fresh
./artisan seed
```

```typescript
// Example seeder with Faker data
import BaseSchema from 'sosise-core/build/Database/BaseSchema';
import { faker } from '@faker-js/faker';

export default class UsersTableSeeder extends BaseSchema {
    protected tableName = 'users';
    protected onlyInLocalEnvironment = true; // Safety first!

    public async run(): Promise<void> {
        const users = Array.from({ length: 50 }, () => ({
            name: faker.person.fullName(),
            email: faker.internet.email(),
            password: '$2b$10$hashedpassword...',
            created_at: faker.date.past(),
            updated_at: new Date()
        }));

        await this.dbConnection.table(this.tableName).insert(users);
    }
}
```

Generate realistic test data for better development and testing!

## Introduction

Database seeding allows you to populate your database with test data programmatically. This is invaluable for development, testing, and demonstrations where you need realistic data to work with.

Sosise seeders use TypeScript and integrate with popular libraries like Faker.js to generate realistic test data. You can create seeders for individual tables, establish relationships between data, and ensure your development environment always has the data you need.

## Creating Seeders

### Basic Seeder Generation

Create seeders using Artisan commands:

```bash
# Create a basic seeder
./artisan make:seed UsersTableSeeder

# Create seeder for specific table
./artisan make:seed ProductsTableSeeder

# Create seeder with descriptive name
./artisan make:seed SampleCustomersSeeder
```

Seeders are created in `database/seeds/` with timestamps:
```
2023_12_25_143022_users_table_seeder.ts
2023_12_25_143045_products_table_seeder.ts
```

### Basic Seeder Structure

```typescript
import BaseSchema from 'sosise-core/build/Database/BaseSchema';
import { faker } from '@faker-js/faker';

export default class UsersTableSeeder extends BaseSchema {
    // Table to seed
    protected tableName = 'users';
    
    // Environment safety - only run in local/development
    protected onlyInLocalEnvironment = true;

    public async run(): Promise<void> {
        // Clear existing data (optional)
        await this.dbConnection.table(this.tableName).del();

        // Generate test data
        const users = [];
        for (let i = 0; i < 100; i++) {
            users.push({
                name: faker.person.fullName(),
                email: faker.internet.email(),
                password: '$2b$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // "password"
                active: faker.datatype.boolean(),
                created_at: faker.date.past(),
                updated_at: new Date()
            });
        }

        // Insert data in batches for performance
        await this.dbConnection.batchInsert(this.tableName, users, 50);
    }
}
```

## Realistic Data with Faker

### User Data Seeding

```typescript
export default class UsersTableSeeder extends BaseSchema {
    protected tableName = 'users';
    protected onlyInLocalEnvironment = true;

    public async run(): Promise<void> {
        const users = Array.from({ length: 200 }, () => {
            const firstName = faker.person.firstName();
            const lastName = faker.person.lastName();
            const email = faker.internet.email({ firstName, lastName }).toLowerCase();

            return {
                first_name: firstName,
                last_name: lastName,
                email,
                username: faker.internet.userName({ firstName, lastName }),
                password: '$2b$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi',
                phone: faker.phone.number(),
                date_of_birth: faker.date.birthdate({ min: 18, max: 80, mode: 'age' }),
                avatar: faker.image.avatar(),
                bio: faker.lorem.paragraph(),
                website: faker.internet.url(),
                location: `${faker.location.city()}, ${faker.location.state()}`,
                email_verified_at: faker.datatype.boolean() ? faker.date.past() : null,
                active: faker.helpers.weightedArrayElement([
                    { weight: 0.8, value: true },
                    { weight: 0.2, value: false }
                ]),
                created_at: faker.date.past({ years: 2 }),
                updated_at: faker.date.recent()
            };
        });

        await this.dbConnection.batchInsert(this.tableName, users, 50);
    }
}
```

### Product Data Seeding

```typescript
export default class ProductsTableSeeder extends BaseSchema {
    protected tableName = 'products';
    protected onlyInLocalEnvironment = true;

    public async run(): Promise<void> {
        const categories = await this.dbConnection('categories').select('id');
        const categoryIds = categories.map(cat => cat.id);

        const products = Array.from({ length: 500 }, () => {
            const name = faker.commerce.productName();
            const price = parseFloat(faker.commerce.price({ min: 10, max: 1000 }));
            
            return {
                name,
                slug: faker.helpers.slugify(name).toLowerCase(),
                description: faker.commerce.productDescription(),
                price,
                cost: price * 0.6, // 40% markup
                sku: faker.string.alphanumeric({ length: 8, casing: 'upper' }),
                category_id: faker.helpers.arrayElement(categoryIds),
                stock_quantity: faker.number.int({ min: 0, max: 1000 }),
                weight: faker.number.float({ min: 0.1, max: 50, fractionDigits: 2 }),
                dimensions: JSON.stringify({
                    length: faker.number.int({ min: 1, max: 100 }),
                    width: faker.number.int({ min: 1, max: 100 }),
                    height: faker.number.int({ min: 1, max: 100 })
                }),
                images: JSON.stringify([
                    faker.image.url(),
                    faker.image.url(),
                    faker.image.url()
                ]),
                tags: JSON.stringify(faker.helpers.arrayElements([
                    'new', 'popular', 'sale', 'featured', 'trending', 'bestseller'
                ], { min: 0, max: 3 })),
                rating: faker.number.float({ min: 1, max: 5, fractionDigits: 1 }),
                review_count: faker.number.int({ min: 0, max: 500 }),
                active: faker.helpers.weightedArrayElement([
                    { weight: 0.9, value: true },
                    { weight: 0.1, value: false }
                ]),
                featured: faker.helpers.weightedArrayElement([
                    { weight: 0.2, value: true },
                    { weight: 0.8, value: false }
                ]),
                created_at: faker.date.past({ years: 1 }),
                updated_at: faker.date.recent()
            };
        });

        await this.dbConnection.batchInsert(this.tableName, products, 50);
    }
}
```

## Relational Data Seeding

### Orders with Items

```typescript
export default class OrdersTableSeeder extends BaseSchema {
    protected tableName = 'orders';
    protected onlyInLocalEnvironment = true;

    public async run(): Promise<void> {
        // Get existing users and products
        const users = await this.dbConnection('users').select('id');
        const products = await this.dbConnection('products').select('id', 'price');

        const orders = [];
        const orderItems = [];

        for (let i = 0; i < 1000; i++) {
            const userId = faker.helpers.arrayElement(users).id;
            const orderDate = faker.date.past({ years: 1 });
            const status = faker.helpers.weightedArrayElement([
                { weight: 0.6, value: 'delivered' },
                { weight: 0.2, value: 'processing' },
                { weight: 0.1, value: 'shipped' },
                { weight: 0.1, value: 'cancelled' }
            ]);

            // Create order
            const order = {
                id: i + 1, // Explicit ID for relationship
                user_id: userId,
                order_number: `ORD-${faker.string.numeric(8)}`,
                status,
                subtotal: 0, // Will be calculated
                tax_amount: 0,
                shipping_cost: faker.number.float({ min: 5, max: 25, fractionDigits: 2 }),
                total: 0, // Will be calculated
                shipping_address: JSON.stringify({
                    name: faker.person.fullName(),
                    street: faker.location.streetAddress(),
                    city: faker.location.city(),
                    state: faker.location.state(),
                    zipCode: faker.location.zipCode(),
                    country: 'US'
                }),
                created_at: orderDate,
                updated_at: status === 'delivered' ? faker.date.between({ from: orderDate, to: new Date() }) : orderDate
            };

            // Create 1-5 order items per order
            const itemCount = faker.number.int({ min: 1, max: 5 });
            let subtotal = 0;

            for (let j = 0; j < itemCount; j++) {
                const product = faker.helpers.arrayElement(products);
                const quantity = faker.number.int({ min: 1, max: 3 });
                const price = parseFloat(product.price);
                const total = price * quantity;

                orderItems.push({
                    order_id: order.id,
                    product_id: product.id,
                    quantity,
                    price,
                    total,
                    created_at: orderDate,
                    updated_at: orderDate
                });

                subtotal += total;
            }

            // Calculate totals
            const taxRate = 0.08; // 8% tax
            order.subtotal = subtotal;
            order.tax_amount = parseFloat((subtotal * taxRate).toFixed(2));
            order.total = parseFloat((subtotal + order.tax_amount + order.shipping_cost).toFixed(2));

            orders.push(order);
        }

        // Insert orders first
        await this.dbConnection.batchInsert('orders', orders, 100);
        
        // Then insert order items
        await this.dbConnection.batchInsert('order_items', orderItems, 200);
    }
}
```

### Categories with Hierarchy

```typescript
export default class CategoriesTableSeeder extends BaseSchema {
    protected tableName = 'categories';
    protected onlyInLocalEnvironment = true;

    public async run(): Promise<void> {
        // Main categories
        const mainCategories = [
            'Electronics',
            'Clothing',
            'Home & Garden',
            'Sports',
            'Books',
            'Health & Beauty'
        ];

        const categories = [];
        let categoryId = 1;

        // Create main categories
        for (const mainCat of mainCategories) {
            categories.push({
                id: categoryId++,
                name: mainCat,
                slug: faker.helpers.slugify(mainCat).toLowerCase(),
                description: faker.lorem.sentence(),
                parent_id: null,
                sort_order: categories.length,
                active: true,
                created_at: new Date(),
                updated_at: new Date()
            });
        }

        // Create subcategories
        const subcategoriesMap = {
            'Electronics': ['Smartphones', 'Laptops', 'Tablets', 'Accessories'],
            'Clothing': ['Men\'s', 'Women\'s', 'Kids', 'Shoes'],
            'Home & Garden': ['Furniture', 'Kitchen', 'Garden', 'Decor'],
            'Sports': ['Fitness', 'Outdoor', 'Team Sports', 'Water Sports'],
            'Books': ['Fiction', 'Non-Fiction', 'Children', 'Textbooks'],
            'Health & Beauty': ['Skincare', 'Makeup', 'Hair Care', 'Vitamins']
        };

        categories.forEach(parent => {
            const subcats = subcategoriesMap[parent.name];
            if (subcats) {
                subcats.forEach((subcat, index) => {
                    categories.push({
                        id: categoryId++,
                        name: subcat,
                        slug: faker.helpers.slugify(subcat).toLowerCase(),
                        description: faker.lorem.sentence(),
                        parent_id: parent.id,
                        sort_order: index,
                        active: true,
                        created_at: new Date(),
                        updated_at: new Date()
                    });
                });
            }
        });

        await this.dbConnection.batchInsert(this.tableName, categories, 50);
    }
}
```

## Advanced Seeding Patterns

### Conditional Seeding

```typescript
export default class ConditionalSeeder extends BaseSchema {
    protected tableName = 'users';
    protected onlyInLocalEnvironment = false; // Allow in staging too

    public async run(): Promise<void> {
        // Only seed if table is empty
        const existingCount = await this.dbConnection(this.tableName).count('id as count').first();
        if (existingCount.count > 0) {
            console.log(`${this.tableName} already has data, skipping seeding`);
            return;
        }

        // Environment-specific seeding
        const recordCount = process.env.NODE_ENV === 'production' ? 10 : 1000;
        
        const users = Array.from({ length: recordCount }, () => ({
            name: faker.person.fullName(),
            email: faker.internet.email(),
            password: '$2b$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi',
            created_at: faker.date.past(),
            updated_at: new Date()
        }));

        await this.dbConnection.batchInsert(this.tableName, users, 100);
    }
}
```

### Factory Pattern

```typescript
// Create a factory helper
class UserFactory {
    public static create(overrides: Partial<any> = {}): any {
        const defaultUser = {
            name: faker.person.fullName(),
            email: faker.internet.email(),
            password: '$2b$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi',
            active: true,
            created_at: faker.date.past(),
            updated_at: new Date()
        };

        return { ...defaultUser, ...overrides };
    }

    public static createAdmin(overrides: Partial<any> = {}): any {
        return this.create({
            role: 'admin',
            email_verified_at: new Date(),
            ...overrides
        });
    }

    public static createInactive(overrides: Partial<any> = {}): any {
        return this.create({
            active: false,
            email_verified_at: null,
            ...overrides
        });
    }
}

// Use in seeder
export default class UsersWithRolesSeeder extends BaseSchema {
    protected tableName = 'users';
    protected onlyInLocalEnvironment = true;

    public async run(): Promise<void> {
        const users = [
            // Create specific admin users
            UserFactory.createAdmin({ email: 'admin@example.com', name: 'Admin User' }),
            UserFactory.createAdmin({ email: 'superadmin@example.com', name: 'Super Admin' }),
            
            // Create regular users
            ...Array.from({ length: 50 }, () => UserFactory.create()),
            
            // Create some inactive users
            ...Array.from({ length: 10 }, () => UserFactory.createInactive())
        ];

        await this.dbConnection.batchInsert(this.tableName, users, 50);
    }
}
```

### Performance-Optimized Seeding

```typescript
export default class LargeDatasetSeeder extends BaseSchema {
    protected tableName = 'analytics_events';
    protected onlyInLocalEnvironment = true;

    public async run(): Promise<void> {
        console.log('Starting large dataset seeding...');
        
        const batchSize = 1000;
        const totalRecords = 100000;
        const batches = Math.ceil(totalRecords / batchSize);

        for (let batch = 0; batch < batches; batch++) {
            console.log(`Processing batch ${batch + 1}/${batches}`);
            
            const events = Array.from({ length: batchSize }, () => ({
                user_id: faker.number.int({ min: 1, max: 1000 }),
                event_type: faker.helpers.arrayElement([
                    'page_view', 'click', 'purchase', 'signup', 'login'
                ]),
                event_data: JSON.stringify({
                    page: faker.internet.url(),
                    timestamp: faker.date.recent(),
                    properties: {
                        browser: faker.helpers.arrayElement(['Chrome', 'Firefox', 'Safari']),
                        device: faker.helpers.arrayElement(['desktop', 'mobile', 'tablet'])
                    }
                }),
                created_at: faker.date.past({ years: 1 }),
                updated_at: new Date()
            }));

            await this.dbConnection.batchInsert(this.tableName, events, batchSize);
        }

        console.log('Large dataset seeding completed');
    }
}
```

## Running Seeders

### Basic Seeding Commands

```bash
# Run all seeders
./artisan seed

# Force seed even when restricted to local environment
./artisan seed --force
```

### Seeder Organization

```bash
# Create seeders in logical order
./artisan make:seed 01_CategoriesTableSeeder
./artisan make:seed 02_UsersTableSeeder
./artisan make:seed 03_ProductsTableSeeder
./artisan make:seed 04_OrdersTableSeeder

# Or use descriptive names
./artisan make:seed MasterDataSeeder
./artisan make:seed TestUsersSeeder
./artisan make:seed SampleProductsSeeder
```

## Environment Safety

### Development-Only Seeders

```typescript
export default class DevelopmentDataSeeder extends BaseSchema {
    protected onlyInLocalEnvironment = true; // Critical for safety
    
    public async run(): Promise<void> {
        // This will only run in APP_ENV=local
        // Never in production or staging
    }
}
```

### Environment-Aware Seeding

```typescript
export default class EnvironmentSpecificSeeder extends BaseSchema {
    protected onlyInLocalEnvironment = false;
    
    public async run(): Promise<void> {
        const env = process.env.NODE_ENV;
        
        switch (env) {
            case 'local':
                await this.seedDevelopmentData();
                break;
                
            case 'staging':
                await this.seedStagingData();
                break;
                
            case 'production':
                await this.seedMinimalProductionData();
                break;
        }
    }
    
    private async seedDevelopmentData(): Promise<void> {
        // Large dataset for development
        const users = Array.from({ length: 1000 }, () => UserFactory.create());
        await this.dbConnection.batchInsert('users', users, 100);
    }
    
    private async seedStagingData(): Promise<void> {
        // Moderate dataset for staging
        const users = Array.from({ length: 100 }, () => UserFactory.create());
        await this.dbConnection.batchInsert('users', users, 50);
    }
    
    private async seedMinimalProductionData(): Promise<void> {
        // Only essential data for production
        const adminUser = UserFactory.createAdmin({
            email: 'admin@myapp.com',
            name: 'System Administrator'
        });
        
        await this.dbConnection.table('users').insert([adminUser]);
    }
}
```

## Best Practices

### 1. Use Environment Protection

```typescript
// ✅ Good: Protected from production accidents
export default class TestDataSeeder extends BaseSchema {
    protected onlyInLocalEnvironment = true; // Safety first!
    
    public async run(): Promise<void> {
        // Safe to generate lots of test data
    }
}

// ❌ Dangerous: Could run in production
export default class TestDataSeeder extends BaseSchema {
    protected onlyInLocalEnvironment = false; // Dangerous!
    
    public async run(): Promise<void> {
        // This could accidentally run in production
    }
}
```

### 2. Use Batch Inserts for Performance

```typescript
// ✅ Good: Efficient batch processing
const users = Array.from({ length: 1000 }, () => UserFactory.create());
await this.dbConnection.batchInsert('users', users, 100); // 100 records per batch

// ❌ Bad: Inefficient individual inserts
for (const user of users) {
    await this.dbConnection.table('users').insert(user); // Very slow
}
```

### 3. Clear Data Responsibly

```typescript
// ✅ Good: Clear specific test data
export default class UsersTableSeeder extends BaseSchema {
    public async run(): Promise<void> {
        // Only clear test data, keep real data
        await this.dbConnection('users').where('email', 'like', '%@example.com').del();
        
        // Or truncate if seeder owns the table completely
        await this.dbConnection.raw('TRUNCATE TABLE users RESTART IDENTITY CASCADE');
    }
}

// ❌ Bad: Indiscriminate deletion
await this.dbConnection('users').del(); // Deletes ALL users including real ones
```

### 4. Handle Relationships Correctly

```typescript
// ✅ Good: Maintain referential integrity
export default class OrdersSeeder extends BaseSchema {
    public async run(): Promise<void> {
        // Get existing users first
        const users = await this.dbConnection('users').select('id');
        
        if (users.length === 0) {
            throw new Error('No users found. Run UsersTableSeeder first.');
        }
        
        const orders = Array.from({ length: 100 }, () => ({
            user_id: faker.helpers.arrayElement(users).id, // Valid foreign key
            // ... other fields
        }));
        
        await this.dbConnection.batchInsert('orders', orders, 50);
    }
}
```

## Summary

Sosise database seeding provides:

- ✅ Realistic test data generation with Faker.js
- ✅ Environment safety with protection flags
- ✅ Batch processing for performance
- ✅ Relational data handling
- ✅ TypeScript support with IDE assistance
- ✅ Flexible seeder organization
- ✅ Production deployment safety
- ✅ Easy development and testing setup

Keep your development environment populated with realistic data using Sosise seeders!