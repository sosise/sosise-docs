# Error Handling

## Introduction

In a Sosise project, error and exception handling is pre-configured for you. The `src/app/Exceptions/Handler.ts` class is responsible for logging and rendering all exceptions thrown by your application. This documentation will provide insights into this class and its functionalities.

## Writing Exceptions

### Creating an Exception using Artisan

To create a new exception, you can use the following Artisan command:

```sh
./artisan make:exception CustomerDoesNotExistException
```

The generated exception class will extend the `Exception` class from `sosise-core/build/Exceptions/Exception`.

Let's explore an example of an exception class:

```typescript
import Exception from 'sosise-core/build/Exceptions/Exception';
import ExceptionResponse from 'sosise-core/build/Types/ExceptionResponse';

export default class CustomerNotFoundException extends Exception {

    // This variable is optional; you may remove it if not needed
    public exampleVariable: string;

    // HTTP Code of the response with this exception
    protected httpCode = 500;

    // Error code which is rendered in the response
    protected code = 3000;

    // If set to false, no exception will be sent to Sentry
    protected sendToSentry = true;

    // In which logging channel should this exception be logged; see src/config/logging.ts
    protected loggingChannel = 'default';

    /**
     * Constructor
     */
    constructor(message: string, exampleVariable: string) {
        super(message);

        // This is just an example
        this.exampleVariable = exampleVariable;
    }

    /**
     * Handle exception
     */
    public handle(exception: this): ExceptionResponse {
        const response: ExceptionResponse = {
            code: exception.code,
            httpCode: exception.httpCode,
            message: exception.message,
            data: {
                yourCustomData: exception.exampleVariable
            }
        };
        return response;
    }
}
```

## Exception Codes

Since HTTP codes are not unique "error" codes, they cannot provide detailed information about the exact nature of the error. Using human-readable error messages as unique identifiers is also not recommended since they can change over time.

To address this, you should use the `code` property of an exception and create a table that describes what each code means. This allows your customers to precisely identify the error and decide how to handle it in their applications.

For instance, you have used `protected code = 3001;` as an example in this documentation.

## Response Code Ranges

To maintain consistency in your application's error handling, consider using the following response code ranges:

| Code Range | Description |
| ---------- | ----------- |
| 1000-1999  | Use these codes for "good" responses |
| 2000-2999  | Reserved by the framework for Exceptions that the framework can throw |
| 3000-...   | Use this range for displaying application-specific errors |

## Examples

Example of a response coming from the controller:

```json
{
    "code": 1000,
    "message": "Cart item deleted",
    "data": null
}
```

Example of a response coming from the exception:

```json
{
    "code": 3001,
    "httpCode": 404,
    "message": "Cart does not exist, you should create it first",
    "data": null
}
```

Using meaningful error codes allows you to communicate precise information to your customers, making it easier for them to handle and respond to errors effectively.