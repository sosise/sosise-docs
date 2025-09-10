# Logging

## Quick Start

Logging in Sosise is simple - just inject LoggerService and start logging!

```typescript
// In any service or controller
export default class UserService {
    constructor(
        private logger: LoggerService,
        private userRepository: UserRepository
    ) {}

    public async createUser(userData: CreateUserData): Promise<User> {
        this.logger.info('Creating new user', { email: userData.email });
        
        try {
            const user = await this.userRepository.create(userData);
            this.logger.info('User created successfully', { userId: user.id });
            return user;
        } catch (error) {
            this.logger.error('Failed to create user', { error: error.message, userData });
            throw error;
        }
    }
}
```

Logs automatically go to daily files and console. Perfect for debugging and monitoring!

## Introduction

Logging is essential for understanding your application's behavior, debugging issues, and monitoring performance in production. Sosise provides a comprehensive logging system that supports multiple channels, different log levels, and environment-specific formatting.

The logging system automatically handles daily rotation, structured logging, and integrates seamlessly with your services through dependency injection.

## Basic Usage

### Inject LoggerService

The recommended way is to inject LoggerService through IoC:

```typescript
// src/app/Services/OrderService.ts
import LoggerService from 'sosise-core/build/Services/Logger/LoggerService';

export default class OrderService {
    constructor(
        private logger: LoggerService,
        private orderRepository: OrderRepository,
        private paymentService: PaymentService
    ) {}

    public async processOrder(order: Order): Promise<void> {
        this.logger.info('Starting order processing', { orderId: order.id });
        
        // Your business logic here
    }
}
```

Register in IoC:

```typescript
// src/config/ioc.ts
const iocConfig = {
    nonSingletons: {
        OrderService: () => new OrderService(
            IOC.make(LoggerService),
            new OrderRepository(),
            new PaymentService()
        )
    }
};
```

### Log Levels

Use appropriate log levels for different situations:

```typescript
export default class PaymentService {
    constructor(private logger: LoggerService) {}

    public async processPayment(amount: number): Promise<Payment> {
        // Debug: Detailed information for debugging
        this.logger.debug('Payment processing started', { amount });
        
        // Info: General operational messages
        this.logger.info('Processing payment', { amount });
        
        // Warning: Something unexpected but not critical
        this.logger.warn('Payment processor response slow', { responseTime: 2000 });
        
        // Error: Error occurred but application continues
        this.logger.error('Payment validation failed', { amount, reason: 'invalid card' });
        
        // Critical: Critical error that may cause application issues
        this.logger.critical('Payment processor unavailable', { processor: 'stripe' });
        
        return payment;
    }
}
```

## Structured Logging

### Log with Context Data

Always include relevant context in your logs:

```typescript
export default class AuthService {
    constructor(private logger: LoggerService) {}

    public async login(email: string, password: string): Promise<User> {
        const startTime = Date.now();
        
        this.logger.info('Login attempt started', { 
            email, 
            ip: request.ip,
            userAgent: request.get('User-Agent')
        });
        
        try {
            const user = await this.authenticateUser(email, password);
            
            this.logger.info('Login successful', {
                userId: user.id,
                email: user.email,
                duration: Date.now() - startTime
            });
            
            return user;
        } catch (error) {
            this.logger.error('Login failed', {
                email,
                error: error.message,
                duration: Date.now() - startTime,
                ip: request.ip
            });
            throw error;
        }
    }
}
```

### Request Tracking

Log request flows for better debugging:

```typescript
export default class OrderController {
    constructor(
        private logger: LoggerService,
        private orderService: OrderService
    ) {}

    public async create(request: Request, response: Response): Promise<void> {
        const requestId = request.id; // Assuming request ID middleware
        
        this.logger.info('Order creation request received', {
            requestId,
            userId: request.user?.id,
            orderData: request.body
        });
        
        try {
            const orderData = new CreateOrderUnifier(request.body);
            const order = await this.orderService.createOrder(orderData);
            
            this.logger.info('Order created successfully', {
                requestId,
                orderId: order.id,
                total: order.total
            });
            
            response.json({
                code: 1000,
                message: 'Order created',
                data: order
            });
        } catch (error) {
            this.logger.error('Order creation failed', {
                requestId,
                error: error.message,
                orderData: request.body
            });
            throw error;
        }
    }
}
```

## Logging Channels

### Configure Custom Channels

Create specialized channels in `src/config/logging.ts`:

```typescript
export default {
    channels: {
        default: {
            driver: 'file',
            level: 'info'
        },
        
        // Payment-specific logs
        payments: {
            driver: 'file',
            level: 'info',
            filename: 'payments'
        },
        
        // Security events
        security: {
            driver: 'file',
            level: 'warn',
            filename: 'security'
        },
        
        // API access logs
        api: {
            driver: 'file',
            level: 'info',
            filename: 'api-access'
        }
    }
};
```

### Use Different Channels

```typescript
export default class PaymentService {
    constructor(private logger: LoggerService) {}

    public async processPayment(payment: PaymentData): Promise<void> {
        // Log to payments channel
        this.logger.info('Payment processing started', { 
            amount: payment.amount,
            currency: payment.currency
        }, 'payments');
        
        if (payment.amount > 10000) {
            // Log large transactions to security channel
            this.logger.warn('Large payment detected', {
                amount: payment.amount,
                userId: payment.userId
            }, 'security');
        }
    }
}

export default class AuthMiddleware {
    constructor(private logger: LoggerService) {}

    public async authenticate(request: Request): Promise<void> {
        // Log authentication attempts to security channel
        this.logger.info('Authentication attempt', {
            ip: request.ip,
            userAgent: request.get('User-Agent'),
            endpoint: request.path
        }, 'security');
    }
}
```

## Environment-Specific Logging

### Development vs Production

Sosise automatically formats logs based on `APP_ENV`:

```typescript
// APP_ENV=local (pretty printed)
2023-12-25 14:30:45 [INFO] User login successful
  userId: 123
  email: "user@example.com"
  duration: 245ms

// APP_ENV=production (JSON format)
{"timestamp":"2023-12-25T14:30:45.123Z","level":"info","message":"User login successful","userId":123,"email":"user@example.com","duration":245}
```

### Environment-Specific Log Levels

```typescript
export default class DatabaseService {
    constructor(private logger: LoggerService) {}

    public async query(sql: string): Promise<any> {
        // Only log SQL queries in development
        if (process.env.APP_ENV === 'local') {
            this.logger.debug('Executing SQL query', { sql });
        }
        
        const startTime = Date.now();
        const result = await this.executeQuery(sql);
        const duration = Date.now() - startTime;
        
        // Log slow queries in all environments
        if (duration > 1000) {
            this.logger.warn('Slow query detected', { sql, duration });
        }
        
        return result;
    }
}
```

## Advanced Logging Patterns

### Exception Logging

Exceptions automatically log to specified channels:

```typescript
export default class PaymentFailureException extends Exception {
    protected loggingChannel = 'payments'; // Logs to payments channel
    
    constructor(paymentId: string, reason: string) {
        super(`Payment ${paymentId} failed: ${reason}`);
    }
}

export default class SecurityViolationException extends Exception {
    protected loggingChannel = 'security'; // Logs to security channel
    
    constructor(userId: number, violation: string) {
        super(`Security violation by user ${userId}: ${violation}`);
    }
}
```

### Conditional Logging

```typescript
export default class CacheService {
    constructor(private logger: LoggerService) {}

    public async get(key: string): Promise<any> {
        const result = await this.retrieveFromCache(key);
        
        // Only log cache misses
        if (!result) {
            this.logger.debug('Cache miss', { key });
        }
        
        return result;
    }
    
    public async invalidatePattern(pattern: string): Promise<void> {
        const keys = await this.findKeysByPattern(pattern);
        
        // Log significant cache operations
        if (keys.length > 100) {
            this.logger.info('Mass cache invalidation', { 
                pattern, 
                keysCount: keys.length 
            });
        }
        
        await this.deleteKeys(keys);
    }
}
```

### Performance Logging

```typescript
export default class ReportService {
    constructor(private logger: LoggerService) {}

    public async generateReport(filters: ReportFilters): Promise<Report> {
        const startTime = Date.now();
        const reportId = Math.random().toString(36).substring(7);
        
        this.logger.info('Report generation started', { 
            reportId,
            filters 
        });
        
        try {
            const data = await this.fetchData(filters);
            const processedData = await this.processData(data);
            const report = await this.formatReport(processedData);
            
            const duration = Date.now() - startTime;
            
            this.logger.info('Report generation completed', {
                reportId,
                duration,
                recordCount: data.length,
                reportSize: JSON.stringify(report).length
            });
            
            // Log slow report generation
            if (duration > 30000) { // 30 seconds
                this.logger.warn('Slow report generation', {
                    reportId,
                    duration,
                    filters
                });
            }
            
            return report;
        } catch (error) {
            this.logger.error('Report generation failed', {
                reportId,
                duration: Date.now() - startTime,
                error: error.message,
                filters
            });
            throw error;
        }
    }
}
```

## Log File Management

### Daily Rotation

Logs automatically rotate daily:

```
storage/logs/
├── sosise-2023-12-23.log
├── sosise-2023-12-24.log
├── sosise-2023-12-25.log  ← Today's logs
├── payments-2023-12-25.log
└── security-2023-12-25.log
```

### Log Cleanup Service

```typescript
export default class LogCleanupService {
    constructor(private logger: LoggerService) {}

    public async cleanupOldLogs(): Promise<void> {
        const cutoffDate = new Date();
        cutoffDate.setDate(cutoffDate.getDate() - 30); // Keep 30 days
        
        this.logger.info('Starting log cleanup', { cutoffDate });
        
        const deletedFiles = await this.deleteOldLogFiles(cutoffDate);
        
        this.logger.info('Log cleanup completed', { 
            deletedFiles: deletedFiles.length 
        });
    }
}
```

## Best Practices

### 1. Use Structured Logging

```typescript
// ✅ Good: Structured data for easy parsing
this.logger.info('User action completed', {
    userId: 123,
    action: 'purchase',
    amount: 99.99,
    duration: 245
});

// ❌ Bad: Unstructured string interpolation
this.logger.info(`User 123 completed purchase of $99.99 in 245ms`);
```

### 2. Include Relevant Context

```typescript
// ✅ Good: Rich context for debugging
this.logger.error('Payment processing failed', {
    orderId: order.id,
    userId: order.userId,
    amount: order.total,
    paymentMethod: order.paymentMethod,
    error: error.message,
    requestId: request.id
});

// ❌ Bad: Minimal context
this.logger.error('Payment failed', { error: error.message });
```

### 3. Use Appropriate Log Levels

```typescript
// ✅ Good: Appropriate levels
this.logger.debug('Cache hit', { key }); // Development debugging
this.logger.info('User logged in', { userId }); // Normal operations
this.logger.warn('API rate limit approaching', { current: 950, limit: 1000 }); // Potential issues
this.logger.error('Database connection failed', { error }); // Errors
this.logger.critical('Payment processor down', { processor }); // System-critical issues

// ❌ Bad: Wrong levels
this.logger.critical('User logged in'); // Not critical
this.logger.debug('Payment processor unavailable'); // Should be critical
```

### 4. Avoid Logging Sensitive Data

```typescript
// ✅ Good: Log safe data only
this.logger.info('Payment processed', {
    userId: user.id,
    amount: payment.amount,
    cardLast4: payment.card.slice(-4) // Only last 4 digits
});

// ❌ Bad: Logging sensitive information
this.logger.info('Payment processed', {
    creditCard: payment.fullCardNumber, // Never log full card numbers
    password: user.password, // Never log passwords
    ssn: user.socialSecurityNumber // Never log PII
});
```

### 5. Use Channels for Organization

```typescript
// ✅ Good: Organized by domain
this.logger.info('Payment processed', paymentData, 'payments');
this.logger.warn('Suspicious login attempt', securityData, 'security');
this.logger.info('API request', requestData, 'api');

// ✅ Good: Default channel for general logs
this.logger.info('Application started');
```

## Summary

Sosise's logging system provides:

- ✅ Automatic daily log rotation
- ✅ Environment-specific formatting
- ✅ Multiple logging channels
- ✅ Structured logging with context
- ✅ Integration with exception handling
- ✅ Easy service injection
- ✅ Performance and security monitoring

Use logging extensively to monitor your application's health and debug issues efficiently!