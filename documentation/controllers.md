# Controllers

## Introduction

Controllers offer a structured approach to handle request logic in your application. Instead of defining all request handling logic as closures in your route files, you can group related behavior into "controller" classes. A controller is responsible for handling specific types of requests and can encapsulate methods for showing, creating, updating, and deleting related resources. For instance, a `UserController` class could handle all incoming requests related to users.

Controllers in this documentation are assumed to be used in an Express-based web application with TypeScript. By default, controllers are stored in the `app/Http/Controllers` directory.

## Writing Controllers

### Creating a Controller using Artisan

To create a new controller, you can use the following Artisan command:

```sh
./artisan make:controller ExampleController
```

After creating the controller, remember to create a route that maps to your controller in `src/routes/api.ts`.

## Basic Controllers

Here's an example of a basic controller:

```typescript
import { Request, Response, NextFunction } from 'express';
import HttpResponse from 'sosise-core/build/Types/HttpResponse';

export default class ExampleController {
    /**
     * Example method
     */
    public async example(request: Request, response: Response, next: NextFunction) {
        try {
            // Prepare the HTTP response
            const httpResponse: HttpResponse = {
                code: 1000,
                message: 'Some example',
                data: null
            };

            // Send the response
            return response.send(httpResponse);
        } catch (error) {
            next(error);
        }
    }
}
```

To use this controller, you need to define a route in `src/routes/api.ts`, like so:

```typescript
const exampleController = new ExampleController();
router.get('/example', (request: Request, response: Response, next: NextFunction) => {
    exampleController.example(request, response, next);
});
```

When an incoming request matches the specified route URI, the `example` method on the `ExampleController` class will be invoked, and any route parameters will be passed to the method.

> Important: Ensure that any asynchronous operations in your controller methods are wrapped with a `try/catch` block, as shown in the example above. This is necessary to ensure proper exception handling and avoid unhandled rejections.

By using controllers, you can organize your request handling logic more efficiently, making your codebase cleaner and more maintainable.