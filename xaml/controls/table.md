# Table

> A markup table — rows and cells written by hand or repeated over a collection, with merging, row marking and grid lines.

## Overview

`Table` is a table whose rows and cells are written as markup: `Table.Header`, the body rows, and `Table.Footer`, each made of `TableRow` elements holding `TableCell` elements. With `ItemsSource` the body rows repeat over a collection.

It inherits from `Control → UIElement → UIElementBase`. Content property: `Rows`.

Unlike [DataGrid](https://docs-llm.a2v10.com/xaml/controls/datagrid.md), a table is described cell by cell rather than column by column: a cell may span columns or rows, a row may be marked, and the body may mix bound rows with hand-written ones.

## Use When

- The layout is not one row per record — merged cells, several header rows, totals written by hand.
- A fixed small table of labels and values is easier to read as markup than as a grid of columns.
- Body rows come from a collection but the surrounding rows do not.

## Do Not Use When

- The data is a uniform collection shown row per record, with sorting, marking and row commands — use [DataGrid](https://docs-llm.a2v10.com/xaml/controls/datagrid.md) instead.
- The table is a report with sections, groups and Excel export — use [Sheet](https://docs-llm.a2v10.com/xaml/layouts/sheet.md) instead.
- This is a printed form — the report template has its own table element, see [Report Table](https://docs-llm.a2v10.com/report/table.md).

## Syntax

```xml
<Table ItemsSource="{Bind Document.Rows}" GridLines="Both" Compact="True">
  <Table.Columns>
    <TableColumn Fit="True" />
    <TableColumn />
    <TableColumn Width="120px" />
  </Table.Columns>
  <Table.Header>
    <TableRow>
      <TableCell Content="#" />
      <TableCell Content="Item" />
      <TableCell Content="Sum" Align="Right" />
    </TableRow>
  </Table.Header>
  <TableRow>
    <TableCell Content="{Bind RowNo}" />
    <TableCell Content="{Bind Item.Name}" />
    <TableCell Content="{Bind Sum, DataType=Currency}" Align="Right" />
  </TableRow>
</Table>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Rows` | Collection of `TableRow` | Content property. The body rows |
| `Header` | Collection of `TableRow` | The header rows |
| `Footer` | Collection of `TableRow` | The footer rows |
| `Columns` | Collection of `TableColumn` | The columns of the table |
| `ItemsSource` | Array | Binding only. Data source for the body rows |
| `Border` | Boolean | A border around the table |
| `Compact` | Boolean | Compact display |
| `Hover` | Boolean | Highlight rows under the mouse pointer |
| `Striped` | Boolean | Alternate the background of odd and even rows |
| `GridLines` | GridLinesVisibility | Grid lines: `None` (default), `Horizontal`, `Vertical`, `Both` |
| `Background` | TableBackgroundStyle | Background: `None` (default, transparent), `Paper`, `Yellow`, `Cyan`, `Rose`, `WhiteSmoke` |
| `CellSpacing` | CellSpacingMode | Space between cells; when set, every cell is rendered separately: `None` (default), `Small`, `Medium`, `Large` |

### TableRow

A row of the table. Inherits `UIElement → UIElementBase`. Content property: `Cells`.

| Property | Type | Description |
|----------|------|-------------|
| `Cells` | Collection of `TableCell` | The cells of the row |
| `Align` | TextAlign? | Horizontal alignment for every cell of the row. Can be overridden per cell |
| `VAlign` | VerticalAlign | Vertical alignment. Can be overridden per cell |
| `Mark` | Object | Binding only. Row marking, see below |

### TableCell

A cell of the table. Inherits `UIContentElement → UIElementBase`.

| Property | Type | Description |
|----------|------|-------------|
| `ItemsSource` | Array | Binding only. Data source for emitting several cells in the row |
| `ColSpan` | Int32? | Merge columns |
| `RowSpan` | Int32? | Merge rows |
| `Bold` | Boolean? | Bold font. Inherited from the parent row when not set |
| `Italic` | Boolean? | Italic font. Inherited from the parent row when not set |
| `Gray` | Boolean | Grey text. The exact colour depends on the UI theme |
| `Align` | TextAlign | Horizontal text alignment |
| `VAlign` | VerticalAlign | Vertical text alignment |

### TableColumn

One column of the table.

| Property | Type | Description |
|----------|------|-------------|
| `Width` | Length | Column width |
| `Fit` | Boolean | Size the column to its content |
| `Background` | ColumnBackgroundStyle | Column background colour |
| `If` | Boolean? | May be a binding. Whether to include the column — excluding a column excludes every cell in it |

### Row marking

The binding expression of `Mark` must return a string, which is added to the row as an extra CSS class. The default theme supports `"danger"`, `"error"`, `"red"`; `"warning"`, `"orange"`; `"success"`, `"green"`; `"info"`, `"cyan"`.

Marking only adds a CSS class, so the supported values are not a closed list: an expression returning `"danger bold"` marks the row red and renders it bold.

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties.

## Example

A table of document rows with a hand-written total row, a merged cell and marking by row state:

```xml
<Table ItemsSource="{Bind Document.Rows}" GridLines="Horizontal" Hover="True" Border="True">
  <Table.Columns>
    <TableColumn Fit="True" />
    <TableColumn />
    <TableColumn Width="100px" />
    <TableColumn Width="120px" />
  </Table.Columns>

  <Table.Header>
    <TableRow>
      <TableCell Content="#" />
      <TableCell Content="Item" />
      <TableCell Content="Qty" Align="Right" />
      <TableCell Content="Sum" Align="Right" />
    </TableRow>
  </Table.Header>

  <TableRow Mark="{Bind RowColor}">
    <TableCell Content="{Bind RowNo}" />
    <TableCell Content="{Bind Item.Name}" />
    <TableCell Content="{Bind Qty, DataType=Number}" Align="Right" />
    <TableCell Content="{Bind Sum, DataType=Currency}" Align="Right" />
  </TableRow>

  <Table.Footer>
    <TableRow>
      <TableCell ColSpan="3" Content="Total" Align="Right" Bold="True" />
      <TableCell Content="{Bind Document.Sum, DataType=Currency}" Align="Right" Bold="True" />
    </TableRow>
  </Table.Footer>
</Table>
```

## Notes

- `Rows` is the content property, so body rows are written directly inside `Table` while the header and footer need the explicit `Table.Header` / `Table.Footer` syntax.
- `ItemsSource` on a cell is the horizontal counterpart of `ItemsSource` on the table: one cell definition produces as many cells in the row as the collection has items.
- `If` on a `TableColumn` removes the whole column, cells included; there is no need to repeat the condition on each cell.
- `Bold` and `Italic` on a cell are nullable on purpose: left unset they follow the row, which is what makes a bold footer row work without repeating it per cell.
- `CellSpacing` changes the visual model from a grid to separate cells, so it and `GridLines` are alternatives rather than a combination.
