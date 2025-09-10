# Mailer

## Quick Start

Send emails easily with Sosise's built-in mailer! Perfect for notifications, confirmations, and user communication.

```typescript
import Mailer from 'sosise-core/build/Mailer/Mailer';

// Simple email
await Mailer.sendMail({
    from: "noreply@myapp.com",
    to: "user@example.com",
    subject: "Welcome to MyApp!",
    html: "<h1>Welcome!</h1><p>Thanks for joining us.</p>"
});

// In a service
export default class NotificationService {
    public async sendWelcomeEmail(user: User): Promise<void> {
        await Mailer.sendMail({
            to: user.email,
            subject: "Welcome to our platform!",
            html: await this.generateWelcomeTemplate(user)
        });
    }
}
```

Built on Nodemailer for reliability and flexibility!

## Introduction

Email communication is essential for most applications - from user registration confirmations to order receipts and password resets. Sosise provides a clean, simple email API built on top of the robust Nodemailer library.

Whether you need to send simple text emails, rich HTML templates, or emails with attachments, Sosise's mailer handles it all with a consistent, easy-to-use interface. The system supports SMTP configuration, development testing, and production-ready email delivery.

## Basic Email Sending

### Simple Email Service

Create a dedicated service for email operations:

```typescript
// src/app/Services/EmailService.ts
import Mailer from 'sosise-core/build/Mailer/Mailer';
import LoggerService from 'sosise-core/build/Services/Logger/LoggerService';

export default class EmailService {
    constructor(
        private logger: LoggerService
    ) {}

    public async sendEmail(options: EmailOptions): Promise<void> {
        this.logger.info('Sending email', { 
            to: options.to, 
            subject: options.subject 
        });

        try {
            await Mailer.sendMail({
                from: process.env.MAIL_FROM || "noreply@myapp.com",
                ...options
            });

            this.logger.info('Email sent successfully', { to: options.to });
        } catch (error) {
            this.logger.error('Failed to send email', {
                to: options.to,
                error: error.message
            });
            throw new EmailSendingError(`Failed to send email to ${options.to}`);
        }
    }
}

interface EmailOptions {
    to: string;
    subject: string;
    text?: string;
    html?: string;
    attachments?: any[];
}
```

### Use in Business Logic

```typescript
// src/app/Services/UserService.ts
export default class UserService {
    constructor(
        private userRepository: UserRepository,
        private emailService: EmailService
    ) {}

    public async createUser(userData: CreateUserData): Promise<User> {
        const user = await this.userRepository.create(userData);

        // Send welcome email
        await this.emailService.sendEmail({
            to: user.email,
            subject: "Welcome to our platform!",
            html: `
                <h1>Welcome ${user.name}!</h1>
                <p>Thank you for joining our platform. Get started by exploring our features:</p>
                <ul>
                    <li>Complete your profile</li>
                    <li>Browse our marketplace</li>
                    <li>Connect with other users</li>
                </ul>
                <p>Best regards,<br>The Team</p>
            `
        });

        return user;
    }

    public async resetPassword(email: string): Promise<void> {
        const user = await this.userRepository.findByEmail(email);
        if (!user) {
            throw new UserNotFoundError(email);
        }

        const resetToken = await this.generateResetToken(user.id);

        await this.emailService.sendEmail({
            to: user.email,
            subject: "Password Reset Request",
            html: `
                <h2>Password Reset</h2>
                <p>Hi ${user.name},</p>
                <p>You requested a password reset. Click the link below to reset your password:</p>
                <a href="${process.env.APP_URL}/reset-password?token=${resetToken}">Reset Password</a>
                <p>This link expires in 1 hour.</p>
                <p>If you didn't request this, please ignore this email.</p>
            `
        });
    }
}
```

## Email Templates and Formatting

### HTML Email Templates

Create reusable email templates:

```typescript
export default class EmailTemplateService {
    public generateOrderConfirmation(order: Order): string {
        return `
            <div style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">
                <header style="background-color: #2563eb; color: white; padding: 20px; text-align: center;">
                    <h1>Order Confirmation</h1>
                </header>
                
                <main style="padding: 20px;">
                    <h2>Thank you for your order!</h2>
                    <p>Order #${order.id} has been confirmed and will be processed soon.</p>
                    
                    <table style="width: 100%; border-collapse: collapse; margin: 20px 0;">
                        <thead>
                            <tr style="background-color: #f8f9fa;">
                                <th style="padding: 12px; text-align: left; border: 1px solid #dee2e6;">Item</th>
                                <th style="padding: 12px; text-align: right; border: 1px solid #dee2e6;">Quantity</th>
                                <th style="padding: 12px; text-align: right; border: 1px solid #dee2e6;">Price</th>
                            </tr>
                        </thead>
                        <tbody>
                            ${order.items.map(item => `
                                <tr>
                                    <td style="padding: 12px; border: 1px solid #dee2e6;">${item.name}</td>
                                    <td style="padding: 12px; text-align: right; border: 1px solid #dee2e6;">${item.quantity}</td>
                                    <td style="padding: 12px; text-align: right; border: 1px solid #dee2e6;">$${item.price.toFixed(2)}</td>
                                </tr>
                            `).join('')}
                        </tbody>
                        <tfoot>
                            <tr style="font-weight: bold; background-color: #f8f9fa;">
                                <td colspan="2" style="padding: 12px; border: 1px solid #dee2e6;">Total</td>
                                <td style="padding: 12px; text-align: right; border: 1px solid #dee2e6;">$${order.total.toFixed(2)}</td>
                            </tr>
                        </tfoot>
                    </table>
                    
                    <p>Shipping Address:</p>
                    <address style="margin-left: 20px;">
                        ${order.shippingAddress.name}<br>
                        ${order.shippingAddress.street}<br>
                        ${order.shippingAddress.city}, ${order.shippingAddress.state} ${order.shippingAddress.zipCode}
                    </address>
                </main>
                
                <footer style="background-color: #f8f9fa; padding: 20px; text-align: center; color: #6b7280;">
                    <p>Questions? Contact us at support@myapp.com</p>
                </footer>
            </div>
        `;
    }

    public generateInvoice(invoice: Invoice): string {
        return `
            <div style="font-family: 'Courier New', monospace; max-width: 600px; margin: 0 auto;">
                <h1>INVOICE</h1>
                <p><strong>Invoice #:</strong> ${invoice.id}</p>
                <p><strong>Date:</strong> ${invoice.date}</p>
                <p><strong>Due Date:</strong> ${invoice.dueDate}</p>
                
                <!-- Invoice content here -->
            </div>
        `;
    }
}
```

### Dynamic Email Generation

```typescript
export default class NotificationService {
    constructor(
        private emailService: EmailService,
        private templateService: EmailTemplateService
    ) {}

    public async sendOrderConfirmation(order: Order): Promise<void> {
        const htmlContent = this.templateService.generateOrderConfirmation(order);
        
        await this.emailService.sendEmail({
            to: order.customerEmail,
            subject: `Order Confirmation #${order.id}`,
            html: htmlContent
        });
    }

    public async sendAccountAlert(user: User, alertType: string, details: any): Promise<void> {
        const subject = this.getAlertSubject(alertType);
        const html = this.generateAlertEmail(user, alertType, details);

        await this.emailService.sendEmail({
            to: user.email,
            subject,
            html
        });
    }

    private getAlertSubject(alertType: string): string {
        const subjects = {
            'login': 'New login to your account',
            'password_change': 'Password changed successfully',
            'suspicious_activity': 'Suspicious activity detected',
            'payment_failed': 'Payment failed - Action required'
        };
        return subjects[alertType] || 'Account notification';
    }
}
```

## Attachments and Advanced Features

### Sending Files

```typescript
export default class ReportService {
    constructor(private emailService: EmailService) {}

    public async emailReport(userId: number, reportPath: string): Promise<void> {
        const user = await this.getUserById(userId);
        
        await Mailer.sendMail({
            to: user.email,
            subject: "Your monthly report is ready",
            html: `
                <h2>Monthly Report</h2>
                <p>Hi ${user.name},</p>
                <p>Your monthly report is attached to this email.</p>
                <p>The report covers the period from last month and includes:</p>
                <ul>
                    <li>Activity summary</li>
                    <li>Performance metrics</li>
                    <li>Recommendations</li>
                </ul>
            `,
            attachments: [
                {
                    filename: 'monthly-report.pdf',
                    path: reportPath,
                    contentType: 'application/pdf'
                }
            ]
        });
    }

    public async sendInvoiceWithPDF(invoice: Invoice): Promise<void> {
        const pdfBuffer = await this.generateInvoicePDF(invoice);
        
        await Mailer.sendMail({
            to: invoice.customerEmail,
            subject: `Invoice ${invoice.number}`,
            html: `
                <h2>Invoice ${invoice.number}</h2>
                <p>Please find your invoice attached.</p>
                <p><strong>Amount Due:</strong> $${invoice.total}</p>
                <p><strong>Due Date:</strong> ${invoice.dueDate}</p>
            `,
            attachments: [
                {
                    filename: `invoice-${invoice.number}.pdf`,
                    content: pdfBuffer,
                    contentType: 'application/pdf'
                }
            ]
        });
    }
}
```

### Multiple Recipients and CC/BCC

```typescript
export default class BulkEmailService {
    public async sendNewsletter(newsletter: Newsletter, subscribers: User[]): Promise<void> {
        // Send to batches to avoid overwhelming email service
        const batchSize = 50;
        
        for (let i = 0; i < subscribers.length; i += batchSize) {
            const batch = subscribers.slice(i, i + batchSize);
            
            await Mailer.sendMail({
                from: 'newsletter@myapp.com',
                bcc: batch.map(user => user.email), // Use BCC for privacy
                subject: newsletter.subject,
                html: newsletter.content
            });
            
            // Wait between batches to respect rate limits
            await new Promise(resolve => setTimeout(resolve, 1000));
        }
    }

    public async sendTeamNotification(subject: string, message: string): Promise<void> {
        await Mailer.sendMail({
            to: 'team@myapp.com',
            cc: ['manager@myapp.com', 'admin@myapp.com'],
            subject,
            html: `
                <h2>Team Notification</h2>
                <p>${message}</p>
                <p>Timestamp: ${new Date().toISOString()}</p>
            `
        });
    }
}
```

## Configuration

### SMTP Configuration

Configure your email settings in `src/config/mailer.ts`:

```typescript
export default {
    // SMTP connection settings
    connection: {
        host: process.env.MAIL_HOST || 'smtp.gmail.com',
        port: parseInt(process.env.MAIL_PORT || '587'),
        secure: process.env.MAIL_SECURE === 'true', // true for 465, false for other ports
        auth: {
            user: process.env.MAIL_USERNAME,
            pass: process.env.MAIL_PASSWORD
        }
    },

    // Default sender
    defaults: {
        from: process.env.MAIL_FROM || 'noreply@myapp.com'
    },

    // Development settings
    dryrun: process.env.NODE_ENV === 'development' || process.env.MAIL_DRYRUN === 'true',

    // Email templates directory
    templatesPath: 'resources/email-templates'
};
```

### Environment Variables

```bash
# .env
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_FROM="My App <noreply@myapp.com>"
MAIL_SECURE=false
MAIL_DRYRUN=true  # For development
```

## Development and Testing

### Dry Run Mode

Enable dry run mode for development:

```typescript
// In src/config/mailer.ts
export default {
    dryrun: true, // Emails won't be sent, just logged
    // ... other config
};
```

When dry run is enabled, you'll see output like:

```bash
2023-12-25 14:30:45 INFO  Mailer dryrun mode is active... No emails would be sent
2023-12-25 14:30:45 INFO  Test SMTP account obtained successfully {
  user: 'test123@ethereal.email',
  pass: 'testpassword',
  smtp: { host: 'smtp.ethereal.email', port: 587, secure: false },
  web: 'https://ethereal.email'
}
2023-12-25 14:30:46 INFO  Fake email was sent successfully { 
  messageId: '<abc123@server.com>' 
}
2023-12-25 14:30:46 INFO  Preview URL: https://ethereal.email/message/xyz789
```

### Email Testing Service

```typescript
export default class EmailTestingService {
    constructor(private logger: LoggerService) {}

    public async testEmailConfiguration(): Promise<boolean> {
        try {
            await Mailer.sendMail({
                to: 'test@example.com',
                subject: 'Email Configuration Test',
                text: 'If you receive this, email configuration is working!'
            });
            
            this.logger.info('Email configuration test successful');
            return true;
        } catch (error) {
            this.logger.error('Email configuration test failed', {
                error: error.message
            });
            return false;
        }
    }

    public async sendTestEmails(): Promise<void> {
        const testCases = [
            {
                name: 'Plain text email',
                options: {
                    to: 'test@example.com',
                    subject: 'Plain text test',
                    text: 'This is a plain text email test.'
                }
            },
            {
                name: 'HTML email',
                options: {
                    to: 'test@example.com',
                    subject: 'HTML test',
                    html: '<h1>HTML Test</h1><p>This is an <strong>HTML</strong> email.</p>'
                }
            }
        ];

        for (const testCase of testCases) {
            try {
                await Mailer.sendMail(testCase.options);
                this.logger.info(`✅ ${testCase.name} - Success`);
            } catch (error) {
                this.logger.error(`❌ ${testCase.name} - Failed`, {
                    error: error.message
                });
            }
        }
    }
}
```

## Queue Integration

### Sending Emails Asynchronously

Combine with queues for better performance:

```typescript
// In your service
export default class UserService {
    public async createUser(userData: CreateUserData): Promise<User> {
        const user = await this.userRepository.create(userData);
        
        // Queue welcome email instead of sending immediately
        await Queue.add('email-queue', {
            type: 'welcome-email',
            userId: user.id,
            email: user.email,
            name: user.name
        });
        
        return user; // Returns immediately
    }
}

// In your email worker
export default class EmailWorker {
    private async handle(job: Job): Promise<void> {
        const { type, ...data } = job.data;
        
        switch (type) {
            case 'welcome-email':
                await this.sendWelcomeEmail(data);
                break;
            case 'order-confirmation':
                await this.sendOrderConfirmation(data);
                break;
            case 'password-reset':
                await this.sendPasswordReset(data);
                break;
        }
    }
    
    private async sendWelcomeEmail(data: any): Promise<void> {
        await Mailer.sendMail({
            to: data.email,
            subject: 'Welcome!',
            html: `<h1>Welcome ${data.name}!</h1>`
        });
    }
}
```

## Best Practices

### 1. Use Service Layer for Email Logic

```typescript
// ✅ Good: Centralized email service
export default class EmailService {
    public async sendWelcomeEmail(user: User): Promise<void> {
        await Mailer.sendMail({
            to: user.email,
            subject: this.getWelcomeSubject(user),
            html: this.generateWelcomeTemplate(user)
        });
    }
}

// ❌ Bad: Email logic scattered in controllers
export default class UserController {
    public async create(request: Request): Promise<void> {
        const user = await this.userService.createUser(request.body);
        
        // Email logic shouldn't be in controller
        await Mailer.sendMail({
            to: user.email,
            subject: 'Welcome!',
            html: '<h1>Welcome!</h1>'
        });
    }
}
```

### 2. Handle Email Failures Gracefully

```typescript
// ✅ Good: Proper error handling
export default class NotificationService {
    public async sendOrderNotification(order: Order): Promise<void> {
        try {
            await this.emailService.sendOrderConfirmation(order);
        } catch (error) {
            this.logger.error('Failed to send order confirmation', {
                orderId: order.id,
                error: error.message
            });
            
            // Don't let email failure break the order process
            // Maybe queue for retry later
            await Queue.add('email-retry-queue', {
                type: 'order-confirmation',
                orderId: order.id,
                attempt: 1
            }, { delay: 60000 }); // Retry in 1 minute
        }
    }
}
```

### 3. Use Environment-Specific Configuration

```typescript
// ✅ Good: Environment-specific settings
const emailConfig = {
    production: {
        host: 'smtp.sendgrid.net',
        dryrun: false
    },
    development: {
        host: 'smtp.mailtrap.io',
        dryrun: true
    },
    test: {
        dryrun: true,
        silent: true
    }
};

export default emailConfig[process.env.NODE_ENV || 'development'];
```

### 4. Validate Email Addresses

```typescript
// ✅ Good: Validate before sending
export default class EmailService {
    public async sendEmail(to: string, subject: string, content: string): Promise<void> {
        if (!this.isValidEmail(to)) {
            throw new InvalidEmailError(`Invalid email address: ${to}`);
        }
        
        await Mailer.sendMail({
            to,
            subject,
            html: content
        });
    }
    
    private isValidEmail(email: string): boolean {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }
}
```

## Summary

Sosise's mailer provides:

- ✅ Simple email sending API
- ✅ Built on reliable Nodemailer
- ✅ HTML and text email support
- ✅ File attachments
- ✅ Development dry-run mode
- ✅ Easy SMTP configuration
- ✅ Service layer integration
- ✅ Queue system compatibility

Use the mailer for all your email communication needs with confidence!