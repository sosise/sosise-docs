# Helpers

## Quick Start

Sosise helpers make common tasks super easy! Need to debug, sleep, or parse dates? We've got you covered.

```typescript
import Helper from 'sosise-core/build/Helper/Helper';

// Debug variables (dumps and stops execution)
Helper.dd(userObject); 

// Sleep for 2 seconds
await Helper.sleep(2000);

// Parse any date format to standard format
const standardDate = Helper.parseDate('25.12.2023'); // "2023-12-25"

// Get only specific fields from objects
const userInfo = Helper.pluckMany(user, ['name', 'email']);
```

Perfect for development, debugging, and data manipulation!

## Introduction

Helpers are utility functions that solve common development problems with minimal code. Instead of writing the same utility logic over and over, Sosise provides battle-tested helpers that handle edge cases and provide consistent results.

These functions are particularly useful for debugging, data transformation, file path operations, date handling, and performance profiling.

## Debugging Helpers

### Debug and Die (dd)

Stop execution and inspect variables in detail:

```typescript
import Helper from 'sosise-core/build/Helper/Helper';

// Debug a single variable
Helper.dd(complexObject);

// Debug multiple variables at once
Helper.dd(user, order, paymentDetails);

// Great for debugging in services
export default class OrderService {
    public async processOrder(order: Order): Promise<void> {
        const calculations = await this.calculateTotal(order);
        Helper.dd(calculations); // Stops here and shows the data
        
        // This line won't execute
        await this.saveOrder(order);
    }
}
```

### Debug Without Stopping

Use when you want to inspect but continue execution:

```typescript
// Dump variable and continue
Helper.dump(responseData);
console.log('This will still execute');

// Useful in loops
users.forEach(user => {
    Helper.dump(user.permissions);
    // Process user...
});
```

### Performance Profiling

Measure execution time for optimization:

```typescript
export default class ReportService {
    public async generateReport(): Promise<Report> {
        Helper.startProfiling();
        
        const data = await this.fetchLargeDataset();
        Helper.stopProfiling(); // Shows time elapsed
        
        Helper.startProfiling();
        const processedData = await this.processData(data);
        Helper.stopProfiling(); // Shows time for processing step
        
        return this.formatReport(processedData);
    }
}
```

## Async Utilities

### Sleep Function

Clean delays without Promise boilerplate:

```typescript
// Instead of this verbose approach
await new Promise(resolve => setTimeout(resolve, 1000));

// Use this clean syntax
await Helper.sleep(1000);

// Practical examples
export default class EmailService {
    public async sendWithRetry(email: Email): Promise<void> {
        for (let attempt = 1; attempt <= 3; attempt++) {
            try {
                await this.sendEmail(email);
                return;
            } catch (error) {
                if (attempt < 3) {
                    console.log(`Attempt ${attempt} failed, retrying in 2 seconds...`);
                    await Helper.sleep(2000); // Wait before retry
                } else {
                    throw error;
                }
            }
        }
    }
}
```

## Date and Time Helpers

### Date Parsing

Handle various date formats consistently:

```typescript
// Parse different date formats to standard YYYY-MM-DD
Helper.parseDate('25.12.2023');    // "2023-12-25"
Helper.parseDate('12/25/2023');    // "2023-12-25" 
Helper.parseDate('2023-12-25');    // "2023-12-25"
Helper.parseDate('invalid');       // null

// Parse date with time
Helper.parseDateTime('25.12.2023 14:30:45'); // "2023-12-25 14:30:45"
Helper.parseDateTime('12/25/2023 2:30 PM');  // "2023-12-25 14:30:00"

// Current timestamp
const now = Helper.getCurrentDateTime(); // "2023-12-25 14:30:45"
```

### Real-world Date Usage

```typescript
export default class EventService {
    public async createEvent(eventData: CreateEventData): Promise<Event> {
        // Parse user-provided dates safely
        const startDate = Helper.parseDateTime(eventData.startDate);
        const endDate = Helper.parseDateTime(eventData.endDate);
        
        if (!startDate || !endDate) {
            throw new InvalidDateFormatError('Event dates must be valid');
        }
        
        return await this.eventRepository.create({
            ...eventData,
            startDate,
            endDate,
            createdAt: Helper.getCurrentDateTime()
        });
    }
}
```

## Path Helpers

### Project and Storage Paths

Get consistent file paths across environments:

```typescript
// Get project root with trailing slash
const projectRoot = Helper.projectPath();
// Returns: "/var/www/my-app/"

// Get storage directory with trailing slash  
const storageDir = Helper.storagePath();
// Returns: "/var/www/my-app/storage/"

// Use in file operations
export default class FileService {
    public async saveUpload(filename: string, content: Buffer): Promise<string> {
        const filePath = Helper.storagePath() + 'uploads/' + filename;
        await fs.writeFile(filePath, content);
        return filePath;
    }
    
    public async readConfig(configFile: string): Promise<any> {
        const configPath = Helper.projectPath() + 'config/' + configFile;
        return await fs.readJSON(configPath);
    }
}
```

## Data Manipulation Helpers

### Extract Specific Fields

Get only the data you need:

```typescript
const user = {
    id: 123,
    name: 'John Doe',
    email: 'john@example.com',
    password: 'secret',
    internalNotes: 'VIP customer'
};

// Extract safe fields for API response
const safeUser = Helper.pluckMany(user, ['id', 'name', 'email']);
// Result: { id: 123, name: 'John Doe', email: 'john@example.com' }

// Works with arrays too
const users = [
    { id: 1, name: 'Alice', role: 'admin', password: 'secret' },
    { id: 2, name: 'Bob', role: 'user', password: 'secret' }
];

const publicUsers = Helper.pluckMany(users, ['id', 'name', 'role']);
// Result: [
//   { id: 1, name: 'Alice', role: 'admin' },
//   { id: 2, name: 'Bob', role: 'user' }
// ]
```

### Array Pagination

Paginate data in memory:

```typescript
const allProducts = [
    { id: 1, name: 'Product 1' },
    { id: 2, name: 'Product 2' },
    { id: 3, name: 'Product 3' },
    { id: 4, name: 'Product 4' },
    { id: 5, name: 'Product 5' }
];

// Get page 1 with 2 items per page
const page1 = Helper.paginateArray(allProducts, 1, 2);
// Result: [{ id: 1, name: 'Product 1' }, { id: 2, name: 'Product 2' }]

// Get page 2 
const page2 = Helper.paginateArray(allProducts, 2, 2);
// Result: [{ id: 3, name: 'Product 3' }, { id: 4, name: 'Product 4' }]

// Generate pagination metadata
const pagination = Helper.assemblePagination(allProducts, 1, 2);
// Result: { page: 1, pageSize: 2, totalPages: 3, totalElements: 5 }
```

### Practical Pagination Example

```typescript
export default class SearchService {
    public async searchInMemory(query: string, page: number = 1): Promise<PaginatedResults> {
        // Get all matching results
        const allResults = await this.performComplexSearch(query);
        
        // Paginate the results
        const pageSize = 20;
        const items = Helper.paginateArray(allResults, page, pageSize);
        const pagination = Helper.assemblePagination(allResults, page, pageSize);
        
        return {
            items,
            pagination
        };
    }
}
```

## Advanced Usage Patterns

### Combining Helpers

```typescript
export default class AnalyticsService {
    public async generateReport(filters: ReportFilters): Promise<Report> {
        Helper.startProfiling();
        
        // Parse date filters
        const startDate = Helper.parseDate(filters.startDate);
        const endDate = Helper.parseDate(filters.endDate);
        
        if (!startDate || !endDate) {
            Helper.dd({ filters, startDate, endDate }); // Debug invalid dates
        }
        
        const data = await this.fetchAnalyticsData(startDate, endDate);
        
        // Extract only needed fields for report
        const reportData = Helper.pluckMany(data, [
            'date', 'revenue', 'orders', 'customers'
        ]);
        
        Helper.stopProfiling(); // See how long it took
        
        return {
            data: reportData,
            generatedAt: Helper.getCurrentDateTime(),
            period: { startDate, endDate }
        };
    }
}
```

### Debugging Complex Objects

```typescript
export default class OrderProcessor {
    public async processComplexOrder(order: ComplexOrder): Promise<void> {
        // Debug the full order structure
        Helper.dump(order);
        
        // Process each item
        for (const item of order.items) {
            try {
                await this.processItem(item);
            } catch (error) {
                // Debug problematic items
                Helper.dd({
                    item,
                    error: error.message,
                    order: Helper.pluckMany(order, ['id', 'customer', 'total'])
                });
            }
        }
    }
}
```

## Best Practices

### 1. Remove Debug Helpers Before Production

```typescript
// ✅ Good: Use environment checks
if (process.env.NODE_ENV === 'development') {
    Helper.dd(debugData);
}

// ✅ Good: Use for specific debugging sessions
// Helper.dd(userData); // TODO: Remove after debugging login issue

// ❌ Bad: Leaving debug calls in production
Helper.dd(sensitiveData); // Will expose data and crash app!
```

### 2. Use Profiling for Optimization

```typescript
// ✅ Good: Profile expensive operations
export default class DataProcessor {
    public async processLargeDataset(data: any[]): Promise<any[]> {
        Helper.startProfiling();
        
        const result = await this.expensiveTransformation(data);
        
        Helper.stopProfiling(); // Monitor performance
        return result;
    }
}
```

### 3. Consistent Date Handling

```typescript
// ✅ Good: Always validate parsed dates
const parsedDate = Helper.parseDate(userInput);
if (!parsedDate) {
    throw new InvalidDateError('Invalid date format provided');
}

// ✅ Good: Use for consistent formatting
const event = {
    startDate: Helper.parseDateTime(input.startDate),
    createdAt: Helper.getCurrentDateTime()
};
```

### 4. Safe Data Extraction

```typescript
// ✅ Good: Extract only needed fields for API responses
const userResponse = Helper.pluckMany(user, ['id', 'name', 'email', 'avatar']);

// ✅ Good: Remove sensitive data from logs
const logData = Helper.pluckMany(requestData, ['userId', 'action', 'timestamp']);
this.logger.info('User action', logData);
```

## Summary

Sosise helpers provide:

- ✅ Quick debugging with `dd()` and `dump()`
- ✅ Clean async delays with `sleep()`
- ✅ Robust date parsing and formatting
- ✅ Consistent file path handling
- ✅ Easy data extraction and pagination
- ✅ Performance profiling tools
- ✅ Type-safe utility functions

Use helpers to write cleaner, more maintainable code with less boilerplate!