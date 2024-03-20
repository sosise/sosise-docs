# Unifiers

## Introduction
Unifiers provide a convenient way to validate incoming requests and map them to class parameters or DTOs (Data Transfer Objects). For example, an order creation middleware can be used to validate that the required customer ID and shop information are provided in the request. Unifiers are located in the `app/Unifiers` directory.

## Defining a Unifier
To create a new unifier, you can use the `make:unifier` Artisan command:

```sh
./artisan make:unifier CreateOrderUnifier
```

Let's take a look at an example below:

```typescript
import Validator from 'sosise-core/build/Validator/Validator';
import ValidationException from 'sosise-core/build/Exceptions/Validation/ValidationException';

/**
 * For more information, refer to: https://sosise.github.io/sosise-docs/#/documentation/unifiers
 */
export default class CreateOrderUnifier {

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
        const validator = new Validator(this.params);

        // Validate
        validator
            .field('customerId')
            .required()
            .shouldBeNumber()
            .min(0);

        validator
            .field('shop')
            .required()
            .shouldBeString()
            .min(1);

        // If validation fails, throw an exception
        if (validator.fails()) {
            throw new ValidationException('Validation exception', validator.errors);
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

# ValidatorRules

The framework provides a robust solution for validating data in your applications. It supports a wide range of validation scenarios, from simple presence checks to complex type validations and custom rule definitions.

## Quick Start

### Creating a Validator Instance

First, create an instance of the `Validator` class by passing the data object you wish to validate:

```typescript
import Validator from "./Validator";

const data = {
    username: "exampleUser",
    age: 25,
    email: "user@example.com"
};

const validator = new Validator(data);
```

### Applying Validation Rules

To apply validation rules, use the `field` method on your `Validator` instance to specify the field you're validating, then chain any of the available validation methods provided by `ValidatorRules`:

```typescript
validator.field('username').required();
validator.field('age').shouldBeNumber().min(18);
validator.field('email').required().email();
```

### Checking for Errors

After applying your rules, use the `fails` method to check if any validation errors occurred:

```typescript
if (validator.fails()) {
    console.error("Validation errors:", validator.errors);
} else {
    console.log("Validation passed.");
}
```

## Validation Methods

The `ValidatorRules` class offers a variety of methods to cover common validation needs:

### `required(customErrorMessage?: string)`

Ensures the field is present and not undefined.

### `sometimes()`

It just tells that the field may be undefined.

### `min(num: number, customErrorMessage?: string)`

Validates the minimum value or length of the field. Supports strings, numbers, and arrays.

### `max(num: number, customErrorMessage?: string)`

Validates the maximum value or length of the field. Supports strings, numbers, and arrays.

### `notNullable(customErrorMessage?: string)`

Ensures the field is not null.

### `shouldBeString(customErrorMessage?: string)`

Checks if the field value is of type string.

### `shouldBeNumber(customErrorMessage?: string)`

Checks if the field value is of type number.

### `shouldBeBoolean(customErrorMessage?: string)`

Checks if the field value is of type boolean.

### `shouldBeArray(customErrorMessage?: string)`

Checks if the field value is an array.

### `shouldBeObject(customErrorMessage?: string)`

Checks if the field value is an object (and not null or an array).

### `validateObject(validationRules: (validator: Validator) => void)`

Validates a single nested object using provided validation rules.

### `validateArrayOfObjects(validationRules: (validator: Validator) => void)`

Validates each object in an array of objects using provided validation rules.

### `validateArrayOfStrings(customErrorMessage?: string)`

Ensures each value in an array is a string.

### `validateArrayOfNumbers(customErrorMessage?: string)`

Ensures each value in an array is a number.

### `email(customErrorMessage?: string)`

Validates the field value as a valid email address.

### `inList(values: any[], customErrorMessage?: string)`

Ensures the field value is within a predefined list of values.

### `regex(pattern: RegExp, customErrorMessage?: string)`

Validates the field value against a custom regular expression pattern.

### `different(otherParamName: string, customErrorMessage?: string)`

Validates that the value of the current field is different from another field's value.

### `enum(enumObject: object, customErrorMessage?: string)`

Ensures the field value matches one of the predefined options in an enumeration.

### `customValidation(validationFunction: (value: any) => boolean, customErrorMessage?: string)`

Allows for custom validation logic defined by the user. The user-provided function should return `true` if the validation passes or `false` otherwise. This method offers maximum flexibility, enabling the enforcement of complex or specific validation rules that are not covered by the predefined methods.

- **`validationFunction`**: A function that contains the custom validation logic. It receives the field value as its argument.
- **`customErrorMessage`**: Optional. A custom error message that overrides the default error message if the validation fails.

## Advanced Usage Examples

### Nested Object Validation

```typescript
validator
    .field('user')
    .validateObject(userValidator => {
        userValidator
            .field('firstName')
            .required()
            .shouldBeString();
        
        userValidator
            .field('lastName')
            .required()
            .shouldBeString();
    });
```

### Array of Objects Validation

```typescript
validator
    .field('tags')
        .validateArrayOfObjects(tagValidator => {
            tagValidator
                .field('id')
                .shouldBeNumber();
            
            tagValidator
                .field('name')
                .shouldBeString();
        });
```

### Custom Validation
```typescript
validator
    .field('user')
    .required()
    .shouldBeObject()
    .customValidation((value: any) => {
        if (value.mobile === null) {
            return false;
        }
        return true;
    }, 'Custom error message!!!');
```

### Custom Error Messages

You can provide custom error messages for any rule, which can include a `:paramName` placeholder, replaced with the field name:

```typescript
validator.field('age').min(18, "Sorry, you must be at least 18 years old. :paramName is too young.");
```