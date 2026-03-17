# Project Directory Structure

## Quick Overview

Sosise follows a clean, intuitive architecture that scales from simple APIs to complex enterprise applications!

```
my-sosise-project/
├── src/                    # 📁 Source code (TypeScript)
│   ├── app/                # 🏢 Application logic
│   │   ├── Http/           # 🌐 Controllers & Middlewares
│   │   ├── Services/       # 🔧 Business logic layer
│   │   ├── Repositories/   # 💾 Data access layer
│   │   ├── Unifiers/       # ✨ Request validation & mapping
│   │   ├── Exceptions/     # 🚫 Custom error classes
│   │   ├── Types/          # 📝 TypeScript type definitions
│   │   ├── Enums/          # 🎯 Enumeration constants
│   │   ├── Libraries/      # 📚 Custom utility libraries
│   │   └── Console/        # 💻 Artisan commands & queue workers
│   ├── config/             # ⚙️ Configuration files
│   ├── database/           # 🗄️ Migrations & seeders
│   ├── routes/             # 🗺️ API route definitions
│   └── tests/              # ✅ Test files
├── build/                  # 🏗️ Compiled JavaScript
├── storage/                # 📋 File storage & logs
├── docker/                 # 🐳 Docker configuration
├── docs/                   # 📖 Project documentation
└── artisan                 # 🪄 Command-line tool
```

Perfect for building scalable TypeScript APIs with proper separation of concerns!

## Architecture Philosophy

Sosise embraces the **Controller → Service → Repository** pattern:

```typescript
// 🌐 Controller: Handles HTTP requests
export default class UserController {
    constructor(private userService: UserService) {}
    
    public async getUser(request: Request, response: Response): Promise<void> {
        const user = await this.userService.getUserById(request.params.id);
        response.json({ user });
    }
}

// 🔧 Service: Contains business logic
export default class UserService {
    constructor(private userRepository: UserRepository) {}
    
    public async getUserById(id: number): Promise<User> {
        const user = await this.userRepository.findById(id);
        if (!user) {
            throw new UserNotFoundException('User not found');
        }
        return user;
    }
}

// 💾 Repository: Handles data access
export default class UserRepository {
    public async findById(id: number): Promise<User | null> {
        return await this.db('users').where('id', id).first();
    }
}
```

This ensures clean, testable, and maintainable code!

## Core Directories

### `/src` - Source Code Root

All your TypeScript source code lives here. This is where the magic happens!

**Key Features:**
- 🔥 Full TypeScript support with strict typing
- 🔄 Hot reload during development
- 🏗️ Automatic compilation to `/build`
- 🧩 Intelligent import resolution

### `/src/app` - Application Logic

The heart of your application where all business logic resides:

```typescript
// Example structure
src/app/
├── Http/
│   ├── Controllers/        # HTTP request handlers
│   │   ├── Api/            # API controllers
│   │   └── Web/            # Web controllers
│   ├── Middlewares/        # Request middleware
│   └── Kernel.ts           # Middleware registration
├── Services/               # Business logic services
│   ├── UserService.ts
│   └── EmailService.ts
├── Repositories/           # Data access layer
│   ├── UserRepository.ts
│   └── ProductRepository.ts
└── Console/                # CLI commands & workers
    ├── Commands/           # Custom artisan commands
    └── QueueWorkers/       # Background job workers
```

### `/src/config` - Configuration Hub

Centralized configuration management:

```typescript
// src/config/app.ts
export default {
    name: process.env.APP_NAME || 'Sosise App',
    port: parseInt(process.env.APP_PORT || '10000'),
    environment: process.env.NODE_ENV || 'development',
    timezone: 'UTC'
};

// src/config/database.ts
export default {
    default: process.env.DB_CONNECTION || 'mysql',
    connections: {
        mysql: {
            client: 'mysql2',
            connection: {
                host: process.env.DB_HOST,
                database: process.env.DB_DATABASE,
                user: process.env.DB_USERNAME,
                password: process.env.DB_PASSWORD
            }
        }
    }
};
```

**Configuration Files:**
- `app.ts` - Application settings
- `database.ts` - Database connections
- `cache.ts` - Cache configuration
- `session.ts` - Session management
- `logging.ts` - Logging configuration
- `mailer.ts` - Email settings
- `queue.ts` - Background job configuration

### `/src/routes` - API Route Definitions

Define your API endpoints with clean, expressive routing:

```typescript
// src/routes/api.ts
export default function apiRoutes(router: Router): void {
    // Health check
    router.get('/health', 'HealthController@check');
    
    // User management
    router.group('/users', () => {
        router.get('/', 'UserController@index');
        router.post('/', 'UserController@create');
        router.get('/:id', 'UserController@show');
        router.put('/:id', 'UserController@update');
        router.delete('/:id', 'UserController@destroy');
    }).middleware(['auth', 'throttle:100']);
    
    // Authentication routes
    router.group('/auth', () => {
        router.post('/login', 'AuthController@login');
        router.post('/logout', 'AuthController@logout').middleware('auth');
        router.post('/register', 'AuthController@register');
    });
}
```

### `/src/database` - Database Management

Complete database lifecycle management:

```typescript
// Database structure
database/
├── migrations/             # Database schema changes
│   ├── 20240101000000_CreateUsersTable.ts
│   └── 20240101000001_CreateProductsTable.ts
└── seeders/                # Test data generation
    ├── UsersTableSeeder.ts
    └── ProductsTableSeeder.ts
```

## Application Layer Details

### HTTP Layer (`src/app/Http`)

Handles all HTTP-related functionality:

#### Controllers

```typescript
// src/app/Http/Controllers/Api/UserController.ts
export default class UserController {
    constructor(
        private userService: UserService,
        private logger: LoggerService
    ) {}

    public async index(request: Request, response: Response): Promise<void> {
        const filters = new GetUsersUnifier(request.query);
        const users = await this.userService.getUsers(filters);
        
        response.json({
            code: 1000,
            message: 'Users retrieved successfully',
            data: users
        });
    }
    
    public async store(request: Request, response: Response): Promise<void> {
        const userData = new CreateUserUnifier(request.body);
        const user = await this.userService.createUser(userData);
        
        response.status(201).json({
            code: 1000,
            message: 'User created successfully',
            data: user
        });
    }
}
```

#### Middlewares

```typescript
// src/app/Http/Middlewares/AuthMiddleware.ts
export default class AuthMiddleware {
    public async handle(
        request: Request, 
        response: Response, 
        next: NextFunction
    ): Promise<void> {
        const token = request.header('Authorization')?.replace('Bearer ', '');
        
        if (!token) {
            return response.status(401).json({
                code: 2001,
                message: 'Authentication required'
            });
        }
        
        try {
            const user = await this.authService.validateToken(token);
            request.user = user;
            next();
        } catch (error) {
            response.status(401).json({
                code: 2002,
                message: 'Invalid token'
            });
        }
    }
}
```

### Service Layer (`src/app/Services`)

Contains all business logic and orchestrates data operations:

```typescript
// src/app/Services/UserService.ts
export default class UserService {
    constructor(
        private userRepository: UserRepository,
        private emailService: EmailService,
        private logger: LoggerService
    ) {}

    public async createUser(userData: CreateUserUnifier): Promise<User> {
        // Validation
        const existingUser = await this.userRepository.findByEmail(userData.email);
        if (existingUser) {
            throw new UserAlreadyExistsException('Email already registered');
        }
        
        // Hash password
        userData.password = await this.hashPassword(userData.password);
        
        // Create user
        const user = await this.userRepository.create(userData.toObject());
        
        // Send welcome email
        await this.emailService.sendWelcomeEmail(user);
        
        // Log activity
        this.logger.info('New user created', { userId: user.id, email: user.email });
        
        return user;
    }
    
    public async getUserById(id: number): Promise<User> {
        const user = await this.userRepository.findById(id);
        if (!user) {
            throw new UserNotFoundException(`User with ID ${id} not found`);
        }
        return user;
    }
}
```

### Repository Layer (`src/app/Repositories`)

Handles all data access and database operations:

```typescript
// src/app/Repositories/UserRepository.ts
export default class UserRepository {
    constructor(
        private db: Knex = Database.getConnection().client
    ) {}

    public async create(userData: Partial<User>): Promise<User> {
        const [id] = await this.db('users')
            .insert(userData)
            .returning('id');
        
        return await this.findById(id);
    }
    
    public async findById(id: number): Promise<User | null> {
        return await this.db('users')
            .where('id', id)
            .where('deleted_at', null)
            .first();
    }
    
    public async findByEmail(email: string): Promise<User | null> {
        return await this.db('users')
            .where('email', email)
            .where('deleted_at', null)
            .first();
    }
    
    public async getUsers(filters: GetUsersFilters): Promise<User[]> {
        let query = this.db('users')
            .where('deleted_at', null)
            .orderBy('created_at', 'desc');
            
        if (filters.search) {
            query = query.where(function() {
                this.where('name', 'like', `%${filters.search}%`)
                    .orWhere('email', 'like', `%${filters.search}%`);
            });
        }
        
        if (filters.role) {
            query = query.where('role', filters.role);
        }
        
        return await query.limit(filters.limit || 50);
    }
}
```

## Support Directories

### Console Commands (`src/app/Console`)

Custom CLI commands and background workers:

```typescript
// src/app/Console/Commands/SendDailyReportCommand.ts
export default class SendDailyReportCommand {
    public signature = 'report:daily {--email=}';
    public description = 'Send daily analytics report';
    
    public async handle(): Promise<void> {
        const reportData = await this.analyticsService.getDailyReport();
        await this.emailService.sendDailyReport(reportData);
        
        this.success('Daily report sent successfully!');
    }
}
```

### Data Validation (`src/app/Unifiers`)

Request validation and data transformation:

```typescript
// src/app/Unifiers/CreateUserUnifier.ts
import Validator from 'sosise-core/build/Validator/Validator';
import ValidationException from 'sosise-core/build/Exceptions/Validation/ValidationException';

export default class CreateUserUnifier {
    public name: string;
    public email: string;
    public password: string;
    public role: string;

    constructor(params: any) {
        // Validate input
        const validator = new Validator(params);
        validator.field('name').required().shouldBeString().min(2).max(100);
        validator.field('email').required().email();
        validator.field('password').required().shouldBeString().min(8);
        validator.field('role').shouldBeString().inList(['admin', 'user']);

        if (validator.fails()) {
            throw new ValidationException('Validation failed', validator.errors);
        }

        // Assign validated data
        this.name = params.name;
        this.email = params.email.toLowerCase();
        this.password = params.password;
        this.role = params.role || 'user';
    }
}
```

### Exception Handling (`src/app/Exceptions`)

Custom exception classes:

```typescript
// src/app/Exceptions/UserNotFoundException.ts
export default class UserNotFoundException extends Exception {
    protected code = 4001;
    protected statusCode = 404;
    protected message = 'User not found';
}
```

## Code Generation

Sosise provides powerful code generators:

```bash
# Generate a complete CRUD structure
./artisan make:controller UserController
./artisan make:service UserService
./artisan make:repository UserRepository
./artisan make:unifier CreateUserUnifier
./artisan make:migration CreateUsersTable
./artisan make:seeder UsersTableSeeder
./artisan make:exception UserNotFoundException
./artisan make:middleware AuthMiddleware
./artisan make:command SendReportCommand
```

## Best Practices

### ✅ **DO: Follow Layer Separation**
```typescript
// Controller handles HTTP concerns only
export default class ProductController {
    public async create(request: Request, response: Response): Promise<void> {
        const productData = new CreateProductUnifier(request.body);
        const product = await this.productService.createProduct(productData);
        response.status(201).json({ product });
    }
}

// Service contains business logic
export default class ProductService {
    public async createProduct(data: CreateProductUnifier): Promise<Product> {
        await this.validateProductRules(data);
        return await this.productRepository.create(data.toObject());
    }
}
```

### ✅ **DO: Use Dependency Injection**
```typescript
export default class UserController {
    constructor(
        private userService: UserService,
        private logger: LoggerService
    ) {}
}
```

### ❌ **DON'T: Direct Database Access in Controllers**
```typescript
// Bad
export default class UserController {
    public async getUser(request: Request, response: Response): Promise<void> {
        const user = await Database.getConnection().client('users').first();
        response.json({ user });
    }
}

// Good
export default class UserController {
    public async getUser(request: Request, response: Response): Promise<void> {
        const user = await this.userService.getUserById(request.params.id);
        response.json({ user });
    }
}
```

## File Naming Conventions

- **Controllers**: `PascalCase` + `Controller` suffix (`UserController.ts`)
- **Services**: `PascalCase` + `Service` suffix (`EmailService.ts`)
- **Repositories**: `PascalCase` + `Repository` suffix (`UserRepository.ts`)
- **Middlewares**: `PascalCase` + `Middleware` suffix (`AuthMiddleware.ts`)
- **Unifiers**: `PascalCase` + `Unifier` suffix (`CreateUserUnifier.ts`)
- **Exceptions**: `PascalCase` + `Exception` suffix (`UserNotFoundException.ts`)
- **Migrations**: `timestamp_DescriptiveName` (`20240101000000_CreateUsersTable.ts`)

## Summary

Sosise's directory structure promotes:

- 🎨 **Clean Architecture** - Clear separation between layers
- 🔧 **Maintainability** - Easy to understand and modify
- 🚀 **Scalability** - Grows with your application
- ✅ **Testability** - Each layer can be tested in isolation
- 🔄 **Reusability** - Services and repositories can be shared
- 👥 **Team Collaboration** - Consistent structure for all developers

Build enterprise-grade TypeScript APIs with confidence using Sosise's proven architecture!
