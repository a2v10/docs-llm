# Validators

> The `validators` section of a template — validation rules bound to model data rather than to the user interface.

## Overview

Validators check the correctness of the model. They are bound to data, not to controls — the controls themselves track the validity of the data they display and show the corresponding messages. The same rule therefore applies wherever a value is rendered.

The descriptor is an ordinary JavaScript object. Each property name is a path to a data element in the model. When the path crosses an array, the array segment is marked with the `[]` suffix; a validator for the `Sum` property of a document row is named `Document.Rows[].Sum`.

A validator value can be a single validator or an array of validators — when it is an array, all of them are applied. Each validator is a string, a function, or an object. The object is the full form; the string and function forms are shorthand for it.

## Syntax

```ts
declare const enum Severity {
  error   = 'error', /* default */
  warning = 'warning',
  info    = 'info'
}

declare const enum StdValidator {
  notBlank = 'notBlank',
  email    = 'email',
  url      = 'url',
  isTrue   = 'isTrue',
  regExp   = 'regExp'
}

type templateValidatorResult = { msg: string, severity: Severity };

interface templateValidatorFunc {
  (elem: IElement, value?: any): boolean | string | templateValidatorResult | Promise<any>;
}

interface templateValidatorObj {
  valid:     templateValidatorFunc | StdValidator,
  async?:    boolean,
  msg?:      string,
  regExp?:   RegExp,
  severity?: Severity,
  applyIf?:  (elem: IElement, value?: any) => boolean
}

type templateValidator = string | templateValidatorFunc | templateValidatorObj;

validators: {
  [prop: string]: templateValidator | templateValidator[]
}
```

### Validator object properties

| Property | Description |
|----------|-------------|
| `valid` | Standard validator name (string) or a validation function |
| `async` | Optional. Marks the validator asynchronous |
| `msg` | Error message; a function may also return the message as a string |
| `severity` | Severity of the result; defaults to `'error'`. Affects appearance and can be read in code |
| `regExp` | Regular expression for the standard `'regExp'` validator |
| `applyIf` | Function deciding whether the validator should apply |

### Standard validators

A string `valid` value selects a standard validator:

| Value | Checks |
|-------|--------|
| `notBlank` | Value is not empty |
| `email` | Value is a syntactically valid email address |
| `url` | Value is a syntactically valid web address |
| `isTrue` | Value is exactly `true` |
| `regExp` | Value matches `regExp` (which must be supplied) |

### Shorthand forms

A string validator is an empty-value check; the string itself is the error message. It is equivalent to:

```js
{ valid: 'notBlank', msg: 'the given string', severity: Severity.error }
```

A function validator is equivalent to:

```js
{ valid: func, severity: Severity.error }
```

### Validation function

```ts
valid(elem: IElement, value?: any): boolean | string | templateValidatorResult | Promise<any>;
```

The function receives `elem` (the element being validated) and `value` (the value to check). It returns:

| Return | Meaning |
|--------|---------|
| `true` | Value is valid |
| `false` | Value is invalid |
| `string` | Error message — overrides `msg` |
| `templateValidatorResult` | Object specifying message and severity |
| `Promise` | For an async validator; must resolve to a boolean or string |

The `applyIf` function has the same `(elem, value)` signature and returns `true` when the validator is active and should be applied.

## Example

```js
const template = {
  validators: {
    'Document.Agent':       'Agent is required',
    'Document.Email':       { valid: 'email', msg: 'Invalid email address' },
    'Document.Rows[].Price': {
      valid(elem, value) { return value > 0; },
      msg: 'Price must be positive',
      severity: 'warning',
      applyIf(elem) { return elem.Qty > 0; }
    }
  }
};

module.exports = template;
```

## Notes

- Validator names are data paths. Array segments use the `[]` suffix — `Document.Rows[].Sum`, not `Document.Rows.Sum`.
- A property's value may be one validator or an array of validators; an array applies all of them.
- `severity` defaults to `'error'`. `'warning'` and `'info'` are advisory and do not block saving the way an error does.
- `applyIf` lets a validator depend on other fields — skip the price check on rows with no quantity, for example.

## Hints

- Use an async validator (`async: true`) when correctness must be confirmed on the server. The function returns a `Promise` resolving to a boolean or string, typically via the controller method `$asyncValid`.
- Async results are cached to avoid flooding the server: the validator runs once per change of the values it depends on.
