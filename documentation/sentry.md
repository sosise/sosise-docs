# Sentry Integration

## Quick Start

Monitor your application errors in real-time with Sentry! Just add your DSN and get instant error tracking.

```bash
# Add to your .env file
SENTRY_DSN=https://your-key@o123456.ingest.sentry.io/123456
```

```typescript
// That's it! Errors are automatically tracked
export default class UserService {
    public async createUser(userData: CreateUserData): Promise<User> {
        try {
            return await this.userRepository.create(userData);
        } catch (error) {
            // This error automatically goes to Sentry!
            throw new UserCreationError('Failed to create user');
        }
    }
}
```

Get alerts, stack traces, and insights automatically!

## Introduction

Error monitoring is crucial for production applications. You need to know when things break, what caused the error, and how to fix it. Sentry provides powerful error tracking, performance monitoring, and alerting capabilities that integrate seamlessly with Sosise.

With Sentry integration, you get automatic error reporting, detailed stack traces, performance insights, and real-time alerts when issues occur. This helps you fix problems before users notice them and maintain high application reliability.

## Setup and Configuration

### Getting Started with Sentry

1. **Create Sentry Account**
   - Sign up at [sentry.io](https://sentry.io)
   - Create a new project for your application
   - Choose "Node.js" as the platform

2. **Get Your DSN**
   - Copy the DSN from your Sentry project settings
   - It looks like: `https://key@o123456.ingest.sentry.io/123456`

3. **Configure Sosise**
   ```bash
   # .env
   SENTRY_DSN=https://your-key@o123456.ingest.sentry.io/123456
   SENTRY_ENVIRONMENT=production  # or staging, development
   SENTRY_RELEASE=1.0.0          # Your app version
   ```

### Environment-Specific Configuration

```typescript
// src/config/sentry.ts
export default {
    dsn: process.env.SENTRY_DSN,
    environment: process.env.SENTRY_ENVIRONMENT || process.env.NODE_ENV,
    release: process.env.SENTRY_RELEASE || process.env.npm_package_version,
    
    // Sample rate for performance monitoring
    tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
    
    // Don't send errors from development by default
    enabled: process.env.NODE_ENV !== 'development' || process.env.SENTRY_ENABLED === 'true',
    
    // Configure which errors to ignore
    ignoreErrors: [
        'ValidationError',
        'UnauthorizedError'
    ]
};
```

## Automatic Error Tracking

### Built-in Integration

Sosise automatically captures and reports:

```typescript
// All unhandled exceptions are sent to Sentry
export default class PaymentService {
    public async processPayment(paymentData: PaymentData): Promise<Payment> {
        // If this throws an uncaught error, it goes to Sentry
        const paymentResult = await this.paymentProcessor.charge(paymentData);
        
        if (!paymentResult.success) {
            // Custom errors also go to Sentry
            throw new PaymentProcessingError('Payment failed', {
                paymentId: paymentData.id,
                errorCode: paymentResult.errorCode
            });
        }
        
        return paymentResult.payment;
    }
}
```

### Exception Configuration

Control which exceptions get sent to Sentry:

```typescript
// Custom exception with Sentry configuration
export default class DatabaseConnectionError extends Exception {
    protected sendToSentry = true;  // Send to Sentry
    protected loggingChannel = 'database';
    
    constructor(connectionString: string) {
        super(`Failed to connect to database: ${connectionString}`);
    }
}

export default class ValidationError extends Exception {
    protected sendToSentry = false; // Don't spam Sentry with validation errors
    protected loggingChannel = 'validation';
    
    constructor(field: string, message: string) {
        super(`Validation failed for ${field}: ${message}`);
    }
}
```

## Enhanced Error Context

### Adding User Context

```typescript
export default class UserService {
    constructor(
        private userRepository: UserRepository,
        private logger: LoggerService
    ) {}

    public async updateUserProfile(userId: number, profileData: any): Promise<User> {
        try {
            // Add user context for better debugging
            Sentry.setUser({
                id: userId.toString(),
                email: profileData.email
            });
            
            return await this.userRepository.update(userId, profileData);
        } catch (error) {
            // Add additional context
            Sentry.setContext('profileUpdate', {
                userId,
                fields: Object.keys(profileData),
                timestamp: new Date().toISOString()
            });
            
            throw new UserUpdateError(`Failed to update profile for user ${userId}`);
        }
    }
}
```

### Request Context in Controllers

```typescript
export default class OrderController {
    constructor(
        private orderService: OrderService,
        private logger: LoggerService
    ) {}

    public async create(request: Request, response: Response): Promise<void> {
        try {
            // Set request context
            Sentry.setContext('request', {
                url: request.url,
                method: request.method,
                userAgent: request.get('User-Agent'),
                ip: request.ip,
                userId: request.user?.id
            });
            
            const orderData = new CreateOrderUnifier(request.body);
            const order = await this.orderService.createOrder(orderData);
            
            response.json({
                code: 1000,
                message: 'Order created',
                data: order
            });
        } catch (error) {
            // Error context automatically included
            throw error;
        }
    }
}
```

### Service-Level Context

```typescript
export default class ReportService {
    public async generateReport(reportType: string, filters: any): Promise<Report> {
        const reportId = Math.random().toString(36).substring(7);
        
        // Set operation context
        Sentry.setContext('reportGeneration', {
            reportId,
            reportType,
            filterCount: Object.keys(filters).length,
            startTime: Date.now()
        });
        
        try {
            const data = await this.fetchReportData(reportType, filters);
            
            // Add success metrics
            Sentry.setContext('reportSuccess', {
                recordCount: data.length,
                processingTime: Date.now() - Date.now()
            });
            
            return await this.formatReport(data);
        } catch (error) {
            // Add failure context
            Sentry.setContext('reportFailure', {
                reportId,
                failurePoint: 'data_fetch',
                error: error.message
            });
            
            throw new ReportGenerationError(`Report generation failed: ${reportId}`);
        }
    }
}
```

## Performance Monitoring

### Transaction Tracking

```typescript
export default class OrderProcessingService {
    public async processOrder(orderId: string): Promise<void> {
        // Start performance transaction
        const transaction = Sentry.startTransaction({
            op: 'order.processing',
            name: 'Process Order'
        });
        
        try {
            // Track individual operations
            const paymentSpan = transaction.startChild({
                op: 'payment.process',
                description: 'Process payment'
            });
            
            await this.processPayment(orderId);
            paymentSpan.finish();
            
            const inventorySpan = transaction.startChild({
                op: 'inventory.update',
                description: 'Update inventory'
            });
            
            await this.updateInventory(orderId);
            inventorySpan.finish();
            
            const shippingSpan = transaction.startChild({
                op: 'shipping.create',
                description: 'Create shipping label'
            });
            
            await this.createShippingLabel(orderId);
            shippingSpan.finish();
            
        } catch (error) {
            transaction.setStatus('internal_error');
            throw error;
        } finally {
            transaction.finish();
        }
    }
}
```

### Database Query Monitoring

```typescript
export default class UserRepository {
    public async findActiveUsers(filters: UserFilters): Promise<User[]> {
        const span = Sentry.startTransaction({
            op: 'db.query',
            name: 'Find Active Users'
        });
        
        try {
            span.setData('filters', filters);
            span.setData('table', 'users');
            
            const startTime = Date.now();
            const users = await this.query('SELECT * FROM users WHERE active = true', filters);
            
            span.setData('rows_returned', users.length);
            span.setData('query_time', Date.now() - startTime);
            
            return users;
        } catch (error) {
            span.setStatus('internal_error');
            throw error;
        } finally {
            span.finish();
        }
    }
}
```

## Custom Error Tracking

### Manual Error Reporting

```typescript
export default class IntegrationService {
    public async syncWithThirdParty(): Promise<void> {
        try {
            await this.performSync();
        } catch (error) {
            // Report to Sentry with custom context
            Sentry.withScope(scope => {
                scope.setLevel('error');
                scope.setContext('integration', {
                    service: 'third-party-api',
                    endpoint: '/api/sync',
                    lastSuccessfulSync: this.getLastSyncTime()
                });
                
                Sentry.captureException(error);
            });
            
            // Don't re-throw - handle gracefully
            this.logger.warn('Third-party sync failed, will retry later');
        }
    }
}
```

### Custom Messages and Breadcrumbs

```typescript
export default class PaymentService {
    public async processPayment(paymentData: PaymentData): Promise<Payment> {
        // Add breadcrumbs for debugging
        Sentry.addBreadcrumb({
            message: 'Payment processing started',
            category: 'payment',
            level: 'info',
            data: {
                amount: paymentData.amount,
                currency: paymentData.currency
            }
        });
        
        try {
            const validation = await this.validatePayment(paymentData);
            
            Sentry.addBreadcrumb({
                message: 'Payment validation completed',
                category: 'payment',
                level: 'info',
                data: { valid: validation.isValid }
            });
            
            if (!validation.isValid) {
                // Send custom message to Sentry
                Sentry.captureMessage('Payment validation failed', 'warning');
                throw new PaymentValidationError(validation.errors);
            }
            
            return await this.chargePayment(paymentData);
            
        } catch (error) {
            Sentry.addBreadcrumb({
                message: 'Payment processing failed',
                category: 'payment',
                level: 'error',
                data: { error: error.message }
            });
            
            throw error;
        }
    }
}
```

## Alerting and Notifications

### Configure Alerts

Set up alerts in your Sentry dashboard:

1. **Error Rate Alerts**
   - Alert when error rate exceeds 5%
   - Notify via email, Slack, or webhook

2. **New Issue Alerts**
   - Get notified of new error types
   - Set up for critical services only

3. **Performance Alerts**
   - Alert on slow transactions (>2s)
   - Monitor database query performance

### Integration with Team Communication

```typescript
// Custom webhook handler for Slack notifications
export default class SentryWebhookHandler {
    public async handleAlert(alertData: any): Promise<void> {
        if (alertData.level === 'error' && alertData.project === 'production') {
            await this.sendSlackAlert({
                channel: '#alerts',
                message: `🚨 Production Error: ${alertData.title}`,
                fields: [
                    { title: 'Project', value: alertData.project },
                    { title: 'Environment', value: alertData.environment },
                    { title: 'Error Count', value: alertData.count },
                    { title: 'Link', value: alertData.url }
                ]
            });
        }
    }
}
```

## Release Tracking

### Deploy Integration

```typescript
// Deploy script integration
export default class DeploymentService {
    public async deployRelease(version: string): Promise<void> {
        // Create Sentry release
        await this.createSentryRelease(version);
        
        // Deploy application
        await this.deployApplication(version);
        
        // Mark release as deployed
        await this.markReleaseDeployed(version);
    }
    
    private async createSentryRelease(version: string): Promise<void> {
        const release = {
            version,
            projects: ['my-app'],
            refs: [{
                repository: 'my-org/my-app',
                commit: process.env.GIT_COMMIT
            }]
        };
        
        // Create release via Sentry API
        await this.sentryClient.createRelease(release);
    }
}
```

### Automatic Release Detection

```bash
# In your deployment pipeline
export SENTRY_RELEASE=$(git rev-parse --short HEAD)
export SENTRY_ENVIRONMENT=production

# Upload source maps (for frontend)
sentry-cli releases files $SENTRY_RELEASE upload-sourcemaps ./dist

# Mark release as deployed
sentry-cli releases deploys $SENTRY_RELEASE new -e production
```

## Best Practices

### 1. Filter Noise

```typescript
// ✅ Good: Don't send validation errors to Sentry
export default class ValidationError extends Exception {
    protected sendToSentry = false; // Keep Sentry focused on real issues
}

// ✅ Good: Send critical infrastructure errors
export default class DatabaseConnectionError extends Exception {
    protected sendToSentry = true; // This needs immediate attention
}
```

### 2. Add Meaningful Context

```typescript
// ✅ Good: Rich context for debugging
Sentry.setContext('orderProcessing', {
    orderId: order.id,
    customerId: order.customerId,
    totalAmount: order.total,
    paymentMethod: order.paymentMethod,
    processingStep: 'payment_validation'
});

// ❌ Bad: Minimal context
Sentry.captureException(error);
```

### 3. Use Appropriate Sample Rates

```typescript
// ✅ Good: Balanced sampling for performance monitoring
export default {
    tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0, // 10% in prod
    profilesSampleRate: process.env.NODE_ENV === 'production' ? 0.05 : 0.5 // 5% in prod
};

// ❌ Bad: Too high sampling in production (expensive)
export default {
    tracesSampleRate: 1.0 // 100% - will be very expensive
};
```

### 4. Environment Separation

```typescript
// ✅ Good: Separate projects/environments
const sentryConfig = {
    production: {
        dsn: 'https://prod-key@sentry.io/prod-project',
        environment: 'production'
    },
    staging: {
        dsn: 'https://staging-key@sentry.io/staging-project',
        environment: 'staging'
    },
    development: {
        enabled: false // Don't send dev errors
    }
};
```

## Troubleshooting

### Common Issues

1. **No Errors Showing Up**
   ```bash
   # Check DSN is correct
   echo $SENTRY_DSN
   
   # Verify network access
   curl -I https://sentry.io
   
   # Check if errors are being ignored
   console.log('sendToSentry setting:', exception.sendToSentry);
   ```

2. **Too Many Errors**
   ```typescript
   // Implement error filtering
   export default {
     beforeSend(event) {
       // Don't send validation errors
       if (event.exception?.values?.[0]?.type === 'ValidationError') {
         return null;
       }
       return event;
     }
   };
   ```

3. **Missing Context**
   ```typescript
   // Ensure context is set before errors occur
   Sentry.setUser({ id: userId });
   Sentry.setContext('operation', { name: 'user_update' });
   
   // Now errors will include this context
   await this.updateUser(userData);
   ```

## Summary

Sentry integration with Sosise provides:

- ✅ Automatic error tracking and reporting
- ✅ Performance monitoring and insights
- ✅ Real-time alerts and notifications
- ✅ Rich error context and debugging info
- ✅ Release tracking and deployment insights
- ✅ Team collaboration and issue management
- ✅ Easy setup with minimal configuration

Monitor your production applications with confidence and catch issues before they impact users!