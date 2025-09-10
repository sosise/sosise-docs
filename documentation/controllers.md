# Controllers

## Quick Start

Controllers handle your HTTP requests and return responses. Think of them as the bridge between your routes and business logic.

### Simple Example

```typescript
import { Request, Response, NextFunction } from 'express';

export default class UserController {
    public async getUsers(request: Request, response: Response, next: NextFunction) {
        try {
            const users = [
                { id: 1, name: 'Alice' },
                { id: 2, name: 'Bob' }
            ];
            
            response.json({ users });
        } catch (error) {
            next(error);
        }
    }
}
```

That's it! Your controller handles the request and sends back data.

## Introduction

Controllers organize your application's request handling logic. Instead of writing all your code directly in route files, controllers group related functionality together. For example, a `UserController` handles all user-related requests: listing users, creating new ones, updating profiles, and deleting accounts.

Controllers live in the `app/Http/Controllers` directory and work seamlessly with Express.js and TypeScript.

## Creating Controllers

### Generate a Controller

Use Artisan to quickly create a new controller:

```sh
./artisan make:controller ProductController
```

This creates a new controller file in `app/Http/Controllers/ProductController.ts`.

### Basic Controller Structure

Here's a typical controller with common methods:

```typescript
import { Request, Response, NextFunction } from 'express';
import HttpResponse from 'sosise-core/build/Types/HttpResponse';

export default class ProductController {
    /**
     * List all products
     */
    public async index(request: Request, response: Response, next: NextFunction) {
        try {
            const products = await this.getProducts();
            
            const httpResponse: HttpResponse = {
                code: 1000,
                message: 'Products retrieved successfully',
                data: products
            };
            
            return response.send(httpResponse);
        } catch (error) {
            next(error);
        }
    }

    /**
     * Get single product
     */
    public async show(request: Request, response: Response, next: NextFunction) {
        try {
            const productId = request.params.id;
            const product = await this.getProduct(productId);
            
            return response.json({ product });
        } catch (error) {
            next(error);
        }
    }

    private async getProducts() {
        // Your logic to fetch products
        return [];
    }

    private async getProduct(id: string) {
        // Your logic to fetch single product
        return { id, name: 'Sample Product' };
    }
}
```

## Connecting Controllers to Routes

After creating a controller, connect it to your routes in `src/routes/api.ts`:

```typescript
import express from 'express';
import { Request, Response, NextFunction } from 'express';
import ProductController from '../app/Http/Controllers/ProductController';

const router = express.Router();
const productController = new ProductController();

// Connect routes to controller methods
router.get('/products', (request: Request, response: Response, next: NextFunction) => {
    productController.index(request, response, next);
});

router.get('/products/:id', (request: Request, response: Response, next: NextFunction) => {
    productController.show(request, response, next);
});

export default router;
```

## Sosise Architecture Pattern

Sosise follows a clean architecture pattern with clear separation of concerns:

1. **Controllers** → Handle HTTP requests/responses only
2. **Services** → Contain all business logic
3. **Repositories** → Handle data access (database, APIs)
4. **Unifiers** → Validate and map request data

### Controller (HTTP Layer Only)

```typescript
import { Request, Response, NextFunction } from 'express';
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import UserService from '../../Services/UserService';
import CreateUserUnifier from '../../Unifiers/CreateUserUnifier';

export default class UserController {
    private userService: UserService;

    constructor() {
        this.userService = IOC.make(UserService) as UserService;
    }

    public async store(request: Request, response: Response, next: NextFunction) {
        try {
            // 1. Validate input with Unifier
            const userData = new CreateUserUnifier(request.body);
            
            // 2. Pass to Service (business logic)
            const user = await this.userService.createUser(userData);
            
            // 3. Return HTTP response
            return response.status(201).json({ user });
        } catch (error) {
            next(error);
        }
    }
}
```

### Service (Business Logic)

```typescript
// src/app/Services/UserService.ts
import UserRepository from '../Repositories/UserRepository';
import EmailService from './EmailService';

export default class UserService {
    constructor(
        private userRepository: UserRepository,
        private emailService: EmailService
    ) {}

    public async createUser(data: CreateUserData): Promise<User> {
        // Business logic here
        const existingUser = await this.userRepository.findByEmail(data.email);
        if (existingUser) {
            throw new UserAlreadyExistsException('Email already in use');
        }

        // Create user through repository
        const user = await this.userRepository.create({
            name: data.name,
            email: data.email,
            password: await this.hashPassword(data.password)
        });

        // Additional business logic
        await this.emailService.sendWelcomeEmail(user.email);

        return user;
    }

    private async hashPassword(password: string): Promise<string> {
        // Password hashing logic
        return bcrypt.hash(password, 10);
    }
}
```

### Repository (Data Access)

```typescript
// src/app/Repositories/UserRepository.ts
import { Database } from '../Database/Database';

export default class UserRepository {
    public async findByEmail(email: string): Promise<User | null> {
        return Database.table('users')
            .where('email', email)
            .first();
    }

    public async create(data: UserData): Promise<User> {
        const [id] = await Database.table('users').insert(data);
        return this.findById(id);
    }

    public async findById(id: number): Promise<User | null> {
        return Database.table('users')
            .where('id', id)
            .first();
    }
}
```

## Working with Request Data

### Route Parameters

Access URL parameters from `request.params`:

```typescript
// Route: /users/:id
public async show(request: Request, response: Response, next: NextFunction) {
    try {
        const userId = request.params.id; // Get 'id' from URL
        const user = await this.userService.getUser(userId);
        
        return response.json({ user });
    } catch (error) {
        next(error);
    }
}
```

### Query Parameters

Access query strings from `request.query`:

```typescript
// Route: /products?category=electronics&limit=10
public async index(request: Request, response: Response, next: NextFunction) {
    try {
        const category = request.query.category; // 'electronics'
        const limit = request.query.limit;       // '10'
        
        const products = await this.productService.getProducts({
            category,
            limit: parseInt(limit as string)
        });
        
        return response.json({ products });
    } catch (error) {
        next(error);
    }
}
```

### Request Body

Access POST/PUT data from `request.body`:

```typescript
public async store(request: Request, response: Response, next: NextFunction) {
    try {
        const { name, email, password } = request.body;
        
        const user = await this.userService.createUser({
            name,
            email,
            password
        });
        
        return response.status(201).json({ user });
    } catch (error) {
        next(error);
    }
}
```

## Using Unifiers for Validation

Validate request data with unifiers before processing:

```typescript
import CreateUserUnifier from '../../Unifiers/CreateUserUnifier';

export default class UserController {
    public async store(request: Request, response: Response, next: NextFunction) {
        try {
            // Validate and map request data
            const userData = new CreateUserUnifier(request.body);
            
            // Use validated data
            const user = await this.userService.createUser({
                name: userData.name,
                email: userData.email,
                age: userData.age
            });
            
            return response.status(201).json({ user });
        } catch (error) {
            next(error); // Validation errors are handled automatically
        }
    }
}
```

## Response Methods

### JSON Responses

Most common for APIs:

```typescript
// Simple JSON
response.json({ message: 'Success', data: results });

// With status code
response.status(201).json({ message: 'Created', id: newId });
```

### Formatted Responses

Using Sosise's HttpResponse type:

```typescript
const httpResponse: HttpResponse = {
    code: 1000,           // Application-specific code
    message: 'Success',   // Human-readable message
    data: results         // Actual data
};

response.send(httpResponse);
```

### Other Response Types

```typescript
// Redirect
response.redirect('/login');

// Download file
response.download('/path/to/file.pdf');

// Send status only
response.sendStatus(204); // No Content
```

## Error Handling

### Basic Error Handling

Always wrap async operations in try-catch:

```typescript
public async show(request: Request, response: Response, next: NextFunction) {
    try {
        const user = await this.userService.getUser(request.params.id);
        
        if (!user) {
            return response.status(404).json({
                message: 'User not found'
            });
        }
        
        return response.json({ user });
    } catch (error) {
        next(error); // Pass errors to error handling middleware
    }
}
```

### Custom Exceptions

Throw specific exceptions for better error handling:

```typescript
import UserNotFoundException from '../../Exceptions/UserNotFoundException';

public async show(request: Request, response: Response, next: NextFunction) {
    try {
        const user = await this.userService.getUser(request.params.id);
        
        if (!user) {
            throw new UserNotFoundException('User not found', request.params.id);
        }
        
        return response.json({ user });
    } catch (error) {
        next(error);
    }
}
```

## Best Practices

### 1. Follow the Architecture Pattern

Always maintain separation of concerns:

```typescript
// ✅ CORRECT: Controller → Service → Repository
export default class ProductController {
    private productService: ProductService;

    public async getDiscountedPrice(request: Request, response: Response, next: NextFunction) {
        try {
            // Controller: Handle HTTP only
            const unifier = new GetDiscountUnifier(request.body);
            
            // Service: Business logic
            const price = await this.productService.calculateDiscountedPrice(
                unifier.productId,
                unifier.customerId
            );
            
            // Controller: Return response
            return response.json({ price });
        } catch (error) {
            next(error);
        }
    }
}

// ❌ WRONG: Business logic in controller
export default class ProductController {
    public async getDiscountedPrice(request: Request, response: Response, next: NextFunction) {
        try {
            // DON'T DO THIS - Database access in controller!
            const product = await Database.table('products').where('id', request.body.productId).first();
            
            // DON'T DO THIS - Business logic in controller!
            const customer = await Database.table('customers').where('id', request.body.customerId).first();
            let discount = 0;
            
            if (customer.level === 'gold') {
                discount = product.price * 0.2;
            }
            
            return response.json({ price: product.price - discount });
        } catch (error) {
            next(error);
        }
    }
}
```

### 2. Use Consistent Naming

Follow RESTful conventions for method names:

```typescript
export default class UserController {
    public async index() {}    // GET /users - List all
    public async show() {}     // GET /users/:id - Show one
    public async store() {}    // POST /users - Create new
    public async update() {}   // PUT /users/:id - Update existing
    public async destroy() {}  // DELETE /users/:id - Delete
}
```

### 3. Always Handle Errors

Never let errors go unhandled:

```typescript
public async update(request: Request, response: Response, next: NextFunction) {
    try {
        const result = await this.service.update(request.params.id, request.body);
        return response.json(result);
    } catch (error) {
        // Always pass errors to the error handler
        next(error);
    }
}
```

### 4. Validate Input

Always validate user input before processing:

```typescript
public async store(request: Request, response: Response, next: NextFunction) {
    try {
        // Use unifier for validation
        const validatedData = new CreateProductUnifier(request.body);
        
        // Now safe to use
        const product = await this.productService.create(validatedData);
        
        return response.status(201).json({ product });
    } catch (error) {
        next(error);
    }
}
```

### 5. Use Type Safety

Leverage TypeScript for better type safety:

```typescript
interface CreateUserRequest {
    name: string;
    email: string;
    password: string;
}

public async store(
    request: Request<{}, {}, CreateUserRequest>,
    response: Response,
    next: NextFunction
) {
    try {
        const { name, email, password } = request.body;
        // TypeScript knows the types!
        
        const user = await this.userService.create({ name, email, password });
        return response.status(201).json({ user });
    } catch (error) {
        next(error);
    }
}
```

## Summary

Controllers are the heart of your HTTP request handling in Sosise:

- ✅ Organize request handling logic in classes
- ✅ Keep controllers thin - delegate to services
- ✅ Always handle errors with try-catch
- ✅ Validate input with unifiers
- ✅ Follow RESTful naming conventions
- ✅ Use TypeScript for type safety