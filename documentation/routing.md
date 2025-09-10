# Routing

## Quick Start

Routing defines how your application responds to client requests. In Sosise, you define routes in the `src/routes` directory using simple, readable syntax:

```typescript
// Basic GET route
router.get('/', (request: Request, response: Response, next: NextFunction) => {
    response.send('Hello World!');
});

// Route with a controller
const userController = new UserController();
router.get('/users', (request: Request, response: Response, next: NextFunction) => {
    userController.getAllUsers(request, response, next);
});
```

## Introduction

Sosise routing is based on Express.js, providing a familiar and powerful way to handle HTTP requests. Routes are defined in files within the `src/routes` directory and are automatically loaded by the application.

## Common HTTP Methods

Most web applications use these four main HTTP methods:

```typescript
const userController = new UserController();

// GET - Retrieve data
router.get('/users', (request: Request, response: Response, next: NextFunction) => {
    userController.getUsers(request, response, next);
});

// POST - Create new data
router.post('/users', (request: Request, response: Response, next: NextFunction) => {
    userController.createUser(request, response, next);
});

// PUT - Update existing data
router.put('/users/:id', (request: Request, response: Response, next: NextFunction) => {
    userController.updateUser(request, response, next);
});

// DELETE - Remove data
router.delete('/users/:id', (request: Request, response: Response, next: NextFunction) => {
    userController.deleteUser(request, response, next);
});
```

## Route Parameters

Capture dynamic values from the URL using route parameters (marked with `:`):

```typescript
// Single parameter
router.get('/users/:id', (request: Request, response: Response, next: NextFunction) => {
    const userId = request.params.id; // "123" from /users/123
    response.send(`User ID: ${userId}`);
});

// Multiple parameters
router.get('/users/:userId/posts/:postId', (request: Request, response: Response, next: NextFunction) => {
    const { userId, postId } = request.params;
    // From /users/123/posts/456: userId = "123", postId = "456"
});
```

## Accessing Request Data

### Query Parameters (URL parameters after `?`)

```typescript
// GET /search?q=hello&limit=10
router.get('/search', (request: Request, response: Response, next: NextFunction) => {
    const query = request.query.q;        // "hello"
    const limit = request.query.limit;    // "10"
});
```

### Request Body (POST, PUT data)

```typescript
// POST /users with JSON body
router.post('/users', (request: Request, response: Response, next: NextFunction) => {
    const { name, email } = request.body;
    // Handle user creation
});
```

## Response Methods

Send responses back to the client using these common methods:

```typescript
router.get('/api/users', (request: Request, response: Response, next: NextFunction) => {
    // Send JSON data (most common for APIs)
    response.json({ users: [], total: 0 });
    
    // Send plain text or HTML
    response.send('Hello World');
    
    // Redirect to another URL
    response.redirect('/login');
    
    // Send with specific status code
    response.status(404).send('Not Found');
});
```

## Using Middleware in Routes

Apply middleware to specific routes for authentication, validation, etc.:

```typescript
import AuthMiddleware from '../app/Http/Middlewares/AuthMiddleware';

const authMiddleware = new AuthMiddleware();
const userController = new UserController();

// Protected route with middleware
router.get('/profile', authMiddleware.handle, (request: Request, response: Response, next: NextFunction) => {
    userController.getProfile(request, response, next);
});
```

## Advanced Routing

### Route Patterns

For more complex URL matching:

```typescript
// Optional segments with ?
router.get('/posts/:year/:month?', handler); // /posts/2024 or /posts/2024/12

// Wildcards with *
router.get('/files/*', handler); // Matches /files/docs/readme.txt
```

### All HTTP Methods

```typescript
// Handle any HTTP method
router.all('/api/*', (request: Request, response: Response, next: NextFunction) => {
    // This runs for GET, POST, PUT, DELETE, etc.
});
```

### Available HTTP Methods

While most applications only need GET, POST, PUT, and DELETE, Sosise supports all Express.js methods including: `checkout`, `copy`, `head`, `lock`, `merge`, `mkactivity`, `mkcol`, `move`, `m-search`, `notify`, `options`, `patch`, `purge`, `report`, `search`, `subscribe`, `trace`, `unlock`, `unsubscribe`.

### Complex Pattern Matching

For advanced use cases, you can use regular expressions:

```typescript
// Matches anything ending with 'fly'
router.get(/.*fly$/, handler);

// Matches specific patterns
router.get(/ab(cd)?e/, handler); // Matches 'abe' and 'abcde'
```

> **Note**: For complex routing patterns, refer to the [Express.js routing documentation](https://expressjs.com/en/guide/routing.html) and [path-to-regexp documentation](https://www.npmjs.com/package/path-to-regexp).

## Best Practices

1. **Use controllers** instead of inline functions for complex logic
2. **Group related routes** in separate route files
3. **Apply middleware** consistently for authentication and validation
4. **Use descriptive route names** that clearly indicate their purpose
5. **Handle errors** with try-catch blocks in your route handlers