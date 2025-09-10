# Unifiers

## Quick Start

Unifiers solve a common problem: validating and organizing incoming request data. Instead of validating data directly in your controllers, unifiers handle this automatically.

### Simple Example

Create a unifier to validate user registration:

```typescript
export default class RegisterUserUnifier {
    public name: string;
    public email: string;

    constructor(params: any) {
        // Validate the data
        const validator = new Validator(params);
        validator.field('name').required().shouldBeString().min(2);
        validator.field('email').required().shouldBeString().isEmail();
        
        if (validator.fails()) {
            throw new ValidationException('Invalid data', validator.errors);
        }

        // Map validated data to properties
        this.name = params.name;
        this.email = params.email;
    }
}
```

Use it in your controller:

```typescript
router.post('/register', (request: Request, response: Response, next: NextFunction) => {
    try {
        // Data is automatically validated and mapped
        const userData = new RegisterUserUnifier(request.body);
        
        // Use clean, validated data
        console.log(userData.name);  // Safe to use
        console.log(userData.email); // Safe to use
        
    } catch (error) {
        next(error); // Validation errors are handled automatically
    }
});
```

## What Are Unifiers?

Unifiers are classes that **validate** and **organize** incoming request data. They solve two main problems:

1. **Validation**: Ensure data meets your requirements (required fields, correct types, etc.)
2. **Organization**: Convert messy request data into clean, typed properties

### Why Use Unifiers?

**Without unifiers** (messy and error-prone):
```typescript
router.post('/users', (request: Request, response: Response, next: NextFunction) => {
    // Manual validation - easy to forget or get wrong
    if (!request.body.name || request.body.name.length < 2) {
        return response.status(400).send('Name is required');
    }
    if (!request.body.email || !isValidEmail(request.body.email)) {
        return response.status(400).send('Valid email is required');
    }
    
    // Using non-validated data directly
    createUser(request.body.name, request.body.email);
});
```

**With unifiers** (clean and safe):
```typescript
router.post('/users', (request: Request, response: Response, next: NextFunction) => {
    try {
        const userData = new CreateUserUnifier(request.body);
        createUser(userData.name, userData.email); // Clean, validated data
    } catch (error) {
        next(error); // Consistent error handling
    }
});
```

## Creating Unifiers

### Generate a New Unifier

```sh
./artisan make:unifier CreateUserUnifier
```

This creates a template unifier in `app/Unifiers/CreateUserUnifier.ts`.

### Basic Unifier Structure

```typescript
import Validator from 'sosise-core/build/Validator/Validator';
import ValidationException from 'sosise-core/build/Exceptions/Validation/ValidationException';

export default class CreateUserUnifier {
    // Define the properties you want to extract
    public name: string;
    public email: string;
    public age?: number; // Optional property

    constructor(params: any) {
        // Step 1: Validate the incoming data
        this.validate(params);
        
        // Step 2: Map validated data to properties
        this.map(params);
    }

    private validate(params: any) {
        const validator = new Validator(params);
        
        // Define validation rules
        validator.field('name').required().shouldBeString().min(2);
        validator.field('email').required().shouldBeString().isEmail();
        validator.field('age').optional().shouldBeNumber().min(18);
        
        if (validator.fails()) {
            throw new ValidationException('Validation failed', validator.errors);
        }
    }

    private map(params: any) {
        // Map validated data to clean properties
        this.name = params.name;
        this.email = params.email;
        this.age = params.age;
    }
}
```

## Validation Rules

### Common Validation Methods

```typescript
validator.field('fieldName')
    .required()                    // Field must be present
    .optional()                    // Field is optional
    .shouldBeString()              // Must be a string
    .shouldBeNumber()              // Must be a number
    .shouldBeBoolean()             // Must be true/false
    .min(5)                        // Minimum length/value
    .max(100)                      // Maximum length/value
    .isEmail()                     // Must be valid email
    .in(['option1', 'option2'])    // Must be one of these values
```

### Validation Examples

```typescript
export default class ProductUnifier {
    public name: string;
    public price: number;
    public category: string;
    public description?: string;

    constructor(params: any) {
        const validator = new Validator(params);
        
        // Product name: required string, 2-100 characters
        validator.field('name')
            .required()
            .shouldBeString()
            .min(2)
            .max(100);
            
        // Price: required number, minimum $0.01
        validator.field('price')
            .required()
            .shouldBeNumber()
            .min(0.01);
            
        // Category: must be one of these options
        validator.field('category')
            .required()
            .shouldBeString()
            .in(['electronics', 'clothing', 'books']);
            
        // Description: optional string
        validator.field('description')
            .optional()
            .shouldBeString()
            .max(500);

        if (validator.fails()) {
            throw new ValidationException('Invalid product data', validator.errors);
        }

        // Map the data
        this.name = params.name;
        this.price = params.price;
        this.category = params.category;
        this.description = params.description;
    }
}
```

## Using Unifiers in Controllers

### Basic Usage

```typescript
import CreateUserUnifier from '../../Unifiers/CreateUserUnifier';

export default class UserController {
    public async createUser(request: Request, response: Response, next: NextFunction) {
        try {
            // Validate and map request data
            const userData = new CreateUserUnifier(request.body);
            
            // Use the clean data
            const user = await this.userService.create({
                name: userData.name,
                email: userData.email,
                age: userData.age
            });

            response.json({ user });
        } catch (error) {
            next(error);
        }
    }
}
```

### Handling Different Request Data Sources

```typescript
// For URL parameters
const unifier = new GetUserUnifier(request.params);

// For query parameters  
const unifier = new SearchUsersUnifier(request.query);

// For POST body data
const unifier = new CreateUserUnifier(request.body);

// For mixed data sources
const unifier = new UpdateUserUnifier({
    ...request.params,  // User ID from URL
    ...request.body     // Update data from body
});
```

## Error Handling

When validation fails, unifiers automatically throw a `ValidationException` with helpful error details:

```typescript
// If validation fails, you get structured errors
try {
    const userData = new CreateUserUnifier(request.body);
} catch (error) {
    // error.errors contains detailed validation failures:
    // {
    //   "name": ["Name is required", "Name must be at least 2 characters"],
    //   "email": ["Email is required", "Email must be valid"]
    // }
}
```

## Advanced Examples

### Complex Validation Logic

```typescript
export default class OrderUnifier {
    public customerId: number;
    public items: Array<{productId: number, quantity: number}>;
    public shippingAddress: string;

    constructor(params: any) {
        const validator = new Validator(params);
        
        validator.field('customerId').required().shouldBeNumber().min(1);
        validator.field('shippingAddress').required().shouldBeString().min(10);
        
        // Validate array of items
        validator.field('items').required().shouldBeArray();
        
        if (validator.fails()) {
            throw new ValidationException('Invalid order data', validator.errors);
        }

        // Additional custom validation
        if (!Array.isArray(params.items) || params.items.length === 0) {
            throw new ValidationException('Order must contain at least one item', {});
        }

        this.customerId = params.customerId;
        this.items = params.items;
        this.shippingAddress = params.shippingAddress;
    }
}
```

### Transforming Data During Mapping

```typescript
export default class UserPreferencesUnifier {
    public theme: 'light' | 'dark';
    public notifications: boolean;
    public timezone: string;

    constructor(params: any) {
        // Validate...
        const validator = new Validator(params);
        validator.field('theme').required().in(['light', 'dark']);
        // ... other validations

        if (validator.fails()) {
            throw new ValidationException('Invalid preferences', validator.errors);
        }

        // Transform and clean data during mapping
        this.theme = params.theme.toLowerCase();
        this.notifications = Boolean(params.notifications);
        this.timezone = params.timezone || 'UTC';
    }
}
```

## Best Practices

1. **One unifier per endpoint**: Create specific unifiers for each API endpoint
2. **Keep validation simple**: Complex business logic belongs in services, not unifiers
3. **Use descriptive names**: `CreateUserUnifier`, `UpdateProductUnifier`, etc.
4. **Handle optional fields**: Use `.optional()` for fields that aren't required
5. **Transform consistently**: Apply the same data transformations across your app
6. **Test your unifiers**: Write tests to ensure validation works correctly

## Summary

Unifiers provide a clean, consistent way to validate and organize incoming request data. They help you:

- ✅ Validate data automatically
- ✅ Get clean, typed properties to work with  
- ✅ Handle errors consistently
- ✅ Keep controllers focused on business logic
- ✅ Make your code more maintainable and testable