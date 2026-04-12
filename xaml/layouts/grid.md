# Grid

> A CSS-grid-based layout container for arranging child elements in rows and columns.

## Overview

`Grid` is the primary layout element. It inherits from `Container → UIElement → UIElementBase`. It maps directly to CSS Grid — `Columns` defines column widths, `Rows` defines row heights, and child elements are placed in cells either automatically (left-to-right, top-to-bottom) or explicitly via attached properties (`Grid.Row`, `Grid.Col`).

`Grid` is for UI layout only — use `DataGrid` or `Table` for data display.

## Syntax

```xml
<!-- Two equal columns -->
<Grid Columns="1*,1*">
  <TextBox Label="First Name" Value="{Bind Person.FirstName}" />
  <TextBox Label="Last Name" Value="{Bind Person.LastName}" />
</Grid>

<!-- Three columns: auto, flexible, fixed -->
<Grid Columns="auto,1*,200px">
  ...
</Grid>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Columns` | ColumnDefinitions | Column widths. Comma-separated `GridLength` values. |
| `Rows` | RowDefinitions | Row heights. Comma-separated `GridLength` values. |
| `Gap` | GapSize | Spacing between rows and columns. |
| `AlignItems` | AlignItem | Vertical alignment of items in cells: `Top`, `Middle`, `Bottom`, `Baseline`, `Stretch` |
| `AutoFlow` | AutoFlowMode | Auto-placement mode: `Default`, `Row`, `Column`, `RowDense`, `ColumnDense` |
| `Background` | BackgroundStyle | Grid background color |
| `DropShadow` | ShadowStyle | Shadow effect: `Shadow1`–`Shadow5` |
| `Height` | Length | Grid height |
| `Width` | Length | Grid width (overrides default 100%) |
| `MinWidth` | Length | Minimum grid width |
| `MinHeight` | Length | Minimum grid height |
| `Overflow` | Overflow | `Visible`, `Hidden`, `Auto` |

### GridLength Values

| Syntax | Meaning |
|--------|---------|
| `1*` | 1 fractional unit (proportional) |
| `2*` | 2 fractional units (double the `1*` columns) |
| `200px` | Fixed pixel width |
| `auto` | Size to content |
| `minmax(100px,1*)` | CSS minmax expression |

### Attached Properties (on child elements)

| Property | Type | Description |
|----------|------|-------------|
| `Grid.Col` | Int32 | 1-based column number |
| `Grid.Row` | Int32 | 1-based row number |
| `Grid.ColSpan` | Int32 | Number of columns to span (default: 1) |
| `Grid.RowSpan` | Int32 | Number of rows to span (default: 1) |
| `Grid.VAlign` | AlignItem | Per-cell vertical alignment override |

## Example

```xml
<!-- Standard two-column form -->
<Grid Columns="1*,1*">
  <TextBox Label="First Name" Value="{Bind Person.FirstName}" />
  <TextBox Label="Last Name" Value="{Bind Person.LastName}" />
  <DatePicker Label="Birth Date" Value="{Bind Person.BirthDate}" />
  <TextBox Label="Tax ID" Value="{Bind Person.TaxId}" />
  <TextBox Label="Notes" Value="{Bind Person.Notes}"
           Multiline="True" Rows="3"
           Grid.ColSpan="2" />
</Grid>

<!-- Three-column form with explicit positioning -->
<Grid Columns="1*,1*,1*">
  <TextBox Label="Code" Value="{Bind Doc.Code}" />
  <DatePicker Label="Date" Value="{Bind Doc.Date}" />
  <ComboBox Label="Status" Value="{Bind Doc.Status}"
            ItemsSource="{Bind Statuses}" DisplayProperty="Name" />

  <!-- Full-width row -->
  <Selector Label="Customer" Value="{Bind Doc.Customer}"
            DisplayProperty="Name" ItemsSource="{Bind Customers}"
            Grid.ColSpan="3" />
</Grid>

<!-- Grid with explicit row heights -->
<Grid Columns="1*" Rows="auto,1*,40px" Height="100%">
  <Toolbar>...</Toolbar>
  <DataGrid ItemsSource="{Bind Items}" FixedHeader="True" />
  <StackPanel Orientation="Horizontal">...</StackPanel>
</Grid>

<!-- Nested grids -->
<Grid Columns="1*,1*" Gap="8,16">
  <FieldSet Title="Main Info">
    <Grid Columns="1*">
      <TextBox Label="Name" Value="{Bind Item.Name}" />
      <TextBox Label="Code" Value="{Bind Item.Code}" />
    </Grid>
  </FieldSet>
  <FieldSet Title="Details">
    <Grid Columns="1*">
      <CheckBox Label="Active" Value="{Bind Item.IsActive}" />
      <DatePicker Label="Valid From" Value="{Bind Item.ValidFrom}" />
    </Grid>
  </FieldSet>
</Grid>
```

## Notes

- Column widths use CSS grid `fr` unit syntax — `1*` means `1fr`.
- By default, children flow left-to-right, then top-to-bottom filling one row at a time. To change, use `AutoFlow`.
- `Grid.Col` and `Grid.Row` are 1-based (not 0-based).
- `AlignItems="Top"` is useful when columns have different heights and you want fields to align to the top.
- `Gap` takes one or two values: `Gap="8"` (uniform) or `Gap="8,16"` (row-gap, column-gap).
- `Grid` is not scrollable by default. For scrollable content use `Overflow="Auto"` or wrap in `FullHeightPanel`.
