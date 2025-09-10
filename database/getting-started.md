# Database - Getting Started

## Quick Start

Connect to your database and start querying immediately with Sosise's powerful query builder!

```typescript
// Configure in src/config/database.ts
export default {
    default: 'main',
    connections: {
        main: {
            client: 'postgresql', // or 'mysql', 'sqlite3', etc.
            connection: {
                host: process.env.DB_HOST,
                port: process.env.DB_PORT,
                user: process.env.DB_USER,
                password: process.env.DB_PASSWORD,
                database: process.env.DB_NAME
            }
        }
    }
};

// Use in your repository
export default class UserRepository {
    constructor(private db: Knex = Database.getConnection().client) {}

    public async getAllUsers(): Promise<User[]> {
        return await this.db('users')
            .select('*')
            .where('active', true)
            .orderBy('created_at', 'desc');
    }
}
```

Built on Knex.js for maximum flexibility and database compatibility!

## Introduction

Modern applications need reliable, scalable database access. Sosise provides a clean abstraction layer over [Knex.js](http://knexjs.org), giving you the power of raw SQL with the convenience of a fluent query builder.

Whether you're building a simple blog or a complex e-commerce platform, Sosise's database layer handles connection management, query building, migrations, and seeding across multiple database engines. The repository pattern keeps your business logic separated from database concerns.

## Supported Databases

Sosise supports all major databases through Knex.js:

- **PostgreSQL** - Recommended for production
- **MySQL/MariaDB** - Popular web application choice  
- **SQLite3** - Perfect for development and testing
- **Microsoft SQL Server** - Enterprise applications
- **Oracle Database** - Large-scale enterprise
- **Amazon Redshift** - Data warehousing

## Database Configuration

### Basic Setup

Configure your database in `src/config/database.ts`:

```typescript
export default {
    // Default connection name
    default: process.env.DB_CONNECTION || 'main',
    
    connections: {
        // Primary database
        main: {
            client: 'postgresql',
            connection: {
                host: process.env.DB_HOST || 'localhost',
                port: parseInt(process.env.DB_PORT || '5432'),
                user: process.env.DB_USER || 'postgres',
                password: process.env.DB_PASSWORD,
                database: process.env.DB_NAME || 'sosise_app'
            },
            pool: {
                min: 2,
                max: 10
            },
            migrations: {
                tableName: 'knex_migrations',
                directory: 'database/migrations'
            },
            seeds: {
                directory: 'database/seeds'
            }
        },
        
        // Read replica for performance
        readonly: {
            client: 'postgresql',
            connection: {
                host: process.env.DB_READ_HOST,
                port: parseInt(process.env.DB_PORT || '5432'),
                user: process.env.DB_READ_USER,
                password: process.env.DB_READ_PASSWORD,
                database: process.env.DB_NAME
            }
        },
        
        // Analytics database
        analytics: {
            client: 'postgresql',
            connection: {
                host: process.env.ANALYTICS_DB_HOST,
                port: parseInt(process.env.ANALYTICS_DB_PORT || '5432'),
                user: process.env.ANALYTICS_DB_USER,
                password: process.env.ANALYTICS_DB_PASSWORD,
                database: process.env.ANALYTICS_DB_NAME
            }
        }
    }
};
```

### Environment Variables

```bash
# .env
DB_CONNECTION=main
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=your-password
DB_NAME=sosise_app

# Read replica
DB_READ_HOST=read-replica.example.com
DB_READ_USER=readonly_user
DB_READ_PASSWORD=readonly_password

# Analytics
ANALYTICS_DB_HOST=analytics.example.com
ANALYTICS_DB_USER=analytics_user
ANALYTICS_DB_PASSWORD=analytics_password
ANALYTICS_DB_NAME=analytics
```

## Repository Pattern

### Basic Repository

Create repositories to encapsulate database logic:

```typescript
// src/app/Repositories/UserRepository.ts
import Database from 'sosise-core/build/Database/Database';
import { Knex } from 'knex';
import User from '../Models/User';

export default class UserRepository {
    private db: Knex;

    constructor() {
        this.db = Database.getConnection().client;
    }

    public async findById(id: number): Promise<User | null> {
        const row = await this.db('users')
            .select('*')
            .where('id', id)
            .first();
            
        return row ? new User(row) : null;
    }

    public async findByEmail(email: string): Promise<User | null> {
        const row = await this.db('users')
            .select('*')
            .where('email', email)
            .first();
            
        return row ? new User(row) : null;
    }

    public async create(userData: CreateUserData): Promise<User> {
        const [insertedId] = await this.db('users')
            .insert({
                name: userData.name,
                email: userData.email,
                password: userData.hashedPassword,
                created_at: new Date(),
                updated_at: new Date()
            })
            .returning('id');

        return await this.findById(insertedId);
    }

    public async update(id: number, updates: Partial<User>): Promise<User> {
        await this.db('users')
            .where('id', id)
            .update({
                ...updates,
                updated_at: new Date()
            });

        return await this.findById(id);
    }

    public async delete(id: number): Promise<void> {
        await this.db('users')
            .where('id', id)
            .delete();
    }

    public async findActiveUsers(limit: number = 50): Promise<User[]> {
        const rows = await this.db('users')
            .select('*')
            .where('active', true)
            .orderBy('created_at', 'desc')
            .limit(limit);

        return rows.map(row => new User(row));
    }
}
```

### Advanced Repository with Relationships

```typescript
// src/app/Repositories/OrderRepository.ts
export default class OrderRepository {
    constructor(private db: Knex = Database.getConnection().client) {}

    public async findWithItems(orderId: number): Promise<OrderWithItems | null> {
        // Main order query
        const order = await this.db('orders')
            .select([
                'orders.*',
                'users.name as customer_name',
                'users.email as customer_email'
            ])
            .leftJoin('users', 'users.id', 'orders.user_id')
            .where('orders.id', orderId)
            .first();

        if (!order) return null;

        // Order items with product details
        const items = await this.db('order_items')
            .select([
                'order_items.*',
                'products.name as product_name',
                'products.price as product_price'
            ])
            .leftJoin('products', 'products.id', 'order_items.product_id')
            .where('order_items.order_id', orderId);

        return {
            ...order,
            items
        };
    }

    public async findUserOrders(userId: number, options: PaginationOptions = {}): Promise<PaginatedOrders> {
        const { page = 1, limit = 20 } = options;
        const offset = (page - 1) * limit;

        // Get orders with pagination
        const orders = await this.db('orders')
            .select([
                'orders.*',
                this.db.raw('COUNT(order_items.id) as item_count')
            ])
            .leftJoin('order_items', 'order_items.order_id', 'orders.id')
            .where('orders.user_id', userId)
            .groupBy('orders.id')
            .orderBy('orders.created_at', 'desc')
            .limit(limit)
            .offset(offset);

        // Get total count for pagination
        const totalResult = await this.db('orders')
            .count('id as count')
            .where('user_id', userId)
            .first();

        const total = parseInt(totalResult.count.toString());

        return {
            orders,
            pagination: {
                page,
                limit,
                total,
                totalPages: Math.ceil(total / limit)
            }
        };
    }
}
```

## Query Building

### Basic Queries

```typescript
export default class ProductRepository {
    constructor(private db: Knex = Database.getConnection().client) {}

    // Select single record
    public async findById(id: number): Promise<Product | null> {
        const product = await this.db('products')
            .select('*')
            .where('id', id)
            .first();
            
        return product ? new Product(product) : null;
    }

    // Select multiple records
    public async findAll(): Promise<Product[]> {
        const products = await this.db('products')
            .select(['id', 'name', 'price', 'category_id'])
            .where('active', true)
            .orderBy('name');
            
        return products.map(row => new Product(row));
    }

    // Select with conditions
    public async findByCategory(categoryId: number): Promise<Product[]> {
        return await this.db('products')
            .select('*')
            .where('category_id', categoryId)
            .andWhere('active', true)
            .orderBy('price', 'asc');
    }

    // Select with complex conditions
    public async searchProducts(filters: ProductFilters): Promise<Product[]> {
        let query = this.db('products').select('*');

        if (filters.name) {
            query = query.where('name', 'ilike', `%${filters.name}%`);
        }

        if (filters.categoryId) {
            query = query.where('category_id', filters.categoryId);
        }

        if (filters.priceRange) {
            query = query.whereBetween('price', [
                filters.priceRange.min, 
                filters.priceRange.max
            ]);
        }

        if (filters.inStock) {
            query = query.where('stock_quantity', '>', 0);
        }

        return await query
            .orderBy(filters.sortBy || 'name', filters.sortOrder || 'asc')
            .limit(filters.limit || 50);
    }
}
```

### Joins and Relationships

```typescript
export default class ProductRepository {
    // Inner join
    public async findWithCategory(): Promise<ProductWithCategory[]> {
        return await this.db('products')
            .select([
                'products.*',
                'categories.name as category_name',
                'categories.slug as category_slug'
            ])
            .innerJoin('categories', 'categories.id', 'products.category_id')
            .where('products.active', true);
    }

    // Left join with null handling
    public async findAllWithOptionalCategory(): Promise<Product[]> {
        return await this.db('products')
            .select([
                'products.*',
                'categories.name as category_name'
            ])
            .leftJoin('categories', 'categories.id', 'products.category_id')
            .orderBy('products.name');
    }

    // Complex join with aggregations
    public async findBestSellers(limit: number = 10): Promise<BestSellerProduct[]> {
        return await this.db('products')
            .select([
                'products.id',
                'products.name',
                'products.price',
                this.db.raw('SUM(order_items.quantity) as total_sold'),
                this.db.raw('COUNT(DISTINCT orders.id) as order_count'),
                this.db.raw('AVG(reviews.rating) as avg_rating')
            ])
            .innerJoin('order_items', 'order_items.product_id', 'products.id')
            .innerJoin('orders', 'orders.id', 'order_items.order_id')
            .leftJoin('reviews', 'reviews.product_id', 'products.id')
            .where('orders.status', 'completed')
            .groupBy(['products.id', 'products.name', 'products.price'])
            .orderBy('total_sold', 'desc')
            .limit(limit);
    }
}
```

### Advanced Query Patterns

```typescript
export default class AnalyticsRepository {
    constructor(
        private db: Knex = Database.getConnection().client,
        private analyticsDb: Knex = Database.getConnection('analytics').client
    ) {}

    // Subqueries
    public async findUsersWithRecentOrders(): Promise<User[]> {
        const recentOrdersSubquery = this.db('orders')
            .select('user_id')
            .where('created_at', '>=', this.db.raw('NOW() - INTERVAL \'30 days\''))
            .groupBy('user_id');

        return await this.db('users')
            .select('*')
            .whereIn('id', recentOrdersSubquery);
    }

    // Window functions
    public async getMonthlyRevenue(): Promise<MonthlyRevenue[]> {
        return await this.db('orders')
            .select([
                this.db.raw('DATE_TRUNC(\'month\', created_at) as month'),
                this.db.raw('SUM(total) as revenue'),
                this.db.raw('COUNT(*) as order_count'),
                this.db.raw('LAG(SUM(total), 1) OVER (ORDER BY DATE_TRUNC(\'month\', created_at)) as previous_month_revenue')
            ])
            .where('status', 'completed')
            .groupBy(this.db.raw('DATE_TRUNC(\'month\', created_at)'))
            .orderBy('month', 'desc');
    }

    // Common Table Expressions (CTE)
    public async findTopCustomersByRegion(): Promise<TopCustomer[]> {
        return await this.db
            .with('customer_stats', (qb) => {
                qb.select([
                    'users.id',
                    'users.name',
                    'users.region',
                    this.db.raw('SUM(orders.total) as total_spent'),
                    this.db.raw('COUNT(orders.id) as order_count')
                ])
                .from('users')
                .innerJoin('orders', 'orders.user_id', 'users.id')
                .where('orders.status', 'completed')
                .groupBy(['users.id', 'users.name', 'users.region']);
            })
            .with('ranked_customers', (qb) => {
                qb.select([
                    '*',
                    this.db.raw('ROW_NUMBER() OVER (PARTITION BY region ORDER BY total_spent DESC) as rank')
                ])
                .from('customer_stats');
            })
            .select('*')
            .from('ranked_customers')
            .where('rank', '<=', 3)
            .orderBy(['region', 'rank']);
    }
}
```

## Database Operations

### Create Operations

```typescript
export default class UserRepository {
    // Insert single record
    public async create(userData: CreateUserData): Promise<User> {
        const [user] = await this.db('users')
            .insert({
                name: userData.name,
                email: userData.email,
                password: userData.hashedPassword,
                created_at: this.db.fn.now(),
                updated_at: this.db.fn.now()
            })
            .returning('*');

        return new User(user);
    }

    // Insert multiple records
    public async createBulk(usersData: CreateUserData[]): Promise<User[]> {
        const users = await this.db('users')
            .insert(usersData.map(userData => ({
                name: userData.name,
                email: userData.email,
                password: userData.hashedPassword,
                created_at: this.db.fn.now(),
                updated_at: this.db.fn.now()
            })))
            .returning('*');

        return users.map(user => new User(user));
    }

    // Upsert (insert or update)
    public async upsert(userData: UpsertUserData): Promise<User> {
        const [user] = await this.db('users')
            .insert({
                email: userData.email,
                name: userData.name,
                created_at: this.db.fn.now(),
                updated_at: this.db.fn.now()
            })
            .onConflict('email')
            .merge({
                name: userData.name,
                updated_at: this.db.fn.now()
            })
            .returning('*');

        return new User(user);
    }
}
```

### Update Operations

```typescript
export default class ProductRepository {
    // Update single record
    public async update(id: number, updates: Partial<Product>): Promise<Product> {
        const [product] = await this.db('products')
            .where('id', id)
            .update({
                ...updates,
                updated_at: this.db.fn.now()
            })
            .returning('*');

        return new Product(product);
    }

    // Bulk update
    public async updatePrices(categoryId: number, priceMultiplier: number): Promise<number> {
        const affectedRows = await this.db('products')
            .where('category_id', categoryId)
            .update({
                price: this.db.raw('price * ?', [priceMultiplier]),
                updated_at: this.db.fn.now()
            });

        return affectedRows;
    }

    // Conditional update
    public async updateStock(productId: number, quantityChange: number): Promise<void> {
        await this.db('products')
            .where('id', productId)
            .update({
                stock_quantity: this.db.raw('stock_quantity + ?', [quantityChange]),
                updated_at: this.db.fn.now()
            })
            .where('stock_quantity', '>=', Math.abs(quantityChange));
    }
}
```

### Delete Operations

```typescript
export default class UserRepository {
    // Soft delete
    public async softDelete(id: number): Promise<void> {
        await this.db('users')
            .where('id', id)
            .update({
                deleted_at: this.db.fn.now(),
                updated_at: this.db.fn.now()
            });
    }

    // Hard delete
    public async delete(id: number): Promise<void> {
        await this.db('users')
            .where('id', id)
            .delete();
    }

    // Bulk delete
    public async deleteInactive(): Promise<number> {
        const deletedCount = await this.db('users')
            .where('last_login_at', '<', this.db.raw('NOW() - INTERVAL \'1 year\''))
            .andWhere('active', false)
            .delete();

        return deletedCount;
    }
}
```

## Raw Queries

### When to Use Raw Queries

Use raw queries for complex operations that are difficult with the query builder:

```typescript
export default class ReportRepository {
    // Complex analytics query
    public async getCustomerLifetimeValue(): Promise<CustomerLTV[]> {
        const query = `
            WITH customer_orders AS (
                SELECT 
                    u.id,
                    u.name,
                    u.email,
                    COUNT(o.id) as order_count,
                    SUM(o.total) as total_spent,
                    MIN(o.created_at) as first_order,
                    MAX(o.created_at) as last_order,
                    EXTRACT(DAYS FROM MAX(o.created_at) - MIN(o.created_at)) as customer_lifespan
                FROM users u
                INNER JOIN orders o ON o.user_id = u.id
                WHERE o.status = 'completed'
                GROUP BY u.id, u.name, u.email
            ),
            ltv_calculations AS (
                SELECT 
                    *,
                    CASE 
                        WHEN customer_lifespan > 0 
                        THEN total_spent / (customer_lifespan / 365.0)
                        ELSE total_spent
                    END as annual_value,
                    CASE
                        WHEN customer_lifespan > 0
                        THEN (total_spent / (customer_lifespan / 30.0)) * 12
                        ELSE total_spent * 12
                    END as projected_ltv
                FROM customer_orders
            )
            SELECT 
                id,
                name,
                email,
                order_count,
                total_spent,
                customer_lifespan,
                annual_value,
                projected_ltv,
                CASE 
                    WHEN projected_ltv >= 1000 THEN 'High Value'
                    WHEN projected_ltv >= 500 THEN 'Medium Value'
                    ELSE 'Low Value'
                END as customer_segment
            FROM ltv_calculations
            ORDER BY projected_ltv DESC
            LIMIT ?
        `;

        const result = await this.db.raw(query, [100]);
        return result.rows;
    }

    // Database-specific optimizations
    public async refreshMaterializedView(): Promise<void> {
        if (this.db.client.config.client === 'postgresql') {
            await this.db.raw('REFRESH MATERIALIZED VIEW customer_summary');
        }
    }

    // Bulk operations with raw SQL
    public async syncProductCounts(): Promise<void> {
        await this.db.raw(`
            UPDATE categories 
            SET product_count = (
                SELECT COUNT(*) 
                FROM products 
                WHERE products.category_id = categories.id 
                AND products.active = true
            )
        `);
    }
}
```

## Multiple Database Connections

### Using Different Connections

```typescript
export default class DataService {
    private mainDb: Knex;
    private readOnlyDb: Knex;
    private analyticsDb: Knex;

    constructor() {
        this.mainDb = Database.getConnection('main').client;
        this.readOnlyDb = Database.getConnection('readonly').client;
        this.analyticsDb = Database.getConnection('analytics').client;
    }

    // Use read replica for heavy queries
    public async generateReport(): Promise<Report> {
        const data = await this.readOnlyDb('orders')
            .select([
                this.readOnlyDb.raw('DATE(created_at) as date'),
                this.readOnlyDb.raw('COUNT(*) as order_count'),
                this.readOnlyDb.raw('SUM(total) as revenue')
            ])
            .where('created_at', '>=', '2023-01-01')
            .groupBy(this.readOnlyDb.raw('DATE(created_at)'))
            .orderBy('date');

        return { data };
    }

    // Write to main database
    public async createUser(userData: CreateUserData): Promise<User> {
        return await this.mainDb('users')
            .insert(userData)
            .returning('*');
    }

    // Log to analytics database
    public async logEvent(eventData: AnalyticsEvent): Promise<void> {
        await this.analyticsDb('events').insert({
            user_id: eventData.userId,
            event_type: eventData.type,
            properties: JSON.stringify(eventData.properties),
            created_at: new Date()
        });
    }
}
```

## Best Practices

### 1. Use Repository Pattern

```typescript
// ✅ Good: Encapsulated database logic
export default class UserService {
    constructor(private userRepository: UserRepository) {}

    public async createUser(userData: CreateUserData): Promise<User> {
        return await this.userRepository.create(userData);
    }
}

// ❌ Bad: Direct database queries in service
export default class UserService {
    constructor(private db: Knex) {}

    public async createUser(userData: CreateUserData): Promise<User> {
        return await this.db('users').insert(userData); // Database logic in service
    }
}
```

### 2. Handle Errors Gracefully

```typescript
// ✅ Good: Proper error handling
export default class UserRepository {
    public async findByEmail(email: string): Promise<User | null> {
        try {
            const user = await this.db('users')
                .select('*')
                .where('email', email)
                .first();
                
            return user ? new User(user) : null;
        } catch (error) {
            this.logger.error('Failed to find user by email', { email, error });
            throw new DatabaseQueryError('User lookup failed');
        }
    }
}
```

### 3. Use Transactions for Related Operations

```typescript
// ✅ Good: Transactional consistency
export default class OrderRepository {
    public async createOrder(orderData: CreateOrderData): Promise<Order> {
        return await this.db.transaction(async (trx) => {
            // Create order
            const [order] = await trx('orders')
                .insert({
                    user_id: orderData.userId,
                    total: orderData.total,
                    created_at: new Date()
                })
                .returning('*');

            // Create order items
            await trx('order_items').insert(
                orderData.items.map(item => ({
                    order_id: order.id,
                    product_id: item.productId,
                    quantity: item.quantity,
                    price: item.price
                }))
            );

            // Update product stock
            for (const item of orderData.items) {
                await trx('products')
                    .where('id', item.productId)
                    .decrement('stock_quantity', item.quantity);
            }

            return new Order(order);
        });
    }
}
```

### 4. Optimize Query Performance

```typescript
// ✅ Good: Efficient queries with proper indexing
export default class ProductRepository {
    // Use specific columns instead of *
    public async findForListing(): Promise<ProductListing[]> {
        return await this.db('products')
            .select(['id', 'name', 'price', 'thumbnail'])
            .where('active', true)
            .orderBy('created_at', 'desc')
            .limit(50);
    }

    // Use indexes for filtering
    public async findByCategory(categoryId: number): Promise<Product[]> {
        return await this.db('products')
            .select('*')
            .where('category_id', categoryId) // Ensure index on category_id
            .andWhere('active', true)        // Ensure composite index
            .orderBy('name');
    }
}
```

## Summary

Sosise's database layer provides:

- ✅ Multiple database support through Knex.js
- ✅ Repository pattern for clean architecture
- ✅ Connection pooling and management
- ✅ Query builder with raw SQL support
- ✅ Transaction support for data consistency
- ✅ Multiple connection support
- ✅ Migration and seeding tools
- ✅ Type-safe query building

Build robust, scalable applications with Sosise's powerful database capabilities!