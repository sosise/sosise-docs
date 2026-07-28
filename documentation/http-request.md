# HTTP Request

Sosise controllers receive the standard Express `Request` object.

```typescript
import { NextFunction, Request, Response } from 'express';

router.post('/users/:id', (request: Request, response: Response, next: NextFunction) => {
    const userId = request.params.id;
    const dryRun = request.query.dryRun === 'true';
    const { name, email } = request.body;

    response.send({ userId, dryRun, name, email });
});
```

## Request Data

| Property | Source |
|---|---|
| `request.params` | Route parameters such as `:id` |
| `request.query` | URL query string values |
| `request.body` | Parsed request body |
| `request.headers` | HTTP request headers |
| `request.ip` | Client IP resolved by Express |

Treat all request data as untrusted input and validate it before passing it to services or repositories.

## Parsed Bodies

Sosise parses JSON and URL-encoded bodies before route handlers and dynamic middleware. Clients must send the matching `Content-Type` header:

```bash
curl http://localhost:10000/users \
  -H 'Content-Type: application/json' \
  -d '{"name":"Alice","email":"alice@example.com"}'
```

The global `HTTP_JSON_BODY_LIMIT` setting applies to both JSON and URL-encoded bodies. It does not control multipart file uploads. See [HTTP Server](http-server.md) for body-limit syntax, timeout settings, reverse proxy alignment, and oversized-body handling.

## File Uploads

Use dedicated multipart middleware for uploads. Do not increase the JSON body limit to accommodate files. Prefer streamed processing or direct object-storage uploads for large payloads.
