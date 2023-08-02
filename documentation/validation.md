# Validation

Validation in Sosise is implemented using the popular `validatorjs` library. It provides a wide range of validation rules that you can use to ensure the integrity and correctness of incoming data.

## Using Validation Rules

Validation rules are applied using the `validatorjs` library, which provides a simple and expressive way to define rules for validating data. These rules are commonly used in Unifiers to validate incoming request data and ensure it meets the required criteria.

## Available Rules

For a comprehensive list of all available validation rules and their usage, please refer to the official `validatorjs` documentation at: [https://github.com/mikeerickson/validatorjs](https://github.com/mikeerickson/validatorjs).

## Examples

As mentioned in the Unifiers page, validation is used in Unifiers to ensure that incoming data is in the correct format and adheres to the defined rules. Here's a brief example of how validation rules are used in a Unifier:

```typescript
import Validator from 'validatorjs';
import ValidationException from 'sosise-core/build/Exceptions/Validation/ValidationException';

export default class OrderCreatingUnifier {
    // ... Other code ...

    /**
     * Request data validation
     */
    private validate() {
        // Create validator with the data to be validated and the corresponding rules
        const validator = new Validator(this.params, {
            customerId: ['required', 'numeric', 'min:1'],
            shop: ['required', 'string'],
        });

        // If validation fails, throw a ValidationException with the error messages
        if (validator.fails()) {
            throw new ValidationException('Validation exception', (validator.errors.all() as any));
        }
    }

    // ... Other code ...
}
```

In this example, the `validatorjs` library is used to validate the `customerId` and `shop` parameters. The `required`, `numeric`, and `min:1` rules are applied to the `customerId`, while the `required` and `string` rules are applied to the `shop` parameter. If any of the validation rules fail, a `ValidationException` is thrown with the corresponding error messages.

For more complex validation scenarios, you can refer to the `validatorjs` documentation, which provides a comprehensive list of available rules and their usage. This allows you to build robust and reliable validation logic for your application, ensuring that only valid and properly formatted data is accepted.