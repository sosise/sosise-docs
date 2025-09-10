# Error Handling

## Quick Start

Error handling in Sosise is straightforward - create custom exceptions and let the framework handle the rest!

```typescript
import Exception from 'sosise-core/build/Exceptions/Exception';

// Create custom exception
class UserNotFoundError extends Exception {
    protected httpCode = 404;
    protected code = 3001;
    
    constructor(userId: number) {
        super(`User with ID ${userId} not found`);
    }
}

// Use in service
export default class UserService {
    public async getUser(id: number): Promise<User> {
        const user = await this.userRepository.findById(id);
        
        if (!user) {
            throw new UserNotFoundError(id); // Automatically handled!
        }
        
        return user;
    }
}
```

That's it! Your API returns proper error responses automatically.

## Introduction

Error handling is crucial for building robust applications. Sosise provides a comprehensive error handling system that catches exceptions, logs them appropriately, and returns consistent API responses to your clients.

The framework handles all exceptions through `src/app/Exceptions/Handler.ts` and provides tools for creating custom exceptions with meaningful error codes, proper HTTP status codes, and structured responses.

## Creating Custom Exceptions

### Using Artisan Command

Generate a new exception class:

```bash
./artisan make:exception ProductNotFoundError
```

This creates a new exception class in `src/app/Exceptions/`:

```typescript
import Exception from 'sosise-core/build/Exceptions/Exception';
import ExceptionResponse from 'sosise-core/build/Types/ExceptionResponse';

export default class ProductNotFoundError extends Exception {
    protected httpCode = 404;
    protected code = 3001;
    protected sendToSentry = true;
    protected loggingChannel = 'default';

    constructor(productId: number) {
        super(`Product with ID ${productId} not found`);
    }

    public handle(exception: this): ExceptionResponse {
        return {
            code: exception.code,
            httpCode: exception.httpCode,
            message: exception.message,
            data: null
        };
    }
}
```

### Exception Properties

Configure your exception behavior:

```typescript
export default class ValidationError extends Exception {
    // HTTP status code returned to client
    protected httpCode = 400;
    
    // Unique error code for client identification
    protected code = 3002;
    
    // Whether to report to Sentry monitoring
    protected sendToSentry = false; // Don't spam Sentry with validation errors
    
    // Which logging channel to use (see src/config/logging.ts)
    protected loggingChannel = 'validation';
    
    constructor(field: string, value: any) {
        super(`Invalid value '${value}' for field '${field}'`);
    }
}
```

## Using Exceptions in Services

### Basic Exception Throwing

```typescript
// src/app/Services/OrderService.ts
export default class OrderService {
    constructor(
        private orderRepository: OrderRepository,
        private userService: UserService
    ) {}

    public async getOrder(orderId: number, userId: number): Promise<Order> {
        // Verify user exists
        await this.userService.getUser(userId); // Throws UserNotFoundError if needed
        
        const order = await this.orderRepository.findById(orderId);
        
        if (!order) {
            throw new OrderNotFoundError(orderId);
        }
        
        if (order.userId !== userId) {
            throw new UnauthorizedAccessError('You can only view your own orders');
        }
        
        return order;
    }
}
```

### Exception with Additional Data

```typescript
class InsufficientStockError extends Exception {
    protected httpCode = 400;
    protected code = 3003;
    
    constructor(
        productId: number, 
        private requested: number, 
        private available: number
    ) {
        super(`Insufficient stock for product ${productId}`);
    }

    public handle(exception: this): ExceptionResponse {
        return {
            code: exception.code,
            httpCode: exception.httpCode,
            message: exception.message,
            data: {
                requested: this.requested,
                available: this.available,
                shortage: this.requested - this.available
            }
        };
    }
}

// Usage in service
public async createOrder(items: OrderItem[]): Promise<Order> {
    for (const item of items) {
        const stock = await this.getProductStock(item.productId);
        
        if (stock < item.quantity) {
            throw new InsufficientStockError(
                item.productId, 
                item.quantity, 
                stock
            );
        }
    }
    
    // Process order...
}
```

## Error Response Format

### Successful Response

```json
{
    "code": 1000,
    "message": "Order created successfully",
    "data": {
        "orderId": 12345,
        "total": 299.99
    }
}
```

### Error Response

```json
{
    "code": 3001,
    "httpCode": 404,
    "message": "Product with ID 999 not found",
    "data": null
}
```

### Error with Additional Data

```json
{
    "code": 3003,
    "httpCode": 400,
    "message": "Insufficient stock for product 123",
    "data": {
        "requested": 10,
        "available": 5,
        "shortage": 5
    }
}
```

## Error Code Conventions

Organize your error codes systematically:

### Code Ranges

| Range | Purpose | Example |
|-------|---------|---------|
| 1000-1999 | Success responses | 1000: Success, 1001: Created |
| 2000-2999 | Framework errors | 2001: Validation failed |
| 3000-3999 | Business logic errors | 3001: Not found, 3002: Unauthorized |
| 4000-4999 | Integration errors | 4001: Payment failed, 4002: Email service down |

### Example Error Code Registry

```typescript
// src/app/Enums/ErrorCodes.ts
export enum ErrorCodes {
    // Success
    SUCCESS = 1000,
    CREATED = 1001,
    
    // Not Found Errors (3000s)
    USER_NOT_FOUND = 3001,
    PRODUCT_NOT_FOUND = 3002,
    ORDER_NOT_FOUND = 3003,
    
    // Permission Errors (3100s)
    UNAUTHORIZED_ACCESS = 3101,
    INSUFFICIENT_PERMISSIONS = 3102,
    
    // Business Logic Errors (3200s)
    INSUFFICIENT_STOCK = 3201,
    ORDER_ALREADY_SHIPPED = 3202,
    INVALID_DISCOUNT_CODE = 3203,
    
    // Integration Errors (4000s)
    PAYMENT_PROCESSOR_ERROR = 4001,
    EMAIL_SERVICE_ERROR = 4002,
    SHIPPING_API_ERROR = 4003
}
```

## Advanced Exception Patterns

### Exception Chaining

```typescript
export default class OrderService {
    public async processPayment(order: Order): Promise<void> {
        try {
            await this.paymentService.charge(order);
        } catch (error) {
            // Wrap external service error
            throw new PaymentProcessingError(
                `Failed to process payment for order ${order.id}`,
                error // Chain the original error
            );
        }
    }
}
```

### Conditional Sentry Reporting

```typescript
class DatabaseConnectionError extends Exception {
    protected httpCode = 500;
    protected code = 2001;
    protected loggingChannel = 'database';
    
    // Only send to Sentry in production
    protected get sendToSentry(): boolean {
        return process.env.NODE_ENV === 'production';
    }
}
```

### Context-Aware Exceptions

```typescript
class RateLimitExceededError extends Exception {
    protected httpCode = 429;
    protected code = 3301;
    
    constructor(
        private userId: number,
        private action: string,
        private limit: number,
        private resetTime: Date
    ) {
        super(`Rate limit exceeded for ${action}`);
    }

    public handle(exception: this): ExceptionResponse {
        return {
            code: exception.code,
            httpCode: exception.httpCode,
            message: exception.message,
            data: {
                userId: this.userId,
                action: this.action,
                limit: this.limit,
                resetAt: this.resetTime.toISOString()
            }
        };
    }
}
```

## Exception Handler Customization

### Custom Logging

```typescript
// src/app/Exceptions/Handler.ts
export default class Handler {
    public async handle(exception: Exception): Promise<ExceptionResponse> {
        // Custom logging for critical errors
        if (exception.code >= 4000) {
            await this.alertOpsTeam(exception);
        }
        
        // Custom metrics
        this.metricsService.incrementErrorCount(exception.code);
        
        return exception.handle(exception);
    }
    
    private async alertOpsTeam(exception: Exception): Promise<void> {
        // Send to Slack, PagerDuty, etc.
    }
}
```

## Best Practices

### 1. Use Meaningful Error Codes

```typescript
// ✅ Good: Specific, meaningful codes
class PRODUCT_OUT_OF_STOCK extends Exception {
    protected code = 3201;
}

class INVALID_COUPON_CODE extends Exception {
    protected code = 3202;
}

// ❌ Bad: Generic codes
class BusinessError extends Exception {
    protected code = 3000; // Too generic
}
```

### 2. Include Helpful Context

```typescript
// ✅ Good: Helpful error information
class OrderValidationError extends Exception {
    constructor(orderId: number, violations: string[]) {
        super(`Order ${orderId} validation failed`);
        this.data = { violations };
    }
}

// ❌ Bad: No context
throw new Error('Validation failed'); // Unhelpful
```

### 3. Handle Exceptions at Service Level

```typescript
// ✅ Good: Services throw specific exceptions
export default class UserService {
    public async createUser(data: CreateUserData): Promise<User> {
        if (await this.userRepository.emailExists(data.email)) {
            throw new EmailAlreadyExistsError(data.email);
        }
        
        return await this.userRepository.create(data);
    }
}

// Controller just lets exceptions bubble up
export default class UserController {
    public async create(request: Request, response: Response): Promise<void> {
        const userData = new CreateUserUnifier(request.body);
        const user = await this.userService.createUser(userData); // Exception handled automatically
        
        response.json({ code: 1001, message: 'User created', data: user });
    }
}
```

### 4. Don't Catch Unless You Can Handle

```typescript
// ✅ Good: Only catch when you can provide value
public async processOrder(order: Order): Promise<void> {
    try {
        await this.paymentService.charge(order);
        await this.inventoryService.reserveItems(order);
        await this.shippingService.createLabel(order);
    } catch (error) {
        // Rollback any partial changes
        await this.rollbackOrder(order);
        throw error; // Re-throw so framework can handle properly
    }
}

// ❌ Bad: Catching and losing information
public async getUser(id: number): Promise<User | null> {
    try {
        return await this.userRepository.findById(id);
    } catch (error) {
        return null; // Lost important error information!
    }
}
```

## Summary

Sosise's error handling system provides:

- ✅ Automatic exception catching and response formatting
- ✅ Structured error codes for client applications
- ✅ Proper HTTP status codes
- ✅ Configurable logging and monitoring
- ✅ Type-safe exception handling
- ✅ Easy testing and debugging

Focus on creating meaningful exceptions in your services, and let Sosise handle the rest!