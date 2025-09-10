# IoC (Inversion of Control)

## Quick Start

The IoC (Inversion of Control) container helps you manage dependencies in your application. Instead of creating services manually, you register them once and the container provides them when needed.

### Basic Example

1. **Register a service** in `src/config/ioc.ts`:

```typescript
const iocConfig = {
    nonSingletons: {
        UserService: () => new UserService(),
    },
};
```

2. **Use the service** in your controller:

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import UserService from '../Services/UserService';

export default class UserController {
    public async getUsers(request: Request, response: Response, next: NextFunction) {
        // Get service from container
        const userService = IOC.make(UserService) as UserService;
        
        // Use the service
        const users = await userService.getAllUsers();
        response.json(users);
    }
}
```

That's it! The IoC container handles creating and providing your services.

## Introduction

The IoC (Inversion of Control) container in Sosise is a powerful dependency management mechanism that allows you to create a loosely coupled application architecture. Instead of classes creating their own dependencies, the IoC container takes this responsibility.

### Benefits of Using IoC

- **Loose coupling**: Classes don't depend on concrete implementations, but on abstractions
- **Testability**: Easy to substitute dependencies during testing
- **Centralized management**: All dependencies are configured in one place
- **Lazy initialization**: Services are created only when they are actually needed
- **Singleton pattern**: Automatic management of single instances

## Configuration

The IoC container is configured in the `src/config/ioc.ts` file.

```typescript
const iocConfig = {
    // Singleton services - created once and reused
    singletons: {
        // Register singleton services here
    },

    // Non-singleton services - new instance created on each request
    nonSingletons: {
        // Register non-singleton services here
    },
};

export default iocConfig;
```

## Service Registration

### Simple Registration

The simplest case - registering a service without dependencies:

```typescript
import UploadsService from '../app/Services/Local/Uploads/UploadsService';

const iocConfig = {
    nonSingletons: {
        UploadsService: () => new UploadsService(),
    },
};
```

### Registration with Dependencies

Often services depend on other services. The IoC container makes it easy to inject these dependencies:

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import CategoryService from '../app/Services/Local/Category/CategoryService';
import CategoryApiService from '../app/Services/Microservices/CategoryApi/CategoryApiService';

const iocConfig = {
    nonSingletons: {
        CategoryApiService: () => new CategoryApiService(new CategoryApiRepository()),
        CategoryService: () => new CategoryService(IOC.make(CategoryApiService)),
    },
};
```

### Complex Dependencies

Real-world services often have multiple dependencies. Here's a complete example showing how all services are registered:

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import ProductCardService from '../app/Services/Local/ProductCard/ProductCardService';
import CategoryApiService from '../app/Services/Microservices/CategoryApi/CategoryApiService';
import ProductApiService from '../app/Services/Microservices/ProductApi/ProductApiService';
import StockApiService from '../app/Services/Microservices/StockApi/StockApiService';
import DeliveryApiService from '../app/Services/Microservices/DeliveryApi/DeliveryApiService';

const iocConfig = {
    nonSingletons: {
        // First register the dependency services
        CategoryApiService: () => new CategoryApiService(
            new CategoryApiRepository(),
            IOC.makeSingleton(CacheService)
        ),
        ProductApiService: () => new ProductApiService(
            new ProductApiRepository(),
            IOC.makeSingleton(CacheService),
            IOC.make(LoggerService)
        ),
        StockApiService: () => new StockApiService(
            new StockApiRepository(),
            IOC.makeSingleton(CacheService)
        ),
        DeliveryApiService: () => new DeliveryApiService(
            new DeliveryApiRepository(),
            IOC.makeSingleton(CacheService)
        ),
        
        // Then register the service that uses them
        ProductCardService: () =>
            new ProductCardService(
                IOC.make(CategoryApiService),
                IOC.make(ProductApiService),
                IOC.make(StockApiService),
                IOC.make(DeliveryApiService),
            ),
    },
};
```

### Singleton Services

Some services should exist as a single instance (e.g., cache):

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import CacheService from 'sosise-core/build/Services/Cache/CacheService';
import VacancyService from '../app/Services/Local/Vacancy/VacancyService';

const iocConfig = {
    nonSingletons: {
        VacancyService: () => new VacancyService(
            new HeadHunterApiRepository(),
            IOC.makeSingleton(CacheService) // Using singleton
        ),
    },
};
```

## Retrieving Services from Container

### IOC.make() Method

Used to retrieve non-singleton services. Each call creates a new instance:

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import LoggerService from 'sosise-core/build/Services/Logger/LoggerService';

// In a controller or elsewhere
const logger = IOC.make(LoggerService) as LoggerService;
logger.info('Message');
```

### IOC.makeSingleton() Method

Used to retrieve singleton services. Returns the same instance:

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import CacheService from 'sosise-core/build/Services/Cache/CacheService';

// First call creates the instance
const cache1 = IOC.makeSingleton(CacheService) as CacheService;

// Second call returns the same instance
const cache2 = IOC.makeSingleton(CacheService) as CacheService;

// cache1 === cache2 // true
```

### Type Casting

Always use type casting for TypeScript:

```typescript
const service = IOC.make(CategoryService) as CategoryService;
```

### Using Class Constructor

IoC supports passing both strings and class constructors:

```typescript
// Using string
const service = IOC.make('CategoryService') as CategoryService;

// Using class (recommended)
const service = IOC.make(CategoryService) as CategoryService;
```

## Usage in Controllers

Typical pattern for using IoC in controllers:

```typescript
import { Request, Response, NextFunction } from 'express';
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import CategoryService from '../../Services/Local/Category/CategoryService';

export default class CategoryController {
    
    private categoryService: CategoryService;

    /**
     * Constructor
     */
    constructor() {
        this.categoryService = IOC.make(CategoryService);
    }

    /**
     * Get category tree
     */
    public async getCategoryTree(request: Request, response: Response, next: NextFunction) {
        try {
            // Use the service
            const result = await this.categoryService.getCategoryTree();

            // Return result
            return response.send({
                code: 1000,
                message: 'Category tree',
                data: result,
            });
        } catch (error) {
            next(error);
        }
    }
}
```

## Usage in Commands

IoC is also used in console commands:

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import SitemapGenerationService from '../../Services/Local/Sitemap/SitemapGenerationService';

export default class SitemapGenerationCommand extends Command {
    /**
     * Command handler
     */
    public async handle(): Promise<void> {
        // Get the service
        const service = IOC.make(SitemapGenerationService) as SitemapGenerationService;
        
        // Generate sitemap
        await service.generateSitemap();
        
        this.info('Sitemap generated successfully');
    }
}
```

## Built-in Services

Sosise provides several built-in services available without registration:

### LoggerService

```typescript
const logger = IOC.make(LoggerService) as LoggerService;
logger.info('Information message');
logger.error('Error message');
logger.warning('Warning message');
```

### CacheService

```typescript
const cache = IOC.makeSingleton(CacheService) as CacheService;
await cache.set('key', 'value', 3600);
const value = await cache.get('key');
```

## Error Handling

If a service is not registered, IoC will throw an `IOCMakeException`:

```typescript
try {
    const service = IOC.make('NonExistentService');
} catch (error) {
    // IOCMakeException: IOC could not resolve class, 
    // please check your config/ioc.ts config and register needed name there
}
```