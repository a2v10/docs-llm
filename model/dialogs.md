# Dialogs

> The `dialogs` object in model.json — configures modal dialogs that load data and render inside an overlay window.

## Overview

Dialogs are modal windows that open on top of the current page. They follow the same data-loading pattern as actions: a stored procedure is called, the result is bound to a XAML view, and the user interacts with the dialog before closing or saving.

Dialogs inherit `source`, `schema`, and `model` from the root of `model.json` and can override any of them. The stored procedure suffix follows the same convention as actions: `.Load` by default, `.Index` when `index: true`, and `.Copy` when `copy: true`.

A dialog's XAML view must use `Dialog` as its root element. Dialogs are invoked by commands in the view layer — they are not directly navigable by URL.

## Syntax

```json
"dialogs": {
  "<dialogName>": {
    "source":     "",
    "schema":     "",
    "model":      "",
    "index":      false,
    "copy":       false,
    "view":       "",
    "template":   "",
    "checkTypes": "",
    "parameters": {},
    "merge":      {}
  }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `source` | string | Overrides the root `source` for this dialog |
| `schema` | string | Overrides the root `schema` for this dialog |
| `model` | string | Overrides the root `model` for this dialog |
| `index` | boolean | If `true`, calls `[model].Index` instead of `[model].Load` |
| `copy` | boolean | If `true`, calls `[model].Copy` |
| `view` | string | Path to the XAML view file (without extension) |
| `template` | string | Path to the JavaScript template file (without extension) |
| `checkTypes` | string | Path to a TypeScript `.d.ts` file for dynamic type checking |
| `parameters` | object | Static key-value pairs passed to the stored procedure |
| `merge` | object | Secondary model merged into the primary model after loading |

### merge object

Works identically to the `merge` in actions. Parameter values can reference the primary model using double-brace syntax: `"{{Path.To.Property}}"`.

## Example

### Edit dialog

```json
"dialogs": {
  "edit": {
    "view":     "edit",
    "template": "edit"
  }
}
```

Calls `[a2].[Agent.Load]` and renders `edit.xaml` inside a modal dialog.

### Lookup dialog (index mode)

```json
"dialogs": {
  "browse": {
    "index": true,
    "view":  "browse"
  }
}
```

Calls `[a2].[Agent.Index]` to populate a list shown inside the dialog.

### Copy dialog with merge

```json
"dialogs": {
  "copy": {
    "copy": true,
    "view": "edit",
    "merge": {
      "model": "AgentCategory",
      "parameters": {
        "Id": "{{Agent.Category.Id}}"
      }
    }
  }
}
```

Calls `[a2].[Agent.Copy]` and then merges `[a2].[AgentCategory.Load]` using the agent's category id.

## Notes

- `index` and `copy` are mutually exclusive.
- The XAML view for a dialog should use `<Dialog>` as its root element, not `<Page>`.
- Dialogs do not appear in the URL; they are opened programmatically from the view layer.
- `parameters` are always static. Dynamic parameters (based on model data) must be passed from the invoking view via `BindCmd` arguments.
