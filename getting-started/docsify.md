# Project Documentation with Docsify

## Quick Start

Document your API like a pro! Sosise comes with built-in documentation powered by Docsify - beautiful, searchable docs that live alongside your code.

```bash
# Start your Sosise app
npm run serve

# Visit your documentation
open http://localhost:10000/docs
```

Your documentation is automatically available at `/docs` with authentication protection! 📚

## Why Built-in Documentation?

Great APIs deserve great documentation:

- ✅ **Lives with Your Code** - Documentation stays in sync with development
- ✅ **Zero Configuration** - Ready to use out of the box
- ✅ **Beautiful UI** - Clean, searchable, mobile-friendly interface
- ✅ **Markdown Powered** - Write docs in familiar Markdown syntax
- ✅ **Protected Access** - Basic authentication keeps sensitive docs secure
- ✅ **Version Control** - Documentation versioned with your code

## Configuration

### Documentation Settings

Configure documentation in `src/config/documentation.ts`:

```typescript
// src/config/documentation.ts
export default {
    // Enable or disable documentation route
    enabled: process.env.DOCUMENTATION_ENABLED === 'true',
    
    // Basic authentication settings
    basicAuth: {
        enabled: process.env.DOCUMENTATION_BASIC_AUTH_ENABLED === 'true',
        username: process.env.DOCUMENTATION_BASIC_AUTH_USERNAME || 'admin',
        password: process.env.DOCUMENTATION_BASIC_AUTH_PASSWORD || 'password'
    },
    
    // Documentation route path
    path: '/docs',
    
    // Docsify configuration
    docsify: {
        name: 'My API Documentation',
        repo: 'https://github.com/mycompany/my-api',
        loadSidebar: true,
        subMaxLevel: 3,
        auto2top: true,
        search: {
            maxAge: 86400000, // 1 day
            paths: 'auto',
            placeholder: 'Search documentation...'
        }
    }
};
```

### Environment Configuration

```bash
# .env
# Enable documentation
DOCUMENTATION_ENABLED=true

# Basic authentication (recommended for production)
DOCUMENTATION_BASIC_AUTH_ENABLED=true
DOCUMENTATION_BASIC_AUTH_USERNAME=your-username
DOCUMENTATION_BASIC_AUTH_PASSWORD=your-secure-password
```

### Route Configuration

Documentation routes are defined in `src/routes/api.ts`:

```typescript
// src/routes/api.ts
import documentationConfig from '../config/documentation';
import { DocumentationBasicAuthMiddleware } from 'sosise-core';

export default function apiRoutes(router: Router): void {
    // Other routes...
    
    // Documentation route with optional basic auth
    if (documentationConfig.enabled) {
        const middlewares = [];
        
        if (documentationConfig.basicAuth.enabled) {
            middlewares.push(
                new DocumentationBasicAuthMiddleware(
                    documentationConfig.basicAuth.username,
                    documentationConfig.basicAuth.password
                )
            );
        }
        
        router.get('/docs/*', middlewares, 'DocumentationController@serve');
    }
}
```

## Documentation Structure

Your documentation lives in the `docs/` directory:

```
docs/
├── index.html          # Main Docsify configuration
├── README.md          # Homepage content
├── _sidebar.md        # Navigation sidebar
├── _navbar.md         # Top navigation (optional)
├── api/               # API documentation
│   ├── authentication.md
│   ├── users.md
│   └── orders.md
├── guides/            # User guides
│   ├── getting-started.md
│   └── deployment.md
└── assets/            # Images and other assets
    └── logo.png
```

## Writing Documentation

### Homepage (README.md)

```markdown
# My Awesome API

> A powerful REST API built with Sosise framework

## Features

- 🚀 Fast and scalable TypeScript API
- 🔐 JWT authentication
- 📊 Built-in analytics
- 🔄 Real-time updates
- 📱 Mobile-friendly responses

## Quick Start

```bash
# Install dependencies
npm install

# Start development server
npm run serve

# Your API is now running at http://localhost:10000
```

## Authentication

All API endpoints require authentication via JWT tokens.
See [Authentication Guide](api/authentication.md) for details.
```

### Sidebar Navigation (_sidebar.md)

```markdown
<!-- docs/_sidebar.md -->
- Getting Started
  - [Quick Start](/)
  - [Installation](guides/installation.md)
  - [Configuration](guides/configuration.md)

- API Reference
  - [Authentication](api/authentication.md)
  - [Users](api/users.md)
  - [Orders](api/orders.md)
  - [Payments](api/payments.md)

- Guides
  - [Error Handling](guides/error-handling.md)
  - [Rate Limiting](guides/rate-limiting.md)
  - [Webhooks](guides/webhooks.md)
  - [Deployment](guides/deployment.md)

- [Changelog](changelog.md)
```

### API Documentation Example

```markdown
<!-- docs/api/users.md -->
# Users API

## Get All Users

Retrieve a paginated list of users.

**Endpoint:** `GET /api/users`

**Authentication:** Required

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `limit` | integer | No | Items per page (default: 20) |
| `search` | string | No | Search by name or email |
| `role` | string | No | Filter by user role |

**Example Request:**

```bash
curl -X GET "http://localhost:10000/api/users?page=1&limit=10" \
  -H "Authorization: Bearer your-jwt-token"
```

**Example Response:**

```json
{
  "code": 1000,
  "message": "Users retrieved successfully",
  "data": {
    "users": [
      {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com",
        "role": "admin",
        "created_at": "2024-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "current_page": 1,
      "total_pages": 5,
      "total_items": 100,
      "per_page": 20
    }
  }
}
```

**Error Responses:**

| Code | Message | Description |
|------|---------|-------------|
| 2001 | Authentication required | Missing or invalid JWT token |
| 2003 | Insufficient permissions | User lacks required permissions |
| 5000 | Internal server error | Unexpected server error |

## Create User

Create a new user account.

**Endpoint:** `POST /api/users`

**Authentication:** Required (Admin only)

**Request Body:**

```json
{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "password": "securePassword123",
  "role": "user"
}
```

**Validation Rules:**

- `name`: Required, string, 2-100 characters
- `email`: Required, valid email, unique
- `password`: Required, string, minimum 8 characters
- `role`: Optional, enum: `admin|user` (default: `user`)

**Example Response:**

```json
{
  "code": 1000,
  "message": "User created successfully",
  "data": {
    "user": {
      "id": 2,
      "name": "Jane Doe",
      "email": "jane@example.com",
      "role": "user",
      "created_at": "2024-01-02T00:00:00Z"
    }
  }
}
```
```

## Advanced Docsify Configuration

### Custom Index.html

```html
<!-- docs/index.html -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>My API Documentation</title>
  <meta name="description" content="Complete API documentation for My Awesome API">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="icon" href="assets/favicon.ico">
  <link rel="stylesheet" href="//cdn.jsdelivr.net/npm/docsify@4/lib/themes/vue.css">
  <style>
    .app-name-link {
      color: #42b983 !important;
    }
    .sidebar-nav strong {
      color: #42b983;
    }
  </style>
</head>
<body>
  <div id="app"></div>
  <script>
    window.$docsify = {
      name: 'My API Documentation',
      repo: 'https://github.com/mycompany/my-api',
      homepage: 'README.md',
      loadSidebar: true,
      loadNavbar: true,
      subMaxLevel: 4,
      auto2top: true,
      maxLevel: 4,
      
      // Search plugin
      search: {
        maxAge: 86400000, // 1 day
        paths: 'auto',
        placeholder: 'Search documentation...',
        noData: 'No results found.',
        depth: 6
      },
      
      // Copy code plugin
      copyCode: {
        buttonText: 'Copy',
        errorText: 'Error',
        successText: 'Copied!'
      },
      
      // Pagination
      pagination: {
        previousText: 'Previous',
        nextText: 'Next',
        crossChapter: true,
        crossChapterText: true
      },
      
      // Custom 404 page
      notFoundPage: true,
      
      // Tabs plugin
      tabs: {
        persist: true,
        sync: true,
        theme: 'material',
        tabComments: true,
        tabHeadings: true
      }
    };
  </script>
  
  <!-- Docsify core -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4"></script>
  
  <!-- Plugins -->
  <script src="//cdn.jsdelivr.net/npm/docsify@4/lib/plugins/search.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-copy-code@2"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-pagination@2/dist/docsify-pagination.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/docsify-tabs@1"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-bash.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-typescript.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-json.min.js"></script>
  <script src="//cdn.jsdelivr.net/npm/prismjs@1/components/prism-yaml.min.js"></script>
</body>
</html>
```

## Advanced Features

### API Code Examples

Use tabs for multi-language examples:

```markdown
<!-- tabs:start -->

#### **cURL**

```bash
curl -X POST "http://localhost:10000/api/users" \
  -H "Authorization: Bearer your-token" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "securePassword123"
  }'
```

#### **JavaScript**

```javascript
const response = await fetch('http://localhost:10000/api/users', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer your-token',
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'John Doe',
    email: 'john@example.com',
    password: 'securePassword123'
  })
});

const data = await response.json();
console.log(data);
```

#### **Python**

```python
import requests

url = "http://localhost:10000/api/users"
headers = {
    "Authorization": "Bearer your-token",
    "Content-Type": "application/json"
}
data = {
    "name": "John Doe",
    "email": "john@example.com",
    "password": "securePassword123"
}

response = requests.post(url, json=data, headers=headers)
print(response.json())
```

<!-- tabs:end -->
```

### Interactive API Testing

Add testing instructions:

```markdown
## Try It Out

You can test this API endpoint directly:

1. **Get an API token** from the [Authentication](api/authentication.md) endpoint
2. **Open your terminal** or API client (Postman, Insomnia)
3. **Run the example** with your token:

?> **Testing Tip:** Replace `your-token` with an actual JWT token from `/api/auth/login`

!> **Security Note:** Never expose real API tokens in documentation. Use placeholders only.
```

### Custom Alerts and Tips

```markdown
> **Info:** This endpoint requires admin privileges.

!> **Warning:** Rate limiting applies - maximum 100 requests per minute.

?> **Tip:** Use pagination for better performance with large datasets.
```

## Documentation Workflow

### Development Process

```bash
# 1. Write code and tests
./artisan make:controller UserController
./artisan make:service UserService

# 2. Update API documentation
vim docs/api/users.md

# 3. Test documentation locally
npm run serve
open http://localhost:10000/docs

# 4. Commit everything together
git add .
git commit -m "Add user management API with documentation"
```

### Automated Documentation

Generate API docs from code comments:

```typescript
// src/app/Http/Controllers/Api/UserController.ts
/**
 * @api {get} /api/users Get Users
 * @apiName GetUsers
 * @apiGroup Users
 * @apiVersion 1.0.0
 * 
 * @apiDescription Retrieve a paginated list of users.
 * 
 * @apiHeader {String} Authorization Bearer JWT token
 * 
 * @apiParam {Number} [page=1] Page number
 * @apiParam {Number} [limit=20] Items per page
 * @apiParam {String} [search] Search by name or email
 * 
 * @apiSuccess {Number} code Response code (1000)
 * @apiSuccess {String} message Success message
 * @apiSuccess {Object} data Response data
 * @apiSuccess {Object[]} data.users Array of users
 * @apiSuccess {Number} data.users.id User ID
 * @apiSuccess {String} data.users.name User name
 * @apiSuccess {String} data.users.email User email
 */
export default class UserController {
    public async index(request: Request, response: Response): Promise<void> {
        // Implementation...
    }
}
```

## Production Deployment

### Security Configuration

```typescript
// src/config/documentation.ts (Production)
export default {
    enabled: process.env.NODE_ENV !== 'production' || process.env.DOCUMENTATION_ENABLED === 'true',
    
    basicAuth: {
        enabled: true, // Always enable in production
        username: process.env.DOCUMENTATION_BASIC_AUTH_USERNAME,
        password: process.env.DOCUMENTATION_BASIC_AUTH_PASSWORD
    }
};
```

### Environment Variables

```bash
# .env.production
DOCUMENTATION_ENABLED=true
DOCUMENTATION_BASIC_AUTH_ENABLED=true
DOCUMENTATION_BASIC_AUTH_USERNAME=api-docs-admin
DOCUMENTATION_BASIC_AUTH_PASSWORD=super-secure-password-123
```

### Nginx Configuration

```nginx
# nginx.conf - Additional security for docs
location /docs {
    # Rate limiting
    limit_req zone=docs_limit burst=10 nodelay;
    
    # IP whitelist (optional)
    allow 203.0.113.0/24;  # Office network
    allow 198.51.100.42;    # VPN IP
    deny all;
    
    proxy_pass http://localhost:10000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

## Best Practices

### ✅ **DO: Keep Docs Current**
```markdown
# Update docs with every API change
- Add new endpoints immediately
- Update examples when response format changes
- Remove deprecated endpoints
```

### ✅ **DO: Use Real Examples**
```json
// Use realistic example data
{
  "name": "Sarah Johnson",
  "email": "sarah.johnson@example.com",
  "created_at": "2024-01-15T14:30:00Z"
}
```

### ✅ **DO: Document Error Cases**
```markdown
## Error Responses

| Code | Message | When |
|------|---------|------|
| 4001 | User not found | Invalid user ID |
| 4002 | Email already exists | Duplicate email |
| 4003 | Invalid password | Password too weak |
```

### ❌ **DON'T: Expose Sensitive Data**
```markdown
<!-- Bad: Real credentials in docs -->
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

<!-- Good: Placeholder credentials -->
Authorization: Bearer your-jwt-token
```

### ❌ **DON'T: Forget Authentication**
```bash
# Always protect documentation in production
DOCUMENTATION_BASIC_AUTH_ENABLED=true
```

## Summary

Sosise's built-in documentation system provides:

- 📚 **Beautiful Interface** - Clean, searchable documentation with Docsify
- 🔒 **Secure Access** - Basic authentication protection for sensitive docs  
- 🎨 **Customizable** - Full control over appearance and structure
- 🔄 **Version Controlled** - Documentation lives with your code
- 📱 **Mobile Friendly** - Responsive design works on all devices
- 🔍 **Full-Text Search** - Built-in search across all documentation
- 💡 **Interactive Examples** - Multi-language code samples and testing guides
- ⚡ **Zero Setup** - Works out of the box with sensible defaults

Create comprehensive, beautiful API documentation that developers love to use! 🚀