# Queues

## Quick Start

Move time-consuming tasks to background jobs - your API stays fast and responsive!

```typescript
import Queue from 'sosise-core/build/Queue/Queue';

// In your service - dispatch job instantly
export default class OrderService {
    public async createOrder(orderData: CreateOrderData): Promise<Order> {
        const order = await this.orderRepository.create(orderData);
        
        // Send confirmation email in background
        await Queue.add('email-queue', {
            type: 'order-confirmation',
            orderId: order.id,
            customerEmail: order.customerEmail
        });
        
        return order; // Returns immediately!
    }
}
```

Then create a worker to handle the jobs:

```bash
./artisan make:queueworker EmailWorker
./artisan queue:listen email-queue
```

Perfect for emails, file processing, reports, and any heavy operations!

## Introduction

Queues solve the problem of slow API responses by moving time-consuming tasks to background workers. Instead of making users wait for emails to send, files to process, or reports to generate, you queue these tasks and handle them asynchronously.

Sosise uses Redis and BullMQ for robust job queuing with features like delayed jobs, retries, priority handling, and job monitoring. Your web server stays responsive while background workers handle the heavy lifting.

## Basic Queue Operations

### Dispatching Jobs from Services

Add jobs to queues from your business logic:

```typescript
// src/app/Services/UserService.ts
import Queue from 'sosise-core/build/Queue/Queue';

export default class UserService {
    constructor(
        private userRepository: UserRepository,
        private logger: LoggerService
    ) {}

    public async createUser(userData: CreateUserData): Promise<User> {
        const user = await this.userRepository.create(userData);
        
        // Queue welcome email
        await Queue.add('email-queue', {
            type: 'welcome-email',
            userId: user.id,
            email: user.email,
            name: user.name
        });
        
        // Queue profile image processing if uploaded
        if (userData.profileImage) {
            await Queue.add('image-processing-queue', {
                userId: user.id,
                imageUrl: userData.profileImage,
                operations: ['resize', 'optimize', 'thumbnail']
            });
        }
        
        this.logger.info('User created, background jobs queued', { 
            userId: user.id 
        });
        
        return user;
    }
}
```

### Job Options and Configuration

Control how jobs are processed:

```typescript
export default class NotificationService {
    public async sendImportantNotification(userId: number, message: string): Promise<void> {
        // High priority, immediate processing
        await Queue.add('notifications-queue', {
            userId,
            message,
            priority: 'high'
        }, {
            priority: 10, // Higher priority
            attempts: 5,  // More retry attempts
            backoff: {
                type: 'exponential',
                delay: 1000
            }
        });
    }
    
    public async scheduleReminder(userId: number, reminder: string, sendAt: Date): Promise<void> {
        // Delayed job
        await Queue.add('reminders-queue', {
            userId,
            reminder
        }, {
            delay: sendAt.getTime() - Date.now(), // Delay until sendAt
            attempts: 3
        });
    }
    
    public async generateReport(reportId: string): Promise<void> {
        // Heavy processing job with longer timeout
        await Queue.add('reports-queue', {
            reportId,
            generatedAt: new Date()
        }, {
            attempts: 2,
            backoff: {
                type: 'fixed',
                delay: 30000 // 30 seconds between retries
            },
            removeOnComplete: 5, // Keep only 5 completed jobs
            removeOnFail: 10     // Keep 10 failed jobs for debugging
        });
    }
}
```

## Creating Queue Workers

### Generate Worker with Artisan

```bash
./artisan make:queueworker EmailWorker
```

This creates a worker in `src/app/Console/QueueWorkers/EmailWorker.ts`:

```typescript
import { Worker, QueueScheduler, Job } from 'bullmq';
import queueConfig from '../../../config/queue';
import Handler from '../../Exceptions/Handler';
import LoggerService from 'sosise-core/build/Services/Logger/LoggerService';
import IOC from 'sosise-core/build/ServiceProviders/IOC';

export default class EmailWorker {
    private queueName = 'email-queue';
    private redisConnection = queueConfig.connections.default;
    private logger = IOC.make(LoggerService) as LoggerService;

    constructor() {
        this.listen();
    }

    /**
     * Handle incoming job
     */
    private async handle(job: Job): Promise<void> {
        this.logger.info('Processing email job', { 
            jobId: job.id, 
            type: job.data.type 
        });
        
        try {
            switch (job.data.type) {
                case 'welcome-email':
                    await this.sendWelcomeEmail(job.data);
                    break;
                    
                case 'order-confirmation':
                    await this.sendOrderConfirmation(job.data);
                    break;
                    
                case 'password-reset':
                    await this.sendPasswordReset(job.data);
                    break;
                    
                default:
                    throw new Error(`Unknown email type: ${job.data.type}`);
            }
            
            this.logger.info('Email job completed successfully', { 
                jobId: job.id 
            });
        } catch (error) {
            this.logger.error('Email job failed', {
                jobId: job.id,
                error: error.message,
                data: job.data
            });
            throw error; // Re-throw to trigger retry
        }
    }
    
    private async sendWelcomeEmail(data: any): Promise<void> {
        const emailService = IOC.make(EmailService) as EmailService;
        await emailService.send({
            to: data.email,
            template: 'welcome',
            data: { name: data.name }
        });
    }
    
    private async sendOrderConfirmation(data: any): Promise<void> {
        const emailService = IOC.make(EmailService) as EmailService;
        const orderService = IOC.make(OrderService) as OrderService;
        
        const order = await orderService.getOrder(data.orderId);
        await emailService.send({
            to: data.customerEmail,
            template: 'order-confirmation',
            data: { order }
        });
    }
    
    /**
     * Listen to the queue
     */
    private async listen(): Promise<void> {
        // Instantiate queue scheduler for delayed jobs
        const scheduler = new QueueScheduler(this.queueName, { 
            connection: this.redisConnection 
        });

        // Create worker
        const worker = new Worker(
            this.queueName, 
            async (job) => this.handle(job), 
            { 
                connection: this.redisConnection,
                concurrency: 5 // Process 5 jobs concurrently
            }
        );

        // Handle failed jobs
        worker.on('failed', async (job: Job, error) => {
            this.logger.error('Queue job failed', {
                jobId: job?.id,
                queueName: this.queueName,
                error: error.message,
                attempts: job?.attemptsMade
            });
            
            const exceptionHandler = new Handler();
            exceptionHandler.reportCommandException(error);
        });
        
        // Handle completed jobs
        worker.on('completed', async (job: Job) => {
            this.logger.info('Queue job completed', {
                jobId: job.id,
                queueName: this.queueName,
                duration: Date.now() - job.processedOn!
            });
        });
        
        this.logger.info(`EmailWorker listening on ${this.queueName} queue`);
    }
}
```

## Specialized Workers

### File Processing Worker

```typescript
export default class FileProcessingWorker {
    private queueName = 'file-processing-queue';
    
    private async handle(job: Job): Promise<void> {
        const { fileUrl, operations, userId } = job.data;
        
        switch (job.data.type) {
            case 'csv-import':
                await this.processCsvImport(job.data);
                break;
                
            case 'image-processing':
                await this.processImage(job.data);
                break;
                
            case 'pdf-generation':
                await this.generatePdf(job.data);
                break;
        }
    }
    
    private async processCsvImport(data: any): Promise<void> {
        const csvService = IOC.make(CsvService) as CsvService;
        const userService = IOC.make(UserService) as UserService;
        
        // Process large CSV file
        const records = await csvService.parse(data.fileUrl);
        let processed = 0;
        
        for (const record of records) {
            try {
                await userService.importUser(record);
                processed++;
                
                // Update job progress
                if (processed % 100 === 0) {
                    await job.updateProgress(processed / records.length * 100);
                }
            } catch (error) {
                this.logger.warn('Failed to import user record', { 
                    record, 
                    error: error.message 
                });
            }
        }
        
        this.logger.info('CSV import completed', { 
            totalRecords: records.length,
            processedRecords: processed
        });
    }
}
```

### Report Generation Worker

```typescript
export default class ReportWorker {
    private queueName = 'reports-queue';
    
    private async handle(job: Job): Promise<void> {
        const reportService = IOC.make(ReportService) as ReportService;
        const { reportId, reportType, filters } = job.data;
        
        this.logger.info('Starting report generation', { reportId, reportType });
        
        try {
            // Update progress
            await job.updateProgress(10);
            
            // Generate report data
            const reportData = await reportService.generateData(reportType, filters);
            await job.updateProgress(60);
            
            // Format report
            const formattedReport = await reportService.formatReport(reportData);
            await job.updateProgress(80);
            
            // Save to storage
            const filePath = await reportService.saveReport(reportId, formattedReport);
            await job.updateProgress(90);
            
            // Notify user
            await Queue.add('email-queue', {
                type: 'report-ready',
                userId: job.data.userId,
                reportId,
                downloadUrl: filePath
            });
            
            await job.updateProgress(100);
            
            this.logger.info('Report generation completed', { 
                reportId, 
                filePath 
            });
        } catch (error) {
            this.logger.error('Report generation failed', {
                reportId,
                error: error.message
            });
            
            // Notify user of failure
            await Queue.add('email-queue', {
                type: 'report-failed',
                userId: job.data.userId,
                reportId,
                error: error.message
            });
            
            throw error;
        }
    }
}
```

## Queue Management

### Running Workers

```bash
# Start specific worker
./artisan queue:listen email-queue

# Run multiple workers
./artisan queue:listen email-queue &
./artisan queue:listen file-processing-queue &
./artisan queue:listen reports-queue &
```

### Monitoring Queues

```bash
# List jobs in queue
./artisan queue:list email-queue

# Show queue statistics
./artisan queue:stats email-queue

# Monitor failed jobs
./artisan queue:list email-queue --failed
```

### Queue Maintenance

```bash
# Retry all failed jobs
./artisan queue:retry email-queue

# Retry specific job
./artisan queue:retry email-queue --job-id=123

# Clear failed jobs
./artisan queue:flush email-queue

# Clear all jobs
./artisan queue:clear email-queue
```

## Advanced Queue Patterns

### Job Batching

Process related jobs together:

```typescript
export default class BulkEmailService {
    public async sendCampaign(campaignId: string, recipients: string[]): Promise<void> {
        // Break into batches of 100
        const batches = this.chunkArray(recipients, 100);
        
        for (let i = 0; i < batches.length; i++) {
            await Queue.add('email-campaign-queue', {
                campaignId,
                recipients: batches[i],
                batchNumber: i + 1,
                totalBatches: batches.length
            }, {
                delay: i * 1000 // Stagger batches to avoid overwhelming email service
            });
        }
    }
}
```

### Job Chaining

Chain jobs that depend on each other:

```typescript
export default class OrderProcessingService {
    public async processOrder(orderId: string): Promise<void> {
        // Step 1: Validate payment
        await Queue.add('payment-processing-queue', {
            orderId,
            nextStep: 'inventory-check'
        });
    }
}

// In PaymentWorker
private async handle(job: Job): Promise<void> {
    const { orderId, nextStep } = job.data;
    
    await this.processPayment(orderId);
    
    // Chain to next step
    if (nextStep) {
        await Queue.add('inventory-queue', {
            orderId,
            nextStep: 'shipping-label'
        });
    }
}
```

### Priority Queues

```typescript
export default class NotificationService {
    public async sendUrgentAlert(message: string): Promise<void> {
        await Queue.add('notifications-queue', {
            type: 'urgent-alert',
            message
        }, {
            priority: 100 // Highest priority
        });
    }
    
    public async sendDailyDigest(userId: number): Promise<void> {
        await Queue.add('notifications-queue', {
            type: 'daily-digest',
            userId
        }, {
            priority: 1 // Lowest priority
        });
    }
}
```

## Configuration

### Queue Configuration

Configure queues in `src/config/queue.ts`:

```typescript
export default {
    connections: {
        default: {
            host: process.env.REDIS_HOST || '127.0.0.1',
            port: parseInt(process.env.REDIS_PORT || '6379'),
            database: parseInt(process.env.REDIS_QUEUE_DB || '1'),
            password: process.env.REDIS_PASSWORD || undefined,
            keyPrefix: process.env.APP_NAME + ':queue:',
        }
    },
    
    queues: {
        'email-queue': {
            connection: 'default',
            defaultJobOptions: {
                attempts: 3,
                backoff: {
                    type: 'exponential',
                    delay: 2000
                },
                removeOnComplete: 10,
                removeOnFail: 50
            }
        },
        
        'file-processing-queue': {
            connection: 'default',
            defaultJobOptions: {
                attempts: 2,
                timeout: 300000, // 5 minutes
                removeOnComplete: 5,
                removeOnFail: 20
            }
        }
    }
};
```

## Best Practices

### 1. Keep Jobs Small and Focused

```typescript
// ✅ Good: Focused job responsibility
await Queue.add('email-queue', {
    type: 'welcome-email',
    userId: user.id
});

await Queue.add('profile-setup-queue', {
    userId: user.id
});

// ❌ Bad: Too many responsibilities
await Queue.add('user-setup-queue', {
    userId: user.id,
    sendWelcomeEmail: true,
    setupProfile: true,
    createStripeCustomer: true,
    sendSlackNotification: true
});
```

### 2. Handle Failures Gracefully

```typescript
// ✅ Good: Proper error handling
private async handle(job: Job): Promise<void> {
    try {
        await this.processJob(job.data);
    } catch (error) {
        if (error instanceof RecoverableError) {
            // Let it retry
            throw error;
        } else {
            // Log and don't retry
            this.logger.error('Non-recoverable error', {
                jobId: job.id,
                error: error.message
            });
            return; // Don't throw, prevents retry
        }
    }
}
```

### 3. Use Appropriate Retry Strategies

```typescript
// ✅ Good: Context-appropriate retry settings
// Email sending - retry with backoff
await Queue.add('email-queue', data, {
    attempts: 5,
    backoff: { type: 'exponential', delay: 1000 }
});

// File processing - fewer retries, longer delay
await Queue.add('file-processing-queue', data, {
    attempts: 2,
    backoff: { type: 'fixed', delay: 30000 }
});

// Payment processing - immediate retry once, then manual review
await Queue.add('payment-queue', data, {
    attempts: 2,
    backoff: { type: 'fixed', delay: 1000 }
});
```

### 4. Monitor Queue Health

```typescript
export default class QueueMonitoringService {
    constructor(private logger: LoggerService) {}
    
    public async checkQueueHealth(): Promise<void> {
        const queues = ['email-queue', 'file-processing-queue', 'reports-queue'];
        
        for (const queueName of queues) {
            const waiting = await this.getWaitingJobsCount(queueName);
            const failed = await this.getFailedJobsCount(queueName);
            
            if (waiting > 1000) {
                this.logger.warn('Queue backlog detected', { queueName, waiting });
            }
            
            if (failed > 100) {
                this.logger.error('High failure rate detected', { queueName, failed });
            }
        }
    }
}
```

## Summary

Sosise queues provide:

- ✅ Background job processing
- ✅ Delayed and scheduled jobs
- ✅ Automatic retries with backoff
- ✅ Job priorities and batching
- ✅ Progress tracking
- ✅ Failed job handling
- ✅ Redis-based reliability
- ✅ Easy monitoring and management

Use queues to keep your API fast and handle heavy operations in the background!