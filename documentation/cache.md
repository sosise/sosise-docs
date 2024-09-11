# Cache

## Introduction
Your application may execute certain data retrieval or processing tasks that are CPU-intensive or time-consuming. In these scenarios, caching the data temporarily can offer rapid retrieval for future requests that require the same data.

Sosise provides a robust API for various cache backends, allowing you to leverage their fast data retrieval capabilities to boost your application's performance.

## Supported Cache Drivers
- `Filesystem`
- `Memory (RAM)`
- `Redis`

## Configuration
The cache configuration file is located at `src/config/cache.ts`. This file allows you to specify the default cache driver and contains other options. By default, Sosise uses the `Memory (RAM)` cache driver.

## Accessing Cache

### Accessing the `CacheService`
The `CacheService` is registered as a singleton in the Sosise core Inversion of Control (IOC). You can use it in your IOC configuration or directly from your code.

### Example: Accessing via `IOC`
`src/config/ioc.ts`:
```typescript
import IOC from "sosise-core/build/ServiceProviders/IOC";
import CacheService from "sosise-core/build/Services/Cache/CacheService";
import CustomerService from "../app/Services/CustomerService";

const iocConfig = {
    singletons: {
        // Register CacheService as a singleton
    },
    nonSingletons: {
        CustomerService: () => new CustomerService(IOC.makeSingleton(CacheService)),
    }
};

export default iocConfig;
```

`src/app/services/CustomerService.ts`:
```typescript
import CacheService from "sosise-core/build/Services/Cache/CacheService";

export default class CustomerService {

    private cacheService: CacheService;

    constructor(cacheService: CacheService) {
        this.cacheService = cacheService;
    }

    public async getAllCustomers(): Promise<any> {
        const allCustomers = await this.cacheService.get('all-customers');
        return allCustomers;
    }
}
```

### Example: Accessing Directly in Code
```typescript
import IOC from "sosise-core/build/ServiceProviders/IOC";
import CacheService from "sosise-core/build/Services/Cache/CacheService";

[...]

public async getAllCustomers(): Promise<any> {
    const cacheService = IOC.makeSingleton(CacheService) as CacheService;
    const allCustomers = await cacheService.get('all-customers');
    console.log(allCustomers);
}
```

---

## Cache Usage

### Fetching Items from the Cache
```typescript
get(key: string): Promise<any | null>
```
#### Example:
```typescript
const cachedData = await cacheService.get('all-customers');
```

### Fetch & Remove
```typescript
pull(key: string): Promise<any | null>
```
#### Example:
```typescript
const cachedData = await cacheService.pull('all-customers');
```

### Saving Items in the Cache
```typescript
put(key: string, data: any, ttlInSeconds?: number): Promise<void>
```
#### Example:
```typescript
const allCustomers = [
    { id: 1, name: 'Foo', mobile: '123' },
    { id: 2, name: 'Bar', mobile: '456' },
];
await cacheService.put('all-customers', allCustomers);
```

### Saving Items Permanently in the Cache
```typescript
putForever(key: string, data: any): Promise<void>
```
#### Example:
```typescript
await cacheService.putForever('all-customers', allCustomers);
```

### Checking If an Item Exists
```typescript
has(key: string): Promise<boolean>
```
#### Example:
```typescript
const exists = await cacheService.has('all-customers');
if (exists) {
    console.log('Returning customers from cache!');
}
```

### Fetching Cache Keys
```typescript
keys(regex?: RegExp): Promise<string[]>
```
#### Example:
```typescript
const customerKeys = await cacheService.keys(new RegExp('customer-13[0-9]{1,3}'));
```

### Removing Cache Item
```typescript
delete(key: string): Promise<void>
```
#### Example:
```typescript
await cacheService.delete('all-customers');
```

### Removing Multiple Cache Items
```typescript
deleteMany(keys: string[]): Promise<void>
```
#### Example:
```typescript
await cacheService.deleteMany(['all-customers', 'foo', 'bar']);
```

### Flush Cache
```typescript
flush(): Promise<void>
```
#### Example:
```typescript
await cacheService.flush();
```

---

## Redis-Specific Methods

### Fetching Multiple Items from Redis Cache (Redis Only)
This method is **only applicable** to the Redis cache driver and allows you to retrieve multiple cache items by their keys.

```typescript
getMany(keys: string[]): Promise<any[]>
```

#### Example:
```typescript
const customers = await cacheService.getMany(['customer-1', 'customer-2', 'customer-3']);
console.log(customers);  // Outputs an array of the cached data
```

---

## Filesystem-Specific Methods

### Fetching All Cache Keys with Expiration Timestamps (Filesystem Only)
This method is **only applicable** to the Filesystem cache driver and allows you to retrieve all cache keys along with their expiration timestamps.

```typescript
getAllCacheKeysWithTimestamps(): Promise<{ key: string, expiresAtTimestamp: number }[]>
```

#### Example:
```typescript
const cacheKeysWithTimestamps = await cacheService.getAllCacheKeysWithTimestamps();
console.log(cacheKeysWithTimestamps);
```