# Artisan

## Quick Start

Artisan is Sosise's command-line tool that helps you build and manage your application. Think of it like a helpful assistant that can generate files, run tasks, and automate repetitive work.

### Basic Commands

```sh
# See all available commands
./artisan

# Get help for a specific command
./artisan make:controller --help

# Generate a new controller
./artisan make:controller UserController

# Generate a new unifier
./artisan make:unifier CreateUserUnifier

# Run a custom command
./artisan my:custom-task
```

That's the basics! Artisan makes development faster and more consistent.

## Introduction

Artisan is the command-line interface for Sosise applications. It provides helpful commands for generating files, running tasks, and managing your application. Every Sosise project comes with Artisan built-in.

## Using Built-in Commands

### View All Commands

```sh
./artisan
```

This shows all available Artisan commands with brief descriptions.

### Get Command Help

```sh
./artisan make:controller --help
./artisan make:unifier --help
```

Every command includes help information showing available options and usage examples.

### Common Built-in Commands

```sh
# Generate files
./artisan make:controller ProductController
./artisan make:unifier CreateProductUnifier
./artisan make:command ProcessOrdersCommand

# These create files with proper templates and naming
```

## Creating Custom Commands

Custom commands let you automate tasks specific to your application, like data processing, report generation, or maintenance tasks.

### Generate a Custom Command

```sh
./artisan make:command SendEmailReportCommand
```

> **Important**: Always end custom command names with "Command" or they won't be registered properly.

This creates a new command file in `app/Console/Commands/SendEmailReportCommand.ts`.

### Simple Command Example

Here's what a basic custom command looks like:

```typescript
import commander from 'commander';
import BaseCommand from 'sosise-core/build/Command/BaseCommand';

export default class SendEmailReportCommand extends BaseCommand {
    // The command name (how you'll call it)
    protected signature: string = 'email:report';

    // Description shown in command list
    protected description: string = 'Send daily email report to managers';

    // The actual work the command does
    public async handle(cli: commander.Command): Promise<void> {
        console.log('Generating daily report...');
        
        // Your command logic here
        const reportData = await this.generateReport();
        await this.sendEmail(reportData);
        
        console.log('Report sent successfully!');
    }

    private async generateReport() {
        // Generate your report data
        return { users: 100, orders: 50 };
    }

    private async sendEmail(data: any) {
        // Send the email
        console.log('Sending email with data:', data);
    }
}
```

### Using Your Command

```sh
./artisan email:report
```

## Command Options

Options let users customize how your command runs:

### Command with Simple Options

```typescript
export default class BackupCommand extends BaseCommand {
    protected signature: string = 'backup:database';
    protected description: string = 'Backup the database';

    // Define command options
    protected options: OptionType[] = [
        // Simple flag option (true/false)
        { 
            flag: '-c, --compress', 
            description: 'Compress the backup file', 
            required: false 
        },
        
        // Option that accepts a value
        { 
            flag: '-p, --path <path>', 
            description: 'Backup destination path', 
            default: './backups', 
            required: false 
        },
    ];

    public async handle(cli: commander.Command): Promise<void> {
        // Check if compress flag was provided
        if (cli.compress) {
            console.log('Creating compressed backup...');
        }

        // Get the path value (uses default if not provided)
        const backupPath = cli.path;
        console.log(`Backing up to: ${backupPath}`);
        
        // Your backup logic here
    }
}
```

### Using Commands with Options

```sh
# Basic usage
./artisan backup:database

# With compression
./artisan backup:database --compress

# With custom path
./artisan backup:database --path /custom/backup/location

# With both options
./artisan backup:database --compress --path /custom/backup/location
```

## Real-World Examples

### Data Processing Command

```typescript
export default class ProcessOrdersCommand extends BaseCommand {
    protected signature: string = 'orders:process';
    protected description: string = 'Process pending orders';

    protected options: OptionType[] = [
        { 
            flag: '-l, --limit <limit>', 
            description: 'Number of orders to process', 
            default: '100', 
            required: false 
        },
        { 
            flag: '--dry-run', 
            description: 'Show what would be processed without actually doing it', 
            required: false 
        }
    ];

    public async handle(cli: commander.Command): Promise<void> {
        const limit = parseInt(cli.limit);
        const isDryRun = cli.dryRun;

        console.log(`Processing ${limit} orders${isDryRun ? ' (dry run)' : ''}...`);

        // Get pending orders from database
        const orders = await this.getPendingOrders(limit);
        
        for (const order of orders) {
            if (isDryRun) {
                console.log(`Would process order ${order.id}`);
            } else {
                await this.processOrder(order);
                console.log(`Processed order ${order.id}`);
            }
        }

        console.log('Done!');
    }

    private async getPendingOrders(limit: number) {
        // Get orders from your database
        return []; // Your implementation
    }

    private async processOrder(order: any) {
        // Process the order
        console.log('Processing order...');
    }
}
```

### Cleanup Command

```typescript
export default class CleanupFilesCommand extends BaseCommand {
    protected signature: string = 'cleanup:files';
    protected description: string = 'Clean up old temporary files';

    protected options: OptionType[] = [
        { 
            flag: '-d, --days <days>', 
            description: 'Delete files older than N days', 
            default: '30', 
            required: false 
        }
    ];

    public async handle(cli: commander.Command): Promise<void> {
        const days = parseInt(cli.days);
        
        console.log(`Cleaning up files older than ${days} days...`);

        const deletedCount = await this.cleanupFiles(days);
        
        console.log(`Deleted ${deletedCount} files`);
    }

    private async cleanupFiles(days: number): Promise<number> {
        // Your file cleanup logic
        return 0; // Return number of files deleted
    }
}
```

## Command Safety Features

### Prevent Double Execution

For commands that shouldn't run multiple times simultaneously:

```typescript
export default class ImportDataCommand extends BaseCommand {
    protected signature: string = 'import:data';
    protected description: string = 'Import data from external source';
    
    // Prevent this command from running if it's already running
    protected singleExecution: boolean = true;

    public async handle(cli: commander.Command): Promise<void> {
        console.log('Starting data import...');
        // Long-running import process
        await this.importData();
        console.log('Import completed!');
    }

    private async importData() {
        // Your import logic that might take a while
    }
}
```

## Best Practices

### 1. Use Clear Command Names

```typescript
// Good: Clear and specific
protected signature: string = 'orders:process-pending';
protected signature: string = 'users:send-welcome-emails';
protected signature: string = 'reports:generate-monthly';

// Avoid: Vague or confusing
protected signature: string = 'do-stuff';
protected signature: string = 'task';
```

### 2. Provide Good Descriptions

```typescript
// Good: Explains what the command does and when to use it
protected description: string = 'Process pending orders and send confirmation emails';

// Avoid: Too brief or unclear
protected description: string = 'Process orders';
```

### 3. Use IoC Services

```typescript
import IOC from 'sosise-core/build/ServiceProviders/IOC';
import EmailService from '../Services/EmailService';
import OrderService from '../Services/OrderService';

export default class ProcessOrdersCommand extends BaseCommand {
    public async handle(cli: commander.Command): Promise<void> {
        // Use your services through IoC
        const orderService = IOC.make(OrderService) as OrderService;
        const emailService = IOC.make(EmailService) as EmailService;
        
        const orders = await orderService.getPending();
        for (const order of orders) {
            await orderService.process(order);
            await emailService.sendConfirmation(order.customerEmail);
        }
    }
}
```

### 4. Handle Errors Gracefully

```typescript
public async handle(cli: commander.Command): Promise<void> {
    try {
        console.log('Starting process...');
        await this.doSomethingThatMightFail();
        console.log('Process completed successfully!');
    } catch (error) {
        console.error('Process failed:', error.message);
        process.exit(1); // Exit with error code
    }
}
```

### 5. Provide Progress Feedback

```typescript
public async handle(cli: commander.Command): Promise<void> {
    const items = await this.getItemsToProcess();
    
    console.log(`Processing ${items.length} items...`);
    
    for (let i = 0; i < items.length; i++) {
        await this.processItem(items[i]);
        
        // Show progress
        if ((i + 1) % 10 === 0) {
            console.log(`Processed ${i + 1}/${items.length} items`);
        }
    }
    
    console.log('All items processed!');
}
```

## Summary

Artisan commands are a powerful way to automate tasks in your Sosise application:

- ✅ Generate files quickly and consistently
- ✅ Automate repetitive tasks
- ✅ Process data in batches
- ✅ Run maintenance tasks
- ✅ Create deployment scripts
- ✅ Integrate with your existing services through IoC

Start with simple commands and gradually add more complex features as needed!