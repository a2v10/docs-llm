# Properties

> The `properties` section of a template — additional scalar and computed properties added to model elements.

## Overview

The `properties` section adds extra attributes to model elements. An attribute can be a scalar value (string, number, boolean) or a computed property — a function evaluated when the property is read, with its return value used as the property's value. Computed properties are a powerful way to express business logic such as line totals and document sums without storing them.

The descriptor is an ordinary JavaScript object. Each property name is a fully qualified name that includes the element's type name (for example `TRow.Sum`). Each value is a scalar, a getter function, or an object with `get` and `set` functions.

For a computed property the function is executed on every access, and `this` is the containing object. Never use an arrow function for a computed property — an arrow function has no own `this` and cannot read the element's data. Avoid server calls inside a computed property; cache the result if a lookup is unavoidable.

## Syntax

```ts
properties: {
  'TypeName.property': String | Number | Boolean, /* scalar property */
  'TypeName.property': Function,                   /* computed property (getter) */
  'TypeName.property': {                           /* computed property with setter */
    get: Function,
    set: Function
  }
}
```

### Naming rules

| Rule | Effect |
|------|--------|
| `TypeName` prefix | Must match the type name used when the model is built; by convention type names start with `T` (for "Type") |
| Name matches existing property | Replaces the model property instead of adding a new one |
| Name starts with `$` or `_` | Internal — excluded from client/server data exchange |
| Name starts with `$$` | Does not set the `$isDirty` flag when changed |

## Example

A computed row sum and a document sum that aggregates the rows:

```js
const template = {
  properties: {
    'TRow.Sum'() { return this.Qty * this.Price; },  /* row sum */
    'TDocument.Sum': getDocumentSum                   /* document sum */
  }
};

function getDocumentSum() {
  return this.Rows.reduce((prev, curr) => prev + curr.Sum, 0);
}

module.exports = template;
```

## Notes

- A computed property is a function, not a value — it runs on every read. Keep it cheap and side-effect free.
- Never use arrow functions for computed properties; they have no `this` binding and cannot access the element.
- Do not call the server inside a computed property. If a remote value is required, cache it; for server-dependent validation use an [async validator](https://docs-llm.a2v10.com/template/validators.md) instead.
- Properties with setters can recurse: a setter that changes a value the getter depends on may loop. Use setters carefully.
- A setter receives `this` (the element) and `value`.
- Naming a property the same as an existing model property replaces it — use this to override a value coming from SQL.

## Hints

- Use `$$`-prefixed names for derived display values that should not mark the model dirty (for example a formatted caption).
- Use `$`/`_` prefixes for purely client-side scratch state you do not want sent back to the server on save.
