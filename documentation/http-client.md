# HttpClient

## Quick Start

The HttpClient makes it easy to call external APIs from your Sosise application. 

### Simple Example

```typescript
import HttpClient from "sosise-core/build/HttpClient/HttpClient";

// Create client
const httpClient = new HttpClient({
    baseURL: 'https://api.example.com'
});

// Make a GET request
const response = await httpClient.request({
    method: 'get',
    url: '/users'
});

console.log(response.data); // API response data
```

That's it! You're ready to make API calls.

## Introduction

Making HTTP requests to external APIs is common in web applications. Sosise provides the `HttpClient` class to simplify this process. Built on top of Axios, it provides additional features like automatic retries, error handling, and easy configuration.

## Basic Usage

### Creating an HttpClient

```typescript
import HttpClient from "sosise-core/build/HttpClient/HttpClient";

const httpClient = new HttpClient({
    baseURL: 'https://api.example.com',
    headers: {
        'User-Agent': 'My App 1.0'
    }
});
```

### Making Requests

#### GET Request

```typescript
// Simple GET
const users = await httpClient.request({
    method: 'get',
    url: '/users'
});

// GET with query parameters
const filteredUsers = await httpClient.request({
    method: 'get',
    url: '/users',
    params: {
        status: 'active',
        limit: 10
    }
});
// This calls: GET /users?status=active&limit=10
```

#### POST Request (Create Data)

```typescript
const newUser = await httpClient.request({
    method: 'post',
    url: '/users',
    data: {
        name: 'John Doe',
        email: 'john@example.com'
    }
});
```

#### PUT Request (Update Data)

```typescript
const updatedUser = await httpClient.request({
    method: 'put',
    url: '/users/123',
    data: {
        name: 'John Smith',
        email: 'johnsmith@example.com'
    }
});
```

#### DELETE Request

```typescript
const result = await httpClient.request({
    method: 'delete',
    url: '/users/123'
});
```

### Error Handling

Always wrap HTTP requests in try-catch blocks:

```typescript
try {
    const response = await httpClient.request({
        method: 'get',
        url: '/users'
    });
    
    console.log('Success:', response.data);
} catch (error) {
    console.error('Request failed:', error.message);
    // Handle the error appropriately
}
```

## Common Use Cases

### Calling a REST API

```typescript
export default class UserApiService {
    private httpClient: HttpClient;

    constructor() {
        this.httpClient = new HttpClient({
            baseURL: 'https://jsonplaceholder.typicode.com'
        });
    }

    async getUser(id: number) {
        const response = await this.httpClient.request({
            method: 'get',
            url: `/users/${id}`
        });
        return response.data;
    }

    async createUser(userData: {name: string, email: string}) {
        const response = await this.httpClient.request({
            method: 'post',
            url: '/users',
            data: userData
        });
        return response.data;
    }
}
```

### Authentication Headers

```typescript
const httpClient = new HttpClient({
    baseURL: 'https://api.example.com',
    headers: {
        'Authorization': 'Bearer your-api-token',
        'Content-Type': 'application/json'
    }
});
```

### Custom Error Responses

Handle specific HTTP status codes gracefully:

```typescript
const response = await httpClient.request({
    method: 'get',
    url: '/users/999',
    // Return null instead of throwing error for 404
    returnInCaseOfStatusCodes: {
        404: null
    }
});

if (response.data === null) {
    console.log('User not found');
} else {
    console.log('User:', response.data);
}
```

## Automatic Retries

For unreliable networks, you can automatically retry failed requests:

### Basic Retry

```typescript
const retryConfig = {
    requestMaxRetries: 3,           // Try up to 3 times
    requestRetryInterval: 1000,     // Wait 1 second between retries
    requestRetryStrategy: 'linear'  // Wait same amount each time
};

try {
    const response = await httpClient.requestWithRetry({
        method: 'get',
        url: '/users'
    }, retryConfig);
} catch (error) {
    console.error('Failed after 3 retries:', error);
}
```

### Advanced Retry Configuration

```typescript
const retryConfig = {
    requestMaxRetries: 5,
    requestRetryInterval: 1000,
    requestRetryStrategy: 'exponential',  // 1s, 2s, 4s, 8s, 16s
    // Don't retry for these error codes (client errors)
    requestDoNotRetryForHttpCodes: [400, 401, 403, 404]
};

const response = await httpClient.requestWithRetry({
    method: 'post',
    url: '/process-payment',
    data: paymentData
}, retryConfig);
```

## Advanced Configuration

### Complete Configuration Options

```typescript
import HttpClient from "sosise-core/build/HttpClient/HttpClient";

const httpClient = new HttpClient({
    baseURL: 'https://api.example.com',
    timeout: 10000,  // 10 second timeout
    headers: {
        'User-Agent': 'My App 1.0',
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    },
    
    // SSL/TLS options
    ignoreSelfSignedCertificates: true,
    
    // Connection pooling
    keepAlive: true,
    keepAliveMsecs: 30000,
    
    // Debug logging
    debug: {
        enabled: true,
        loggingChannel: 'api-requests'
    }
});
```

### Request-Level Configuration

You can override client settings for individual requests:

```typescript
const response = await httpClient.request({
    method: 'post',
    url: '/upload',
    data: fileData,
    timeout: 60000,  // 60 second timeout for file upload
    headers: {
        'Content-Type': 'multipart/form-data'
    }
});
```

## Error Handling Strategies

### Custom Exceptions

Define your own exception types for better error handling:

```typescript
class ApiException extends Exception {
    constructor(message: string, public httpCode: number, public apiData: any) {
        super(message);
    }
}

const response = await httpClient.request({
    method: 'get',
    url: '/users',
    defaultException: ApiException  // Throw this instead of generic error
});
```

### Handling Different Status Codes

```typescript
try {
    const response = await httpClient.request({
        method: 'get',
        url: '/users/123',
        returnInCaseOfStatusCodes: {
            404: { error: 'User not found' },
            403: { error: 'Access denied' },
            500: { error: 'Server error' }
        }
    });
    
    if (response.data.error) {
        console.error('API Error:', response.data.error);
    } else {
        console.log('User:', response.data);
    }
} catch (error) {
    // Only network errors or unexpected status codes reach here
    console.error('Network error:', error);
}
```

## Best Practices

### 1. Create Service Classes

Don't use HttpClient directly in controllers. Create dedicated API service classes:

```typescript
// Good: Dedicated service class
export default class PaymentApiService {
    private httpClient: HttpClient;

    constructor() {
        this.httpClient = new HttpClient({
            baseURL: process.env.PAYMENT_API_URL,
            headers: {
                'Authorization': `Bearer ${process.env.PAYMENT_API_KEY}`
            }
        });
    }

    async processPayment(amount: number, cardToken: string) {
        return this.httpClient.request({
            method: 'post',
            url: '/payments',
            data: { amount, cardToken }
        });
    }
}
```

### 2. Handle Timeouts

Set appropriate timeouts for different types of requests:

```typescript
// Quick API calls
const quickClient = new HttpClient({
    baseURL: 'https://fast-api.com',
    timeout: 5000  // 5 seconds
});

// File uploads/processing
const slowClient = new HttpClient({
    baseURL: 'https://processing-api.com',
    timeout: 60000  // 60 seconds
});
```

### 3. Use Environment Variables

Store API URLs and keys in environment variables:

```typescript
const httpClient = new HttpClient({
    baseURL: process.env.EXTERNAL_API_URL,
    headers: {
        'Authorization': `Bearer ${process.env.EXTERNAL_API_KEY}`,
        'User-Agent': `${process.env.APP_NAME}/${process.env.APP_VERSION}`
    }
});
```

### 4. Log Important Requests

Enable debug logging for troubleshooting:

```typescript
const httpClient = new HttpClient({
    baseURL: 'https://api.example.com',
    debug: {
        enabled: process.env.NODE_ENV === 'development',
        loggingChannel: 'external-api'
    }
});
```

### 5. Register in IoC Container

Register your HTTP clients in the IoC container for dependency injection:

```typescript
// In src/config/ioc.ts
const iocConfig = {
    nonSingletons: {
        PaymentApiService: () => new PaymentApiService(),
        UserApiService: () => new UserApiService(),
    }
};
```

## Summary

The HttpClient provides a powerful, flexible way to make HTTP requests in your Sosise application:

- ✅ Simple API for common HTTP methods
- ✅ Automatic retries for network reliability  
- ✅ Flexible error handling
- ✅ Built-in debugging and logging
- ✅ Connection pooling and SSL options
- ✅ Integrates well with IoC container