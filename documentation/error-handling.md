# Error Handling
## Introduction
When you start a new Sosise project, error and exception handling is already configured for you. The `src/app/Exceptions/Handler.ts` class is where all exceptions thrown by your application are logged or/and rendered to the user. We'll dive deeper into this class throughout this documentation.

## Writing Exceptions
### Create using artisan
```sh
./artisan make:exception CustomerDoesNotExistsException
```

Let's take a look at an example of an exception class.

```typescript
import Exception from 'sosise-core/build/Exceptions/Exception';
import ExceptionResponse from 'sosise-core/build/Types/ExceptionResponse';

export default class CustomerDoesNotExistsException extends Exception {

    // This variables are optional, you may remove them
    public exampleVariable: string;
    protected httpCode = 500;
    protected code = 3001;

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
            code: this.code, // optional
            httpCode: this.httpCode, // optional
            message: this.message,
            data: {
                yourCustomData: this.exampleVariable
            }
        };
        return response;
    }
}
```

## Exception codes
Since HTTP code is not a unique "error" code you cannot tell applications that are using your REST-API what exactly happened and your human-readable error messages must not be used as a unique identifier, since they can change.

That's why you should use the `code` property of an exception and write down a table that describes what this code means. That allows your customers to be sure what exactly happened and decide what to show to the end customers.

> We are talking about `protected code = 3001;`

### Response code ranges
| Code Range | Description |
| ----------- | ----------- |
| 1000-1999 | You can use these codes for "good" responses |
| 2000-2999 | Are reserved by a framework, and are used for Exceptions which framework can throw |
| 3000-... | Feel free to use this range to display any error in your application |

#### Examples
Comes from controller
```json
{
    "code": 1000,
    "message": "Cart item deleted",
    "data": null
}
```

Comes from exception
```json
{
    "code": 3001,
    "httpCode": 404,
    "message": "Cart does not exist, you should create it first",
    "data": null
}
```