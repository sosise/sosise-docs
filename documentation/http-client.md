# HttpClient

## Introduction

In web application development, making HTTP requests is an essential operation. Sosise provides an `HttpClient` class that simplifies the HTTP request process and provides more flexible control over the request process. The `HttpClient` provides a convenient API for fetching, posting, and sending multiple types of HTTP requests.

## Configuration

The `HttpClient` class utilizes an instance of Axios under the hood and can be configured using the `HttpClientConfig` interface.

```typescript
interface HttpClientConfig extends CreateAxiosDefaults {
    ignoreSelfSignedCertificates?: boolean;
}
```

## Creating an Instance

Creating an instance of the `HttpClient` class:

```typescript
import HttpClient from "sosise-core/build/HttpClient/HttpClient";
import HttpClientConfig from "sosise-core/build/Types/HttpClientConfig";

const httpClientConfig: HttpClientConfig = {
    baseURL: 'https://api.example.com',
    headers: {'User-Agent': 'Custom User-Agent'}
};

const httpClient = new HttpClient(httpClientConfig);
```

In the above code, an instance of `HttpClient` is created with a custom base URL and a custom user-agent header.

## Making HTTP Requests

The `HttpClient` provides the `request` method to send HTTP requests.

### Syntax

```typescript
request(config: HttpClientRequestConfig): Promise<AxiosResponse>
```

### Example

```typescript
try {
    const response = await httpClient.request({
        url: '/users',
        method: 'get',
        params: {
            id: 13,
            name: 'Max',
        }
    });
    
    console.log(response.data);
} catch (error) {
    console.error(error);
}
```

The `request` method accepts an `HttpClientRequestConfig` object and returns a Promise that resolves to the response.

## Request with Retry

For scenarios where a network request might fail due to temporary issues (like network flakiness), the `HttpClient` provides the `requestWithRetry` method that automatically retries the request if it fails.

### Syntax

```typescript
requestWithRetry(config: HttpClientRequestConfig, retryConfig: HttpClientRetryConfig): Promise<AxiosResponse>
```

### Example

```typescript
const retryConfig: HttpClientRetryConfig = {
    requestMaxRetries: 3,
    requestRetryInterval: 2000,
    requestRetryStrategy: 'linear',
    requestDoNotRetryForHttpCodes: [400, 401, 403]
};

try {
    const response = await httpClient.requestWithRetry({
        url: '/users',
        method: 'get'
    }, retryConfig);
    
    console.log(response.data);
} catch (error) {
    console.error(error);
}
```

In this example, the request is retried up to 3 times in a linear interval of 2 seconds. However, if the response contains HTTP status codes 400, 401, or 403, the request will not be retried and will fail immediately.

## HttpClientRequestConfig

This interface is used to configure the request in the `request` and `requestWithRetry` methods.

```typescript
interface HttpClientRequestConfig extends AxiosRequestConfig {
    returnInCaseOfStatusCodes?: {
        [key: number]: any;
    };
    defaultException?: new (message: string, httpCode: number, data: any) => Exception;
}
```

The `returnInCaseOfStatusCodes` specifies the behavior for what should be returned in case of specific HTTP status codes.

The `defaultException` specifies which exception should be thrown in case of an error.

### Example

```typescript
const response = await httpClient.request({
    url: '/users',
    method: 'get',
    params: {
        id: 13,
        name: 'Max',
    },
    // In case of a 404 error, null will be returned as response.data
    returnInCaseOfStatusCodes: {
        404: null
    },
    // In case of any other exception CartApiException will be thrown
    defaultException: CartApiException,
});
```

## HttpClientRetryConfig

This interface is used to configure retry behavior in the `requestWithRetry` method.

```typescript
type Milliseconds = number;

interface HttpClientRetryConfig {
    requestMaxRetries: number;
    requestRetryInterval: Milliseconds;
    requestRetryStrategy: 'linear' | 'exponential';
    requestDoNotRetryForHttpCodes?: number[];
}
```

- `requestMaxRetries`: Sets the maximum number of retry attempts.
- `requestRetryInterval`: The delay (in milliseconds) between retry attempts.
- `requestRetryStrategy`: The strategy for calculating the delay between retries (`linear` or `exponential`).
- `requestDoNotRetryForHttpCodes`: A list of HTTP status codes for which requests should not be retried.

## Usage Examples

### GET Request with Query Parameters

```typescript
const config = {
    method: 'get',
    url: '/endpoint',
    params: { key1: 'value1', key2: 'value2' },
};

const response = await httpClient.request(config);

// Equivalent to: GET /endpoint?key1=value1&key2=value2
```

### POST Request with Body

```typescript
const config = {
    method: 'post',
    url: '/endpoint',
    data: { key1: 'value1', key2: 'value2' },
};

const response = await httpClient.request(config);

// Equivalent to: POST /endpoint with body: { "key1": "value1", "key2": "value2" }
```

### PUT Request

```typescript
const config = {
    method: 'put',
    url: '/endpoint',
    data: { key1: 'new-value1', key2: 'new-value2' },
};

const response = await httpClient.request(config);
```

### DELETE Request

```typescript
const config = {
    method: 'delete',
    url: '/endpoint',
};

const response = await httpClient.request(config);
```

### PATCH Request

```typescript
const config = {
    method: 'patch',
    url: '/endpoint',
    data: { key1: 'partially-new-value1' },
};

const response = await httpClient.request(config);
```