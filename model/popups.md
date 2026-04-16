# Popups

> The `popups` object in model.json — configures lightweight popup windows for quick lookups and inline selection.

## Overview

Popups are smaller, lighter-weight overlays compared to dialogs. They are typically used for lookup lists, autocomplete panels, and inline pickers — cases where the user selects a value rather than editing a full form.

Like dialogs, popups load data via stored procedures and render a XAML view. The view root element for a popup is usually `Popup`. Popups inherit `source`, `schema`, and `model` from the root of `model.json`.

The stored procedure suffix follows the same convention: `.Load` by default, `.Index` when `index: true`.

## Syntax

```json
"popups": {
  "<popupName>": {
    "source":     "",
    "schema":     "",
    "model":      "",
    "index":      false,
    "view":       "",
    "template":   "",
    "parameters": {}
  }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `source` | string | Overrides the root `source` for this popup |
| `schema` | string | Overrides the root `schema` for this popup |
| `model` | string | Overrides the root `model` for this popup |
| `index` | boolean | If `true`, calls `[model].Index` instead of `[model].Load` |
| `view` | string | Path to the XAML view file (without extension) |
| `template` | string | Path to the JavaScript template file (without extension) |
| `parameters` | object | Static key-value pairs passed to the stored procedure |

## Example

### Simple lookup popup

```json
"popups": {
  "browse": {
    "index": true,
    "view":  "browse"
  }
}
```

Calls `[a2].[Agent.Index]` and renders `browse.xaml` in a popup panel. Typically bound to a `Selector` control in the parent view.

### Popup with static filter parameter

```json
"popups": {
  "activeAgents": {
    "index": true,
    "view":  "browse",
    "parameters": {
      "Active": true
    }
  }
}
```

Passes `@Active = 1` to the stored procedure to pre-filter the list.

## Notes

- Popups do not support `copy` or `merge` — use a dialog for those patterns.
- The XAML view for a popup typically uses `<Popup>` as its root element.
- Popups are opened from the view layer via `BindCmd` with the appropriate command type; they are not navigable by URL.
- `parameters` are always static; dynamic filtering should be implemented inside the stored procedure using optional parameters passed from the calling view.
