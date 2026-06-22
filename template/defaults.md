# Defaults

> The `defaults` section of a template — initial values applied to a new model.

## Overview

The `defaults` section sets initial values for a new model. A model is considered new when its primary object identifier is zero or empty — for example, when the user creates a document that has not yet been saved.

The descriptor is an ordinary JavaScript object. Each property name is a path to a data element in the model — the property whose value should be set. Each value is the value to assign: a literal (string, number, boolean, date, object) or a function that returns the value.

When the value is a function, it is evaluated against the new model. The function receives the containing element and the property name, and `this` is the model root. This is how dynamic defaults such as the current date are computed.

## Syntax

```ts
/* template defaults */
interface templateDefaultFunc { (this: IRoot, elem: IElement, prop: string): any; }
declare type templateDefault = templateDefaultFunc | string | number | boolean | Date | object;

template: {
  defaults?: {
    [prop: string]: templateDefault
  }
}
```

## Example

Seed a new document with today's date and a default state:

```js
const dateUtils = require('std:utils').date;

const template = {
  defaults: {
    'Document.Date':  dateUtils.today(), /* today's date */
    'Document.State': 'Draft'
  }
};

module.exports = template;
```

## Notes

- Defaults apply only to a new model (primary identifier zero or empty). They do not overwrite values on an existing, loaded record.
- A function default is evaluated when the new model is initialized; `this` is the model root, and the function receives the element and property name.
- The property name is a path into the model, so nested elements are addressed with dot notation (for example `Document.Agent.Id`).
- Compute dynamic values such as dates with the standard utilities — `require('std:utils').date` — rather than constructing them inline.
