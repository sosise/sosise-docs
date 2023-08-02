# Routing

The routing in Sosise is based on `expressjs`. If you need more information about routing, please read the full documentation at [https://expressjs.com/en/guide/routing.html](https://expressjs.com/en/guide/routing.html).

## Basic Routing

The most basic Sosise routes accept a URI and a closure, providing a simple and expressive way of defining routes and behavior without complicated routing configuration files:

```typescript
router.get('/', (request: Request, response: Response, next: NextFunction) => {
    response.send('Hello World!');
});
```

## The Default Route File

All Sosise routes are defined in your route file, which is located in the `src/routes` directory. This file is automatically loaded by the application.

## Available Router Methods

The router allows you to register routes that respond to any HTTP verb:

- `checkout`
- `copy`
- `delete`
- `get`
- `head`
- `lock`
- `merge`
- `mkactivity`
- `mkcol`
- `move`
- `m-search`
- `notify`
- `options`
- `patch`
- `post`
- `purge`
- `put`
- `report`
- `search`
- `subscribe`
- `trace`
- `unlock`
- `unsubscribe`

You can register a route that responds to multiple HTTP verbs using the `match` method. Alternatively, you can register a route that responds to all HTTP verbs using the `any` method:

```typescript
const indexController = new IndexController();
router.all('/', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});
```

## Route Paths

Route paths, in combination with a request method, define the endpoints at which requests can be made. Route paths can be strings, string patterns, or regular expressions. The characters `?`, `+`, `*`, and `()` are subsets of their regular expression counterparts. The hyphen (`-`) and the dot (`.`) are interpreted literally by string-based paths.

If you need to use the dollar character (`$`) in a path string, enclose it escaped within `([ and ])`. For example, the path string for requests at `/data/$book` would be `/data/([\$])book`.

### Examples

```typescript
const indexController = new IndexController();

// Matches the root path
router.get('/', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

// Matches the path '/about'
router.get('/about', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

// Matches the path '/random.text'
router.get('/random.text', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

// Matches 'acd' and 'abcd'
router.get('/ab?cd', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

// Matches 'abcd', 'abbcd', 'abbbcd', and so on
router.get('/ab+cd', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

// Matches 'abcd', 'abxcd', 'abRANDOMcd', 'ab123cd', and so on
router.get('/ab*cd', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

// Matches 'abe' and 'abcde'
router.get('/ab(cd)?e', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

// Matches anything with an "a" in it
router.get(/a/, (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

// Matches 'butterfly' and 'dragonfly', but not 'butterflyman', 'dragonflyman', and so on
router.get(/.*fly$/, (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});
```

Please note that `expressjs` uses path-to-regexp for matching route paths; refer to the [path-to-regexp documentation](https://www.npmjs.com/package/path-to-regexp) for all the possibilities in defining route paths. The [Express Route Tester](https://express-route-tester.whistl.com/) is a handy tool for testing basic Express routes, although it does not support pattern matching.

Please note that query strings are not part of the route path.

## Route Parameters

Route parameters are named URL segments used to capture the values specified at their position in the URL. The captured values are populated in the `req.params` object, with the name of the route parameter specified in the path as their respective keys.

```text
Route path: /users/:userId/books/:bookId
Request URL: http://localhost:10000/users/34/books/8989
req.params: { "userId": "34", "bookId": "8989" }
```

To define routes with route parameters, simply specify the route parameters in the path of the route.

### Example

```typescript
const indexController = new IndexController();

// Matches '/users/:userId/books/:bookId'
router.get('/users/:userId/books/:bookId', (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});
```

To access the route parameters in the controller, use `request.params`.

### Example in Controller

```typescript
import { Request, Response, NextFunction } from 'express';

export default class IndexController {
    public async index(request: Request, response:

 Response, next: NextFunction) {
        try {
            console.log(request.params);
        } catch (error) {
            next(error);
        }
    }
}
```

## Query Parameters

To access query parameters, use `request.query`.

### Example in Controller

```typescript
import { Request, Response, NextFunction } from 'express';

export default class IndexController {
    public async index(request: Request, response: Response, next: NextFunction) {
        try {
            console.log(request.query);
        } catch (error) {
            next(error);
        }
    }
}
```

## POST, PUT, etc. Parameters

To access parameters from POST, PUT, etc. requests, use `request.body`.

### Example in Controller

```typescript
import { Request, Response, NextFunction } from 'express';

export default class IndexController {
    public async index(request: Request, response: Response, next: NextFunction) {
        try {
            console.log(request.body);
        } catch (error) {
            next(error);
        }
    }
}
```

## Response Methods

The methods on the response object (`response`) can send a response to the client and terminate the request-response cycle. If none of these methods are called from a route handler, the client request will be left hanging.

Available response methods:

- `res.download()`: Prompt a file to be downloaded.
- `res.end()`: End the response process.
- `res.json()`: Send a JSON response.
- `res.jsonp()`: Send a JSON response with JSONP support.
- `res.redirect()`: Redirect a request.
- `res.render()`: Render a view template.
- `res.send()`: Send a response of various types.
- `res.sendFile()`: Send a file as an octet stream.
- `res.sendStatus()`: Set the response status code and send its string representation as the response body.

## Using Middlewares in Routes

If you want to use middleware in a specific route, create a new middleware, instantiate it in `src/routes/api.ts`, and use it in a particular route. This approach allows you to use specific middlewares in needed routes, without applying them globally.

### Example

```typescript
import express from 'express';
import { Request, Response, NextFunction } from 'express';
import IndexController from '../app/Http/Controllers/IndexController';
import ExampleMiddleware from '../app/Http/Middlewares/ExampleMiddleware';
const router = express.Router();

// Instantiate example middleware
const exampleMiddleware = new ExampleMiddleware();

// IndexController
const indexController = new IndexController();
router.get('/', exampleMiddleware.handle, (request: Request, response: Response, next: NextFunction) => {
    indexController.index(request, response, next);
});

export default router;
```

In this example, we use the `handle` method as the second argument in a route. This approach allows you to use specific middlewares in needed routes. Additionally, using middleware directly in a route allows you to access `request.route` in this particular middleware, which would be inaccessible otherwise.