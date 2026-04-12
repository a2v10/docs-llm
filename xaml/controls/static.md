# Static

> A read-only display field that looks like a disabled text input.

## Overview

`Static` renders a value for display only — it looks like a disabled `TextBox` (label + grayed-out text), but it is not an input control. It inherits from `ValuedControl → Control → UIElement → UIElementBase`.

Use `Static` when you want to show a model value in the standard form layout (with a label and consistent styling) but the field must not be editable.

## Syntax

```xml
<Static Label="Status" Value="{Bind Document.StatusText}" />
<Static Label="Total" Value="{Bind Document.Total, DataType=Currency}" Align="Right" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Align` | TextAlign | Text alignment: `Left`, `Right`, `Center` |
| `Size` | ControlSize | `Default`/`Normal` or `Large` |
| `MaxChars` | Int32 | Max characters before truncation with ellipsis. Full text shown as tooltip. Cannot be combined with `Tip`. |

### From ValuedControl

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | **Binding only.** The displayed value. |

See [Base Classes](../base-classes.md) for `Label`, `Width`, and other inherited properties.

## Example

```xml
<!-- Read-only field in a form -->
<Grid Columns="1*,1*">
  <TextBox Label="Name" Value="{Bind Document.Name}" />
  <Static Label="Created By" Value="{Bind Document.CreatedBy}" />
  <DatePicker Label="Date" Value="{Bind Document.Date}" />
  <Static Label="Created At" Value="{Bind Document.CreatedAt, DataType=DateTime}" />
</Grid>

<!-- Numeric value right-aligned -->
<Static Label="Balance"
        Value="{Bind Account.Balance, DataType=Currency}"
        Align="Right" />

<!-- Long text truncated -->
<Static Label="Note" Value="{Bind Document.Note}" MaxChars="60" />
```

## Notes

- `Static` is purely for display — unlike `TextBox` with `Disabled="True"`, there is no input mechanism at all.
- The visual appearance is identical to a disabled `TextBox`, making it blend naturally in mixed read/write forms.
- `DataType` formatting is specified on the `Bind`, not on `Static` itself: `Value="{Bind Field, DataType=Currency}"`.
- For displaying calculated or formatted text without the input-field visual style, use `Text` or `Span` elements instead.
