# Throttling

## Introduction

Throttling is an essential mechanism in web application development that helps manage traffic to individual resources, ensuring fairness, preventing abuse, and optimizing resource allocation. The `ThrottlingConfig` allows you to configure request throttling for your HTTP client.

## Configuration

The `ThrottlingConfig` can be set up in `src/config/throttling.ts`:

```ts
const throttlingConfig = {
    isEnabled: true,
    clientIpHeader: 'X-Forwarded-For',
    skipSubnets: ['10.0.0.0/24'],
    routeRules: [
        {
            httpMethod: 'GET',
            path: '/customer/get-all-customers',
            maxRequestsPerMinute: 5,
        },
        {
            httpMethod: 'GET',
            path: '/customer/:id',
            maxRequestsPerMinute: 10,
        },
    ],
};
```

### Configuration Options

* **isEnabled**: Set this to `true` to enable request throttling. The default value is `false`.
* **clientIpHeader**: The HTTP header used to determine the client's IP address. Note that this header must be set by a trusted proxy or load balancer to avoid security risks.
* **skipSubnets**: An array of CIDR subnets that should be exempted from throttling, allowing requests from these subnets to bypass restrictions.
* **routeRules**: An array of specific rules that define throttling behavior based on HTTP methods and URL paths.

> **Important:** The order of the `routeRules` array is significant.
> When you have multiple rules for similar routes, **always define more specific routes (without parameters) before general ones (with parameters)**.
>
> For example:
>
> ```ts
> // Correct order
> { path: '/customer/get-all-customers', ... }
> { path: '/customer/:id', ... }
> ```
>
> If the order is reversed, the parameterized route (`/customer/:id`) may match requests intended for the more specific path, potentially applying the wrong throttling rule.

### Route Rules

Each rule within the `routeRules` array specifies a particular HTTP method and URL path combination along with the maximum number of requests allowed within a rolling minute.

#### Example

```ts
{
    httpMethod: 'GET',
    path: '/customer/get-all-customers',
    maxRequestsPerMinute: 5,
}
```

Another example:

```ts
{
    httpMethod: 'GET',
    path: '/customer/:id',
    maxRequestsPerMinute: 10,
}
```

By implementing throttling configurations, applications can better manage usage patterns, reduce server load, and enhance user experience.