# Cache
# Introduction
Your application may execute certain data retrieval or processing tasks that are CPU-intensive or significantly time-consuming. For these scenarios, caching the fetched data temporarily can offer rapid retrieval for future requests that require the same data.

Sosise provides a robust and consistent API for various cache backends, enabling you to leverage their fast data retrieval capabilities to boost your application's performance.

# Supported cache drivers
- `Filesystem`
- `Memory(RAM)`
- `Redis`

# Configuration
The cache configuration file for your application is located at `src/config/cache.ts`. This file allows you to specify the default cache driver that your application will use.

Additionally, the cache configuration file contains several other options that are well-documented. Therefore, it's crucial to review these settings thoroughly. By default, Sosise is set to use the `Memory (RAM)` cache driver.

# Accessing Cache
## How to Access the `CacheService`
The CacheService is a service registered as a singleton in the Sosise core Inversion of Control (IOC). This means you can utilize it in your IOC configuration or makeSingleton directly from your code.

## Example of Accessing Through `IOC`
`src/config/ioc.ts`
```typescript
import IOC from "sosise-core/build/ServiceProviders/IOC";
import CacheService from "sosise-core/build/Services/Cache/CacheService";
import CustomerService from "../app/Services/CustomerService";

/**
 * IOC Config, please register here your services
 */
const iocConfig = {
    /**
     * Singleton services
     *
     * How to register:
     * YourServiceName: () => new YourServiceName()
     *
     * How to use:
     * const logger = IOC.makeSingleton(LoggerService);
     */
    singletons: {
    },

    /**
     * Non singleton services
     *
     * How to register:
     * YourServiceName: () => new YourServiceName()
     *
     * How to use:
     * const logger = IOC.make(LoggerService);
     */
    nonSingletons: {
        CustomerService: () => new CustomerService(
            IOC.makeSingleton(CacheService),
        ),
    }
};

export default iocConfig;
```

`src/app/services/CustomerService.ts`
```typescript
import CacheService from "sosise-core/build/Services/Cache/CacheService";

export default class CustomerService {

    private cacheService: CacheService;

    /**
     * Constructor
     */
    constructor(cacheService: CacheService) {
        this.cacheService = cacheService;
    }

    /**
     * Get all customers
     */
    public async getAllCustomers(): Promise<any> {
        const allCustomers = await this.cacheService.get('all-customers');
        return allCustomers;
    }
}
```

## Example of Accessing Directly in Your Code
```typescript
import IOC from "sosise-core/build/ServiceProviders/IOC";
import CacheService from "sosise-core/build/Services/Cache/CacheService";

[...]

/**
 * Get all customers
 */
public async getAllCustomers(): Promise<any> {
    const cacheService = IOC.makeSingleton(CacheService) as CacheService;
    const allCustomers = await cacheService.get('all-customers');
    console.log(allCustomers);
}
```

# Cache usage
## Fetching Items From The Cache
### Syntax
```typescript
get(key: string): Promise<any | null>
```

### Example
```typescript
const cachedData = await cacheService.get('all-customers');
```

## Fetch & Remove
### Syntax
```typescript
pull(key: string): Promise<any | null>
```

### Example
```typescript
const cachedData = await cacheService.pull('all-customers');
```

## Saving Items In The Cache
### Syntax
You can configure the default `ttlInSeconds` in `src/config/cache.ts`
```typescript
put(key: string, data: any, ttlInSeconds?: number): Promise<void>
```

### Example
```typescript
const allCustomers = [
    {
        id: 1,
        name: 'Foo',
        mobile: '123'
    },
    {
        id: 2,
        name: 'Bar',
        mobile: '456'
    },
];
await cacheService.put('all-customers', allCustomers);
```

## Saving Items Permanently In The Cache
### Syntax
```typescript
putForever(key: string, data: any): Promise<void>
```

### Example
```typescript
const allCustomers = [
    {
        id: 1,
        name: 'Foo',
        mobile: '123'
    },
    {
        id: 2,
        name: 'Bar',
        mobile: '456'
    },
];
await cacheService.putForever('all-customers', allCustomers);
```

## Checking If An Item Exists
### Syntax
```typescript
has(key: string): Promise<boolean>
```

### Example
```typescript
const allCustomers = await cacheService.has('all-customers');

if (allCustomers) {
    console.log('Returning customers from cache!');
}
```

## Fetching Cache Keys
### Syntax
If a regex is not specified, all keys will be returned.
```typescript
keys(regex?: RegExp): Promise<string[]>
```

### Example
```typescript
const customerKeys = await cacheService.keys(new RegExp('customer-13[0-9]{1,3}'));
```

## Removing Cache Item
### Syntax
```typescript
delete(key: string): Promise<void>
```

### Example
```typescript
await cacheService.delete('all-customers');
```

## Removing Cache Items
### Syntax
```typescript
deleteMany(keys: string[]): Promise<void>
```

### Example
```typescript
await cacheService.deleteMany(['all-customers', 'foo', 'bar']);
```

## Flush Cache
### Syntax
```typescript
flush(): Promise<void>
```

### Example
```typescript
await cacheService.flush();
```