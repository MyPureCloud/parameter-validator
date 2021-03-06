# Parameter Validator

Parameter-validator makes it easy to verify that a JavaScript object contains required, valid parameters.

## Examples

### Basic Examples

```js
import { validate, ParameterValidationError } from 'parameter-validator';

let params = { name: 'Paula PureCloud', id: 'user1' };

let { name, id } = validate(params, [ 'name', 'id' ]);
// parameters exist, so no error is thrown

try {
    let { age } = validate(params, [ 'age' ]);
} catch (error) {
    if (error instanceof ParameterValidationError) {
        console.log(error.message);
        // "Invalid value of 'undefined' was provided for parameter 'age'."
    }
}
```

### Async Example

To ensure that the any errors thrown are wrapped in a Promise, use the async version:

```js
import { validateAsync, ParameterValidationError } from 'parameter-validator';

validateAsync(params, [ 'price', 'quantity' ])
.then(({ price, quantity }) => {

    console.log(`Price: ${price}, Quantity: ${quantity}`) ;
})
.catch(error => {

    if (error instanceof ParameterValidationError) {
        // Handle invalid parameters
    } else {
        throw error;
    }
});
```

### Advanced usage

#### Other types of validation

* You can specify that at least one of a group of parameters must be included by placing those properties together within a nested array (either `username` or `email` must be specified in the example below)

* You can provide a specific validation function for a parameter by providing it in an object

```js
validate(params, [
    'firstName',
    'lastName',
    [ 'username', 'email' ],
    { age: val => val > 30 }
]);
```

#### Optional parameters

##### Passing in an object to which the extracted parameters will be assigned.

```js
// Sets `this.logger` and `this.username`
validate(params, [ 'logger', 'username' ], this);
```

##### Adding a prefix to param names

```js
// Sets `this._logger` and `this._username`
validate(params, [ 'logger', 'username' ], this, { addPrefix: '_' });
```

### ParameterValidator class

For convenience, `validate()` and `validateAsync()` are exported as standalone functions as shown above, but it's also possible to import and instantiate the `ParameterValidator` class that implements those methods.

```js
import ParameterValidator from 'parameter-validator';

let parameterValidator = new ParameterValidator();
let { firstName, lastName } = parameterValidator.validate(options, [ 'firstName', 'lastName' ]);
// or
parameterValidator.validateAsync(options, [ 'firstName', 'lastName' ])
.then(({ firstName, lastName }) => {
   ...
});
```

### Parameters for `validate` and `validateAsync`

```
param:   {Object}       paramsProvided       - The names and values of provided parameters
param:   {Array}        paramRequirements    - Each item in this array is interpretted in order as a validation rule.
                                             - If an item is a string, it's interpretted as the name of a parameter that must be contained in paramsProvided.
                                             - If an item is an Array, it's interpretted as an array of parameter names where at least one of the
                                               parameters in the Array must be in paramsProvided.
                                             - If an item is an Object, it's assumed that the object's only key is the name of a parameter to be validated
                                               and its corresponding value is a function that returns true if that parameter's value in paramsProvided is
                                               valid.
param:   {Object|null}  [extractedParams]    - This method returns an object containing the names and values of the validated parameters extracted.
                                               By default, it creates a new object and assigns the extracted parameters to it, but if you want this
                                               method to add the extracted params to an existing object (such as the class instance that internally
                                               invokes this method), you can optionally supply that object as the extractedParams parameter.

param:   {Object}       [options]            - Object of additional options.
param:   {string}       [options.addPrefix]  - Specifies a prefix that will be added to each param name before it's assigned to the
                                               extractedParams object. This is useful, for example, for prefixing property names with an underscore
                                               to indicate that they're private properties.
param:   {class}        [options.errorClass] - Specifies a specific `Error` subclass to throw instead of the default `ParameterValidationError
                                               when invalid parameters are detected.

returns: {Object}       extractedParams      - The names and values of the validated parameters extracted.

throws:  {ParameterValidationError}          - Indicates that one or more parameter validation rules failed. The error message identifies the names and
                                               values of each invalid parameter.
```

## Installation

```
npm install parameter-validator --save
```

This module ships with two different builds: one for the CommonJS module spec and one for the AMD module spec. The CommonJS module is specified as the default entry point, so you can import the module as expected in Node.js without any additional work.

### Installation in Ember.js

Ember.js uses the AMD module system, and a couple of extra steps are required to correctly import the AMD module.

1. Install parameter-validator using `npm` as specified above.
2. Install the [ember-cli-node-modules-to-vendor](`https://www.npmjs.com/package/ember-cli-node-modules-to-vendor`) addon, which copies specific node modules to your `vendor` directory so that they can be imported.
3. Add the following lines to `ember-cli-build.js`:

```js
let app = new EmberApp(defaults, {
    // Instructs the ember-cli-node-modules-to-vendor addon to copy this module to `/vendor`
    nodeModulesToVendor: [
        'node_modules/parameter-validator/dist/amd'
    ]
});

// Instructs the app to import the AMD module so that you can import it in your code.
app.import('vendor/ParameterValidator.js', {
    using: [
        { transformation: 'amd', as: 'parameter-validator' }
    ]
});
```

## Development

The module is implemented in ES 6 (located in the src directory) but has been transpiled to ES 5 using Babel (located in the dist directory). The package.json file specifies the dist directory for the module's entry point, so the transpiled code will be used automatically.

### Building

```
npm run build
```

There's also a git pre-commit hook that automatically builds upon commit, since the dist directory is committed.

### Running tests

```
npm test
```

