# HttpClient
# Introduction
In web application development, making HTTP requests is an essential operation. Sosise provides an `HttpClient` class that simplifies the HTTP request process and provides more flexible control over the request process. The HttpClient provides a convenient API for fetching, posting, and sending multiple types of HTTP requests.

# Configuration
The HttpClient class utilizes an instance of Axios under the hood and can be configured using the `HttpClientConfig` interface.

```typescript
interface HttpClientConfig extends CreateAxiosDefaults {
    ignoreSelfSignedCertificates?: boolean;
}
```

# Creating an Instance
Creating an instance of HttpClient class:
```typescript
import HttpClient from "sosise-core/build/HttpClient/HttpClient";
import HttpClientConfig from "sosise-core/build/Types/HttpClientConfig";

const httpClientConfig: HttpClientConfig = {
    baseURL: 'https://api.example.com',
    headers: {'User-Agent': 'Custom User-Agent'}
};

const httpClient = new HttpClient(httpClientConfig);
```
In the above code, an instance of HttpClient is created with a custom base URL and a custom user-agent header.

# Making HTTP Requests
The HttpClient provides the `request` method to send HTTP requests.

## Syntax
```typescript
request(config: AxiosRequestConfig): Promise<AxiosResponse>
```
## Example
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
The `request` method accepts an `AxiosRequestConfig` object and returns a Promise that resolves to the response.

# Request with Retry
For scenarios where a network request might fail due to temporary issues (like network flakiness), the HttpClient provides the `requestWithRetry` method that automatically retries the request if it fails.

## Syntax
```typescript
requestWithRetry(config: AxiosRequestConfig, retryConfig: HttpClientRetryConfig): Promise<AxiosResponse>
```
## Example
```typescript
const retryConfig: HttpClientRetryConfig = {
    requestMaxRetries: 3,
    requestRetryInterval: 2000,
    requestRetryStrategy: 'linear'
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
In the above code, if the request fails, it will be retried linearly every 2 seconds, up to a maximum of 3 times. If all retries fail, the promise is rejected with the encountered error.

# HttpClientRetryConfig
This interface is used to configure retry behavior in the `requestWithRetry` method.
```typescript
type Milliseconds = number;

interface HttpClientRetryConfig {
    requestMaxRetries: number;
    requestRetryInterval: Milliseconds;
    requestRetryStrategy: 'linear' | 'exponential';
}
```
The `requestMaxRetries` property sets the maximum number of retry attempts. The `requestRetryInterval` sets the delay (in milliseconds) between retry attempts. The `requestRetryStrategy` property sets the strategy to use when calculating the delay between retries.

# Note
While the `HttpClient` class provides a higher-level API for making HTTP requests, it's important to remember that it's a wrapper around Axios. The methods and configurations available on Axios can also be used with `HttpClient`. For complex use-cases, consider using Axios directly.

# Using HTTP Proxy

HttpClient supports HTTP proxy settings to route your requests through a specified proxy server. This is useful for making requests from a specific geographical location or for debugging purposes. The proxy settings can be included in the HttpClient configuration object using the `proxy` property. 

Here is the `proxy` property definition:

```typescript
interface HttpClientConfig extends CreateAxiosDefaults {
    proxy?: {
        host: string;
        port: number;
        auth?: {
            username: string;
            password: string;
        };
    };
}
```

The `host` and `port` fields define the proxy server through which the requests will be routed. The `auth` field is optional and can be used if the proxy server requires authentication. It includes the `username` and `password` fields.

Here is an example of how to create a HttpClient with proxy settings:

```typescript
const config: HttpClientConfig = {
    baseURL: 'https://api.example.com',
    timeout: 5000,
    proxy: {
        host: '123.123.123.123',
        port: 8080,
        auth: {
            username: 'username',
            password: 'password',
        },
    },
};

const httpClient = new HttpClient(config);
```

In this example, all requests made by `httpClient` will be routed through the proxy server at IP address `123.123.123.123` and port `8080`. The `username` and `password` provided will be used to authenticate with the proxy server. 

# Using HTTPS Proxy

HttpClient also supports HTTPS proxy settings for making secure requests through a specified proxy server. This is ideal when your application requires enhanced security and encryption for its requests. HTTPS proxy is configured similarly to HTTP proxy, with the additional requirement of specifying the `protocol` property as `https`.

Here is the `proxy` property definition with the `protocol` field:

```typescript
interface HttpClientConfig extends CreateAxiosDefaults {
    proxy?: {
        protocol: string;
        host: string;
        port: number;
        auth?: {
            username: string;
            password: string;
        };
    };
}
```

The `protocol` field defines the protocol that will be used for communication with the proxy server. This field should be set to `https` for HTTPS proxies.

Here is an example of how to create a HttpClient with HTTPS proxy settings:

```typescript
const config: HttpClientConfig = {
    baseURL: 'https://api.example.com',
    timeout: 5000,
    proxy: {
        protocol: 'https',
        host: '123.123.123.123',
        port: 8080,
        auth: {
            username: 'username',
            password: 'password',
        },
    },
};

const httpClient = new HttpClient(config);
```

In this example, all requests made by `httpClient` will be routed through the HTTPS proxy server at IP address `123.123.123.123` and port `8080`. The `username` and `password` provided will be used to authenticate with the proxy server.

# Usage Examples
Please note that both `request` and `requestWithRetry` methods can be used.

## GET Request with Query Parameters

You can include query parameters in a GET request by using the `params` field in the config.

```typescript
const config = {
    method: 'get',
    url: '/endpoint',
    params: { key1: 'value1', key2: 'value2' },
};

const response = await httpClient.request(config);

// Equivalent to: GET /endpoint?key1=value1&key2=value2
```

## POST Request with Body

In a POST request, you can send data in the body of the request by using the `data` field in the config.

```typescript
const config = {
    method: 'post',
    url: '/endpoint',
    data: { key1: 'value1', key2: 'value2' },
};

const response = await httpClient.request(config);

// Equivalent to: POST /endpoint with body: { "key1": "value1", "key2": "value2" }
```

## PUT Request

A PUT request can be used to update a resource. Like a POST request, you can include data in the body of the request.

```typescript
const config = {
    method: 'put',
    url: '/endpoint',
    data: { key1: 'new-value1', key2: 'new-value2' },
};

const response = await httpClient.request(config);

// Equivalent to: PUT /endpoint with body: { "key1": "new-value1", "key2": "new-value2" }
```

## DELETE Request

A DELETE request can be used to delete a resource. It does not typically include a body.

```typescript
const config = {
    method: 'delete',
    url: '/endpoint',
};

const response = await httpClient.request(config);

// Equivalent to: DELETE /endpoint
```

## PATCH Request

A PATCH request can be used to partially update a resource. Like a POST and PUT request, you can include data in the body of the request.

```typescript
const config = {
    method: 'patch',
    url: '/endpoint',
    data: { key1: 'partially-new-value1' },
};

const response = await httpClient.request(config);

// Equivalent to: PATCH /endpoint with body: { "key1": "partially-new-value1" }
```