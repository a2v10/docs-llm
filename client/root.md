# Root Object (IRoot)

> The model root (TRoot) — extends a base element with model-wide state, the template reference, and validation and dirty-state control.

## Overview

The root object is the top of the model. It extends [`IElement`](https://docs-llm.a2v10.com/client/element.md) with members that concern the whole model: read-only and copy state, a readiness flag, a reference to the [template](https://docs-llm.a2v10.com/template/overview.md), and methods that drive events, validation, and the dirty flag.

Inside template code the root is the `this` of commands and computed properties at document level, and it is reachable from any object through `$root`.

## Syntax

```ts
interface IRoot extends IElement {
  // properties
  readonly $readOnly:      boolean;
  readonly $stateReadOnly: boolean;
  readonly $isCopy:        boolean;
  readonly $ready:         boolean;
  readonly $template:      Template;
  // methods
  $defer(handler: () => any): void;
  $emit(event: string, ...params: any[]): void;
  $forceValidate(): void;
  $revalidate(elem: IElement, rule: string): void;
  $setDirty(dirty: boolean, path?: string): void;
}
```

### Properties

| Member | Type | Description |
|--------|------|-------------|
| `$readOnly` | boolean | The whole model is read-only |
| `$stateReadOnly` | boolean | The model is read-only because of its current state |
| `$isCopy` | boolean | The model was opened as a copy of another record |
| `$ready` | boolean | No server request is currently running — useful for enabling or disabling UI during server operations |
| `$template` | `Template` | Reference to the [template](https://docs-llm.a2v10.com/template/overview.md) object |

### Methods

| Member | Description |
|--------|-------------|
| `$defer(handler)` | Runs `handler` after the current event cycle completes |
| `$emit(event, ...params)` | Raises a custom [event](https://docs-llm.a2v10.com/template/events.md) on the model |
| `$forceValidate()` | Runs all validators immediately |
| `$revalidate(elem, rule)` | Forcibly runs the asynchronous validators of `rule` for `elem` |
| `$setDirty(dirty, path?)` | Sets or clears the dirty flag, optionally for a specific `path` |

## Example

```js
const template = {
  events: {
    'Document.Rows[].Price.change'() {
      this.$defer(() => this.$forceValidate()); /* this === root */
    }
  },
  commands: {
    notify(doc) { doc.$root.$emit('Document.Recalculated'); }
  }
};

module.exports = template;
```

## Notes

- `$forceValidate()` evaluates all rules at once; `$revalidate(elem, rule)` targets only the named async rule for one element. See [validators](https://docs-llm.a2v10.com/template/validators.md).
- `$setDirty` is the programmatic counterpart to the [`options.skipDirty`](https://docs-llm.a2v10.com/template/options.md) flag — use it to mark the model changed (or not) from code.
- `$emit` raises custom events handled in the template [`events`](https://docs-llm.a2v10.com/template/events.md) section; the event name should not collide with system event names.
- Use `$ready` to disable buttons while a server request is in flight, avoiding double submissions.
