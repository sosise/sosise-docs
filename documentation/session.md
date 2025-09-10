# HTTP Session Management

## Quick Start

Keep user data across requests with Sosise's simple session management!

```typescript
// Enable sessions in src/config/session.ts
export default {
    enabled: true,
    driver: 'file', // or 'memory', 'redis'
    secret: process.env.SESSION_SECRET
};

// Use in any controller
export default class AuthController {
    public async login(request: Request, response: Response): Promise<void> {
        const userData = new LoginUnifier(request.body);
        const user = await this.authService.authenticate(userData);
        
        // Store user in session
        request.session.userId = user.id;
        request.session.email = user.email;
        
        response.json({ message: 'Logged in successfully' });
    }
    
    public async profile(request: Request, response: Response): Promise<void> {
        // Access session data
        if (request.session.userId) {
            const user = await this.userService.getUser(request.session.userId);
            response.json({ user });
        } else {
            response.status(401).json({ error: 'Not authenticated' });
        }
    }
}
```

Perfect for authentication, shopping carts, and user preferences!

## Introduction

HTTP is stateless by design - each request is independent with no memory of previous interactions. Sessions solve this by storing user-specific data server-side and linking it to the client through cookies or tokens.

Sosise provides a unified session API that works with multiple storage backends (file, memory, Redis) and integrates seamlessly with your authentication flow. Whether you're building user authentication, shopping carts, or multi-step forms, sessions make it easy to maintain state across requests.

## Configuration and Setup

### Enable Sessions

Configure sessions in `src/config/session.ts`:

```typescript
export default {
    // Enable session handling
    enabled: true,
    
    // Storage driver
    driver: 'file', // 'file', 'memory', 'redis'
    
    // Session configuration
    name: 'sosise-session',
    secret: process.env.SESSION_SECRET || 'your-secret-key',
    
    // Cookie settings
    cookie: {
        secure: process.env.NODE_ENV === 'production', // HTTPS only in production
        httpOnly: true, // Prevent XSS attacks
        maxAge: 24 * 60 * 60 * 1000, // 24 hours
        sameSite: 'strict' // CSRF protection
    },
    
    // File driver settings
    fileStore: {
        path: 'storage/sessions',
        ttl: 86400 // 24 hours in seconds
    },
    
    // Redis driver settings (if using Redis)
    redisStore: {
        host: process.env.REDIS_HOST || 'localhost',
        port: parseInt(process.env.REDIS_PORT || '6379'),
        password: process.env.REDIS_PASSWORD,
        database: 2
    }
};
```

### Environment Variables

```bash
# .env
SESSION_SECRET=your-super-secret-session-key-min-32-chars
SESSION_DRIVER=file
REDIS_HOST=localhost
REDIS_PORT=6379
```

### Session Middleware

Ensure session middleware is registered in your application:

```typescript
// src/app.ts - Sessions are automatically configured
// No additional setup needed!
```

## Authentication with Sessions

### Login Flow

```typescript
// src/app/Services/AuthService.ts
export default class AuthService {
    constructor(
        private userRepository: UserRepository,
        private logger: LoggerService
    ) {}

    public async login(credentials: LoginCredentials, request: Request): Promise<User> {
        const user = await this.validateCredentials(credentials);
        
        if (!user) {
            throw new InvalidCredentialsError('Invalid email or password');
        }
        
        // Store user data in session
        request.session.userId = user.id;
        request.session.email = user.email;
        request.session.role = user.role;
        request.session.loginTime = new Date().toISOString();
        
        // Security: Regenerate session ID to prevent fixation attacks
        await this.regenerateSession(request);
        
        this.logger.info('User logged in successfully', {
            userId: user.id,
            email: user.email,
            sessionId: request.sessionID
        });
        
        return user;
    }
    
    public async logout(request: Request): Promise<void> {
        const userId = request.session.userId;
        
        // Clear all session data
        request.session.destroy((error) => {
            if (error) {
                this.logger.error('Session destruction failed', { error });
            }
        });
        
        this.logger.info('User logged out', { userId });
    }
    
    private async regenerateSession(request: Request): Promise<void> {
        return new Promise((resolve, reject) => {
            request.session.regenerate((error) => {
                if (error) {
                    this.logger.error('Session regeneration failed', { error });
                    reject(error);
                } else {
                    resolve();
                }
            });
        });
    }
}
```

### Authentication Middleware

```typescript
// src/app/Http/Middlewares/AuthMiddleware.ts
export default class AuthMiddleware {
    constructor(
        private userService: UserService,
        private logger: LoggerService
    ) {}

    public async handle(request: Request, response: Response, next: NextFunction): Promise<void> {
        try {
            // Check if user is logged in via session
            if (!request.session.userId) {
                return response.status(401).json({
                    code: 2001,
                    message: 'Authentication required'
                });
            }
            
            // Load user data and attach to request
            const user = await this.userService.getUser(request.session.userId);
            
            if (!user) {
                // User was deleted, clear invalid session
                request.session.destroy();
                return response.status(401).json({
                    code: 2002,
                    message: 'Invalid session'
                });
            }
            
            // Attach user to request for controllers
            request.user = user;
            
            // Update last activity
            request.session.lastActivity = new Date().toISOString();
            
            next();
        } catch (error) {
            this.logger.error('Authentication middleware error', { error });
            next(error);
        }
    }
}
```

## Session Data Management

### Storing Different Data Types

```typescript
export default class ShoppingCartService {
    public async addToCart(request: Request, productId: number, quantity: number): Promise<void> {
        // Initialize cart if it doesn't exist
        if (!request.session.cart) {
            request.session.cart = {
                items: [],
                total: 0,
                createdAt: new Date().toISOString()
            };
        }
        
        // Add item to cart
        const existingItem = request.session.cart.items.find(item => item.productId === productId);
        
        if (existingItem) {
            existingItem.quantity += quantity;
        } else {
            const product = await this.productService.getProduct(productId);
            request.session.cart.items.push({
                productId,
                name: product.name,
                price: product.price,
                quantity
            });
        }
        
        // Recalculate total
        request.session.cart.total = this.calculateCartTotal(request.session.cart.items);
        request.session.cart.updatedAt = new Date().toISOString();
    }
    
    public getCart(request: Request): ShoppingCart {
        return request.session.cart || { items: [], total: 0 };
    }
    
    public clearCart(request: Request): void {
        delete request.session.cart;
    }
}
```

### User Preferences and Settings

```typescript
export default class UserPreferencesService {
    public async updatePreferences(request: Request, preferences: UserPreferences): Promise<void> {
        // Store preferences in session for immediate use
        request.session.preferences = {
            language: preferences.language || 'en',
            theme: preferences.theme || 'light',
            timezone: preferences.timezone || 'UTC',
            notifications: preferences.notifications || true,
            updatedAt: new Date().toISOString()
        };
        
        // Also save to database for persistence
        if (request.session.userId) {
            await this.userRepository.updatePreferences(
                request.session.userId, 
                request.session.preferences
            );
        }
    }
    
    public getPreferences(request: Request): UserPreferences {
        return request.session.preferences || this.getDefaultPreferences();
    }
}
```

### Multi-Step Form Data

```typescript
export default class RegistrationController {
    public async step1(request: Request, response: Response): Promise<void> {
        const personalData = new PersonalDataUnifier(request.body);
        
        // Store step 1 data in session
        request.session.registration = {
            step: 1,
            personalData,
            completedAt: new Date().toISOString()
        };
        
        response.json({
            code: 1000,
            message: 'Step 1 completed',
            nextStep: '/register/step2'
        });
    }
    
    public async step2(request: Request, response: Response): Promise<void> {
        // Validate previous step exists
        if (!request.session.registration || request.session.registration.step < 1) {
            return response.status(400).json({
                code: 3001,
                message: 'Please complete step 1 first',
                redirectTo: '/register/step1'
            });
        }
        
        const addressData = new AddressDataUnifier(request.body);
        
        // Add step 2 data
        request.session.registration.step = 2;
        request.session.registration.addressData = addressData;
        request.session.registration.updatedAt = new Date().toISOString();
        
        response.json({
            code: 1000,
            message: 'Step 2 completed',
            nextStep: '/register/complete'
        });
    }
    
    public async complete(request: Request, response: Response): Promise<void> {
        const registration = request.session.registration;
        
        if (!registration || registration.step < 2) {
            return response.status(400).json({
                code: 3002,
                message: 'Please complete all previous steps'
            });
        }
        
        try {
            // Create user with all collected data
            const user = await this.userService.createUser({
                ...registration.personalData,
                ...registration.addressData
            });
            
            // Clear registration data and login user
            delete request.session.registration;
            request.session.userId = user.id;
            
            response.json({
                code: 1000,
                message: 'Registration completed',
                user
            });
        } catch (error) {
            throw new RegistrationError('Failed to complete registration');
        }
    }
}
```

## Session Security

### Session Fixation Prevention

```typescript
export default class SecurityService {
    public async regenerateSessionOnLogin(request: Request): Promise<void> {
        return new Promise((resolve, reject) => {
            const userData = { ...request.session };
            
            request.session.regenerate((error) => {
                if (error) {
                    reject(error);
                } else {
                    // Restore session data after regeneration
                    Object.assign(request.session, userData);
                    resolve();
                }
            });
        });
    }
    
    public async validateSession(request: Request): Promise<boolean> {
        // Check session age
        const loginTime = new Date(request.session.loginTime);
        const maxAge = 24 * 60 * 60 * 1000; // 24 hours
        
        if (Date.now() - loginTime.getTime() > maxAge) {
            request.session.destroy();
            return false;
        }
        
        // Check for suspicious activity
        if (request.session.lastIp && request.session.lastIp !== request.ip) {
            this.logger.warn('IP address changed during session', {
                userId: request.session.userId,
                oldIp: request.session.lastIp,
                newIp: request.ip
            });
        }
        
        // Update session tracking
        request.session.lastIp = request.ip;
        request.session.lastActivity = new Date().toISOString();
        
        return true;
    }
}
```

### Session Data Validation

```typescript
export default class SessionValidationService {
    public validateSessionData(request: Request): boolean {
        // Validate user session
        if (request.session.userId && typeof request.session.userId !== 'number') {
            this.logger.warn('Invalid userId in session', {
                sessionId: request.sessionID,
                userId: request.session.userId
            });
            this.clearSession(request);
            return false;
        }
        
        // Validate cart data
        if (request.session.cart && !this.isValidCart(request.session.cart)) {
            this.logger.warn('Invalid cart data in session');
            delete request.session.cart;
        }
        
        return true;
    }
    
    private isValidCart(cart: any): boolean {
        return cart && 
               Array.isArray(cart.items) && 
               typeof cart.total === 'number' &&
               cart.total >= 0;
    }
    
    private clearSession(request: Request): void {
        request.session.destroy((error) => {
            if (error) {
                this.logger.error('Failed to destroy invalid session', { error });
            }
        });
    }
}
```

## Advanced Session Patterns

### Session-Based Flash Messages

```typescript
export default class FlashMessageService {
    public addSuccessMessage(request: Request, message: string): void {
        if (!request.session.flash) {
            request.session.flash = {};
        }
        
        if (!request.session.flash.success) {
            request.session.flash.success = [];
        }
        
        request.session.flash.success.push(message);
    }
    
    public addErrorMessage(request: Request, message: string): void {
        if (!request.session.flash) {
            request.session.flash = {};
        }
        
        if (!request.session.flash.error) {
            request.session.flash.error = [];
        }
        
        request.session.flash.error.push(message);
    }
    
    public getAndClearMessages(request: Request): FlashMessages {
        const messages = request.session.flash || {};
        delete request.session.flash;
        return messages;
    }
}

// Use in controllers
export default class ProductController {
    public async update(request: Request, response: Response): Promise<void> {
        try {
            const productData = new UpdateProductUnifier(request.body);
            await this.productService.updateProduct(productData);
            
            this.flashService.addSuccessMessage(request, 'Product updated successfully');
            response.redirect('/products');
        } catch (error) {
            this.flashService.addErrorMessage(request, 'Failed to update product');
            response.redirect('back');
        }
    }
}
```

### Session Analytics

```typescript
export default class SessionAnalyticsService {
    public trackPageView(request: Request, page: string): void {
        if (!request.session.analytics) {
            request.session.analytics = {
                pageViews: [],
                sessionStart: new Date().toISOString(),
                userAgent: request.get('User-Agent')
            };
        }
        
        request.session.analytics.pageViews.push({
            page,
            timestamp: new Date().toISOString(),
            referrer: request.get('Referrer')
        });
    }
    
    public getSessionStats(request: Request): SessionStats {
        const analytics = request.session.analytics;
        if (!analytics) return null;
        
        const duration = Date.now() - new Date(analytics.sessionStart).getTime();
        
        return {
            duration,
            pageViews: analytics.pageViews.length,
            uniquePages: new Set(analytics.pageViews.map(pv => pv.page)).size,
            startTime: analytics.sessionStart,
            userAgent: analytics.userAgent
        };
    }
}
```

## Storage Drivers

### File Storage (Default)

```typescript
// Automatic - no additional configuration needed
// Sessions stored in storage/sessions/
```

### Memory Storage

```typescript
// src/config/session.ts
export default {
    driver: 'memory',
    // Fast but data lost on restart
    // Good for development/testing
};
```

### Redis Storage

```typescript
// src/config/session.ts
export default {
    driver: 'redis',
    redisStore: {
        host: process.env.REDIS_HOST,
        port: parseInt(process.env.REDIS_PORT),
        password: process.env.REDIS_PASSWORD,
        database: 2,
        keyPrefix: 'session:'
    }
};
```

## Best Practices

### 1. Minimize Session Data

```typescript
// ✅ Good: Store only essential data
request.session.userId = user.id;
request.session.role = user.role;

// ❌ Bad: Storing large objects
request.session.user = user; // Contains passwords, large data, etc.
```

### 2. Validate and Sanitize

```typescript
// ✅ Good: Always validate session data
export default class CartService {
    public getCart(request: Request): ShoppingCart {
        const cart = request.session.cart;
        
        if (cart && this.isValidCart(cart)) {
            return cart;
        }
        
        // Return clean cart if invalid
        return { items: [], total: 0 };
    }
}
```

### 3. Handle Session Expiry

```typescript
// ✅ Good: Graceful session expiry handling
export default class AuthMiddleware {
    public async handle(request: Request, response: Response, next: NextFunction): Promise<void> {
        if (!request.session.userId) {
            return response.status(401).json({
                code: 2001,
                message: 'Session expired, please login again',
                redirectTo: '/login'
            });
        }
        
        next();
    }
}
```

### 4. Use Secure Settings

```typescript
// ✅ Good: Production-ready session config
export default {
    cookie: {
        secure: true,      // HTTPS only
        httpOnly: true,    // No JavaScript access
        sameSite: 'strict' // CSRF protection
    },
    secret: process.env.SESSION_SECRET, // Strong secret
    name: 'sessionId'      // Don't reveal framework
};
```

## Summary

Sosise session management provides:

- ✅ Multiple storage backends (file, memory, Redis)
- ✅ Secure cookie configuration
- ✅ Session fixation protection
- ✅ Easy authentication integration
- ✅ Shopping cart and preferences support
- ✅ Multi-step form handling
- ✅ Flash message system
- ✅ Analytics and tracking

Build stateful applications with confidence using Sosise's flexible session system!