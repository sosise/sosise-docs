# Throttling

## Quick Start

Protect your API from abuse with Sosise's built-in throttling system! 

```typescript
// Enable in src/config/throttling.ts
export default {
    isEnabled: true,
    clientIpHeader: 'X-Forwarded-For',
    skipSubnets: ['10.0.0.0/24'], // Skip internal networks
    routeRules: [
        {
            httpMethod: 'POST',
            path: '/api/login',
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

Automatically returns `429 Too Many Requests` when limits are exceeded!

## Introduction

Rate limiting (throttling) protects your API from abuse by limiting the number of requests from each IP address. Sosise includes a simple but effective throttling system that works at the route level.

The throttling system:
- Tracks requests per client IP address
- Uses a sliding 60-second window
- Supports route-specific rules with pattern matching
- Can exempt trusted IP subnets
- Returns HTTP 429 when limits are exceeded

## Configuration

### Enable Throttling

First, ensure ThrottlingMiddleware is registered in your middleware kernel:

```typescript
// src/app/Http/Middlewares/Kernel.ts
export const middlewares = [
    'ExampleMiddleware',
    'ThrottlingMiddleware', // Add this line
];
```

### Basic Configuration

Configure throttling rules in `src/config/throttling.ts`:

```typescript
const throttlingConfig = {
    /**
     * Enable or disable request throttling
     */
    isEnabled: true,

    /**
     * HTTP header used to determine client's IP address
     * This header must be set by a trusted proxy or load balancer
     */
    clientIpHeader: 'X-Forwarded-For',

    /**
     * List of CIDR subnets that are skipped (not throttled)
     */
    skipSubnets: ['10.0.0.0/24'],

    /**
     * Array of route-specific throttling rules
     */
    routeRules: [
        {
            // HTTP method for this rule
            httpMethod: 'GET',
            // URL path pattern (supports Express.js route patterns)
            path: '/api/users/:id',
            // Maximum requests allowed per rolling minute
            maxRequestsPerMinute: 10,
        },
    ],
};

export default throttlingConfig;
```

## Route Rules

### Basic Route Matching

```typescript
routeRules: [
    {
        httpMethod: 'GET',
        path: '/api/users',
        maxRequestsPerMinute: 50
    },
    {
        httpMethod: 'POST',
        path: '/api/users',
        maxRequestsPerMinute: 10
    }
]
```

### Route Parameters

Throttling supports Express.js route patterns:

```typescript
routeRules: [
    {
        httpMethod: 'GET',
        path: '/api/users/:id',           // Matches /api/users/123
        maxRequestsPerMinute: 100
    },
    {
        httpMethod: 'GET',
        path: '/api/products/:id/reviews', // Matches /api/products/456/reviews
        maxRequestsPerMinute: 20
    }
]
```

### Wildcard Routes

```typescript
routeRules: [
    {
        httpMethod: 'GET',
        path: '/api/admin/*',            // Matches any admin route
        maxRequestsPerMinute: 5
    }
]
```

## IP Address Handling

### Behind Proxy/Load Balancer

When running behind nginx, Cloudflare, or other proxies:

```typescript
// For nginx with X-Forwarded-For
clientIpHeader: 'X-Forwarded-For'

// For Cloudflare
clientIpHeader: 'CF-Connecting-IP'

// For AWS ALB
clientIpHeader: 'X-Forwarded-For'
```

### Skip Internal Networks

Exempt internal services and health checks:

```typescript
skipSubnets: [
    '10.0.0.0/8',        // Internal network
    '172.16.0.0/12',     // Docker networks
    '192.168.0.0/16',    // Private networks  
    '127.0.0.1/32'       // Localhost
]
```

## Practical Examples

### API Protection Setup

```typescript
// src/config/throttling.ts
export default {
    isEnabled: true,
    clientIpHeader: 'X-Forwarded-For',
    skipSubnets: ['10.0.0.0/8', '127.0.0.1/32'],
    routeRules: [
        // Authentication endpoints - strict limits
        {
            httpMethod: 'POST',
            path: '/api/auth/login',
            maxRequestsPerMinute: 5
        },
        {
            httpMethod: 'POST',
            path: '/api/auth/register',
            maxRequestsPerMinute: 3
        },
        
        // General API endpoints - moderate limits
        {
            httpMethod: 'GET',
            path: '/api/users/:id',
            maxRequestsPerMinute: 100
        },
        {
            httpMethod: 'POST',
            path: '/api/users',
            maxRequestsPerMinute: 20
        },
        
        // Admin endpoints - very strict limits
        {
            httpMethod: 'POST',
            path: '/api/admin/*',
            maxRequestsPerMinute: 10
        },
        
        // File uploads - limited
        {
            httpMethod: 'POST',
            path: '/api/upload',
            maxRequestsPerMinute: 5
        }
    ]
};
```

## Error Response

When throttling limit is exceeded, clients receive:

```json
{
    "error": "Too Many Requests",
    "message": "Rate limit exceeded. Try again in 60 seconds.",
    "statusCode": 429
}
```

HTTP headers included:
- `X-RateLimit-Limit`: Maximum requests allowed
- `X-RateLimit-Remaining`: Requests remaining in current window
- `X-RateLimit-Reset`: Time when limit resets

## Monitoring and Debugging

### Check Throttling Status

```typescript
// In your controller or middleware
console.log('Client IP:', request.ip);
console.log('Headers:', request.headers);
```

### Disable Throttling for Testing

```typescript
// src/config/throttling.ts
export default {
    isEnabled: false, // Disable during development/testing
    // ... rest of config
};
```

## Production Considerations

### Performance

- Throttling data is stored in memory
- Each request checks against all rules
- Consider Redis for distributed systems

### Security

- Always use trusted proxy headers only
- Never accept client-provided IP headers directly
- Monitor for circumvention attempts

### Tuning Limits

Start conservative and adjust based on:
- Server capacity
- Legitimate user behavior
- Attack patterns observed

## Limitations

- **In-Memory Storage**: Throttle data doesn't persist across restarts
- **Single Instance**: Works only within one application instance
- **No Distributed Support**: Each instance maintains separate counters

For high-scale distributed applications, consider external rate limiting solutions like Redis-based systems or API gateways.

## Best Practices

1. **Start Conservative**: Begin with low limits and increase based on monitoring
2. **Monitor Legitimate Users**: Ensure real users aren't being blocked
3. **Use Different Limits**: Apply stricter limits to sensitive endpoints
4. **Log Blocked Requests**: Track throttling events for analysis
5. **Test Thoroughly**: Verify rules work as expected in development

## Summary

Sosise's throttling system provides:

- ✅ **Easy Configuration** - Simple config file setup
- ✅ **Route-Specific Rules** - Different limits for different endpoints
- ✅ **Express.js Pattern Support** - Full route pattern matching
- ✅ **IP Subnet Exemptions** - Skip internal networks
- ✅ **Standard HTTP Responses** - Proper 429 status codes
- ✅ **Production Ready** - Lightweight and efficient

Perfect for protecting your API from abuse while maintaining performance!