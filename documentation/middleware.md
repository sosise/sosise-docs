# Middleware
## Introduction
Middleware provide a convenient mechanism for inspecting and filtering HTTP requests entering your application. Middleware can be written to perform a variety of tasks besides authentication. For example, a logging middleware might log all incoming requests to your application. All middlewares are located in the `app/Http/Middlewares` directory.

## Defining Middleware
To create a new middleware, use the `make:middleware` Artisan command:

```sh
./artisan make:middleware EnsureTokenIsValidMiddleware
```

> Please register created middleware in `app/Http/Middlewares/Kernel.ts`, otherwise it will not participate in the request.

This command will place a new `EnsureTokenIsValidMiddleware` class within your `app/Http/Middlewares` directory. In this middleware, we will only proceed if the supplied token header matches a specified value. Otherwise, we will abort with an error. Take a look at example below:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class EnsureTokenIsValidMiddleware {
    /**
     * This method handles the middleware
     */
    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        
        if (request.header('X-My-Token-Header') !== 'my-secret-token') {
            return response.status(403).json({
                message: 'Please provide correct X-My-Token-Header'
            });
        }

        next();
    }
}
```

As you can see, if the given `token header` does not match our secret token, the middleware will return an HTTP code 403 to the client; otherwise, the request will be passed further into the application. To pass the request deeper into the application (allowing the middleware to "pass"), you should call the `next()` callback.

It's best to envision middleware as a series of "layers" HTTP requests must pass through before they hit your application. Each layer can examine the request and even reject it entirely.

### Middleware & Responses
Of course, a middleware can perform tasks before or after passing the request deeper into the application. For example, the following middleware would perform some task `before` the request is handled by the application:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class BeforeMiddleware {
    /**
     * This method handles the middleware
     */
    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        // Perform action

        next();
    }
}
```

However, this middleware would perform its task `after` the request is handled by the application:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class AfterMiddleware {
    /**
     * This method handles the middleware
     */
    public async handle(request: Request, response: Response, next: NextFunction): Promise<any> {
        next();

        // Perform action
    }
}
```

### How to get response body in the middleware
Sometimes you may need to get `response body` in the middleware. Take a look at example below:

```typescript
import { Request, Response, NextFunction } from 'express';

export default class ExampleMiddleware {
    /**
     * This method handles the middleware
     */
    public handle(request: Request, response: Response, next: NextFunction): void {
        // Patch the response send method to get body out of that
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

> This is a little bit tricky, in this example we are patching the `response.send()` method with additional code

## Registering Middleware
### Global Middleware
If you want a middleware to run during every HTTP request to your application, list the middleware class in the `middlewares` property of your `app/Http/Middlewares/Kernel.ts` class.

## Using middlewares in routes
If you want to use middleware in a specific route, create a new middleware, instantiate it in `src/routes/api.ts` and use in particular route, let's take a loot at example below:

```typescript
import express from 'express';
import { Request, Response, NextFunction } from 'express';
import IndexController from '../app/Http/Controllers/IndexController';
import ExampleMiddleware from '../app/Http/Middlewares/ExampleMiddleware';
const router = express.Router();

// Instantiate example middleware
const exampleMiddleware = new ExampleMiddleware();

// IndexController
router.get('/', exampleMiddleware.handle, (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});

export default router;
```

> As you can see we use the handle method as a second argument in a route.
> This approach allows you to use specific `middlewares` in needed routes, for example if you do not want to use a global middleware.

> Also note that using middleware directly in a route, allows you to get `request.route` in this particular middleware, which would be inaccessable otherwise.
