# Throttling

## Quick Start

Protect your API from abuse and ensure fair resource usage with Sosise's built-in throttling!

```typescript
// Enable in src/config/throttling.ts
export default {
    isEnabled: true,
    clientIpHeader: 'X-Forwarded-For',
    skipSubnets: ['127.0.0.1/32', '10.0.0.0/8'], // Allow internal traffic
    routeRules: [
        {
            httpMethod: 'POST',
            path: '/api/auth/login',
            maxRequestsPerMinute: 5 // Prevent brute force
        },
        {
            httpMethod: 'GET', 
            path: '/api/users/:id',
            maxRequestsPerMinute: 100 // Allow normal usage
        }
    ]
};
```

Automatically returns `429 Too Many Requests` when limits are exceeded. Perfect for preventing abuse and ensuring API stability!

## Introduction

Throttling (also known as rate limiting) is essential for protecting your API from abuse, ensuring fair resource distribution, and maintaining service quality. Without proper throttling, malicious users could overwhelm your servers, legitimate users might experience poor performance, and your infrastructure costs could skyrocket.

Sosise provides flexible throttling capabilities that work at the route level, allowing you to set different limits for different endpoints based on their sensitivity and resource requirements. The system tracks requests per client IP and enforces limits using a sliding window approach.

## Basic Configuration

### Enable Throttling

Configure throttling in `src/config/throttling.ts`:

```typescript
export default {
    // Enable throttling globally
    isEnabled: true,
    
    // IP detection (important for apps behind proxies)
    clientIpHeader: 'X-Forwarded-For', // or 'X-Real-IP'
    
    // Exempt trusted networks
    skipSubnets: [
        '127.0.0.1/32',    // localhost
        '10.0.0.0/8',      // internal network
        '172.16.0.0/12',   // private network
        '192.168.0.0/16'   // local network
    ],
    
    // Default rule for all unmatched routes
    defaultRule: {
        maxRequestsPerMinute: 60 // 1 request per second average
    },
    
    // Specific rules for different endpoints
    routeRules: [
        // Authentication endpoints - strict limits
        {
            httpMethod: 'POST',
            path: '/api/auth/login',
            maxRequestsPerMinute: 5 // Prevent brute force
        },
        {
            httpMethod: 'POST',
            path: '/api/auth/register',
            maxRequestsPerMinute: 3 // Prevent spam registrations
        },
        
        // Password reset - very strict
        {
            httpMethod: 'POST',
            path: '/api/auth/forgot-password',
            maxRequestsPerMinute: 2
        },
        
        // Public endpoints - moderate limits
        {
            httpMethod: 'GET',
            path: '/api/products',
            maxRequestsPerMinute: 100
        },
        {
            httpMethod: 'GET',
            path: '/api/products/:id',
            maxRequestsPerMinute: 200
        },
        
        // User-specific data - higher limits for authenticated users
        {
            httpMethod: 'GET',
            path: '/api/users/:id/orders',
            maxRequestsPerMinute: 30
        },
        
        // Search endpoints - moderate to prevent abuse
        {
            httpMethod: 'GET',
            path: '/api/search',
            maxRequestsPerMinute: 20
        },
        
        // File upload - very strict
        {
            httpMethod: 'POST',
            path: '/api/files/upload',
            maxRequestsPerMinute: 5
        }
    ]
};
```

## Advanced Configuration

### Environment-Specific Settings

```typescript
// src/config/throttling.ts
const baseConfig = {
    isEnabled: true,
    clientIpHeader: 'X-Forwarded-For',
    skipSubnets: ['127.0.0.1/32']
};

const environmentConfig = {
    development: {
        ...baseConfig,
        isEnabled: false, // Disable in development
        routeRules: []
    },
    
    staging: {
        ...baseConfig,
        routeRules: [
            {
                httpMethod: 'POST',
                path: '/api/auth/login',
                maxRequestsPerMinute: 10 // More lenient for testing
            }
        ]
    },
    
    production: {
        ...baseConfig,
        routeRules: [
            {
                httpMethod: 'POST', 
                path: '/api/auth/login',
                maxRequestsPerMinute: 5 // Strict in production
            },
            {
                httpMethod: 'GET',
                path: '/api/users/:id',
                maxRequestsPerMinute: 100
            }
        ]
    }
};

export default environmentConfig[process.env.NODE_ENV || 'development'];
```

### Dynamic Configuration

```typescript
// src/config/throttling.ts
export default class ThrottlingConfigService {
    public static getConfig(): ThrottlingConfig {
        return {
            isEnabled: process.env.THROTTLING_ENABLED === 'true',
            
            // Adjust limits based on server capacity
            routeRules: [
                {
                    httpMethod: 'GET',
                    path: '/api/heavy-computation',
                    maxRequestsPerMinute: this.getComputeLimit()
                },
                {
                    httpMethod: 'POST',
                    path: '/api/auth/login',
                    maxRequestsPerMinute: this.getAuthLimit()
                }
            ]
        };
    }
    
    private static getComputeLimit(): number {
        // Adjust based on server resources
        const cpuCount = require('os').cpus().length;
        return Math.max(5, Math.floor(cpuCount * 2));
    }
    
    private static getAuthLimit(): number {
        // Stricter limits during high traffic periods
        const hour = new Date().getHours();
        const isPeakTime = hour >= 9 && hour <= 17; // 9 AM to 5 PM
        
        return isPeakTime ? 3 : 5;
    }
}
```

## Route Rule Patterns

### Order Matters!

Route rules are matched in order, so put more specific routes first:

```typescript
// ✅ Correct order - specific to general
routeRules: [
    {
        path: '/api/users/me',        // Most specific
        maxRequestsPerMinute: 60
    },
    {
        path: '/api/users/search',    // Specific endpoint
        maxRequestsPerMinute: 20
    },
    {
        path: '/api/users/:id',       // Parameterized route
        maxRequestsPerMinute: 100
    },
    {
        path: '/api/users',           // General endpoint
        maxRequestsPerMinute: 50
    }
]

// ❌ Wrong order - general rules might match first
routeRules: [
    {
        path: '/api/users/:id',       // This would match /api/users/me!
        maxRequestsPerMinute: 100
    },
    {
        path: '/api/users/me',        // Never reached
        maxRequestsPerMinute: 60
    }
]
```

### HTTP Method Specificity

```typescript
routeRules: [
    // Different limits for different methods
    {
        httpMethod: 'GET',
        path: '/api/products/:id',
        maxRequestsPerMinute: 200 // Read operations - higher limit
    },
    {
        httpMethod: 'PUT',
        path: '/api/products/:id', 
        maxRequestsPerMinute: 10  // Write operations - lower limit
    },
    {
        httpMethod: 'DELETE',
        path: '/api/products/:id',
        maxRequestsPerMinute: 5   // Delete operations - very low limit
    }
]
```

## Real-World Use Cases

### E-commerce API Protection

```typescript
export default class EcommerceThrottlingService {
    public static getConfig(): ThrottlingConfig {
        return {
            isEnabled: true,
            clientIpHeader: 'X-Forwarded-For',
            skipSubnets: ['10.0.0.0/8'], // Internal services
            
            routeRules: [
                // Cart operations - prevent spam
                {
                    httpMethod: 'POST',
                    path: '/api/cart/add',
                    maxRequestsPerMinute: 30
                },
                {
                    httpMethod: 'PUT',
                    path: '/api/cart/update',
                    maxRequestsPerMinute: 20
                },
                
                // Checkout flow - critical protection
                {
                    httpMethod: 'POST',
                    path: '/api/checkout/create',
                    maxRequestsPerMinute: 5 // Prevent payment abuse
                },
                {
                    httpMethod: 'POST',
                    path: '/api/payment/process',
                    maxRequestsPerMinute: 3 // Very strict
                },
                
                // Product browsing - generous limits
                {
                    httpMethod: 'GET',
                    path: '/api/products',
                    maxRequestsPerMinute: 100
                },
                {
                    httpMethod: 'GET',
                    path: '/api/products/:id',
                    maxRequestsPerMinute: 200
                },
                
                // Search - prevent scraping
                {
                    httpMethod: 'GET',
                    path: '/api/products/search',
                    maxRequestsPerMinute: 30
                },
                
                // Reviews - prevent spam
                {
                    httpMethod: 'POST',
                    path: '/api/products/:id/reviews',
                    maxRequestsPerMinute: 5
                },
                
                // Wishlist - reasonable limits
                {
                    httpMethod: 'POST',
                    path: '/api/wishlist/add',
                    maxRequestsPerMinute: 20
                }
            ]
        };
    }
}
```

### API Gateway Configuration

```typescript
export default class APIGatewayThrottling {
    public static getConfig(): ThrottlingConfig {
        return {
            isEnabled: true,
            clientIpHeader: 'X-Forwarded-For',
            
            // Exempt health checks and monitoring
            skipSubnets: [
                '10.0.0.0/8',      // Internal network
                '172.16.0.0/12',   // Docker networks
                '169.254.0.0/16'   // AWS metadata service
            ],
            
            routeRules: [
                // Health checks - unlimited
                {
                    httpMethod: 'GET',
                    path: '/health',
                    maxRequestsPerMinute: 1000
                },
                {
                    httpMethod: 'GET',
                    path: '/metrics',
                    maxRequestsPerMinute: 100
                },
                
                // Public API endpoints
                {
                    httpMethod: 'GET',
                    path: '/api/v1/public/*',
                    maxRequestsPerMinute: 100
                },
                
                // Authenticated API - higher limits
                {
                    httpMethod: 'GET',
                    path: '/api/v1/protected/*',
                    maxRequestsPerMinute: 300
                },
                
                // Admin endpoints - very restrictive
                {
                    httpMethod: 'POST',
                    path: '/api/v1/admin/*',
                    maxRequestsPerMinute: 10
                },
                
                // Bulk operations - extremely limited
                {
                    httpMethod: 'POST',
                    path: '/api/v1/bulk/*',
                    maxRequestsPerMinute: 2
                }
            ]
        };
    }
}
```

### SaaS Application Throttling

```typescript
export default class SaaSThrottling {
    public static getConfig(): ThrottlingConfig {
        return {
            isEnabled: true,
            clientIpHeader: 'X-Forwarded-For',
            skipSubnets: ['127.0.0.1/32'],
            
            routeRules: [
                // User authentication - prevent brute force
                {
                    httpMethod: 'POST',
                    path: '/api/auth/login',
                    maxRequestsPerMinute: 5
                },
                
                // Trial registration - prevent abuse
                {
                    httpMethod: 'POST',
                    path: '/api/auth/register',
                    maxRequestsPerMinute: 3
                },
                
                // Dashboard data - frequent access expected
                {
                    httpMethod: 'GET',
                    path: '/api/dashboard/*',
                    maxRequestsPerMinute: 120
                },
                
                // Data export - resource intensive
                {
                    httpMethod: 'POST',
                    path: '/api/data/export',
                    maxRequestsPerMinute: 2
                },
                
                // File uploads - bandwidth intensive
                {
                    httpMethod: 'POST',
                    path: '/api/files/upload',
                    maxRequestsPerMinute: 10
                },
                
                // Reports generation - CPU intensive
                {
                    httpMethod: 'POST',
                    path: '/api/reports/generate',
                    maxRequestsPerMinute: 5
                },
                
                // API integrations - third party access
                {
                    httpMethod: 'GET',
                    path: '/api/integrations/*',
                    maxRequestsPerMinute: 200
                },
                {
                    httpMethod: 'POST',
                    path: '/api/integrations/*',
                    maxRequestsPerMinute: 50
                }
            ]
        };
    }
}
```

## Custom Error Handling

### Custom Throttling Response

```typescript
// src/app/Http/Middlewares/CustomThrottlingMiddleware.ts
export default class CustomThrottlingMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction): Promise<void> {
        // Check if rate limit exceeded (this is handled automatically by Sosise)
        // This middleware runs after throttling to customize the response
        
        if (response.statusCode === 429) {
            const retryAfter = response.getHeader('Retry-After') || 60;
            
            response.json({
                code: 4001,
                message: 'Rate limit exceeded',
                details: {
                    retryAfter: parseInt(retryAfter.toString()),
                    endpoint: request.path,
                    method: request.method,
                    currentTime: new Date().toISOString()
                },
                suggestions: [
                    'Wait before making another request',
                    'Consider upgrading your plan for higher limits',
                    'Use pagination for bulk operations'
                ]
            });
        }
        
        next();
    }
}
```

### Logging Throttle Events

```typescript
// src/app/Services/ThrottlingMonitorService.ts
export default class ThrottlingMonitorService {
    constructor(private logger: LoggerService) {}
    
    public logThrottleEvent(request: Request, limit: number): void {
        this.logger.warn('Rate limit exceeded', {
            ip: this.getClientIP(request),
            userAgent: request.get('User-Agent'),
            endpoint: request.path,
            method: request.method,
            limit,
            timestamp: new Date().toISOString()
        });
        
        // Track repeat offenders
        this.trackRepeatOffender(this.getClientIP(request));
    }
    
    private getClientIP(request: Request): string {
        return request.headers['x-forwarded-for'] as string || request.ip;
    }
    
    private async trackRepeatOffender(ip: string): Promise<void> {
        // Implement logic to track and potentially block repeat offenders
        const offenseCount = await this.getOffenseCount(ip);
        
        if (offenseCount > 10) {
            this.logger.error('Potential abuse detected', { ip, offenseCount });
            // Consider temporary IP blocking here
        }
    }
}
```

## Performance Considerations

### Memory Usage

```typescript
// Monitor throttling memory usage
export default class ThrottlingPerformanceMonitor {
    public static monitorMemoryUsage(): void {
        setInterval(() => {
            const memoryUsage = process.memoryUsage();
            
            if (memoryUsage.heapUsed > 100 * 1024 * 1024) { // 100MB
                console.warn('High memory usage detected, consider clearing throttling cache');
            }
        }, 60000); // Check every minute
    }
    
    // Clean up old throttling data periodically
    public static setupCleanup(): void {
        setInterval(() => {
            // This would be implemented in the core throttling service
            // to remove expired rate limit counters
        }, 5 * 60 * 1000); // Clean every 5 minutes
    }
}
```

## Best Practices

### 1. Set Appropriate Limits

```typescript
// ✅ Good: Realistic limits based on endpoint purpose
{
    httpMethod: 'GET',
    path: '/api/products',
    maxRequestsPerMinute: 100 // Browsing is common
}

{
    httpMethod: 'POST',
    path: '/api/auth/login',
    maxRequestsPerMinute: 5   // Authentication should be limited
}

// ❌ Bad: Too restrictive for normal usage
{
    httpMethod: 'GET', 
    path: '/api/products',
    maxRequestsPerMinute: 2   // Too low for browsing
}
```

### 2. Consider User Experience

```typescript
// ✅ Good: Different limits for different user types
const getUserTypeLimit = (request: Request): number => {
    if (request.user?.isPremium) {
        return 200; // Premium users get higher limits
    }
    
    if (request.user) {
        return 100; // Authenticated users
    }
    
    return 50; // Anonymous users
};
```

### 3. Use Subnet Exemptions Wisely

```typescript
// ✅ Good: Exempt legitimate internal traffic
skipSubnets: [
    '127.0.0.1/32',     // localhost
    '10.0.0.0/8',       // private network
    '172.16.0.0/12',    // docker networks
]

// ❌ Bad: Exempting too broad ranges
skipSubnets: [
    '0.0.0.0/0'  // This exempts everyone!
]
```

### 4. Monitor and Adjust

```typescript
// ✅ Good: Log and monitor throttling effectiveness
export default class ThrottlingAnalytics {
    public generateReport(): ThrottlingReport {
        return {
            totalRequests: this.getTotalRequests(),
            throttledRequests: this.getThrottledRequests(),
            topOffenders: this.getTopOffenders(),
            mostThrottledEndpoints: this.getMostThrottledEndpoints(),
            recommendations: this.generateRecommendations()
        };
    }
}
```

## Troubleshooting

### Common Issues

1. **Throttling Too Aggressive**
   ```bash
   # Check current limits
   grep -r "maxRequestsPerMinute" src/config/
   
   # Increase limits temporarily
   # Edit src/config/throttling.ts
   ```

2. **IP Detection Issues**
   ```typescript
   // Debug IP detection
   console.log('Detected IP:', request.headers['x-forwarded-for'] || request.ip);
   console.log('All headers:', request.headers);
   ```

3. **Subnet Exemption Not Working**
   ```typescript
   // Verify subnet configuration
   const ip = require('ip');
   console.log('Is in range:', ip.cidrSubnet('10.0.0.0/8').contains('10.1.2.3'));
   ```

## Summary

Sosise throttling provides:

- ✅ Flexible per-route rate limiting
- ✅ IP-based request tracking
- ✅ Subnet exemptions for trusted sources
- ✅ Automatic 429 responses
- ✅ Environment-specific configuration
- ✅ Pattern matching for dynamic routes
- ✅ Easy integration with existing middleware
- ✅ Production-ready performance

Protect your APIs and ensure fair resource usage with Sosise's comprehensive throttling system!