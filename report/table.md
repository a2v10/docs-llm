# Report Table

> The Table element of a report template — header, body and footer, repeating rows over a collection, column widths, cell merging and the ready-made table styles.

## Overview

A table has a header, a body and a footer. When `ItemsSource` is given, the body rows repeat for every element of the collection, and the header repeats on every page the table continues onto.

A table is made of four elements: `Table` itself, [TableColumn](#tablecolumn) for widths, [TableRow](#tablerow) and [TableCell](#tablecell). Styling is set on cells; a row does not paint anything.

| Property | Type | Description |
|----------|------|-------------|
| `ItemsSource` | Array | Binding only. The collection the body rows repeat over. Inside such a row expressions are read from its element |
| `Columns` | TableColumnCollection | The columns of the table — see [TableColumn](#tablecolumn). The shortest form is a comma-separated list of widths: `Columns="30pt,1fr,80pt"` |
| `Header` | TableRowCollection | Header rows. Repeated on every page |
| `Body` | TableRowCollection | Content property. Body rows |
| `Footer` | TableRowCollection | Footer rows. Printed once, after the body, and read from the same scope as the table itself — not from an element of the collection |
| `Style` | TableStyle | Ready-made styling, see below |

Content property: `Body`. Plus the properties of [the base element](https://docs-llm.a2v10.com/report/elements.md#base-element).

## Syntax

The `Style` property switches on a ready-made set of styling, so that borders and padding need not be described on every cell:

| Style | What it gives |
|-------|---------------|
| `Default` | No styling at all: no borders, no padding |
| `Simple` | Thin borders and padding in header and body cells, grey header background |
| `Details` | The same, plus a border around the whole table, space above and below, and a bold footer |

Any styling property can be overridden on a cell — a cell has the full set of base element properties. Styling is not set on a row: a row does not paint it.

### TableColumn

A column of a table. It sets the width only; everything else is described on the cells.

| Property | Type | Description |
|----------|------|-------------|
| `Width` | [Length](https://docs-llm.a2v10.com/report/elements.md#length) | Column width. Fixed (`80pt`, `30mm`) or a fraction of the free space (`1fr`). When no columns are declared at all, the table gets one column of the full width |

Declaring every column as a separate element is usually unnecessary: a comma-separated list of widths becomes the columns automatically.

```xml
<!-- short -->
<Table Columns="30pt,1fr,80pt">

<!-- the same in full -->
<Table>
    <Table.Columns>
        <TableColumn Width="30pt"/>
        <TableColumn Width="1fr"/>
        <TableColumn Width="80pt"/>
    </Table.Columns>
</Table>
```

### TableRow

A row of a table, used inside `Table.Header`, `Table.Body` and `Table.Footer`. Content property: `Cells` (a `TableCellCollection`).

Of the base element properties only `If` has any effect on a row. Styling — `Bold`, `Align`, `Border`, `Background` — is accepted without an error but never reaches the paper.

### TableCell

A cell of a table. Its content may be a string, a binding or another element — a [Text](https://docs-llm.a2v10.com/report/elements.md#text) or a nested `Table`, for instance. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Object | Content property: text, a binding or a nested element |
| `ColSpan` | UInt32 | How many columns the cell occupies |
| `RowSpan` | UInt32 | How many rows the cell occupies |

```xml
<!-- text -->
<TableCell Content="Total"/>

<!-- a value from the model, with a format -->
<TableCell Content="{Bind Sum, DataType=Currency}" Align="Right"/>

<!-- merged cells -->
<TableCell ColSpan="3" Content="Subtotal" Align="Right"/>

<!-- a nested element -->
<TableCell>
    <Text>
        <Span Content="{Bind Item.Name}" Bold="True"/>
        <Break/>
        <Span Content="{Bind Item.Article}"/>
    </Text>
</TableCell>
```

## Example

A document table with a header, rows from a collection and a total:

```xml
<Table Style="Details" Columns="30pt,1fr,60pt,80pt"
       ItemsSource="{Bind Document.Rows}">
    <Table.Header>
        <TableRow>
            <TableCell Content="#" Align="Center"/>
            <TableCell Content="Item" Align="Center"/>
            <TableCell Content="Qty" Align="Center"/>
            <TableCell Content="Sum" Align="Center"/>
        </TableRow>
    </Table.Header>
    <Table.Body>
        <TableRow If="{Bind !Void}">
            <TableCell Content="{Bind RowNo}"/>
            <TableCell Content="{Bind Item.Name}"/>
            <TableCell Content="{Bind Qty, DataType=Number}" Align="Right"/>
            <TableCell Content="{Bind Sum, DataType=Currency}" Align="Right"/>
        </TableRow>
    </Table.Body>
    <Table.Footer>
        <TableRow>
            <TableCell ColSpan="3" Content="Total" Align="Right"/>
            <TableCell Content="{Bind Document.Sum, DataType=Currency}" Align="Right"/>
        </TableRow>
    </Table.Footer>
</Table>
```

## Notes

- `If` on a body row is evaluated for every element of the collection separately, so rows can be hidden selectively: `<TableRow If="{Bind !Void}">`.
- Footer rows are read from the scope of the table, not of a row — that is why a footer says `{Bind Document.Sum}` where the body says `{Bind Sum}`.
- The content of a cell may be another table. Its `ItemsSource` is evaluated from the row it stands in, so nesting works to any depth with nothing extra to declare.
- Fractions (`fr`) share what is left after the fixed columns. When the fixed widths add up to more than the page width, the document is not built — the engine reports contradictory sizes.
- A border on a `TableRow` has no effect. To underline a whole row, set `Border` on each of its cells.
- A table does not number its rows. `RowNo` in the examples is an ordinary field of the model, like any other — the collection behind a table is usually an [Array](https://docs-llm.a2v10.com/sql/array.md) dataset.
