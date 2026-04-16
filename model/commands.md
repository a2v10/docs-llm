# Commands

> The `commands` object in model.json — defines server-side operations triggered from the UI without navigating away from the current page.

## Overview

Commands execute server-side logic in response to user actions — saving a record, deleting an item, calling an external API, starting a business process, or running a custom .NET method. Unlike actions, commands do not load a new view; they perform an operation and return a result to the calling view.

Each command has a required `type` that determines its execution mechanism. The most common type is `sql`, which calls a stored procedure. Other types delegate to .NET code, external HTTP endpoints, or the platform's process engine.

Commands inherit `source` and `schema` from the root of `model.json`.

## Syntax

```json
"commands": {
  "<commandName>": {
    "type":      "sql | clr | callApi | javascript | startProcess | resumeProcess | sendMessage | file",
    "source":    "",
    "schema":    "",
    "procedure": "",
    "clrType":   "",
    "file":      "",
    "parameters": {},
    "signal":    false,
    "debugOnly": false
  }
}
```

### type values

| Value | Description |
|-------|-------------|
| `sql` | Calls a stored procedure (requires `procedure`) |
| `clr` | Calls a .NET object implementing `IInvokeTarget` (requires `clrType`) |
| `callApi` | Makes an outbound HTTP request to an external API |
| `javascript` | Executes a server-side JavaScript module |
| `startProcess` | Starts a workflow/business process |
| `resumeProcess` | Continues a paused workflow/business process |
| `sendMessage` | Sends an internal platform message |
| `file` | Returns a file as a download |

### Common properties

| Property | Type | Description |
|----------|------|-------------|
| `source` | string | Overrides the root `source` |
| `schema` | string | Overrides the root `schema` |
| `procedure` | string | Stored procedure name — required for `sql` type |
| `clrType` | string | Assembly-qualified .NET type descriptor — required for `clr` type |
| `file` | string | File path — used with `file` type |
| `parameters` | object | Static key-value pairs passed to the operation |
| `signal` | boolean | If `true`, sends a SignalR notification on completion (.NET Core only) |
| `debugOnly` | boolean | If `true`, the command is available only in debug configuration |

## Example

### SQL delete command

```json
"commands": {
  "delete": {
    "type":      "sql",
    "procedure": "Agent.Delete"
  }
}
```

Calls `[a2].[Agent.Delete]` with the parameters supplied by the calling view.

### SQL command with static parameter

```json
"commands": {
  "activate": {
    "type":      "sql",
    "procedure": "Agent.SetState",
    "parameters": {
      "State": 1
    }
  }
}
```

### CLR command

```json
"commands": {
  "sendReport": {
    "type":    "clr",
    "clrType": "MyApp.Commands.SendReportCommand, MyApp"
  }
}
```

Instantiates `SendReportCommand` from the `MyApp` assembly and calls its `InvokeAsync` method.

### callApi command

```json
"commands": {
  "syncExternal": {
    "type": "callApi",
    "parameters": {
      "Url":    "https://api.example.com/sync",
      "Method": "POST"
    }
  }
}
```

## Notes

- `procedure` for a `sql` command must be specified without schema — the schema is applied from `schema` (or the root default).
- `debugOnly: true` commands are stripped in production and will return an error if called outside a debug build.
- `signal: true` requires SignalR to be configured in the application; it broadcasts a notification to connected clients after the command completes.
- For `callApi` and `sendMessage`, the `parameters` object controls the request payload and behavior; the exact keys depend on the platform version.
- `startProcess` and `resumeProcess` integrate with the A2v10 workflow engine — the process definition must exist separately.
