# Actions

> The `actions` object in model.json — configures page-level operations that load data and render views.

## Overview

Actions are the primary entry points for user interaction with an endpoint. Each action loads a data model from SQL Server and renders a XAML view. When a user navigates to an endpoint URL, the platform resolves the action name from the URL and executes it.

Each action calls a stored procedure to load its model. By default, an action calls `[schema].[model.Load]`. Setting `index: true` changes the suffix to `.Index`, which is used for list/grid views. Setting `copy: true` uses `.Copy`, for actions that duplicate an existing record.

The `view` and `template` properties locate the XAML file and JavaScript template file respectively. Paths are relative to the endpoint directory and omit the file extension.

Actions inherit `source`, `schema`, and `model` from the root of `model.json`. Any of these can be overridden per action.

## Syntax

```json
"actions": {
  "<actionName>": {
    "source":     "",
    "schema":     "",
    "model":      "",
    "index":      false,
    "copy":       false,
    "view":       "",
    "template":   "",
    "checkTypes": "",
    "indirect":   false,
    "parameters": {},
    "merge":      {},
    "export":     {}
  }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `source` | string | Overrides the root `source` for this action |
| `schema` | string | Overrides the root `schema` for this action |
| `model` | string | Overrides the root `model` for this action |
| `index` | boolean | If `true`, calls `[model].Index` instead of `[model].Load` |
| `copy` | boolean | If `true`, calls `[model].Copy`; cannot be combined with `index` |
| `view` | string | Path to the XAML view file (without extension), relative to the endpoint |
| `template` | string | Path to the JavaScript template file (without extension) |
| `checkTypes` | string | Path to a TypeScript `.d.ts` file for dynamic type checking |
| `indirect` | boolean | Enables two-stage (indirect) execution |
| `parameters` | object | Static key-value pairs passed to the stored procedure as parameters |
| `merge` | object | Secondary model merged into the primary model after loading |
| `export` | object | Configures a downloadable export for this action |

### merge object

The `merge` object loads an additional model and merges it with the primary one. It supports its own `source`, `schema`, `model`, and a `parameters` object. Parameter values in `merge.parameters` can reference primary model values using double-brace syntax: `"{{Root.Id}}"`.

### export object

| Property | Type | Description |
|----------|------|-------------|
| `fileName` | string | Download filename |
| `format` | string | `xlsx`, `dbf`, or `csv` |
| `template` | string | Required for `xlsx` — path to the Excel template file |
| `encoding` | string | Required for `dbf` and `csv` — `utf8`, `1251`, or `866` |

## Example

### Index action (list view)

```json
"actions": {
  "index": {
    "index": true,
    "view":  "index"
  }
}
```

Calls `[a2].[Agent.Index]` and renders `index.xaml`.

### Edit action with export

```json
"actions": {
  "edit": {
    "view":     "edit",
    "template": "edit",
    "export": {
      "fileName": "agent.xlsx",
      "format":   "xlsx",
      "template": "agent_template"
    }
  }
}
```

Calls `[a2].[Agent.Load]`, renders `edit.xaml`, and provides an Excel download using `agent_template.xlsx`.

### Action with static parameters and merge

```json
"actions": {
  "browse": {
    "index":  true,
    "view":   "browse",
    "parameters": {
      "Kind": 1
    },
    "merge": {
      "model": "AgentKind",
      "parameters": {
        "Id": "{{Agent.Kind}}"
      }
    }
  }
}
```

## Notes

- `index` and `copy` are mutually exclusive.
- If `model` is empty at both root and action level, no stored procedure is called — the action renders the view with an empty model.
- `parameters` values are passed as SQL parameters; names must match the stored procedure's parameter names (without the `@` prefix).
- `indirect: true` causes the platform to perform a preliminary request before the main data load — used when the view needs to resolve some state before fetching data.
- `checkTypes` is resolved at development time only; it has no effect in production builds.
