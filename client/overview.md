# Client Object Model

> The service properties and methods the platform adds to every model object on the client — elements, the root, arrays, array elements, and tree elements.

## Overview

When a model loads, the platform wraps the plain JSON returned by SQL into reactive objects and extends each one with service members whose names begin with `$`. These members let template code navigate the model, inspect state, and manipulate data without touching the raw structure.

There are five object shapes, arranged as an interface hierarchy. Every object is an element (`IElement`). The root object (`IRoot`) adds model-wide state and validation control. Any object that lives in an array is an array element (`IArrayElement`); a tree node (`ITreeElement`) extends that further. Arrays themselves (`IElementArray`) are also extended, with their own collection-level members.

Two more pieces complete the client API. The controller (`IController`), reachable from any object via `$ctrl`, drives interaction with the server and the UI. The [`std:utils`](https://docs-llm.a2v10.com/client/utils.md) module provides helper functions for dates, currency, and text. Both are used heavily from inside a [client template](https://docs-llm.a2v10.com/template/overview.md).

## Syntax

```ts
interface IElement { /* base — every object */ }
interface IRoot         extends IElement { /* the model root (TRoot) */ }
interface IArrayElement extends IElement { /* an element inside an array */ }
interface ITreeElement  extends IArrayElement { /* a node of a tree */ }
interface IElementArray<T> extends Array<T> { /* any model array */ }
interface IController { /* reached from any object via $ctrl */ }
```

### Reference

| Shape | Interface | Reference |
|-------|-----------|-----------|
| Object (base) | `IElement` | [Element](https://docs-llm.a2v10.com/client/element.md) |
| Root object | `IRoot` | [Root Object](https://docs-llm.a2v10.com/client/root.md) |
| Array | `IElementArray<T>` | [Array](https://docs-llm.a2v10.com/client/array.md) |
| Array element | `IArrayElement` | [Array Element](https://docs-llm.a2v10.com/client/array-element.md) |
| Tree element | `ITreeElement` | [Tree Element](https://docs-llm.a2v10.com/client/tree-element.md) |
| Controller | `IController` | [Controller](https://docs-llm.a2v10.com/client/controller.md) |
| Utilities | — | [std:utils](https://docs-llm.a2v10.com/client/utils.md), [require](https://docs-llm.a2v10.com/client/require.md) |

## Example

Reaching the service members from a computed property and a command:

```js
const template = {
  properties: {
    'TDocument.RowCount'() { return this.Rows.Count; } /* array $-member */
  },
  commands: {
    reload(doc) { return doc.$ctrl.$reload(); } /* controller via $ctrl */
  }
};

module.exports = template;
```

## Notes

- All service members are prefixed with `$`. Properties of the model that come from SQL never start with `$`, so the two namespaces never collide.
- Most service properties are read-only; the writable ones (such as `$selected`, `$checked` on an array element) are noted on their reference pages.
- Every object can reach the root via `$root` and the controller via `$ctrl`, regardless of how deeply nested it is.
- These members are the API that [client template](https://docs-llm.a2v10.com/template/overview.md) code is written against.
