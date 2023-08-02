# Middleware

## Introduction

Middleware provides a convenient mechanism for inspecting and filtering HTTP requests entering your application. It allows you to perform various tasks besides authentication, such as logging incoming requests or handling specific headers. All middlewares are located in the `app/Http/Middlewares` directory.

## Defining Middleware

To create a new middleware, you can use the `make:middleware` Artisan command:

```sh
./artisan make:middleware EnsureTokenIsValidMiddleware
```

> Don't forget to register the created middleware in `app/Http/Middlewares/Kernel.ts`, otherwise, it will not participate in the request.

This command will generate a new `EnsureTokenIsValidMiddleware` class within your `app/Http/Middlewares` directory. In this middleware, we check if the supplied token header matches a specified value. If not, we return a 403 HTTP status code with an error message. Otherwise, the request will be allowed to proceed further into the application.

```typescript
import { Request, Response, NextFunction } from 'express';

export default class EnsureTokenIsValidMiddleware {
    /**
     * This method handles the middleware
     */
    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        if (request.header('X-My-Token-Header') !== 'my-secret-token') {
            return response.status(403).json({
                message: 'Please provide the correct X-My-Token-Header'
            });
        }

        next();
    }
}
```

### Middleware & Responses

Middleware can perform tasks before or after passing the request further into the application. For example, the following middleware performs a task `before` the request is handled:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class BeforeMiddleware {
    /**
     * This method handles the middleware
     */
    public async handle(request: Request, response: Response, next: NextFunction): Promise<void> {
        // Perform action before handling the request

        next();
    }
}
```

However, this middleware performs its task `after` the request is handled:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class AfterMiddleware {
    /**
     * This method handles the middleware
     */
    public async handle(request: Request, response: Response, next: NextFunction): Promise<void> {
        next();

        // Perform action after handling the request
    }
}
```

## How to Get Response Body in the Middleware

Sometimes you may need to get the `response body` in the middleware. Here's an example of how to achieve this:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class ExampleMiddleware {
    /**
     * This method handles the middleware
     */
    public handle(request: Request, response: Response, next: NextFunction): void {
        // Patch the response send method to get the body
        const originalSend = response.send;
        response.send = function () {
            const responseBody = arguments[0];
            if (typeof responseBody === 'string') {
                console.log(responseBody);
            }
            return originalSend.apply(response, arguments);
        };

        next();
    }
}
```

> This example shows how to patch the `response.send()` method with additional code to access the response body.

## Registering Middleware

### Global Middleware

If you want a middleware to run during every HTTP request to your application, you can list the middleware class in the `middlewares` property of your `app/Http/Middlewares/Kernel.ts` class.