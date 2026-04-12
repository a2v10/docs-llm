# Bind & BindCmd

> Markup extensions for connecting XAML properties to model data and commands.

## Overview

`Bind` and `BindCmd` are the two primary markup extensions in A2v10 XAML. They are written inline as attribute values using curly-brace syntax. Both are resolved server-side and emit Vue.js reactive bindings into the rendered HTML.

**Bind** creates a reactive, two-way connection between a control property and a model data path. When the model updates, the UI updates; when the user edits a field, the model updates.

**BindCmd** attaches a command to a control's action (typically a button click). Commands trigger server-side or client-side operations: saving, opening dialogs, navigating, executing stored procedures, and more.

## Bind

```xml
Value="{Bind Path}"
Value="{Bind Path, DataType=Currency}"
Value="{Bind Path, Format='#,##0.00', HideZeros=True}"
```

`Path` is the default property — it can be written without `Path=`.

### Bind Properties

| Property | Type | Description |
|----------|------|-------------|
| `Path` | String | **Default.** Dot-notation path to the model property (e.g., `Document.Name`, `Row.Amount`) |
| `DataType` | DataType | How the value is formatted for display |
| `Format` | String | Custom format string (overrides DataType formatting) |
| `Mask` | String | Input mask pattern |
| `HideZeros` | Boolean | Suppress display of zero numeric values |
| `NegativeRed` | Boolean | Render negative numbers in red |
| `Filters` | FilterCollection | Transform filters: `Trim`, `Upper`, `Lower`, `Barcode` |

### DataType Values

| Value | Description |
|-------|-------------|
| `String` | Plain text (default) |
| `Number` | Integer number |
| `Currency` | Currency with thousands separator |
| `Date` | Date only (no time) |
| `DateTime` | Date and time |
| `Time` | Time only |
| `Period` | Period object (date range) |
| `Boolean` | True/false |
| `Percent` | Percentage value |
| `PhoneNumber` | Phone number formatting |

## BindCmd

```xml
Command="{BindCmd CommandName}"
Command="{BindCmd CommandName, Argument={Bind Path}}"
Command="{BindCmd Dialog, Action=Edit, Argument={Bind Row}}"
```

`Command` is the default property.

### BindCmd Properties

| Property | Type | Description |
|----------|------|-------------|
| `Command` | CommandType | **Default.** The command to execute |
| `Argument` | Object | Command argument, typically a `Bind` |
| `Data` | Object | Additional data passed to the command |
| `Url` | String | URL for navigation commands (must start with `/`) |
| `CommandName` | String | Template command name for `Execute`/`ExecuteSelected` |
| `Action` | DialogAction | Dialog mode: `Edit`, `Show`, `Browse`, `Append`, `New`, `Copy`, `EditSelected`, `ShowSelected` |
| `SaveRequired` | Boolean | Save model before executing |
| `ValidRequired` | Boolean | Require valid model before executing |
| `CheckReadOnly` | Boolean | Respect model's ReadOnly flag |
| `Confirm` | Confirm | Show confirmation dialog before executing |
| `NewWindow` | Boolean | Open in new browser tab |
| `ReloadAfter` | Boolean | Reload model after command completes |
| `Export` | Boolean | Export instead of display (for Report command) |
| `Print` | Boolean | Print instead of display |
| `Permission` | Permission | Required permission: `CanView`, `CanEdit`, `CanDelete`, `CanApply`, `CanCreate`, etc. |

### CommandType Values

| Command | Description |
|---------|-------------|
| `Open` | Navigate to a form with the given argument as ID |
| `OpenSelected` | Navigate to a form for the selected row |
| `Close` | Close current form or dialog |
| `CloseOk` | Close dialog returning `true` |
| `SaveAndClose` | Save the model, then close |
| `Save` | Save the current model |
| `Reload` / `Refresh` | Reload the model or a portion of it |
| `Requery` | Fully reload the form from server |
| `Execute` | Execute a named template command (stored procedure `Execute` action) |
| `ExecuteSelected` | Execute command for the selected row |
| `Dialog` | Open a dialog with specified Action |
| `Clear` | Clear/reset the argument object |
| `Append` | Append new row to an array |
| `Remove` / `RemoveSelected` | Remove item from array |
| `DbRemove` / `DbRemoveSelected` | Delete from DB via server call |
| `Report` | Execute a report (with `Export`/`Print` options) |
| `Download` | Download a static file |
| `File` | Work with binary objects (Show, Download, Print) |
| `Navigate` | Client-side navigation |
| `Print` | Print the current page |
| `Browse` | Open browse dialog |

## Example

```xml
<!-- Simple data binding -->
<TextBox Label="Name" Value="{Bind Document.Name}" />
<TextBox Label="Amount" Value="{Bind Document.Sum, DataType=Currency}" />

<!-- Boolean -->
<CheckBox Label="Active" Value="{Bind Document.IsActive}" />

<!-- Conditional visibility -->
<Button Content="Delete"
        Command="{BindCmd DbRemoveSelected, Argument={Bind Documents.Selected}}"
        If="{Bind Documents.HasSelected}" />

<!-- Open dialog -->
<Button Content="Edit"
        Command="{BindCmd Dialog, Action=Edit, Argument={Bind Row}}" />

<!-- Execute stored procedure command -->
<Button Content="Apply"
        Command="{BindCmd Execute, CommandName=Apply, Argument={Bind Document},
                           SaveRequired=True, ValidRequired=True}"
        Style="Primary" />

<!-- Confirm before delete -->
<Button Content="Delete"
        Command="{BindCmd DbRemove, Argument={Bind Document},
                           Confirm={Confirm Message='Delete this record?'}}"
        Style="Danger" />
```

## Notes

- `Bind` paths use dot notation; array elements inside `DataGrid`/`Repeater` are automatically scoped to the current row.
- `BindCmd` `Argument` is almost always a `{Bind ...}` — it passes the model object to the server action.
- `SaveRequired=True` triggers a save before the command runs; useful for Execute commands that need fresh data.
- `ValidRequired=True` prevents execution if any validators on the page are failing.
- `Permission` gates the command — the button is disabled (not hidden) if the user lacks the permission.
- `ReloadAfter=True` reloads the model after the command, so the UI reflects server-side changes.
