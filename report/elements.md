# Report Elements

> Flow elements of a hand-written report template — the base element, Length and Thickness, Page, Column, Inlined, Text, List, Line, Checkbox.

## Overview

The content of a page is a sequence of elements that print one after another and move to the next page by themselves. Sizes are computed by the engine; nothing is positioned by coordinates.

Layout containers:

- [Page](#page) — the page, and the root of the template
- [Column](#column) — a vertical sequence of elements
- [Inlined](#inlined) — elements side by side, in a row

Content:

- [Text](#text) — a text paragraph and its pieces: `Span`, `Space`, `Break`
- [List](#list) — a list with bullets
- [Line](#line) — a horizontal rule
- [Checkbox](#checkbox) — a check box

Described separately: [Table and its parts](https://docs-llm.a2v10.com/report/table.md), and [Image, QrCode, Barcode](https://docs-llm.a2v10.com/report/images.md).

Every element shares one set of styling properties, described below as [the base element](#base-element), and every element may carry an `If` condition — see [Expressions and Bindings](https://docs-llm.a2v10.com/report/bind.md).

## Base Element

These properties exist on every element of a report template, from the page down to a table cell.

| Property | Type | Description |
|----------|------|-------------|
| `Align` | TextAlign | Horizontal alignment: `Left`, `Center`, `Right`, `Justify`. `Justify` works on a text paragraph only |
| `VAlign` | VertAlign | Vertical alignment: `Top`, `Middle`, `Bottom` |
| `Bold` | Boolean? | Bold text |
| `Italic` | Boolean? | Italic text |
| `Underline` | Boolean? | Underlined text |
| `FontSize` | Single? | Font size in points |
| `Color` | String | Text colour, for example `"#404040"` |
| `Background` | String | Background colour. The fill is solid and opaque |
| `Margin` | [Thickness](#thickness) | Space outside the element |
| `Padding` | [Thickness](#thickness) | Space inside the element, between the border and the content |
| `Border` | [Thickness](#thickness) | Border width on each side. Zero means no border on that side |
| `If` | Boolean? | Display condition. Usually a binding; an exclamation mark in front of the expression inverts it |
| `ShowEntire` | Boolean | Do not split the element across pages: if it does not fit in what is left of the current page, it is printed whole on the next one |

Styling is applied in exactly this order: `Margin` — `Background` — `Border` — `Padding` — alignment. The background is therefore painted inside the outer margin and under the border, and `Padding` pushes the content away from the border, not the background.

`ShowEntire` affects [Column](#column), [Table](https://docs-llm.a2v10.com/report/table.md), [Text](#text), [List](#list) and a [workbook](https://docs-llm.a2v10.com/report/workbook.md); on the remaining elements it does nothing.

## Length

A length is a number and a unit, written as a string.

| Unit | Description |
|------|-------------|
| `pt` | Point, 1/72 inch. The default unit: `"12"` means `"12pt"` |
| `mm` | Millimetres |
| `cm` | Centimetres |
| `in` | Inches |
| `fr` | A fraction of the free space — table column widths only. A star may be written instead: `"*"` means `"1fr"` |

```xml
<TableColumn Width="80pt"/>
<TableColumn Width="30mm"/>
<TableColumn Width="2fr"/>
<Space Width="6"/>   <!-- 6 points -->
```

An unknown unit is a load error, not a silent zero width. The decimal separator is a dot: `"0.5cm"`.

## Thickness

A width on four sides — for `Margin`, `Padding` and `Border`. Written as a string: one, two or four values separated by commas, each of them a [Length](#length).

| Form | Meaning |
|------|---------|
| `"4pt"` | The same on all sides |
| `"10mm,5mm"` | First top and bottom, second left and right |
| `"1pt,4pt,1pt,4pt"` | Top, right, bottom, left — clockwise, as in CSS |

For `Border`, zero means there is no border on that side. That is how partial borders are drawn — a rule under a cell, for instance:

```xml
<TableCell Border="0,0,.2pt,0" Content="Total"/>
<Column Margin="0,0,10pt,0" Padding="4pt" Border=".2pt"/>
```

Three values is a load error: one, two or four are allowed.

## Page

The root of a report template. Sets the page size and orientation, the default font, the running headers and the watermark. Content property: `Columns`.

| Property | Type | Description |
|----------|------|-------------|
| `Columns` | ColumnCollection | Content property. The columns of the page — see [Column](#column) |
| `Header` | [Column](#column) | Running header: printed on every page |
| `Footer` | [Column](#column) | Running footer: printed on every page |
| `Orientation` | PageOrientation | `Portrait` (default) or `Landscape`. The page size is A4 |
| `FontFamily` | String | Default font for the whole document. `Verdana` when not specified |
| `Title` | String | Document title, which ends up in the PDF file properties. Supports substitution of model values: `Title="Invoice {Document.No}"` |
| `Code` | String | Javascript that runs once before the document is built. Functions declared here are available in expressions — see [Expressions and Bindings](https://docs-llm.a2v10.com/report/bind.md) |
| `Watermark` | String | Watermark: a file name, or bytes from the model through a binding. Drawn under the content of every page — see [Images and Watermark](https://docs-llm.a2v10.com/report/images.md) |

Page margins are given by the `Margin` property of the base element and the font size by `FontSize`. When they are not specified, the defaults are `20mm` top and bottom, `10mm` left and right, and a 9 point font.

```xml
<Page xmlns="clr-namespace:A2v10.Xaml.Report;assembly=A2v10.Xaml.Report"
      Orientation="Landscape" Margin="15mm,10mm" FontSize="10">
    <Page.Header>
        <Column>
            <Text Align="Right"><Span Content="{Bind Company.Name}"/></Text>
        </Column>
    </Page.Header>

    <Column>
        <!-- document content -->
    </Column>
</Page>
```

## Column

A vertical sequence of elements: the content prints top to bottom and moves to the next page when it does not fit. This is the main way to compose a page — the content of `Page` is made of columns. Content property: `Children`.

| Property | Type | Description |
|----------|------|-------------|
| `Children` | FlowElementCollection | Content property. The elements, printed one after another |

```xml
<Column Margin="0,0,10pt,0">
    <Text Style="Title">Certificate of work done</Text>
    <Text><Span Content="{Bind Document.Agent.Name}"/></Text>
    <Table Style="Details" Columns="1fr,80pt">
        <!-- ... -->
    </Table>
</Column>
```

## Inlined

Places its children side by side, in a row, and wraps them to the next line when they do not fit the width. The opposite of [Column](#column), which stacks them. Content property: `Children`.

| Property | Type | Description |
|----------|------|-------------|
| `Children` | FlowElementCollection | Content property. The elements, printed side by side |

```xml
<Inlined>
    <Image Source="{Bind Company.Logo}" Height="15mm"/>
    <Text><Span Content="{Bind Company.Name}" Bold="True"/></Text>
</Inlined>
```

## Text

A text paragraph. Made of pieces — `Span`, `Space`, `Break` — that print in sequence and wrap by words. Content property: `Inlines`.

| Property | Type | Description |
|----------|------|-------------|
| `Inlines` | InlineCollection | Content property. The pieces of text. A plain string becomes a single `Span` |
| `Style` | TextStyle | `Default` or `Title` — a heading, 12 points |

```xml
<!-- constant text -->
<Text>Delivery note</Text>

<!-- a value from the model -->
<Text><Span Content="{Bind Document.Name}"/></Text>

<!-- several pieces with different styling -->
<Text>
    <Span Content="Supplier:"/>
    <Space Width="6pt"/>
    <Span Content="{Bind Document.Company.Name}" Bold="True"/>
</Text>
```

`Text` has no `Content` property: writing `<Text Content="..."/>` stops loading with the error *"Property Content not found"*. The text is given either as a string inside the element or as pieces.

A binding works in an attribute only. `<Text>{Bind Document.No}</Text>` or `<Text>Invoice {Document.No}</Text>` raises no error and substitutes nothing: the paper shows exactly what is written, braces included. Substitution directly in text exists only in [workbook](https://docs-llm.a2v10.com/report/workbook.md) cells.

### Span

A piece of text. Content property: `Content` — a constant string or a binding. It has all the properties of [the base element](#base-element), so styling (bold, italic, colour, size) is set on the `Span` when it is not meant for the whole paragraph.

### Space

A gap of a given width between pieces of text. Its only own property is `Width` ([Length](#length)).

### Break

A line break inside a paragraph.

### Page numbers

Two service values are available in running headers and footers: `$(PageNumber)` — the current page number, and `$(TotalPages)` — the total count. They are written as the content of a `Span` and replaced while printing.

```xml
<Page.Footer>
    <Column>
        <Text Align="Right">
            <Span Content="Page"/>
            <Span Content="$(PageNumber)"/>
            <Span Content="of"/>
            <Span Content="$(TotalPages)"/>
        </Text>
    </Column>
</Page.Footer>
```

## List

A list: a sequence of items, each with a bullet and content. Like a table, it can repeat over a collection. Content property: `Items`.

| Property | Type | Description |
|----------|------|-------------|
| `ItemsSource` | Array | Binding only. The collection the items are repeated over. Inside, expressions are read from the element of the collection |
| `Items` | ListItemCollection | Content property. The items of the list |
| `Spacing` | Single | The gap between a bullet and its content, in points |

`ListItem` — content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Object | Content property: text, a binding or a nested element |
| `Bullet` | Object | The bullet to the left of the content: constant text or a binding, a row number for instance |

```xml
<!-- a constant list -->
<List Spacing="4">
    <ListItem Bullet="—" Content="First condition"/>
    <ListItem Bullet="—" Content="Second condition"/>
</List>

<!-- over a collection: one ListItem repeats -->
<List Spacing="4" ItemsSource="{Bind Document.Rows}">
    <ListItem Bullet="{Bind RowNo}" Content="{Bind Item.Name}"/>
</List>
```

## Line

A horizontal rule across the whole available width. A separator between parts of a document, or a line to sign on.

| Property | Type | Description |
|----------|------|-------------|
| `Thickness` | [Length](#length) | Line width. `1pt` by default |

```xml
<Line Thickness=".5pt" Margin="6pt,0"/>
```

## Checkbox

A check box: a square with or without a tick. It is always drawn — an empty square prints too, which is what leaves room for a mark by hand.

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Boolean? | Whether the tick is there. Usually a binding; any value other than `true` gives an empty square |

```xml
<Checkbox Value="{Bind Document.Done}"/>
```

## Notes

- A bullet does not number itself. When a number is needed, it comes from the model (`{Bind RowNo}`) or from an expression.
- The length of a `Line` cannot be set: it takes the full width of its parent. To shorten it, put it in a [Column](#column) of the required width, or use margins.
- The size of a `Checkbox` is fixed at 12 points. Its `Width` and `Height` properties are declared but have no effect on print.
- The service values `$(PageNumber)` and `$(TotalPages)` are meant for running headers and footers — that is where the engine knows the page number.
