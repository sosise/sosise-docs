# Routing
The routing comes from `expressjs` which sosise is based on. If you need more information about routing please read the full documentation at https://expressjs.com/en/guide/routing.html

## Basic Routing
The most basic Sosise routes accept a URI and a closure, providing a very simple and expressive method of defining routes and behavior without complicated routing configuration files:

```typescript
router.get('/', (request: Request, response: Response, next: NextFunction) => {
    response.send('Hello World!');
});
```

## The Default Route File
All Sosise routes are defined in your route file, which is located in the `src/routes` directory. This file is automatically loaded by the application.

## Available Router Methods
### The router allows you to register routes that respond to any HTTP verb:

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

Sometimes you may need to register a route that responds to multiple HTTP verbs. You may do so using the match method. Or, you may even register a route that responds to all HTTP verbs using the any method:

```typescript
router.all('/', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

## Route paths
Route paths, in combination with a request method, define the endpoints at which requests can be made. Route paths can be strings, string patterns, or regular expressions.
The characters `?`, `+`, `*`, and `()` are subsets of their regular expression counterparts. The hyphen (-) and the dot (.) are interpreted literally by string-based paths.
If you need to use the dollar character ($) in a path string, enclose it escaped within ([ and ]). For example, the path string for requests at `"/data/$book"`, would be `"/data/([\$])book"`.

If it seems to be complicated for you, please see some examples below.

## Example routes
### /
```typescript
router.get('/', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### /about
```typescript
router.get('/about', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### /random.text
```typescript
router.get('/random.text', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### This route path will match `acd` and `abcd`.
```typescript
router.get('/ab?cd', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### This route path will match `abcd`, `abbcd`, `abbbcd`, and so on.
```typescript
router.get('/ab+cd', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### This route path will match `abcd`, `abxcd`, `abRANDOMcd`, `ab123cd`, and so on.
```typescript
router.get('/ab*cd', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### This route path will match `abe` and `abcde`.
```typescript
router.get('/ab(cd)?e', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### This route path will match anything with an `"a"` in it.
```typescript
router.get(/a/, (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### This route path will match `butterfly` and `dragonfly`, but not `butterflyman`, `dragonflyman`, and so on.
```typescript
router.get(/.*fly$/, (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

> Please note, that `expressjs` uses path-to-regexp for matching the route paths; see the path-to-regexp documentation for all the possibilities in defining route paths. Express Route Tester is a handy tool for testing basic Express routes, although it does not support pattern matching.

> Please note, Query strings are not part of the route path.

## Route parameters
Route parameters are named URL segments that are used to capture the values specified at their position in the URL. The captured values are populated in the req.params object, with the name of the route parameter specified in the path as their respective keys. You can read more at https://expressjs.com/en/guide/routing.html (section route parameters)

```text
Route path: /users/:userId/books/:bookId
Request URL: http://localhost:10000/users/34/books/8989
req.params: { "userId": "34", "bookId": "8989" }
```

To define routes with route parameters, simply specify the route parameters in the path of the route as shown below.

```typescript
router.get('/users/:userId/books/:bookId', (request: Request, response: Response, next: NextFunction) => {
    new IndexController().index(request, response, next);
});
```

### How to get the route params
```typescript
import { Request, Response, NextFunction } from 'express';
import HttpResponse from 'sosise-core/build/Types/HttpResponse';

export default class IndexController {
    /**
     * Example method
     */
    public async index(request: Request, response: Response, next: NextFunction) {
        try {
            console.log(request.params);
        } catch (error) {
            next(error);
        }
    }
}
```

## Response methods
The methods on the response object (response) in the following table can send a response to the client, and terminate the request-response cycle. If none of these methods are called from a route handler, the client request will be left hanging.

|Method|Description|
|-|-|
|res.download()|Prompt a file to be downloaded.|
|res.end()|End the response process.|
|res.json()|Send a JSON response.|
|res.jsonp()|Send a JSON response with JSONP support.|
|res.redirect()|Redirect a request.|
|res.render()|Render a view template.|
|res.send()|Send a response of various types.|
|res.sendFile()|Send a file as an octet stream.|
|res.sendStatus()|Set the response status code and send its string representation as the response body.|

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
