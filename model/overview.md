# model.json Overview

> The required configuration file in every endpoint directory — declares all permitted operations and their data-source bindings.

## Overview

Every subdirectory that represents an endpoint in an A2v10 application must contain a `model.json` file. This file describes all operations available at that URL: which views to render, which stored procedures to call, which dialogs and commands are reachable, and how files are handled.

An endpoint maps to a single domain entity. When a user navigates to `/agent/`, the platform reads `/agent/model.json` to determine what is allowed and how to respond. Nothing outside that file can be invoked for that endpoint.

The top-level properties `source`, `schema`, and `model` act as defaults. Every child section (`actions`, `dialogs`, `popups`, `commands`, `reports`, `files`) inherits these values and can override them individually. This makes it easy to work with a single schema and data source while selectively pointing specific operations elsewhere.

Property names within each child object become operation names. For example, an `actions` object with a key `"index"` defines an action reachable as the `index` operation of that endpoint.

## Syntax

```json
{
  "source": "",
  "schema": "",
  "model": "",
  "requireBase": false,
  "actions":   {},
  "dialogs":   {},
  "popups":    {},
  "commands":  {},
  "reports":   {},
  "files":     {},
  "blobs":     {}
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `source` | string | `"Default"` | Connection string name from the application configuration |
| `schema` | string | `"dbo"` | SQL schema used for stored procedure calls |
| `model` | string | — | Base model name, inherited by child sections unless overridden |
| `requireBase` | boolean | `false` | .NET Core only. Every call must carry `base` — the owner's endpoint URL; its roles are checked first. Used for attachments, see [Blobs](https://docs-llm.a2v10.com/model/blobs.md) |
| `actions` | object | — | Named page actions — each renders a view and loads a model |
| `dialogs` | object | — | Named modal dialogs |
| `popups` | object | — | Named popup windows |
| `commands` | object | — | Named server-side commands |
| `reports` | object | — | Named report or export operations |
| `files` | object | — | Named file upload operations |
| `blobs` | object | — | .NET Core only. Fields a file or an image can be uploaded into — see [Blobs](https://docs-llm.a2v10.com/model/blobs.md) |

## Example

A minimal `model.json` for an `agent` endpoint with a list action and an edit dialog:

```json
{
  "schema": "a2",
  "model":  "Agent",
  "actions": {
    "index": {
      "index": true,
      "view": "index"
    },
    "edit": {
      "view": "edit"
    }
  },
  "dialogs": {
    "edit": {
      "view": "edit"
    }
  },
  "commands": {
    "delete": {
      "type": "sql",
      "procedure": "Agent.Delete"
    }
  }
}
```

## Notes

- Omitting `source` or `schema` at root level is fine — the platform uses `"Default"` and `"dbo"` respectively.
- Setting a property to an empty string `""` explicitly clears the inherited value (it does not fall back to the parent).
- Each child section is independent; you can have `files` without `reports`, `commands` without `dialogs`, etc.
- The platform does not allow operations that are not declared in `model.json` — undeclared action names return an error.
- Endpoint directories can be nested; each level may have its own `model.json`.
