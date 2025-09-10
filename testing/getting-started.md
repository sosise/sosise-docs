# Testing Framework

## Quick Start

Build confident, bug-free APIs with Sosise's comprehensive testing framework! Write unit tests, integration tests, and end-to-end API tests with ease.

```bash
# Run all tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch

# Run specific test file
npm test -- --testPathPattern=UserService
```

Your tests are automatically configured and ready to run! ✅

## Introduction

Sosise embraces Test-Driven Development (TDD) with a powerful testing ecosystem:

- ✅ **Jest Framework** - Fast, reliable test runner with excellent TypeScript support
- ✅ **Unit Testing** - Test individual services, repositories, and utilities
- ✅ **Integration Testing** - Test how components work together
- ✅ **API Testing** - End-to-end HTTP request testing
- ✅ **Database Testing** - In-memory database for isolated tests
- ✅ **Mocking & Stubbing** - Easy mocking of dependencies
- ✅ **Code Coverage** - Track test coverage across your codebase
- ✅ **CI/CD Ready** - Seamless integration with GitHub Actions, GitLab CI

## Test Structure

Your tests are organized in the `tests/` directory:

```
tests/
├── Unit/                   # Unit tests for individual components
│   ├── Services/
│   │   ├── UserService.test.ts
│   │   └── EmailService.test.ts
│   ├── Repositories/
│   │   └── UserRepository.test.ts
│   └── Helpers/
│       └── ValidationHelper.test.ts
├── Integration/            # Integration tests
│   ├── Database/
│   │   └── UserFlow.test.ts
│   └── Services/
│       └── AuthenticationFlow.test.ts
├── Functional/             # End-to-end API tests
│   ├── Auth/
│   │   ├── Login.test.ts
│   │   └── Registration.test.ts
│   └── Users/
│       ├── UserCrud.test.ts
│       └── UserPermissions.test.ts
├── Helpers/                # Test utilities and helpers
│   ├── TestHelper.ts
│   ├── DatabaseHelper.ts
│   └── ApiHelper.ts
└── setup.ts                # Global test setup
```

## Writing Unit Tests

### Service Testing

Test your business logic in isolation:

```typescript
// tests/Unit/Services/UserService.test.ts
import UserService from '../../../src/app/Services/UserService';
import UserRepository from '../../../src/app/Repositories/UserRepository';
import EmailService from '../../../src/app/Services/EmailService';
import { UserNotFoundException, UserAlreadyExistsException } from '../../../src/app/Exceptions';

// Mock dependencies
jest.mock('../../../src/app/Repositories/UserRepository');
jest.mock('../../../src/app/Services/EmailService');

describe('UserService', () => {
    let userService: UserService;
    let mockUserRepository: jest.Mocked<UserRepository>;
    let mockEmailService: jest.Mocked<EmailService>;

    beforeEach(() => {
        // Create mocked instances
        mockUserRepository = new UserRepository() as jest.Mocked<UserRepository>;
        mockEmailService = new EmailService() as jest.Mocked<EmailService>;
        
        // Initialize service with mocks
        userService = new UserService(mockUserRepository, mockEmailService);
    });

    describe('createUser', () => {
        it('should create user successfully', async () => {
            // Arrange
            const userData = {
                name: 'John Doe',
                email: 'john@example.com',
                password: 'hashedPassword123'
            };
            
            const expectedUser = {
                id: 1,
                ...userData,
                created_at: new Date()
            };

            mockUserRepository.findByEmail.mockResolvedValue(null);
            mockUserRepository.create.mockResolvedValue(expectedUser);
            mockEmailService.sendWelcomeEmail.mockResolvedValue(undefined);

            // Act
            const result = await userService.createUser(userData);

            // Assert
            expect(result).toEqual(expectedUser);
            expect(mockUserRepository.findByEmail).toHaveBeenCalledWith(userData.email);
            expect(mockUserRepository.create).toHaveBeenCalledWith(userData);
            expect(mockEmailService.sendWelcomeEmail).toHaveBeenCalledWith(expectedUser);
        });

        it('should throw error if email already exists', async () => {
            // Arrange
            const userData = {
                name: 'John Doe',
                email: 'existing@example.com',
                password: 'hashedPassword123'
            };

            const existingUser = { id: 1, email: userData.email };
            mockUserRepository.findByEmail.mockResolvedValue(existingUser);

            // Act & Assert
            await expect(userService.createUser(userData))
                .rejects
                .toThrow(UserAlreadyExistsException);
                
            expect(mockUserRepository.create).not.toHaveBeenCalled();
            expect(mockEmailService.sendWelcomeEmail).not.toHaveBeenCalled();
        });
    });

    describe('getUserById', () => {
        it('should return user when found', async () => {
            // Arrange
            const userId = 1;
            const expectedUser = {
                id: userId,
                name: 'John Doe',
                email: 'john@example.com'
            };

            mockUserRepository.findById.mockResolvedValue(expectedUser);

            // Act
            const result = await userService.getUserById(userId);

            // Assert
            expect(result).toEqual(expectedUser);
            expect(mockUserRepository.findById).toHaveBeenCalledWith(userId);
        });

        it('should throw error when user not found', async () => {
            // Arrange
            const userId = 999;
            mockUserRepository.findById.mockResolvedValue(null);

            // Act & Assert
            await expect(userService.getUserById(userId))
                .rejects
                .toThrow(UserNotFoundException);
        });
    });
});
```

### Repository Testing

Test data access layer with in-memory database:

```typescript
// tests/Unit/Repositories/UserRepository.test.ts
import Database from 'sosise-core/build/Database/Database';
import UserRepository from '../../../src/app/Repositories/UserRepository';
import DatabaseHelper from '../../Helpers/DatabaseHelper';

describe('UserRepository', () => {
    let userRepository: UserRepository;
    let db: any;

    beforeAll(async () => {
        // Setup in-memory SQLite database
        await DatabaseHelper.setupTestDatabase();
        db = Database.getConnection().client;
        userRepository = new UserRepository();
    });

    beforeEach(async () => {
        // Clean database before each test
        await DatabaseHelper.cleanDatabase();
        
        // Seed test data
        await db('users').insert([
            {
                id: 1,
                name: 'John Doe',
                email: 'john@example.com',
                password: 'hashedPassword123',
                role: 'user',
                created_at: new Date(),
                updated_at: new Date()
            },
            {
                id: 2,
                name: 'Jane Admin',
                email: 'jane@example.com',
                password: 'hashedPassword123',
                role: 'admin',
                created_at: new Date(),
                updated_at: new Date()
            }
        ]);
    });

    afterAll(async () => {
        await DatabaseHelper.closeTestDatabase();
    });

    describe('findById', () => {
        it('should find user by id', async () => {
            const user = await userRepository.findById(1);
            
            expect(user).toBeDefined();
            expect(user.id).toBe(1);
            expect(user.name).toBe('John Doe');
            expect(user.email).toBe('john@example.com');
        });

        it('should return null for non-existent user', async () => {
            const user = await userRepository.findById(999);
            
            expect(user).toBeNull();
        });
    });

    describe('findByEmail', () => {
        it('should find user by email', async () => {
            const user = await userRepository.findByEmail('john@example.com');
            
            expect(user).toBeDefined();
            expect(user.id).toBe(1);
            expect(user.email).toBe('john@example.com');
        });

        it('should be case insensitive', async () => {
            const user = await userRepository.findByEmail('JOHN@EXAMPLE.COM');
            
            expect(user).toBeDefined();
            expect(user.email).toBe('john@example.com');
        });
    });

    describe('create', () => {
        it('should create new user', async () => {
            const userData = {
                name: 'New User',
                email: 'newuser@example.com',
                password: 'hashedPassword123',
                role: 'user'
            };

            const user = await userRepository.create(userData);
            
            expect(user).toBeDefined();
            expect(user.id).toBeDefined();
            expect(user.name).toBe(userData.name);
            expect(user.email).toBe(userData.email);
            expect(user.created_at).toBeDefined();
        });
    });

    describe('getUsers', () => {
        it('should return paginated users', async () => {
            const users = await userRepository.getUsers({ 
                limit: 10, 
                offset: 0 
            });
            
            expect(users).toHaveLength(2);
            expect(users[0].name).toBeDefined();
            expect(users[0].email).toBeDefined();
        });

        it('should filter by search term', async () => {
            const users = await userRepository.getUsers({ 
                search: 'Admin',
                limit: 10, 
                offset: 0 
            });
            
            expect(users).toHaveLength(1);
            expect(users[0].name).toBe('Jane Admin');
        });

        it('should filter by role', async () => {
            const adminUsers = await userRepository.getUsers({ 
                role: 'admin',
                limit: 10, 
                offset: 0 
            });
            
            expect(adminUsers).toHaveLength(1);
            expect(adminUsers[0].role).toBe('admin');
        });
    });
});
```

## Integration Testing

Test how multiple components work together:

```typescript
// tests/Integration/Services/AuthenticationFlow.test.ts
import AuthService from '../../../src/app/Services/AuthService';
import UserService from '../../../src/app/Services/UserService';
import DatabaseHelper from '../../Helpers/DatabaseHelper';
import TestHelper from '../../Helpers/TestHelper';

describe('Authentication Flow Integration', () => {
    let authService: AuthService;
    let userService: UserService;

    beforeAll(async () => {
        await DatabaseHelper.setupTestDatabase();
        
        // Use real dependencies for integration testing
        authService = TestHelper.getService(AuthService);
        userService = TestHelper.getService(UserService);
    });

    beforeEach(async () => {
        await DatabaseHelper.cleanDatabase();
    });

    afterAll(async () => {
        await DatabaseHelper.closeTestDatabase();
    });

    describe('Complete User Registration and Login Flow', () => {
        it('should register user and allow login', async () => {
            // Step 1: Register new user
            const registrationData = {
                name: 'Integration Test User',
                email: 'integration@example.com',
                password: 'TestPassword123!'
            };

            const user = await userService.createUser(registrationData);
            
            expect(user).toBeDefined();
            expect(user.id).toBeDefined();
            expect(user.email).toBe(registrationData.email);

            // Step 2: Login with created user
            const loginData = {
                email: registrationData.email,
                password: registrationData.password
            };

            const loginResult = await authService.login(loginData);
            
            expect(loginResult).toBeDefined();
            expect(loginResult.user.id).toBe(user.id);
            expect(loginResult.token).toBeDefined();
            expect(typeof loginResult.token).toBe('string');

            // Step 3: Validate token
            const validatedUser = await authService.validateToken(loginResult.token);
            
            expect(validatedUser).toBeDefined();
            expect(validatedUser.id).toBe(user.id);
            expect(validatedUser.email).toBe(user.email);
        });

        it('should fail login with incorrect password', async () => {
            // Register user
            const user = await userService.createUser({
                name: 'Test User',
                email: 'test@example.com',
                password: 'CorrectPassword123!'
            });

            // Attempt login with wrong password
            const loginData = {
                email: user.email,
                password: 'WrongPassword123!'
            };

            await expect(authService.login(loginData))
                .rejects
                .toThrow('Invalid credentials');
        });
    });
});
```

## API Testing (Functional Tests)

Test your HTTP endpoints end-to-end:

```typescript
// tests/Functional/Users/UserCrud.test.ts
import TestHelper from '../../Helpers/TestHelper';
import ApiHelper from '../../Helpers/ApiHelper';
import DatabaseHelper from '../../Helpers/DatabaseHelper';

describe('User CRUD API', () => {
    let apiHelper: ApiHelper;
    let adminToken: string;
    let userToken: string;

    beforeAll(async () => {
        await DatabaseHelper.setupTestDatabase();
        apiHelper = new ApiHelper();
        
        // Create admin and user tokens for testing
        adminToken = await apiHelper.createAdminToken();
        userToken = await apiHelper.createUserToken();
    });

    beforeEach(async () => {
        await DatabaseHelper.cleanDatabase();
    });

    afterAll(async () => {
        await DatabaseHelper.closeTestDatabase();
        await apiHelper.close();
    });

    describe('GET /api/users', () => {
        it('should return users list for admin', async () => {
            // Seed test data
            await DatabaseHelper.seedUsers([
                { name: 'User 1', email: 'user1@example.com', role: 'user' },
                { name: 'User 2', email: 'user2@example.com', role: 'admin' }
            ]);

            const response = await apiHelper.get('/api/users', {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            expect(response.status).toBe(200);
            expect(response.body.code).toBe(1000);
            expect(response.body.message).toBe('Users retrieved successfully');
            expect(response.body.data.users).toHaveLength(2);
            expect(response.body.data.users[0]).toHaveProperty('id');
            expect(response.body.data.users[0]).toHaveProperty('name');
            expect(response.body.data.users[0]).toHaveProperty('email');
            expect(response.body.data.users[0]).not.toHaveProperty('password');
        });

        it('should return 401 for unauthenticated request', async () => {
            const response = await apiHelper.get('/api/users');

            expect(response.status).toBe(401);
            expect(response.body.code).toBe(2001);
            expect(response.body.message).toBe('Authentication required');
        });

        it('should return 403 for non-admin user', async () => {
            const response = await apiHelper.get('/api/users', {
                headers: { Authorization: `Bearer ${userToken}` }
            });

            expect(response.status).toBe(403);
            expect(response.body.code).toBe(2003);
            expect(response.body.message).toBe('Insufficient permissions');
        });

        it('should support pagination', async () => {
            // Seed 25 users
            const users = Array.from({ length: 25 }, (_, i) => ({
                name: `User ${i + 1}`,
                email: `user${i + 1}@example.com`,
                role: 'user'
            }));
            await DatabaseHelper.seedUsers(users);

            const response = await apiHelper.get('/api/users?page=2&limit=10', {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            expect(response.status).toBe(200);
            expect(response.body.data.users).toHaveLength(10);
            expect(response.body.data.pagination.current_page).toBe(2);
            expect(response.body.data.pagination.total_items).toBe(25);
            expect(response.body.data.pagination.per_page).toBe(10);
        });

        it('should support search functionality', async () => {
            await DatabaseHelper.seedUsers([
                { name: 'John Doe', email: 'john@example.com', role: 'user' },
                { name: 'Jane Smith', email: 'jane@example.com', role: 'user' },
                { name: 'Bob Johnson', email: 'bob@example.com', role: 'user' }
            ]);

            const response = await apiHelper.get('/api/users?search=John', {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            expect(response.status).toBe(200);
            expect(response.body.data.users).toHaveLength(2);
            
            const names = response.body.data.users.map(u => u.name);
            expect(names).toContain('John Doe');
            expect(names).toContain('Bob Johnson');
        });
    });

    describe('POST /api/users', () => {
        it('should create user successfully', async () => {
            const userData = {
                name: 'New User',
                email: 'newuser@example.com',
                password: 'SecurePassword123!',
                role: 'user'
            };

            const response = await apiHelper.post('/api/users', userData, {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            expect(response.status).toBe(201);
            expect(response.body.code).toBe(1000);
            expect(response.body.message).toBe('User created successfully');
            expect(response.body.data.user.id).toBeDefined();
            expect(response.body.data.user.name).toBe(userData.name);
            expect(response.body.data.user.email).toBe(userData.email);
            expect(response.body.data.user).not.toHaveProperty('password');
        });

        it('should validate required fields', async () => {
            const response = await apiHelper.post('/api/users', {}, {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            expect(response.status).toBe(400);
            expect(response.body.code).toBe(3000);
            expect(response.body.message).toContain('validation');
            expect(response.body.errors).toBeDefined();
            expect(response.body.errors.name).toContain('required');
            expect(response.body.errors.email).toContain('required');
            expect(response.body.errors.password).toContain('required');
        });

        it('should prevent duplicate email', async () => {
            const userData = {
                name: 'Test User',
                email: 'duplicate@example.com',
                password: 'SecurePassword123!',
                role: 'user'
            };

            // Create first user
            await apiHelper.post('/api/users', userData, {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            // Attempt to create duplicate
            const response = await apiHelper.post('/api/users', userData, {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            expect(response.status).toBe(400);
            expect(response.body.code).toBe(4002);
            expect(response.body.message).toBe('Email already exists');
        });
    });

    describe('GET /api/users/:id', () => {
        it('should return specific user', async () => {
            const [user] = await DatabaseHelper.seedUsers([
                { name: 'Test User', email: 'test@example.com', role: 'user' }
            ]);

            const response = await apiHelper.get(`/api/users/${user.id}`, {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            expect(response.status).toBe(200);
            expect(response.body.data.user.id).toBe(user.id);
            expect(response.body.data.user.name).toBe(user.name);
            expect(response.body.data.user.email).toBe(user.email);
        });

        it('should return 404 for non-existent user', async () => {
            const response = await apiHelper.get('/api/users/999', {
                headers: { Authorization: `Bearer ${adminToken}` }
            });

            expect(response.status).toBe(404);
            expect(response.body.code).toBe(4001);
            expect(response.body.message).toBe('User not found');
        });
    });
});
```

## Test Helpers and Utilities

### Database Helper

```typescript
// tests/Helpers/DatabaseHelper.ts
import Database from 'sosise-core/build/Database/Database';
import { faker } from '@faker-js/faker';

export default class DatabaseHelper {
    public static async setupTestDatabase(): Promise<void> {
        // Switch to test database configuration
        process.env.NODE_ENV = 'testing';
        
        // Initialize database connection
        await Database.createConnection();
        
        // Run migrations
        await this.runMigrations();
    }

    public static async cleanDatabase(): Promise<void> {
        const db = Database.getConnection().client;
        
        // Disable foreign key checks
        await db.raw('PRAGMA foreign_keys = OFF');
        
        // Clear all tables
        const tables = ['users', 'sessions', 'password_resets'];
        for (const table of tables) {
            await db(table).del();
        }
        
        // Re-enable foreign key checks
        await db.raw('PRAGMA foreign_keys = ON');
    }

    public static async closeTestDatabase(): Promise<void> {
        await Database.closeConnection();
    }

    public static async runMigrations(): Promise<void> {
        const db = Database.getConnection().client;
        
        // Create tables for testing (simplified)
        await db.schema.createTable('users', (table) => {
            table.increments('id');
            table.string('name').notNullable();
            table.string('email').unique().notNullable();
            table.string('password').notNullable();
            table.enum('role', ['admin', 'user']).defaultTo('user');
            table.timestamp('created_at').defaultTo(db.fn.now());
            table.timestamp('updated_at').defaultTo(db.fn.now());
            table.timestamp('deleted_at').nullable();
        });
    }

    public static async seedUsers(users: Partial<User>[]): Promise<User[]> {
        const db = Database.getConnection().client;
        
        const usersToInsert = users.map(user => ({
            name: user.name || faker.person.fullName(),
            email: user.email || faker.internet.email(),
            password: user.password || '$2b$10$hashedPassword',
            role: user.role || 'user',
            created_at: new Date(),
            updated_at: new Date()
        }));

        const insertedIds = await db('users').insert(usersToInsert).returning('id');
        
        return await db('users').whereIn('id', insertedIds);
    }

    public static async createTestUser(overrides: Partial<User> = {}): Promise<User> {
        const db = Database.getConnection().client;
        
        const userData = {
            name: faker.person.fullName(),
            email: faker.internet.email(),
            password: '$2b$10$hashedPassword',
            role: 'user',
            created_at: new Date(),
            updated_at: new Date(),
            ...overrides
        };

        const [id] = await db('users').insert(userData).returning('id');
        return await db('users').where('id', id).first();
    }
}
```

### API Helper

```typescript
// tests/Helpers/ApiHelper.ts
import request from 'supertest';
import app from '../../src/server';
import AuthService from '../../src/app/Services/AuthService';
import DatabaseHelper from './DatabaseHelper';

export default class ApiHelper {
    private app: any;
    private authService: AuthService;

    constructor() {
        this.app = app;
        this.authService = new AuthService();
    }

    public async get(url: string, options: any = {}): Promise<any> {
        const req = request(this.app).get(url);
        
        if (options.headers) {
            Object.keys(options.headers).forEach(key => {
                req.set(key, options.headers[key]);
            });
        }
        
        return req;
    }

    public async post(url: string, data: any = {}, options: any = {}): Promise<any> {
        const req = request(this.app).post(url).send(data);
        
        if (options.headers) {
            Object.keys(options.headers).forEach(key => {
                req.set(key, options.headers[key]);
            });
        }
        
        return req;
    }

    public async put(url: string, data: any = {}, options: any = {}): Promise<any> {
        const req = request(this.app).put(url).send(data);
        
        if (options.headers) {
            Object.keys(options.headers).forEach(key => {
                req.set(key, options.headers[key]);
            });
        }
        
        return req;
    }

    public async delete(url: string, options: any = {}): Promise<any> {
        const req = request(this.app).delete(url);
        
        if (options.headers) {
            Object.keys(options.headers).forEach(key => {
                req.set(key, options.headers[key]);
            });
        }
        
        return req;
    }

    public async createAdminToken(): Promise<string> {
        const admin = await DatabaseHelper.createTestUser({
            name: 'Test Admin',
            email: 'admin@test.com',
            role: 'admin'
        });

        return this.authService.generateToken(admin);
    }

    public async createUserToken(): Promise<string> {
        const user = await DatabaseHelper.createTestUser({
            name: 'Test User',
            email: 'user@test.com',
            role: 'user'
        });

        return this.authService.generateToken(user);
    }

    public async close(): Promise<void> {
        // Close any open connections
        if (this.app && this.app.close) {
            await this.app.close();
        }
    }
}
```

## Test Configuration

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
    preset: 'ts-jest',
    testEnvironment: 'node',
    roots: ['<rootDir>/tests'],
    testMatch: [
        '**/__tests__/**/*.test.ts',
        '**/?(*.)+(spec|test).ts'
    ],
    transform: {
        '^.+\\.ts$': 'ts-jest'
    },
    collectCoverageFrom: [
        'src/**/*.ts',
        '!src/**/*.d.ts',
        '!src/**/index.ts'
    ],
    coverageDirectory: 'coverage',
    coverageReporters: [
        'text',
        'text-summary',
        'lcov',
        'html'
    ],
    coverageThreshold: {
        global: {
            branches: 80,
            functions: 80,
            lines: 80,
            statements: 80
        }
    },
    setupFilesAfterEnv: ['<rootDir>/tests/setup.ts'],
    testTimeout: 10000,
    verbose: true
};
```

### Global Test Setup

```typescript
// tests/setup.ts
import DatabaseHelper from './Helpers/DatabaseHelper';

// Global test setup
beforeAll(async () => {
    // Set test environment
    process.env.NODE_ENV = 'testing';
    process.env.DB_DATABASE = ':memory:';
    
    // Suppress console output during tests
    if (!process.env.DEBUG_TESTS) {
        console.log = jest.fn();
        console.warn = jest.fn();
        console.error = jest.fn();
    }
});

// Global test teardown
afterAll(async () => {
    // Clean up any global resources
    await DatabaseHelper.closeTestDatabase();
});

// Custom Jest matchers
expect.extend({
    toBeValidEmail(received: string) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        const pass = emailRegex.test(received);
        
        return {
            message: () => 
                pass 
                    ? `Expected ${received} not to be a valid email`
                    : `Expected ${received} to be a valid email`,
            pass
        };
    },
    
    toBeValidUUID(received: string) {
        const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;
        const pass = uuidRegex.test(received);
        
        return {
            message: () => 
                pass 
                    ? `Expected ${received} not to be a valid UUID`
                    : `Expected ${received} to be a valid UUID`,
            pass
        };
    }
});
```

## Running Tests

### NPM Scripts

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:unit": "jest tests/Unit",
    "test:integration": "jest tests/Integration", 
    "test:functional": "jest tests/Functional",
    "test:ci": "jest --ci --coverage --watchAll=false"
  }
}
```

### Test Commands

```bash
# Run all tests
npm test

# Run tests in specific directory
npm run test:unit
npm run test:integration
npm run test:functional

# Run specific test file
npm test UserService.test.ts

# Run tests matching pattern
npm test -- --testNamePattern="should create user"

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch

# Run tests for CI/CD
npm run test:ci
```

## Continuous Integration

### GitHub Actions

```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x, 20.x]

    steps:
    - uses: actions/checkout@v3

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'

    - name: Install dependencies
      run: npm ci

    - name: Run linter
      run: npm run lint

    - name: Run type check
      run: npm run typecheck

    - name: Run tests
      run: npm run test:ci

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage/lcov.info
        flags: unittests
        name: codecov-umbrella
```

## Best Practices

### ✅ **DO: Write Tests First (TDD)**
```typescript
// 1. Write failing test
describe('UserService.createUser', () => {
    it('should create user with hashed password', async () => {
        // Test implementation
    });
});

// 2. Write minimal code to pass
// 3. Refactor while keeping tests green
```

### ✅ **DO: Test Edge Cases**
```typescript
// Test boundary conditions
it('should handle empty string email', async () => {
    await expect(userService.createUser({ email: '' }))
        .rejects.toThrow();
});

// Test null/undefined values
it('should handle null user data', async () => {
    await expect(userService.createUser(null))
        .rejects.toThrow();
});
```

### ✅ **DO: Use Descriptive Test Names**
```typescript
// Good: Describes what should happen
it('should throw UserNotFoundException when user ID does not exist', async () => {
    // Test implementation
});

// Bad: Generic test name
it('should test getUserById', async () => {
    // Test implementation
});
```

### ❌ **DON'T: Test Implementation Details**
```typescript
// Bad: Testing how it works internally
it('should call userRepository.findById with correct parameters', async () => {
    await userService.getUserById(1);
    expect(mockUserRepository.findById).toHaveBeenCalledWith(1);
});

// Good: Testing behavior
it('should return user when valid ID is provided', async () => {
    const user = await userService.getUserById(1);
    expect(user).toBeDefined();
    expect(user.id).toBe(1);
});
```

### ❌ **DON'T: Share State Between Tests**
```typescript
// Bad: Tests depend on each other
let sharedUser;

it('should create user', async () => {
    sharedUser = await userService.createUser(userData);
});

it('should update user', async () => {
    await userService.updateUser(sharedUser.id, updateData);
});

// Good: Each test is independent
beforeEach(async () => {
    const testUser = await DatabaseHelper.createTestUser();
});
```

## Summary

Sosise's testing framework provides:

- 🎯 **Comprehensive Testing** - Unit, integration, and functional tests
- ⚡ **Fast Test Execution** - Optimized test runner with in-memory database
- 🔍 **Code Coverage** - Track coverage across your entire codebase
- 🛠️ **Rich Tooling** - Powerful helpers and utilities for testing
- 🔄 **CI/CD Integration** - Seamless integration with popular CI platforms
- 📊 **Detailed Reporting** - Clear test results and coverage reports
- 🧪 **Mocking Support** - Easy mocking of dependencies and external services
- 🎨 **Flexible Structure** - Organize tests however works best for your team

Build robust, reliable APIs with confidence using Sosise's comprehensive testing framework! ✅