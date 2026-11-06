# Element (IElement)

> The base service members added to every model object — references to the root, parent, and controller, plus identity and validation state.

## Overview

`IElement` is the base shape of every object in the client model. Whatever its type, an object carries these members: links upward to its parent and the root, a link to the controller, its identity (`$id`, `$name`), and flags describing whether it is new, empty, and valid.

The root object, array elements, and tree elements all extend this base with extra members of their own. This page covers only the members shared by every object.

## Syntax

```ts
interface IElement {
  // properties
  readonly $root:    IRoot;
  readonly $parent:  IElement | IElementArray<IElement>;
  readonly $id:      any;
  readonly $name:    any;
  readonly $isNew:   boolean;
  readonly $isEmpty: boolean;
  readonly $valid:   boolean;
  readonly $invalid: boolean;
  readonly $ctrl:    IController;
  // methods
  $empty(): IElement;
  $merge(src: Object): IElement;
  $noChangeEvents(fn: (elem: IElement) => any): any;
}
```

### Properties

| Member | Type | Description |
|--------|------|-------------|
| `$root` | `IRoot` | Reference to the [root object](https://docs-llm.a2v10.com/client/root.md) of the model |
| `$parent` | `IElement` \| `IElementArray` | Reference to the parent object or array |
| `$id` | any | The object's identifier (the property marked as the id) |
| `$name` | any | The object's display name (the property marked as the name) |
| `$isNew` | boolean | The object is new (its identifier is zero or empty) |
| `$isEmpty` | boolean | The object has no meaningful data |
| `$valid` | boolean | The object passes all its validators |
| `$invalid` | boolean | The object fails at least one validator |
| `$ctrl` | `IController` | Reference to the [controller](https://docs-llm.a2v10.com/client/controller.md) |

### Methods

| Member | Returns | Description |
|--------|---------|-------------|
| `$empty()` | `IElement` | Clears the object, resetting all properties to defaults; recursive — nested objects are cleared too. Returns itself |
| `$merge(src)` | `IElement` | Copies the properties of `src` into the object. Returns itself |
| `$noChangeEvents(fn)` | any | Calls `fn` so that changing the object's properties raises no [`.changing` / `.change` events](https://docs-llm.a2v10.com/template/events.md). `this` and the argument of `fn` are the object itself. Returns what `fn` returns |

## Example

```js
const template = {
  commands: {
    clearAgent(doc) { doc.Agent.$empty(); },          /* reset a nested object */
    canSubmit(doc) { return doc.$valid && !doc.$isNew; } /* read state flags */
  },
  events: {
    'Document.Rows[].Price.change'(row) {
      // Sum is recalculated without raising Document.Rows[].Sum.change
      row.$noChangeEvents(r => r.Sum = r.Price * r.Qty);
    }
  }
};

module.exports = template;
```

## Notes

- `$id` and `$name` reflect the properties marked as the identifier and name when the model is shaped in SQL — they are not literally named `Id` and `Name` unless the model is.
- `$empty()` is recursive: clearing an object clears every object nested inside it.
- `$valid` / `$invalid` evaluate the [validators](https://docs-llm.a2v10.com/template/validators.md) defined for the object in the template.
- Inside `$noChangeEvents` the model still becomes dirty (`$dirty`). Only the object's own properties are silenced: changes in nested objects and arrays raise events as usual.
- `$root` and `$ctrl` are available on every object, so deeply nested code never has to thread these references manually.
