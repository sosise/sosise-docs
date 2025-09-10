# Cache

## Quick Start

Caching makes your app faster by remembering expensive operations. Instead of hitting the database every time, cache the results!

### Simple Example

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import CacheService from 'sosise-core/build/Services/Cache/CacheService';

// Get cache service
const cache = IOC.makeSingleton(CacheService) as CacheService;

// Save data to cache for 1 hour (3600 seconds)
await cache.put('products', expensiveProducts, 3600);

// Get data from cache
const products = await cache.get('products');
if (products) {
    // Use cached data - fast!
    return products;
}
```

That's it! Your app is now faster with caching.

## Introduction

Caching stores the results of expensive operations so you don't have to repeat them. Think of it as your app's short-term memory. When users request the same data repeatedly, you serve it from cache instead of recalculating or fetching it from the database every time.

Sosise supports multiple cache drivers (Memory, Filesystem, Redis) and provides a simple, consistent API for all of them. The cache configuration is in `src/config/cache.ts`.

## Basic Usage

### Setting Up Cache in Your Service

The recommended way is to inject CacheService through IoC:

```typescript
// src/app/Services/ProductService.ts
import CacheService from 'sosise-core/build/Services/Cache/CacheService';

export default class ProductService {
    constructor(
        private cache: CacheService,
        private productRepository: ProductRepository
    ) {}

    public async getPopularProducts(): Promise<Product[]> {
        const cacheKey = 'popular_products';
        
        // Try to get from cache first
        const cached = await this.cache.get(cacheKey);
        if (cached) {
            return cached; // Fast!
        }

        // Not in cache, fetch from database
        const products = await this.productRepository.getPopular();
        
        // Save to cache for 30 minutes
        await this.cache.put(cacheKey, products, 1800);
        
        return products;
    }
}
```

Register in IoC:

```typescript
// src/config/ioc.ts
const iocConfig = {
    nonSingletons: {
        ProductService: () => new ProductService(
            IOC.makeSingleton(CacheService),
            new ProductRepository()
        ),
    }
};
```

## Common Cache Operations

### Save Data to Cache

```typescript
// Cache for specific time (in seconds)
await cache.put('user:123', userData, 3600); // 1 hour

// Cache forever
await cache.putForever('app:settings', settings);
```

### Get Data from Cache

```typescript
// Get cached data (returns null if not found)
const user = await cache.get('user:123');

if (user) {
    // Use cached data
} else {
    // Fetch fresh data
}
```

### Check if Data Exists

```typescript
if (await cache.has('user:123')) {
    console.log('User is cached');
}
```

### Remove from Cache

```typescript
// Delete single item
await cache.delete('user:123');

// Delete multiple items
await cache.deleteMany(['user:123', 'user:456', 'user:789']);

// Clear entire cache
await cache.flush();
```

### Get and Remove

```typescript
// Get value and remove it from cache in one operation
const tempData = await cache.pull('temporary:data');
```

## Cache Patterns

### Cache-Aside Pattern

The most common pattern - check cache first, fetch if missing:

```typescript
public async getUser(id: number): Promise<User> {
    const cacheKey = `user:${id}`;
    
    // Check cache
    let user = await this.cache.get(cacheKey);
    
    if (!user) {
        // Not in cache, fetch from database
        user = await this.userRepository.findById(id);
        
        if (user) {
            // Save to cache
            await this.cache.put(cacheKey, user, 3600);
        }
    }
    
    return user;
}
```

### Cache Invalidation

Update cache when data changes:

```typescript
public async updateUser(id: number, data: UpdateUserData): Promise<User> {
    // Update in database
    const user = await this.userRepository.update(id, data);
    
    // Invalidate old cache
    await this.cache.delete(`user:${id}`);
    
    // Optionally, cache the new data
    await this.cache.put(`user:${id}`, user, 3600);
    
    return user;
}
```

### Tagged Cache (Manual Implementation)

Group related cache entries:

```typescript
public async clearProductCache(categoryId: number): Promise<void> {
    // Find all cache keys for this category
    const keys = await this.cache.keys(new RegExp(`category:${categoryId}:.*`));
    
    // Delete all related cache entries
    if (keys.length > 0) {
        await this.cache.deleteMany(keys);
    }
}
```

## Configuration

Configure your cache driver in `src/config/cache.ts`:

```typescript
export default {
    // Default cache driver
    default: process.env.CACHE_DRIVER || 'memory',
    
    // Driver configurations
    drivers: {
        memory: {
            driver: 'memory',
            // Memory cache is cleared on app restart
        },
        
        filesystem: {
            driver: 'filesystem',
            path: 'storage/cache',
            // Persists between restarts
        },
        
        redis: {
            driver: 'redis',
            host: process.env.REDIS_HOST || '127.0.0.1',
            port: process.env.REDIS_PORT || 6379,
            database: 0,
            // Best for production
        }
    }
};
```

## Cache Drivers

### Memory (RAM)

- **Pros**: Fastest, no setup required
- **Cons**: Lost on restart, limited by RAM
- **Use for**: Development, small datasets

```typescript
default: 'memory'
```

### Filesystem

- **Pros**: Persists on restart, simple
- **Cons**: Slower than memory, disk I/O
- **Use for**: Medium datasets, single server

```typescript
default: 'filesystem'
```

### Redis

- **Pros**: Fast, scalable, shared between servers
- **Cons**: Requires Redis server
- **Use for**: Production, multi-server setups

```typescript
default: 'redis'
```

## Advanced Features

### Search Cache Keys

Find cache keys matching a pattern:

```typescript
// Find all user cache entries
const userKeys = await cache.keys(new RegExp('user:.*'));

// Find specific range
const orderKeys = await cache.keys(new RegExp('order:2024-.*'));
```

### Redis-Specific Features

When using Redis, you get additional methods:

```typescript
// Get multiple items at once (Redis only)
const users = await cache.getMany(['user:1', 'user:2', 'user:3']);

// Save multiple items at once (Redis only)
await cache.putMany([
    { key: 'user:1', value: user1 },
    { key: 'user:2', value: user2 },
    { key: 'user:3', value: user3 }
], 3600);
```

### Filesystem-Specific Features

When using filesystem cache:

```typescript
// Get all keys with expiration times (Filesystem only)
const keysWithExpiry = await cache.getAllCacheKeysWithTimestamps();
// Returns: [{ key: 'user:1', expiresAtTimestamp: 1234567890 }, ...]
```

## Best Practices

### 1. Use Meaningful Cache Keys

```typescript
// ✅ Good: Descriptive and namespaced
const cacheKey = `product:${productId}:details`;
const cacheKey = `user:${userId}:preferences`;
const cacheKey = `category:${categoryId}:products:page:${page}`;

// ❌ Bad: Unclear or collision-prone
const cacheKey = productId.toString();
const cacheKey = 'data';
```

### 2. Set Appropriate TTL (Time To Live)

```typescript
// Short TTL for frequently changing data
await cache.put('trending:products', products, 300); // 5 minutes

// Long TTL for stable data
await cache.put('country:list', countries, 86400); // 24 hours

// No TTL for permanent data
await cache.putForever('app:config', config);
```

### 3. Handle Cache Misses Gracefully

```typescript
public async getData(key: string): Promise<any> {
    try {
        // Try cache first
        const cached = await this.cache.get(key);
        if (cached) return cached;
        
        // Fetch fresh data
        const data = await this.fetchFromSource(key);
        
        // Cache for next time
        await this.cache.put(key, data, 3600);
        
        return data;
    } catch (error) {
        // If cache fails, still return data
        this.logger.error('Cache error:', error);
        return this.fetchFromSource(key);
    }
}
```

### 4. Invalidate Related Cache

```typescript
public async updateProduct(id: number, data: any): Promise<void> {
    await this.productRepository.update(id, data);
    
    // Clear all related cache
    await this.cache.delete(`product:${id}`);
    await this.cache.delete(`product:${id}:reviews`);
    await this.cache.delete('products:popular');
    await this.cache.delete('products:latest');
}
```

### 5. Use Cache Service in IoC

Always inject cache through IoC for better testing:

```typescript
// ✅ Good: Injected through constructor
constructor(private cache: CacheService) {}

// ❌ Avoid: Direct instantiation
const cache = IOC.makeSingleton(CacheService);
```

## Cache Debugging

### Monitor Cache Usage

```typescript
public async getCachedData(key: string): Promise<any> {
    const startTime = Date.now();
    const data = await this.cache.get(key);
    const duration = Date.now() - startTime;
    
    if (data) {
        this.logger.info(`Cache HIT: ${key} (${duration}ms)`);
    } else {
        this.logger.info(`Cache MISS: ${key}`);
    }
    
    return data;
}
```

### Clear Cache During Development

Add a helper command:

```typescript
// src/app/Console/Commands/CacheClearCommand.ts
export default class CacheClearCommand extends BaseCommand {
    protected signature = 'cache:clear';
    protected description = 'Clear all cache';

    public async handle(): Promise<void> {
        const cache = IOC.makeSingleton(CacheService) as CacheService;
        await cache.flush();
        console.log('Cache cleared successfully!');
    }
}
```

## Summary

Caching is essential for building fast applications:

- ✅ Reduces database load
- ✅ Improves response times
- ✅ Saves computational resources
- ✅ Supports multiple storage backends
- ✅ Simple, consistent API
- ✅ Easy to integrate with services