# Sentry Integration

Sosise offers built-in support for Sentry, a popular error monitoring service. By specifying your `SENTRY_DSN` in the `.env` file, all exceptions and errors in your application will be automatically sent to the Sentry server.

## Setting up Sentry

1. Sign up for a Sentry account if you don't have one already.
2. Once you have an account, log in and create a new project to get the `SENTRY_DSN` for your application.
3. Open your `.env` file in the root directory of your Sosise application.
4. Add the `SENTRY_DSN` variable and set its value to the DSN you obtained from Sentry:

```plaintext
SENTRY_DSN=your_sentry_dsn_here
```

## Automatic Exception Reporting

With the `SENTRY_DSN` configured, Sosise will automatically send all unhandled exceptions, errors, and logs to the specified Sentry project. This allows you to monitor and track issues in your application effectively.

No additional code is required to enable this feature. Once the DSN is set, Sentry integration is automatically activated in your Sosise application.

## Monitoring Your Application

Once Sentry is set up and integrated with your Sosise application, you can visit your Sentry dashboard to monitor exceptions, errors, and logs in real-time. Sentry provides a user-friendly interface for tracking issues and allows you to debug and resolve them efficiently.

By using Sentry, you can proactively identify and address issues before they impact your users, ensuring a smooth and reliable experience for your application.

## Troubleshooting

If you encounter any issues with the Sentry integration, please ensure that you have correctly set the `SENTRY_DSN` in your `.env` file. Also, make sure your Sosise application has access to the internet so it can communicate with the Sentry server.

For more advanced configuration options and customization, you can refer to the official [Sentry documentation](https://docs.sentry.io/).