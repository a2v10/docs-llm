# Events

> The `events` section of a template — handlers for model and element lifecycle events.

## Overview

The `events` section reacts to changes in the model: the model loading or saving, an object being constructed, a property changing, or an element being added to or removed from an array. Like the rest of the template, events are bound to data, not to UI controls.

The descriptor is an ordinary JavaScript object where each property name is an event name and each value is a handler function. Event names are formed in different ways. Whole-model events have fixed names (`Model.load`, `Model.saved`). Object and array events combine a path or type name with a suffix — a construct handler is `{Type}.construct`, a change handler is `{PropertyName}.change`, an array-add handler is `{ArrayName}.add`.

Beyond these system events, custom events can be raised from code with `$emit` and may use any name that does not collide with a system name.

## Syntax

```ts
const template = {
  events: {
    'Model.load':            (this: IRoot, root: IRoot, caller?: IRoot) => void,
    'Model.unload':          (this: IRoot, root: IRoot) => void,
    '{Type}.construct':      (this: IRoot, elem: IElement, prop?: string) => void,
    '{PropertyName}.change':   (this: IRoot, elem?: IElement, newVal?: any, oldVal?: any, prop?: string) => void,
    '{PropertyName}.changing': (this: IRoot, elem?: IElement, newVal?: any, oldVal?: any, prop?: string) => boolean,
    '{ArrayName}.add':       (this: IRoot, array?: IArrayElement, elem?: IElement) => void
  }
};
```

### Model events

| Event | When it fires |
|-------|---------------|
| `Model.load` | The model has loaded |
| `Model.unload` | The model has unloaded |
| `Model.saved` | The model has been saved |
| `Model.beforeSave` | Before the model is saved |

### Object events

| Event | When it fires |
|-------|---------------|
| `{Type}.construct` | An object of the type is created |
| `{PropertyName}.change` | A property value has changed |
| `{PropertyName}.changing` | A property value is about to change — return `false` to cancel |

### Array events

| Event | When it fires |
|-------|---------------|
| `{ArrayName}.adding` | Before an element is added |
| `{ArrayName}.add` | An element has been added |
| `{ArrayName}.change` | An array element has changed |
| `{ArrayName}.remove` | An element has been removed |
| `{ArrayName}.select` | An element has been selected |

## Example

```js
const dateUtils = require('std:utils').date;

const template = {
  events: {
    'TDocument.construct'(doc) {
      doc.Date = dateUtils.today();
    },
    'Document.Rows[].Qty.change'(row) {
      row.Total = row.Qty * row.Price;
    },
    'Document.Rows.add'(rows, row) {
      row.LineNo = rows.length;
    }
  }
};

module.exports = template;
```

## Notes

- Array events fire only when the array is manipulated through the platform's own methods. Direct mutation of the underlying JavaScript array does not raise them.
- `changing` runs before the change and can veto it by returning `false`; `change` runs after the change is applied.
- Object and array event names are built from the model path or type name plus a suffix — `Document.Rows[].Qty.change`, `Document.Rows.add`.
- Custom events raised with `$emit` may carry any name; keep them distinct from the system names above.
- Event handlers are bound to the data model, so they run regardless of which control triggered the change.
