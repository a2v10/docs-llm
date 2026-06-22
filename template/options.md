# Options

> The `options` section of a template — flags that tune the behavior of the whole model.

## Overview

Options set behavior that applies to the entire model rather than to a single property. They control how the change flag is managed, which root properties bind only once, and which arrays preserve their selected element across save and reload.

The most important option is the dirty flag. The model exposes `$isDirty`, which becomes `true` whenever the data changes. The platform uses it to prompt about unsaved changes and to enable controls — a button bound to the `Save` command is automatically enabled once the model becomes dirty. The `noDirty` and `skipDirty` options change when and whether that flag is set.

## Syntax

```js
options: {
  noDirty?:        boolean,
  bindOnce?:       string[],
  persistSelect?:  string[],
  skipDirty?:      string[]
}
```

| Property | Type | Description |
|----------|------|-------------|
| `noDirty` | boolean | Do not set the `$isDirty` flag on any model change |
| `bindOnce` | string[] | Root properties bound to data only on first load |
| `persistSelect` | string[] | Arrays whose selected element position is preserved across save and reload |
| `skipDirty` | string[] | Property paths whose changes do not set the `$isDirty` flag |

## Example

```js
const template = {
  options: {
    bindOnce:      ['Agents', 'Categories'],
    persistSelect: ['Document.Rows'],
    skipDirty:     ['Document.SearchText']
  }
};

module.exports = template;
```

## Notes

- `bindOnce` accepts root properties only — names without a dot. It is useful for binding reference lists (lookups) once, avoiding a rebind on every model reload.
- `persistSelect` keeps the `$selected` element of an array in place when the model is saved or reloaded, so the user's row selection is not lost.
- `skipDirty` takes full property paths; changing one of those properties leaves `$isDirty` unchanged. Use it for transient UI state such as a search box that should not count as an unsaved edit.
- With `noDirty` set, `$isDirty` never becomes `true`, so the platform will not warn about unsaved changes or auto-enable a `Save` command for this model.
- Property names beginning with `$$` in [`properties`](https://docs-llm.a2v10.com/template/properties.md) also avoid setting the dirty flag — a per-property alternative to `skipDirty`.
