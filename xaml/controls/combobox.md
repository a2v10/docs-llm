# ComboBox

> A dropdown selection control for choosing from a static or data-bound list.

## Overview

`ComboBox` renders a dropdown list. It inherits from `ValuedControl → Control → UIElement → UIElementBase`. The selected value is bound to the model via `Value`.

The list can be populated three ways:
1. **Static items** — hardcoded `ComboBoxItem` children
2. **Bound object list** — `ItemsSource` bound to a model array; the selected item (whole object) becomes the value
3. **Bound value list** — `ItemsSource` bound to a model array with nested `ComboBoxItem` to map `Content` and `Value` from array properties

## Syntax

```xml
<!-- Static list -->
<ComboBox Label="Status" Value="{Bind Document.Status}">
  <ComboBoxItem Value="1" Content="Active" />
  <ComboBoxItem Value="2" Content="Inactive" />
  <ComboBoxItem Value="3" Content="Archived" />
</ComboBox>

<!-- Bound list -->
<ComboBox Label="Type" Value="{Bind Document.DocType}"
          ItemsSource="{Bind DocTypes}" DisplayProperty="Name" />
```

## Properties

### ComboBox-Specific

| Property | Type | Description |
|----------|------|-------------|
| `Children` | List\<ComboBoxItem\> | **Content property.** Static list of `ComboBoxItem` elements. |
| `ItemsSource` | Array | **Bind only.** Data source for dynamic list. |
| `DisplayProperty` | String | Property name to display when binding objects. |
| `ShowValue` | Boolean | Display the value field instead of content in the list. |
| `Align` | TextAlign | Text alignment: `Left`, `Right`, `Center` |
| `Style` | ComboBoxStyle | `Default` (standard dropdown) or `Hyperlink` (link-styled) |
| `Size` | ControlSize | `Default`/`Normal` or `Large` |
| `GroupBy` | String | Property name to group list items by |
| `Highlight` | Boolean | Visual emphasis when a value is selected (useful in filters) |

### From ValuedControl

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | **Binding only.** The selected value. |

See [Base Classes](../base-classes.md) for `Label`, `Required`, `Disabled`, and other inherited properties.

## ComboBoxItem Properties

| Property | Type | Description |
|----------|------|-------------|
| `Content` | String | Display text for the item |
| `Value` | Object | The value stored in the model when this item is selected |
| `Bold` | Boolean | Bold display text |
| `CssClass` | Object | CSS class for the item |

## Example

```xml
<!-- Static enum-style list -->
<ComboBox Label="Document Type" Value="{Bind Document.DocTypeId}">
  <ComboBoxItem Value="1" Content="Invoice" />
  <ComboBoxItem Value="2" Content="Credit Note" />
  <ComboBoxItem Value="3" Content="Debit Note" />
</ComboBox>

<!-- Bound to model array — selected object becomes the value -->
<ComboBox Label="Warehouse"
          Value="{Bind Document.Warehouse}"
          ItemsSource="{Bind Warehouses}"
          DisplayProperty="Name" />

<!-- Dynamic list with explicit value mapping -->
<ComboBox Label="Currency"
          Value="{Bind Document.CurrencyId}"
          ItemsSource="{Bind Currencies}">
  <ComboBoxItem Content="{Bind Name}" Value="{Bind Id}" />
</ComboBox>

<!-- Filter combobox with highlight -->
<ComboBox Value="{Bind Filter.Status}"
          ItemsSource="{Bind Statuses}"
          DisplayProperty="Name"
          Highlight="True"
          Placeholder="All statuses" />
```

## Notes

- When `ItemsSource` is used with `DisplayProperty`, the entire selected object is stored in `Value` — not just the display string.
- When you need only the `Id` to be stored, use nested `ComboBoxItem` elements with explicit `Value="{Bind Id}"` and `Content="{Bind Name}"`.
- `Style="Hyperlink"` renders the combobox as a hyperlink-style element — suitable for in-line or compact UI areas.
- `Highlight="True"` is commonly used in filter rows to make it obvious when a filter is active.
- Unlike `Selector`, `ComboBox` does not support server-side search; the entire list is loaded at once.
