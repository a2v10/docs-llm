# TextBox

> A text input field for entering and editing string or numeric data.

## Overview

`TextBox` is the primary text entry control. It renders as an HTML `<input>` or `<textarea>` depending on the `Multiline` flag. It inherits from `ValuedControl → Control → UIElement → UIElementBase` and is bound to a model property via the `Value` property.

By default, `TextBox` is single-line. Setting `Multiline="True"` switches it to a multi-line textarea. The `Password` flag masks input; `Number` restricts input to digits and minus sign.

## Syntax

```xml
<TextBox Label="Name" Value="{Bind Document.Name}" />
<TextBox Label="Notes" Value="{Bind Document.Notes}" Multiline="True" Rows="4" />
<TextBox Label="Amount" Value="{Bind Document.Sum, DataType=Currency}" Align="Right" />
```

## Properties

### TextBox-Specific

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `Multiline` | Boolean | False | Multi-line textarea mode |
| `Rows` | Int32? | — | Row count for multi-line mode |
| `AutoSize` | Boolean | False | Auto-adjust height to content. Multi-line only. |
| `MaxHeight` | Length | — | Maximum height for auto-sized multi-line fields |
| `Password` | Boolean | False | Mask input; disable copy; show placeholder characters |
| `Number` | Boolean | False | Restrict input to digits and minus sign |
| `Align` | TextAlign | Left | Text alignment: `Left`, `Right`, `Center` |
| `Placeholder` | String | — | Hint text shown in empty field; disappears on focus |
| `SpellCheck` | Boolean? | — | Override browser spell-check (default: browser setting) |
| `UpdateTrigger` | UpdateTrigger | Default | When to sync value to model: `Default`, `Blur`, `Input` |
| `EnterCommand` | BindCmd | — | Command fired on Enter key (Ctrl+Enter for multi-line) |
| `ShowClear` | Boolean | False | Show × button to clear the field (hidden when empty) |
| `ShowFilter` | Boolean | False | Show filter icon for `CollectionView` integration |
| `Highlight` | Boolean | False | Visual emphasis when field has a non-empty value |
| `Accel` | Accel | — | Keyboard shortcut that focuses this field |
| `Size` | ControlSize | Default | `Default`/`Normal` or `Large` |

### From ValuedControl

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | **Binding only.** The field value bound to a model property. |
| `ValidateValue` | Object | **Binding only.** Alternative binding used for validation. |

See [Base Classes](../base-classes.md) for `Label`, `Description`, `Required`, `Disabled`, `Width`, and other inherited properties.

### UpdateTrigger Values

| Value | Description |
|-------|-------------|
| `Default` | Sync on blur (when field loses focus) |
| `Blur` | Sync on blur |
| `Input` | Sync on every keystroke (immediate) |

## Example

```xml
<!-- Standard form fields -->
<Grid Columns="2*,1*,1*">
  <TextBox Label="Name" Value="{Bind Document.Name}" Required="True" />
  <TextBox Label="Code" Value="{Bind Document.Code}" />
  <TextBox Label="Amount" Value="{Bind Document.Amount, DataType=Currency}"
           Align="Right" />
</Grid>

<!-- Multi-line notes field -->
<TextBox Label="Description"
         Value="{Bind Document.Description}"
         Multiline="True"
         Rows="5"
         Placeholder="Enter description..." />

<!-- Auto-growing textarea -->
<TextBox Value="{Bind Item.Comment}"
         Multiline="True"
         AutoSize="True"
         MaxHeight="200px" />

<!-- Password field -->
<TextBox Label="Password" Value="{Bind User.Password}" Password="True" />

<!-- Search field that triggers command on Enter -->
<TextBox Placeholder="Search..."
         Value="{Bind Filter.Text}"
         ShowClear="True"
         EnterCommand="{BindCmd Reload}"
         UpdateTrigger="Blur" />

<!-- Inline editable cell in DataGrid -->
<DataGrid ItemsSource="{Bind Items}">
  <DataGridColumn Header="Name" Content="{Bind Name}" Editable="True" />
</DataGrid>
```

## Notes

- `Value` must always be a `Bind` — literal string values are not supported.
- `DataType` is specified on the `Bind` extension, not on `TextBox` directly: `Value="{Bind Field, DataType=Currency}"`.
- `UpdateTrigger="Input"` updates the model on every keystroke — use sparingly as it triggers watchers frequently.
- `EnterCommand` on a multi-line field requires Ctrl+Enter, not plain Enter (which inserts a newline).
- `ShowClear` renders a clear button only when the field has content; it disappears when the field is empty.
- `Number` mode allows only digits and `-`; for formatted currency input, use `DataType=Currency` on the Bind instead.
