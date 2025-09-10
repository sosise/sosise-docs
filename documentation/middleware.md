# Middleware

## Quick Start

Middleware acts as guards and filters for your routes. They can check authentication, log requests, or modify data before it reaches your controller.

### Simple Example

```typescript
import { Request, Response, NextFunction } from 'express';

export default class AuthMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        // Check if user is authenticated
        if (!request.headers.authorization) {
            return response.status(401).json({
                message: 'Please provide authorization token'
            });
        }

        // Continue to the next middleware or controller
        next();
    }
}
```

That's it! Your middleware protects routes from unauthorized access.

## Introduction

Middleware provides a convenient way to filter and inspect HTTP requests entering your application. Think of middleware as layers that a request must pass through before reaching your controller. Each layer can inspect the request, modify it, or even stop it entirely.

Common uses for middleware include authentication, logging, CORS handling, rate limiting, and request transformation. Middleware lives in the `app/Http/Middlewares` directory.

## Creating Middleware

### Generate Middleware

Use Artisan to create new middleware:

```sh
./artisan make:middleware CheckApiKeyMiddleware
```

This creates a new middleware file in `app/Http/Middlewares/CheckApiKeyMiddleware.ts`.

### Basic Middleware Structure

```typescript
import { Request, Response, NextFunction } from 'express';

export default class CheckApiKeyMiddleware {
    /**
     * Handle the incoming request
     */
    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        const apiKey = request.header('X-API-Key');
        
        if (!apiKey || apiKey !== process.env.API_KEY) {
            return response.status(403).json({
                message: 'Invalid API key'
            });
        }

        // Pass control to the next middleware
        next();
    }
}
```

## Registering Middleware

### Global Middleware

To run middleware on EVERY request, register it in `app/Http/Middlewares/Kernel.ts`:

```typescript
import AuthMiddleware from './AuthMiddleware';
import LoggingMiddleware from './LoggingMiddleware';
import CorsMiddleware from './CorsMiddleware';

export default class Kernel {
    /**
     * Global middleware that runs on every request
     */
    public middlewares = [
        new CorsMiddleware(),      // First: Handle CORS
        new LoggingMiddleware(),   // Second: Log requests
        new AuthMiddleware(),      // Third: Check authentication
    ];
}
```

> **Important**: The order matters! Middleware runs in the order you list them.

### Route-Specific Middleware

Apply middleware to specific routes only:

```typescript
// src/routes/api.ts
import express from 'express';
import AdminMiddleware from '../app/Http/Middlewares/AdminMiddleware';
import UserController from '../app/Http/Controllers/UserController';

const router = express.Router();
const adminMiddleware = new AdminMiddleware();
const userController = new UserController();

// Protected route - middleware runs first
router.delete('/users/:id', 
    adminMiddleware.handle,  // Check if admin
    (request, response, next) => {
        userController.destroy(request, response, next);
    }
);

// Public route - no middleware
router.get('/users', (request, response, next) => {
    userController.index(request, response, next);
});
```

## Common Middleware Patterns

### Authentication Middleware

Check if a user is logged in:

```typescript
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

export default class AuthMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        try {
            const token = request.header('Authorization')?.replace('Bearer ', '');
            
            if (!token) {
                return response.status(401).json({
                    message: 'Authentication required'
                });
            }

            // Verify token
            const decoded = jwt.verify(token, process.env.JWT_SECRET);
            
            // Add user to request for use in controllers
            request['user'] = decoded;
            
            next();
        } catch (error) {
            return response.status(401).json({
                message: 'Invalid token'
            });
        }
    }
}
```

### Logging Middleware

Log all incoming requests:

```typescript
import { Request, Response, NextFunction } from 'express';
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import LoggerService from 'sosise-core/build/Services/Logger/LoggerService';

export default class LoggingMiddleware {
    private logger: LoggerService;

    constructor() {
        this.logger = IOC.make(LoggerService) as LoggerService;
    }

    public async handle(request: Request, response: Response, next: NextFunction): Promise<void> {
        const startTime = Date.now();
        
        // Log request
        this.logger.info(`${request.method} ${request.path}`, {
            ip: request.ip,
            userAgent: request.get('user-agent')
        });

        // Continue processing
        next();

        // Log response time after request completes
        const duration = Date.now() - startTime;
        this.logger.info(`Request completed in ${duration}ms`);
    }
}
```

### Rate Limiting Middleware

Prevent API abuse:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class RateLimitMiddleware {
    private requests: Map<string, number[]> = new Map();
    private maxRequests = 100;
    private windowMs = 60000; // 1 minute

    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        const ip = request.ip;
        const now = Date.now();
        
        // Get previous requests from this IP
        const timestamps = this.requests.get(ip) || [];
        
        // Filter out old requests outside the window
        const recentRequests = timestamps.filter(time => now - time < this.windowMs);
        
        if (recentRequests.length >= this.maxRequests) {
            return response.status(429).json({
                message: 'Too many requests, please try again later'
            });
        }
        
        // Add current request
        recentRequests.push(now);
        this.requests.set(ip, recentRequests);
        
        next();
    }
}
```

### CORS Middleware

Handle Cross-Origin requests:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class CorsMiddleware {
    public handle(request: Request, response: Response, next: NextFunction): void {
        // Set CORS headers
        response.header('Access-Control-Allow-Origin', '*');
        response.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
        response.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
        
        // Handle preflight requests
        if (request.method === 'OPTIONS') {
            return response.sendStatus(200);
        }
        
        next();
    }
}
```

## Middleware Execution Order

### Before and After Middleware

Middleware can run code before OR after the main request:

```typescript
export default class TimingMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction): Promise<void> {
        // BEFORE: Runs before the controller
        console.log('Request started');
        const startTime = Date.now();
        
        // Pass control to next middleware/controller
        next();
        
        // AFTER: Runs after the controller
        const duration = Date.now() - startTime;
        console.log(`Request took ${duration}ms`);
    }
}
```

### Modifying the Request

Add data to the request for use in controllers:

```typescript
export default class UserMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction): Promise<void> {
        // Add user information to request
        const userId = request.header('X-User-ID');
        
        if (userId) {
            // Fetch user and add to request
            const user = await this.userService.getUser(userId);
            request['currentUser'] = user;
        }
        
        next();
    }
}

// In controller, you can access:
// request['currentUser']
```

## Accessing Response Body

Sometimes you need to access or modify the response:

```typescript
export default class ResponseFormatterMiddleware {
    public handle(request: Request, response: Response, next: NextFunction): void {
        // Store original send method
        const originalSend = response.send;
        
        // Override send method
        response.send = function(data: any) {
            // Modify response data
            const formattedResponse = {
                success: true,
                timestamp: new Date().toISOString(),
                data: data
            };
            
            // Call original send with modified data
            return originalSend.call(response, formattedResponse);
        };
        
        next();
    }
}
```

## Error Handling in Middleware

Always handle errors properly:

```typescript
export default class DatabaseCheckMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        try {
            // Check database connection
            await this.databaseService.ping();
            next();
        } catch (error) {
            // Don't use next(error) if you're handling the error
            return response.status(503).json({
                message: 'Database unavailable, please try again later'
            });
        }
    }
}
```

## Best Practices

### 1. Keep Middleware Focused

Each middleware should do ONE thing:

```typescript
// ✅ Good: Single responsibility
export default class AuthMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction) {
        // Only handles authentication
        if (!request.headers.authorization) {
            return response.status(401).json({ message: 'Unauthorized' });
        }
        next();
    }
}

// ❌ Bad: Doing too much
export default class SuperMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction) {
        // Don't mix concerns!
        this.checkAuth(request);
        this.logRequest(request);
        this.validateData(request);
        this.checkRateLimit(request);
        next();
    }
}
```

### 2. Order Matters

Register middleware in the correct order:

```typescript
// Kernel.ts
public middlewares = [
    new CorsMiddleware(),        // 1. CORS must be first
    new LoggingMiddleware(),     // 2. Log all requests
    new RateLimitMiddleware(),   // 3. Check rate limits
    new AuthMiddleware(),        // 4. Authenticate user
    new ValidationMiddleware(),  // 5. Validate data
];
```

### 3. Don't Forget to Call next()

Always call `next()` or send a response:

```typescript
public async handle(request: Request, response: Response, next: NextFunction) {
    if (someCondition) {
        return response.status(400).json({ error: 'Bad request' });
    }
    
    // Don't forget this!
    next();
}
```

### 4. Use TypeScript Types

Add types for better safety:

```typescript
interface AuthenticatedRequest extends Request {
    user?: {
        id: number;
        email: string;
        role: string;
    };
}

export default class AdminMiddleware {
    public async handle(request: AuthenticatedRequest, response: Response, next: NextFunction) {
        if (request.user?.role !== 'admin') {
            return response.status(403).json({ message: 'Admin access required' });
        }
        next();
    }
}
```

### 5. Use IoC for Dependencies

Inject services through IoC:

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import AuthService from '../Services/AuthService';

export default class TokenMiddleware {
    private authService: AuthService;

    constructor() {
        this.authService = IOC.make(AuthService) as AuthService;
    }

    public async handle(request: Request, response: Response, next: NextFunction) {
        const token = request.header('Authorization');
        const isValid = await this.authService.validateToken(token);
        
        if (!isValid) {
            return response.status(401).json({ message: 'Invalid token' });
        }
        
        next();
    }
}
```

## Summary

Middleware is a powerful feature in Sosise that helps you:

- ✅ Protect routes with authentication
- ✅ Validate requests before they reach controllers
- ✅ Log and monitor application activity
- ✅ Handle cross-cutting concerns cleanly
- ✅ Transform requests and responses
- ✅ Implement rate limiting and security features