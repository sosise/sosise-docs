# Unifiers
## Introduction
Unfiers provide a way to validate incoming request and map it to the class parameters (DTO). For example, a order creating middleware can be used to validate that customer id and shop was given. All unifiers are located in the `app/Unifiers` directory.

## Defining Unifier
To create a new unifier, use the `make:unifier` Artisan command:

```sh
./artisan make:unifier OrderCreatingUnifier
```

Let's take a look at example below:
```typescript
import Validator from 'validatorjs';
import ValidationException from 'sosise-core/build/Exceptions/Validation/ValidationException';

/**
 * If you need more validation rules, see: https://github.com/mikeerickson/validatorjs
 */
export default class OrderCreatingUnifier {

    private params: any;
    public customerId: number;
    public shop: string;

    /**
     * Constructor
     */
    constructor(params: any) {
        // Remember incoming params
        this.params = params;

        // Validate, await is important otherwise we could not catch the exception
        this.validate();

        // Map data
        this.map();
    }

    /**
     * Request data validation
     */
    private validate() {
        // Create validator
        const validator = new Validator(this.params, {
            customerId: ['required', 'numeric', 'min:1'],
            shop: ['required', 'string'],
        });

        // If it fails throw exception
        if (validator.fails()) {
            throw new ValidationException('Validation exception', (validator.errors.all() as any));
        }
    }

    /**
     * Request data mapping
     */
    private map() {
        this.customerId = this.params.customerId;
        this.shop = this.params.shop;
    }
}
```

> Please visit [validator website](https://github.com/mikeerickson/validatorjs) for full documentation.

## Usage
Now let's see in a example how to use unifiers in `controller`:

```typescript
import { Request, Response, NextFunction } from 'express';
import HttpResponse from 'sosise-core/build/Types/HttpResponse';
import OrderCreatingUnifier from '../../Unifiers/OrderCreatingUnifier';

export default class IndexController {
    /**
     * Example method
     */
    public async index(request: Request, response: Response, next: NextFunction) {
        try {
            // Instantiate unifier
            // At this step all request params are validated and mapped
            const unifier = new OrderCreatingUnifier(request.params);

            // Now we can use unifier params
            console.log(unifier.customerId);
            console.log(unifier.shop);

            // Prepare http response
            const httpResponse: HttpResponse = {
                code: 1000,
                message: 'Some example message',
                data: null
            };

            // Send response
            return response.send(httpResponse);
        } catch (error) {
            next(error);
        }
    }
}
```