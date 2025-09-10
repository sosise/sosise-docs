# Database Migrations

## Quick Start

Manage your database schema like code with Sosise migrations - version control for your database!

```bash
# Create a new table migration
./artisan make:migration create_users_table

# Create a migration to update existing table  
./artisan make:migration add_email_index_to_users_table -u

# Run migrations
./artisan migrate

# Rollback last migration batch
./artisan migrate:rollback
```

```typescript
// Example migration
export default class CreateUsersTable extends BaseSchema {
    protected tableName = 'users';

    public async up(): Promise<void> {
        await this.dbConnection.schema.createTable(this.tableName, (table) => {
            table.increments('id');
            table.string('name').notNullable();
            table.string('email').unique().notNullable();
            table.string('password').notNullable();
            table.boolean('active').defaultTo(true);
            table.timestamps(true, true);
        });
    }

    public async down(): Promise<void> {
        await this.dbConnection.schema.dropTable(this.tableName);
    }
}
```

Keep your database schema in sync across all environments!

## Introduction

Database migrations are version control for your database schema. They allow you to define database changes in code, share them with your team, and apply them consistently across different environments. 

With Sosise migrations, you can create tables, add columns, create indexes, and modify constraints using TypeScript code instead of running manual SQL commands. This ensures your database structure is reproducible, trackable, and deployable alongside your application code.

## Migration Basics

### Creating Migrations

Generate new migrations using Artisan commands:

```bash
# Create new table migration
./artisan make:migration create_products_table

# Update existing table migration  
./artisan make:migration add_price_index_to_products_table -u

# Create migration with timestamp for ordering
./artisan make:migration add_user_preferences_table
```

Migration files are created in `database/migrations/` with timestamps:
```
2023_12_25_143022_create_products_table.ts
2023_12_25_143045_add_price_index_to_products_table.ts
```

### Migration Structure

Every migration has `up()` and `down()` methods:

```typescript
import BaseSchema from 'sosise-core/build/Database/BaseSchema';

export default class CreateProductsTable extends BaseSchema {
    protected tableName = 'products';

    // Apply changes
    public async up(): Promise<void> {
        await this.dbConnection.schema.createTable(this.tableName, (table) => {
            table.increments('id');
            table.string('name').notNullable();
            table.decimal('price', 10, 2).notNullable();
            table.text('description');
            table.integer('category_id').unsigned().references('id').inTable('categories');
            table.integer('stock_quantity').defaultTo(0);
            table.boolean('active').defaultTo(true);
            table.timestamps(true, true);
            
            // Indexes
            table.index(['category_id', 'active']);
            table.index('name');
        });
    }

    // Reverse changes
    public async down(): Promise<void> {
        await this.dbConnection.schema.dropTable(this.tableName);
    }
}
```

## Table Operations

### Creating Tables

```typescript
export default class CreateOrdersTable extends BaseSchema {
    protected tableName = 'orders';

    public async up(): Promise<void> {
        await this.dbConnection.schema.createTable(this.tableName, (table) => {
            // Primary key
            table.increments('id');
            
            // Foreign keys
            table.integer('user_id').unsigned().notNullable()
                .references('id').inTable('users').onDelete('CASCADE');
            
            // Basic columns
            table.string('order_number').unique().notNullable();
            table.enum('status', ['pending', 'processing', 'shipped', 'delivered', 'cancelled'])
                .defaultTo('pending');
            table.decimal('total', 10, 2).notNullable();
            table.decimal('tax_amount', 10, 2).defaultTo(0);
            table.decimal('shipping_cost', 10, 2).defaultTo(0);
            
            // JSON column for metadata
            table.json('shipping_address').notNullable();
            table.json('billing_address');
            table.json('metadata');
            
            // Timestamps
            table.timestamps(true, true);
            table.timestamp('shipped_at').nullable();
            
            // Indexes for performance
            table.index(['user_id', 'status']);
            table.index('order_number');
            table.index('created_at');
        });
    }

    public async down(): Promise<void> {
        await this.dbConnection.schema.dropTable(this.tableName);
    }
}
```

### Column Types

```typescript
export default class CreateComprehensiveTable extends BaseSchema {
    protected tableName = 'comprehensive_example';

    public async up(): Promise<void> {
        await this.dbConnection.schema.createTable(this.tableName, (table) => {
            // Numeric types
            table.increments('id');
            table.integer('count').notNullable();
            table.decimal('price', 10, 2);
            table.float('rating', 3, 2);
            table.boolean('active').defaultTo(true);
            
            // String types
            table.string('name', 100).notNullable();
            table.text('description');
            table.specificType('slug', 'varchar(100)').unique();
            
            // Date/time types
            table.timestamp('created_at').defaultTo(this.dbConnection.fn.now());
            table.datetime('scheduled_at');
            table.date('birth_date');
            table.time('preferred_time');
            
            // Enum
            table.enum('type', ['basic', 'premium', 'enterprise']).defaultTo('basic');
            
            // JSON (PostgreSQL/MySQL 5.7+)
            table.json('settings');
            table.json('tags');
            
            // Binary
            table.binary('file_data');
            
            // UUID (PostgreSQL)
            table.uuid('public_id').defaultTo(this.dbConnection.raw('gen_random_uuid()'));
            
            // Constraints
            table.unique(['name', 'type']);
            table.index(['active', 'created_at']);
        });
    }

    public async down(): Promise<void> {
        await this.dbConnection.schema.dropTable(this.tableName);
    }
}
```

## Modifying Tables

### Adding Columns

```typescript
export default class AddEmailVerificationToUsers extends BaseSchema {
    protected tableName = 'users';

    public async up(): Promise<void> {
        await this.dbConnection.schema.table(this.tableName, (table) => {
            table.timestamp('email_verified_at').nullable();
            table.string('email_verification_token', 100).nullable();
            table.boolean('is_email_verified').defaultTo(false);
            
            // Add index
            table.index('email_verification_token');
        });
    }

    public async down(): Promise<void> {
        await this.dbConnection.schema.table(this.tableName, (table) => {
            table.dropColumn('email_verified_at');
            table.dropColumn('email_verification_token'); 
            table.dropColumn('is_email_verified');
            
            // Note: Indexes are dropped automatically with columns
        });
    }
}
```

### Modifying Columns

```typescript
export default class ModifyUserColumns extends BaseSchema {
    protected tableName = 'users';

    public async up(): Promise<void> {
        await this.dbConnection.schema.table(this.tableName, (table) => {
            // Change column type
            table.string('phone', 20).nullable().alter();
            
            // Add not null constraint
            table.string('name').notNullable().alter();
            
            // Change default value
            table.boolean('active').defaultTo(true).alter();
        });
    }

    public async down(): Promise<void> {
        await this.dbConnection.schema.table(this.tableName, (table) => {
            table.string('phone').alter(); // Remove length constraint
            table.string('name').nullable().alter(); // Remove not null
            table.boolean('active').defaultTo(false).alter(); // Revert default
        });
    }
}
```

### Adding Indexes and Constraints

```typescript
export default class AddIndexesToProducts extends BaseSchema {
    protected tableName = 'products';

    public async up(): Promise<void> {
        await this.dbConnection.schema.table(this.tableName, (table) => {
            // Simple index
            table.index('name', 'idx_products_name');
            
            // Composite index
            table.index(['category_id', 'active', 'created_at'], 'idx_products_category_active_created');
            
            // Unique constraint
            table.unique(['sku'], 'uq_products_sku');
            
            // Foreign key constraint
            table.foreign('category_id', 'fk_products_category')
                .references('id').inTable('categories')
                .onDelete('SET NULL')
                .onUpdate('CASCADE');
        });
    }

    public async down(): Promise<void> {
        await this.dbConnection.schema.table(this.tableName, (table) => {
            table.dropIndex('name', 'idx_products_name');
            table.dropIndex(['category_id', 'active', 'created_at'], 'idx_products_category_active_created');
            table.dropUnique(['sku'], 'uq_products_sku');
            table.dropForeign(['category_id'], 'fk_products_category');
        });
    }
}
```

## Advanced Migration Patterns

### Data Migrations

Sometimes you need to migrate data along with schema changes:

```typescript
export default class MigrateUserRoles extends BaseSchema {
    protected tableName = 'users';

    public async up(): Promise<void> {
        // 1. Add new role column
        await this.dbConnection.schema.table(this.tableName, (table) => {
            table.string('role').defaultTo('user');
        });
        
        // 2. Migrate data
        await this.dbConnection(this.tableName)
            .where('is_admin', true)
            .update({ role: 'admin' });
            
        // 3. Remove old column
        await this.dbConnection.schema.table(this.tableName, (table) => {
            table.dropColumn('is_admin');
        });
    }

    public async down(): Promise<void> {
        // 1. Add back old column
        await this.dbConnection.schema.table(this.tableName, (table) => {
            table.boolean('is_admin').defaultTo(false);
        });
        
        // 2. Migrate data back
        await this.dbConnection(this.tableName)
            .where('role', 'admin')
            .update({ is_admin: true });
            
        // 3. Remove new column
        await this.dbConnection.schema.table(this.tableName, (table) => {
            table.dropColumn('role');
        });
    }
}
```

### Conditional Migrations

```typescript
export default class AddGinIndexIfPostgres extends BaseSchema {
    protected tableName = 'articles';

    public async up(): Promise<void> {
        const client = this.dbConnection.client.config.client;
        
        await this.dbConnection.schema.table(this.tableName, (table) => {
            if (client === 'postgresql') {
                // PostgreSQL-specific GIN index for full-text search
                table.specificType('search_vector', 'tsvector');
            } else {
                // Fallback for other databases
                table.text('search_text');
                table.index('search_text');
            }
        });
        
        // Create trigger for PostgreSQL
        if (client === 'postgresql') {
            await this.dbConnection.raw(`
                CREATE OR REPLACE FUNCTION update_search_vector() RETURNS trigger AS $$
                BEGIN
                    NEW.search_vector = to_tsvector('english', COALESCE(NEW.title, '') || ' ' || COALESCE(NEW.content, ''));
                    RETURN NEW;
                END;
                $$ LANGUAGE plpgsql;
                
                CREATE TRIGGER update_search_vector_trigger
                    BEFORE INSERT OR UPDATE ON ${this.tableName}
                    FOR EACH ROW EXECUTE FUNCTION update_search_vector();
            `);
        }
    }

    public async down(): Promise<void> {
        const client = this.dbConnection.client.config.client;
        
        if (client === 'postgresql') {
            await this.dbConnection.raw(`
                DROP TRIGGER IF EXISTS update_search_vector_trigger ON ${this.tableName};
                DROP FUNCTION IF EXISTS update_search_vector();
            `);
        }
        
        await this.dbConnection.schema.table(this.tableName, (table) => {
            if (client === 'postgresql') {
                table.dropColumn('search_vector');
            } else {
                table.dropColumn('search_text');
            }
        });
    }
}
```

## Running Migrations

### Basic Migration Commands

```bash
# Run all pending migrations
./artisan migrate
```

### Rollback Operations

```bash
# Rollback last batch
./artisan migrate:rollback

# Force rollback without confirmation (bypasses environment check)
./artisan migrate:rollback --force
```

### Fresh Migrations

```bash
# Drop all tables and re-run migrations
./artisan migrate:fresh

# Fresh with force (bypasses environment check)
./artisan migrate:fresh --force
```

## Production Deployment

### Safe Migration Practices

1. **Always backup before production migrations**
```bash
# Example backup command (adjust for your database)
pg_dump myapp_production > backup_$(date +%Y%m%d_%H%M%S).sql
```

2. **Test migrations on staging first**
```bash
# Run on staging environment
NODE_ENV=staging ./artisan migrate
```

3. **Use migration locking for zero-downtime deployments**
```bash
# Migrations are automatically locked during execution to prevent conflicts
./artisan migrate
```

### Environment-Specific Migrations

```typescript
export default class AddIndexForProduction extends BaseSchema {
    protected tableName = 'orders';

    public async up(): Promise<void> {
        // Only create expensive indexes in production
        if (process.env.NODE_ENV === 'production') {
            await this.dbConnection.schema.table(this.tableName, (table) => {
                table.index(['created_at', 'status', 'total'], 'idx_orders_reporting');
            });
        }
    }

    public async down(): Promise<void> {
        if (process.env.NODE_ENV === 'production') {
            await this.dbConnection.schema.table(this.tableName, (table) => {
                table.dropIndex(['created_at', 'status', 'total'], 'idx_orders_reporting');
            });
        }
    }
}
```

## Best Practices

### 1. Always Write Down Methods

```typescript
// ✅ Good: Reversible migration
export default class AddUserPreferences extends BaseSchema {
    public async up(): Promise<void> {
        await this.dbConnection.schema.table('users', (table) => {
            table.json('preferences').defaultTo('{}');
        });
    }

    public async down(): Promise<void> {
        await this.dbConnection.schema.table('users', (table) => {
            table.dropColumn('preferences');
        });
    }
}

// ❌ Bad: No rollback plan
export default class AddUserPreferences extends BaseSchema {
    public async up(): Promise<void> {
        await this.dbConnection.schema.table('users', (table) => {
            table.json('preferences').defaultTo('{}');
        });
    }

    public async down(): Promise<void> {
        // Empty - can't rollback!
    }
}
```

### 2. Use Descriptive Names

```typescript
// ✅ Good: Clear, descriptive names
./artisan make:migration add_email_verification_to_users_table -u
./artisan make:migration create_order_items_table
./artisan make:migration add_fulltext_index_to_articles_table -u

// ❌ Bad: Vague names
./artisan make:migration update_users
./artisan make:migration fix_table
./artisan make:migration add_stuff
```

### 3. Keep Migrations Focused

```typescript
// ✅ Good: Single responsibility
export default class AddEmailVerificationToUsers extends BaseSchema {
    // Only deals with email verification
    public async up(): Promise<void> {
        await this.dbConnection.schema.table('users', (table) => {
            table.timestamp('email_verified_at').nullable();
            table.string('email_verification_token').nullable();
        });
    }
}

// ❌ Bad: Multiple unrelated changes
export default class UpdateUsersAndProducts extends BaseSchema {
    public async up(): Promise<void> {
        // Modifying users
        await this.dbConnection.schema.table('users', (table) => {
            table.string('phone').nullable();
        });
        
        // Also modifying products (should be separate migration)
        await this.dbConnection.schema.table('products', (table) => {
            table.decimal('weight', 8, 2).nullable();
        });
    }
}
```

### 4. Handle Foreign Key Dependencies

```typescript
// ✅ Good: Proper dependency order
export default class CreateOrderItemsTable extends BaseSchema {
    public async up(): Promise<void> {
        await this.dbConnection.schema.createTable('order_items', (table) => {
            table.increments('id');
            
            // Create foreign keys after referenced tables exist
            table.integer('order_id').unsigned().notNullable();
            table.integer('product_id').unsigned().notNullable();
            
            // Add constraints
            table.foreign('order_id').references('id').inTable('orders').onDelete('CASCADE');
            table.foreign('product_id').references('id').inTable('products').onDelete('RESTRICT');
        });
    }
}
```

## Troubleshooting

### Common Issues

1. **Migration fails due to existing data**
```typescript
// Solution: Handle existing data gracefully
public async up(): Promise<void> {
    // Check if column exists
    const hasColumn = await this.dbConnection.schema.hasColumn('users', 'email_verified');
    
    if (!hasColumn) {
        await this.dbConnection.schema.table('users', (table) => {
            table.boolean('email_verified').defaultTo(false);
        });
    }
}
```

2. **Foreign key constraint failures**
```bash
# Check existing data before adding constraints
SELECT COUNT(*) FROM orders WHERE user_id NOT IN (SELECT id FROM users);

# Clean up orphaned records first
DELETE FROM orders WHERE user_id NOT IN (SELECT id FROM users);
```

3. **Schema differences between environments**
```bash
# Check which migrations have been run manually in each environment
# (No built-in status command - check your migrations table directly)
```

## Summary

Sosise migrations provide:

- ✅ Version control for database schema
- ✅ Team collaboration on database changes  
- ✅ Consistent deployments across environments
- ✅ Rollback capabilities for safe changes
- ✅ TypeScript support with full IDE assistance
- ✅ Multi-database compatibility
- ✅ Production-ready deployment tools
- ✅ Data migration capabilities

Keep your database schema organized and deployable with Sosise migrations!