# EventBus

## Quick Start

EventBus enables communication between different parts of your application through events. Components can emit events and listen for them without direct dependencies!

### Simple Example

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import EventBusService from 'sosise-core/build/Services/EventBus/EventBusService';

// Get EventBus service
const eventBus = IOC.makeSingleton(EventBusService) as EventBusService;

// Listen for events
eventBus.on('user.registered', async (payload) => {
    console.log('New user registered:', payload.data);
    // Send welcome email, update statistics, etc.
});

// Emit events from anywhere
await eventBus.emit('user.registered', { 
    id: 123, 
    email: 'user@example.com' 
});
```

That's it! Your components are now loosely coupled through events.

## Introduction

EventBus implements the publish-subscribe pattern, allowing different parts of your application to communicate through events without tight coupling. Think of it as a central message hub where components can broadcast events and others can listen for them.

Sosise EventBus supports multiple drivers (Memory, Redis) with features like wildcard patterns, guaranteed delivery, multi-service support, and TTL for events. The configuration is in `src/config/eventbus.ts`.

## Basic Usage

### Setting Up EventBus in Your Service

The recommended way is to inject EventBusService through IoC:

```typescript
// src/app/Services/OrderService.ts
import EventBusService from 'sosise-core/build/Services/EventBus/EventBusService';

export default class OrderService {
    constructor(
        private eventBus: EventBusService,
        private orderRepository: OrderRepository
    ) {}

    public async createOrder(data: OrderData): Promise<Order> {
        // Create order
        const order = await this.orderRepository.create(data);
        
        // Emit event for other services to react
        await this.eventBus.emit('order.created', {
            orderId: order.id,
            userId: order.userId,
            total: order.total,
            items: order.items
        });
        
        return order;
    }
}
```

Register in IoC:

```typescript
// src/config/ioc.ts
const iocConfig = {
    nonSingletons: {
        OrderService: () => new OrderService(
            IOC.makeSingleton(EventBusService),
            new OrderRepository()
        ),
    }
};
```

## Common EventBus Operations

### Emit Events

```typescript
// Basic event emission
await eventBus.emit('user.login', { userId: 123 });

// Event with TTL (expires after 30 minutes)
await eventBus.emit('session.started', { sessionId: 'abc' }, 30);

// Event without data
await eventBus.emit('cache.cleared');
```

### Listen for Events

```typescript
// Listen for specific event
eventBus.on('order.created', async (payload) => {
    console.log('Order created at:', new Date(payload.event.timestamp));
    console.log('Order data:', payload.data);
});

// Listen with wildcard patterns
eventBus.on('user.*', async (payload) => {
    // Handles user.created, user.updated, user.deleted, etc.
    console.log(`User event: ${payload.event.name}`);
    console.log('Data:', payload.data);
});

// Multi-level wildcards
eventBus.on('payment.**', async (payload) => {
    // Handles payment.processed, payment.gateway.response, etc.
    console.log(`Payment event: ${payload.event.name}`);
});
```

### Durable Subscriptions (Redis only)

Durable subscriptions guarantee message delivery even if the subscriber was offline:

```typescript
// Subscribe with guaranteed delivery
eventBus.onDurable('order.created', async (payload) => {
    // Will receive ALL messages, including those sent while offline
    // When they are not expired
    console.log('Processing order:', payload.data);
    
    // Process the order
    await processOrder(payload.data);
});
```

### Event Payload Structure

All events receive a standardized payload:

```typescript
interface EventPayload {
    event: {
        name: string;        // The event name
        timestamp: number;   // When event was emitted
        expiresAt?: number; // Optional expiration timestamp
    };
    data?: any;             // Your custom data
}

// Example handler
eventBus.on('user.updated', async (payload: EventPayload) => {
    console.log(`Event: ${payload.event.name}`);
    console.log(`Timestamp: ${new Date(payload.event.timestamp)}`);
    console.log(`User data:`, payload.data);
});
```

### Unsubscribe from Events

```typescript
// Create a handler
const handler = async (payload) => {
    console.log('Event received:', payload);
};

// Subscribe
eventBus.on('user.created', handler);

// Unsubscribe specific handler
eventBus.off('user.created', handler);

// Remove all listeners for an event
eventBus.removeAllListeners('user.created');

// Remove all listeners for all events
eventBus.removeAllListeners();
```

## Configuration

Configure your EventBus driver in `src/config/eventbus.ts`:

```typescript
export default {
    // Default EventBus driver
    driver: process.env.EVENTBUS_DRIVER || 'memory',
    
    // Driver configurations
    driverConfiguration: {
        redis: {
            host: process.env.EVENTBUS_REDIS_HOST || '127.0.0.1',
            port: process.env.EVENTBUS_REDIS_PORT || 6379,
            password: process.env.EVENTBUS_REDIS_PASSWORD,
            db: process.env.EVENTBUS_REDIS_DB || 0,
            
            // Service name for multi-service deployments
            serviceName: process.env.EVENTBUS_SERVICE_NAME || 'default-service'
        }
    }
};
```

## EventBus Drivers

### Memory (In-Process)

- **Pros**: Fastest, no setup required
- **Cons**: Events lost on restart, single process only
- **Use for**: Development, single-instance applications

```typescript
driver: 'memory'
```

### Redis (Distributed)

- **Pros**: Distributed events, guaranteed delivery, multi-service support
- **Cons**: Requires Redis server
- **Use for**: Production, microservices, multi-instance deployments

```typescript
driver: 'redis'
```

## Event Patterns

### Basic Events

```typescript
// Emit simple events
await eventBus.emit('app.started');
await eventBus.emit('user.login', { userId: 123 });

// Listen for exact match
eventBus.on('app.started', async () => {
    console.log('Application started');
});
```

### Wildcard Patterns

Use wildcards to listen for multiple related events:

```typescript
// Single-level wildcard (*)
eventBus.on('user.*', async (payload) => {
    // Matches: user.created, user.updated, user.deleted
    // Does NOT match: user.profile.updated
    switch (payload.event.name) {
        case 'user.created':
            await sendWelcomeEmail(payload.data);
            break;
        case 'user.updated':
            await updateSearchIndex(payload.data);
            break;
    }
});

// Multi-level wildcard (**)
eventBus.on('payment.**', async (payload) => {
    // Matches: payment.processed, payment.gateway.response, payment.refund.initiated
    await logPaymentActivity(payload);
});

// Combine wildcards
eventBus.on('order.*.completed', async (payload) => {
    // Matches: order.processing.completed, order.shipping.completed
    await updateOrderStatistics(payload.data);
});
```

## Advanced Features

### Multi-Service Architecture

When using Redis driver with multiple microservices:

```typescript
// Service A configuration
export default {
    driver: 'redis',
    driverConfiguration: {
        redis: {
            serviceName: 'payment-service'
        }
    }
};

// Service B configuration
export default {
    driver: 'redis',
    driverConfiguration: {
        redis: {
            serviceName: 'notification-service'
        }
    }
};

// Both services can subscribe to the same events
// Each maintains independent message position tracking
eventBus.onDurable('order.created', async (payload) => {
    // Each service processes messages independently
});
```

### Event TTL (Time To Live)

Set expiration for events to prevent processing stale data:

```typescript
// Emit event that expires in 5 minutes
await eventBus.emit('verification.code', { code: '123456' }, 5);
```

### Event Inspection

```typescript
// Get all event patterns with listeners
const events = eventBus.eventNames();
console.log('Active event listeners:', events);

// Count listeners for specific event
const count = eventBus.listenerCount('user.created');
console.log(`Listeners for user.created: ${count}`);
```

## Real-World Examples

### E-Commerce Order Processing

```typescript
// OrderService emits events
export default class OrderService {
    constructor(private eventBus: EventBusService) {}
    
    async createOrder(data: OrderData): Promise<Order> {
        const order = await this.orderRepository.create(data);
        
        // Emit event for other services
        await this.eventBus.emit('order.created', {
            orderId: order.id,
            userId: order.userId,
            items: order.items,
            total: order.total
        });
        
        return order;
    }
    
    async shipOrder(orderId: number): Promise<void> {
        await this.orderRepository.markAsShipped(orderId);
        
        await this.eventBus.emit('order.shipped', {
            orderId,
            shippedAt: Date.now()
        });
    }
}

// InventoryService listens and updates stock
export default class InventoryService {
    constructor(private eventBus: EventBusService) {
        this.setupEventListeners();
    }
    
    private setupEventListeners(): void {
        this.eventBus.on('order.created', async (payload) => {
            // Reduce stock for ordered items
            for (const item of payload.data.items) {
                await this.reduceStock(item.productId, item.quantity);
            }
        });
        
        this.eventBus.on('order.cancelled', async (payload) => {
            // Restore stock for cancelled items
            const order = await this.getOrder(payload.data.orderId);
            for (const item of order.items) {
                await this.restoreStock(item.productId, item.quantity);
            }
        });
    }
}

// NotificationService sends emails
export default class NotificationService {
    constructor(private eventBus: EventBusService) {
        this.setupEventListeners();
    }
    
    private setupEventListeners(): void {
        // Use durable subscription to never miss notifications
        this.eventBus.onDurable('order.*', async (payload) => {
            const user = await this.getUser(payload.data.userId);
            
            switch (payload.event.name) {
                case 'order.created':
                    await this.sendEmail(user.email, 'Order Confirmation', payload.data);
                    break;
                case 'order.shipped':
                    await this.sendEmail(user.email, 'Order Shipped', payload.data);
                    break;
                case 'order.delivered':
                    await this.sendEmail(user.email, 'Order Delivered', payload.data);
                    break;
            }
        });
    }
}
```

### User Activity Tracking

```typescript
export default class UserActivityService {
    constructor(private eventBus: EventBusService) {
        this.trackUserActivity();
    }
    
    private trackUserActivity(): void {
        // Track all user events
        this.eventBus.on('user.**', async (payload) => {
            await this.recordActivity({
                event: payload.event.name,
                userId: payload.data.userId,
                timestamp: payload.event.timestamp,
                metadata: payload.data
            });
        });
        
        // Update user last seen
        this.eventBus.on('user.action.*', async (payload) => {
            await this.updateLastSeen(payload.data.userId);
        });
    }
    
    // Emit activity events
    async trackPageView(userId: number, page: string): Promise<void> {
        await this.eventBus.emit('user.action.pageview', {
            userId,
            page,
            timestamp: Date.now()
        });
    }
    
    async trackPurchase(userId: number, amount: number): Promise<void> {
        await this.eventBus.emit('user.action.purchase', {
            userId,
            amount,
            timestamp: Date.now()
        });
    }
}
```

### Cache Invalidation

```typescript
export default class CachedProductService {
    constructor(
        private eventBus: EventBusService,
        private cache: CacheService
    ) {
        this.setupCacheInvalidation();
    }
    
    private setupCacheInvalidation(): void {
        // Clear cache when products change
        this.eventBus.on('product.updated', async (payload) => {
            await this.cache.delete(`product:${payload.data.productId}`);
            await this.cache.delete('products:featured');
        });
        
        this.eventBus.on('product.deleted', async (payload) => {
            await this.cache.delete(`product:${payload.data.productId}`);
            await this.cache.delete('products:all');
        });
        
        // Clear category cache
        this.eventBus.on('category.*', async (payload) => {
            const keys = await this.cache.keys(/^category:.*/);
            if (keys.length > 0) {
                await this.cache.deleteMany(keys);
            }
        });
    }
    
    async updateProduct(id: number, data: any): Promise<void> {
        await this.productRepository.update(id, data);
        
        // Emit event to trigger cache invalidation
        await this.eventBus.emit('product.updated', {
            productId: id,
            changes: data
        });
    }
}
```

### Scheduled Task Results

```typescript
export default class ReportGenerationService {
    constructor(private eventBus: EventBusService) {}
    
    async generateDailyReport(): Promise<void> {
        const report = await this.buildReport();
        
        // Emit with 24-hour TTL
        await this.eventBus.emit('report.daily.generated', {
            reportId: report.id,
            generatedAt: Date.now(),
            data: report.data
        }, 1440); // 24 hours in minutes
    }
}

// Report processor service
export default class ReportProcessorService {
    constructor(private eventBus: EventBusService) {
        // Use durable to process even if service was down
        this.eventBus.onDurable('report.*.generated', async (payload) => {
            await this.processReport(payload.data);
        });
    }
}
```

## Best Practices

### 1. Use Meaningful Event Names

```typescript
// ✅ Good: Clear, hierarchical naming
await eventBus.emit('user.registration.completed', data);
await eventBus.emit('payment.refund.initiated', data);
await eventBus.emit('order.shipping.label.created', data);

// ❌ Bad: Unclear or flat naming
await eventBus.emit('user_done', data);
await eventBus.emit('payment', data);
await eventBus.emit('1', data);
```

### 2. Structure Event Data Consistently

```typescript
// ✅ Good: Consistent structure
await eventBus.emit('user.created', {
    userId: 123,
    email: 'user@example.com',
    createdAt: Date.now()
});

await eventBus.emit('user.updated', {
    userId: 123,
    changes: { email: 'newemail@example.com' },
    updatedAt: Date.now()
});

// ❌ Bad: Inconsistent structure
await eventBus.emit('user.created', { id: 123 });
await eventBus.emit('user.updated', 123); // Just ID
```

### 3. Handle Errors in Event Handlers

```typescript
eventBus.on('order.created', async (payload) => {
    try {
        await processOrder(payload.data);
    } catch (error) {
        // Log error but don't crash the application
        this.logger.error('Failed to process order', {
            error,
            orderId: payload.data.orderId
        });
        
        // Optionally emit error event
        await eventBus.emit('order.processing.failed', {
            orderId: payload.data.orderId,
            error: error.message
        });
    }
});
```

### 4. Use Durable Subscriptions for Critical Events

```typescript
// ✅ Use onDurable for events that must not be missed
eventBus.onDurable('payment.received', async (payload) => {
    // Critical: Update user balance
    await updateUserBalance(payload.data);
});

// ✅ Use regular on() for real-time, non-critical events
eventBus.on('user.online', async (payload) => {
    // Non-critical: Update online status indicator
    await updateOnlineStatus(payload.data.userId, true);
});
```

### 5. Set Appropriate TTL for Time-Sensitive Events

```typescript
// Short TTL for time-sensitive data
await eventBus.emit('otp.generated', { code: '123456' }, 5); // 5 minutes

// Longer TTL for less time-sensitive data
await eventBus.emit('report.weekly.ready', { url: '/reports/123' }, 10080); // 1 week

// No TTL for permanent events
await eventBus.emit('user.deleted', { userId: 123 }); // No expiration
```

## Debugging EventBus

### Monitor Event Flow

```typescript
// Debug helper to log all events
if (process.env.DEBUG_EVENTS === 'true') {
    eventBus.on('**', async (payload) => {
        console.log(`[EVENT] ${payload.event.name}`, {
            timestamp: new Date(payload.event.timestamp),
            data: payload.data
        });
    });
}
```

### Check Active Listeners

```typescript
// List all events with listeners
const events = eventBus.eventNames();
console.log('Active events:', events);

// Check specific event
const count = eventBus.listenerCount('user.created');
console.log(`Listeners for user.created: ${count}`);
```

## Migration from Direct Dependencies

### Before (Tight Coupling)

```typescript
export default class OrderService {
    constructor(
        private emailService: EmailService,
        private inventoryService: InventoryService,
        private analyticsService: AnalyticsService
    ) {}
    
    async createOrder(data: OrderData): Promise<Order> {
        const order = await this.orderRepository.create(data);
        
        // Tight coupling - OrderService knows about all these services
        await this.emailService.sendOrderConfirmation(order);
        await this.inventoryService.reduceStock(order.items);
        await this.analyticsService.trackPurchase(order);
        
        return order;
    }
}
```

### After (Loose Coupling with EventBus)

```typescript
export default class OrderService {
    constructor(private eventBus: EventBusService) {}
    
    async createOrder(data: OrderData): Promise<Order> {
        const order = await this.orderRepository.create(data);
        
        // Loose coupling - OrderService just emits an event
        await this.eventBus.emit('order.created', order);
        
        return order;
    }
}

// Other services subscribe independently
emailService.on('order.created', async (payload) => {
    await sendOrderConfirmation(payload.data);
});

inventoryService.on('order.created', async (payload) => {
    await reduceStock(payload.data.items);
});

analyticsService.on('order.created', async (payload) => {
    await trackPurchase(payload.data);
});
```

## Summary

EventBus provides powerful event-driven architecture for your application:

- ✅ Decouples components through events
- ✅ Supports multiple drivers (Memory, Redis)
- ✅ Wildcard pattern matching for flexible subscriptions
- ✅ Guaranteed delivery with durable subscriptions (Redis)
- ✅ Multi-service support with independent message tracking
- ✅ TTL support for time-sensitive events
- ✅ Simple, consistent API across all drivers
- ✅ Perfect for microservices and distributed systems

Use EventBus to build scalable, maintainable applications where components communicate through well-defined events rather than direct dependencies.