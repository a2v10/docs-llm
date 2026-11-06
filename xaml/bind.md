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
| `Confirm` | Confirm | Confirmation before executing — a string or a `Confirm` element, see [Confirm](#confirm) |
| `NewWindow` | Boolean | Open in new browser tab |
| `ReloadAfter` | Boolean | Reload model after command completes |
| `Report` | String | Report name — the key in the `reports` section (for the `Report` command) |
| `Export` | Boolean | Export instead of display (for Report command) |
| `Print` | Boolean | Print instead of display |
| `Format` | String | Download format for the `Report` command; the available values depend on the report type |
| `Permission` | Permission | Required permission: `CanView`, `CanEdit`, `CanDelete`, `CanApply`, `CanCreate`, etc. |

### Confirm

A confirmation the user must accept before the command runs; if the user declines, the command does not run. A string is enough in most cases — it becomes `Message`. The element form is needed only to change the title or the button captions.

| Property | Type | Description |
|----------|------|-------------|
| `Message` | String | **Default.** The message text |
| `Title` | String | Window title |
| `OkText` | String | Caption of the confirm button |
| `CancelText` | String | Caption of the cancel button |

Any property can be a `Bind`. Unset values come from localization — the same defaults as the [confirm object of template commands](https://docs-llm.a2v10.com/template/commands.md#confirm-object-properties).

```xml
<!-- message only -->
<Button Content="Delete" Command="{BindCmd DbRemoveSelected, Argument={Bind Documents},
        Confirm='@[Confirm.Delete.Element]'}" />

<!-- full form -->
<Button Content="Apply">
  <Button.Command>
    <BindCmd Command="Execute" CommandName="apply">
      <BindCmd.Confirm>
        <Confirm Message="Apply the document?" Title="Apply" OkText="Apply" />
      </BindCmd.Confirm>
    </BindCmd>
  </Button.Command>
</Button>
```

### CommandType Values

| Command | Description |
|---------|-------------|
| `Open` | Navigate to a form with the given argument as ID |
| `OpenSelected` | Navigate to a form for the selected row |
| `Close` | Close current form or dialog |
| `CloseOk` | Close dialog returning `true` |
| `Select` / `SelectChecked` | Browse dialogs only. Close the dialog returning the selected element of the array in `Argument` (required) / the array of its checked elements ([`$checked`](https://docs-llm.a2v10.com/client/array.md)). Disabled while nothing is selected / checked. `ValidRequired` applies to `Select` only |
| `SaveAndClose` | Save the model, then close |
| `Save` | Save the current model |
| `Reload` / `Refresh` | Reload the model or a portion of it |
| `Requery` | Fully reload the form from server |
| `Execute` | Execute a named template command (stored procedure `Execute` action) |
| `ExecuteSelected` | Execute command for the selected row |
| `Dialog` | Open a dialog with specified Action |
| `Clear` | Clear/reset the argument object |
| `Append` / `Prepend` | Add a new empty element at the end / the start of the array in `Argument` (required) and select it. [`{ArrayName}.adding`](https://docs-llm.a2v10.com/template/events.md) fires first — a handler returning `false` cancels the add. Disabled on a read-only model |
| `Remove` / `RemoveSelected` | Remove an element on the client only: the model becomes dirty, the database row goes on save; `{ArrayName}.remove` fires. `Remove` takes the element — inside a row (a `DataGrid` column, for example) it may be omitted and the current row is removed. `RemoveSelected` takes the array and removes its selected element; disabled while nothing is selected. Disabled on a read-only model |
| `DbRemove` / `DbRemoveSelected` | Delete the element in the database at once, without saving the model, then remove it from the array. Calls `[schema].[ModelName.Delete]` with `@Id` — see [Stored Procedures](https://docs-llm.a2v10.com/sql/procedures.md); for a [lazy array](https://docs-llm.a2v10.com/sql/array.md), `[schema].[ModelName.Property.Delete]`, where `Property` is the array name. `DbRemove` takes the element (required); `DbRemoveSelected` takes the array and deletes its selected element, disabled while nothing is selected |
| `Report` | Run a report declared in the [reports](https://docs-llm.a2v10.com/model/reports.md) section. `Report` names it, `Url` is the endpoint folder rather than an action, `Argument` supplies the identifier, `Data` becomes parameters of the report procedure, `Export`/`Print` download or print it. See [Viewing and Printing Reports](https://docs-llm.a2v10.com/report/view.md) |
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

<!-- Delete the selected row; disabled while nothing is selected -->
<Button Content="Delete"
        Command="{BindCmd DbRemoveSelected, Argument={Bind Documents}}" />

<!-- Remove the current row on the client -->
<DataGridColumn>
  <Hyperlink Icon="Delete" Command="{BindCmd Remove, Confirm='Delete the row?'}" />
</DataGridColumn>

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
                           Confirm='Delete this record?'}"
        Style="Danger" />
```

## Notes

- `Bind` paths use dot notation; array elements inside `DataGrid`/`Repeater` are automatically scoped to the current row.
- A parameter value with a space or a comma goes in single quotes, which are removed: `Format='#,##0.00'`, `Confirm='Delete this record?'`. Without them, parsing stops with the error `` Quote a value with spaces: 'text' or `template` ``. `Path` is a path, not an expression: views have no expression engine, so backtick templates and calculations (`` `…${…}` ``, `'Price * Qty'`) work only in [print forms](https://docs-llm.a2v10.com/report/bind.md#quoting-in-markup).
- `BindCmd` `Argument` is almost always a `{Bind ...}` — it passes the model object to the server action.
- `SaveRequired=True` triggers a save before the command runs; useful for Execute commands that need fresh data.
- `ValidRequired=True` prevents execution if any validators on the page are failing.
- `Permission` gates the command — the button is disabled (not hidden) if the user lacks the permission.
- `ReloadAfter=True` reloads the model after the command, so the UI reflects server-side changes.
- The `Dialog` command opens a modal configured in the [model.json `dialogs` section](https://docs-llm.a2v10.com/model/dialogs.md); lighter lookups use [`popups`](https://docs-llm.a2v10.com/model/popups.md).
- `Save` and `SaveAndClose` persist the model through the [TVP + MERGE save pattern](https://docs-llm.a2v10.com/sql/update-model.md) (`.Metadata` → `.Update` → `.Load`).
- Server-side operations invoked from the UI (`Execute` and similar) are declared as [model.json commands](https://docs-llm.a2v10.com/model/commands.md). `DbRemove` / `DbRemoveSelected` need none: they call the `.Delete` procedure by convention.
