# Unifiers

## Introduction
Unifiers provide a convenient way to validate incoming requests and map them to class parameters or DTOs (Data Transfer Objects). For example, an order creation middleware can be used to validate that the required customer ID and shop information are provided in the request. Unifiers are located in the `app/Unifiers` directory.

## Defining a Unifier
To create a new unifier, you can use the `make:unifier` Artisan command:

```sh
./artisan make:unifier OrderCreatingUnifier
```

Let's take a look at an example below:

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

        // If validation fails, throw an exception
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

> For full documentation on available validation rules, visit [the validator.js website](https://github.com/mikeerickson/validatorjs).

## Usage
Now, let's see an example of how to use unifiers in a controller:

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
            // At this step, the request body is validated and mapped
            const unifier = new OrderCreatingUnifier(request.body);

            // Now we can use unifier params
            console.log(unifier.customerId);
            console.log(unifier.shop);

            // Prepare HTTP response
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

Using unifiers in your controllers helps ensure that the incoming request data is properly validated and mapped to the relevant parameters before processing the request. This improves the security and reliability of your application by ensuring that only valid and expected data is used in the subsequent steps of request handling.