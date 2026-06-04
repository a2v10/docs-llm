# Sheet

> Spreadsheet-style table for reports — fixed header/footer, repeating data sections, hierarchical groups, and dynamic cross columns, optimized for export to Excel.

## Overview

`Sheet` renders a table in the style of an electronic spreadsheet. Unlike `DataGrid`, which shows a flat bound collection with interactive columns, `Sheet` is built for reports: it has explicit header and footer regions, one or more data sections, merged cells, totals rows, and indentation for hierarchical data. It is optimized for simple export to Excel. It inherits from `UIElement → UIElementBase`.

A sheet is assembled from a small family of elements:

| Element | Role |
|---------|------|
| `Sheet` | The table itself; holds columns, header, footer, and sections |
| `SheetColumn` | One column definition (width / fit / background) |
| `SheetRow` | One row; holds cells and carries a style |
| `SheetCell` | One cell; content, spanning, alignment, indentation |
| `SheetCellGroup` | Expands a bound array into a variable number of cells (cross columns) |
| `SheetSection` | A body region that repeats its rows for each item in `ItemsSource` |
| `SheetTreeSection` | Like `SheetSection`, but binds recursively for tree / grouping models |
| `SheetGroupCell` | A cell that renders the expand/collapse control inside a tree section |

`Sheet` is the natural renderer for the data shapes described in [Tree & Hierarchy](https://docs-llm.a2v10.com/sql/tree.md), [Grouping Models](https://docs-llm.a2v10.com/sql/grouping.md), and [Cross (Pivot) Models](https://docs-llm.a2v10.com/sql/cross.md).

## Syntax

```xml
<Sheet GridLines="Both" Compact="True">
  <Sheet.Columns>
    <SheetColumn Fit="True" />
    <SheetColumn Width="120px" />
  </Sheet.Columns>
  <Sheet.Header>
    <SheetRow Style="Header"> ... </SheetRow>
  </Sheet.Header>
  <SheetSection ItemsSource="{Bind Items}">
    <SheetRow> ... </SheetRow>
  </SheetSection>
  <Sheet.Footer>
    <SheetRow Style="Total"> ... </SheetRow>
  </Sheet.Footer>
</Sheet>
```

## Properties

### Sheet

| Property | Type | Description |
|----------|------|-------------|
| `Sections` | SheetSections | Body sections (content property — `SheetSection` / `SheetTreeSection` elements). |
| `Columns` | SheetColumnCollection | Column definitions. Also accepts an inline shorthand string, e.g. `Columns="Fit,Auto,Auto"`. |
| `Header` | SheetRows | Header rows (`Sheet.Header`). |
| `Footer` | SheetRows | Footer rows (`Sheet.Footer`). |
| `GridLines` | GridLinesVisibility | `None` (default), `Horizontal`, `Vertical`, `Both`. |
| `Compact` | Boolean | Compact (denser) display. |
| `Hover` | Boolean | Highlight rows on mouse hover. |
| `Striped` | Boolean | Alternating row shading. |
| `AutoGenerate` | SheetAutoGenerate | Generate the table automatically from the data shape. |

### SheetColumn

| Property | Type | Description |
|----------|------|-------------|
| `Width` | Length | Explicit column width. |
| `Fit` | Boolean | Size the column to its content. |
| `Background` | ColumnBackgroundStyle | Column background color. |

### SheetRow

| Property | Type | Description |
|----------|------|-------------|
| `Cells` | SheetCell collection | Cells in the row (content property). |
| `Style` | RowStyle | Visual style: `Default`, `Title`, `Parameter`, `LastParameter`, `Header`, `LightHeader`, `Footer`, `Total`, `NoBorder`. |
| `Align` | TextAlign? | Horizontal alignment for all cells; can be overridden per cell. |
| `Mark` | Object | Bind-only; CSS class for row marking (`danger`/`error`/`red`, `warning`/`orange`, `success`/`green`, `info`/`cyan`, `gray`, combinable e.g. `"danger bold"`). |

### SheetCell

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Object | Cell content; supports `Bind` with `DataType` and `HideZeros`. |
| `ColSpan` | Int32? | Merge columns. |
| `RowSpan` | Int32? | Merge rows. |
| `Align` | TextAlign | Horizontal alignment. |
| `VAlign` | VerticalAlign | Vertical alignment. |
| `GroupIndent` | Boolean | Indent the content by the hierarchy level (tree / grouping rows). |
| `Bold` / `Italic` | Boolean? | Font weight / style; inherited from the row if unset. |
| `Underline` | Boolean | Underline text. |
| `Vertical` | Boolean | Vertical text orientation. |
| `Height` / `MinWidth` | Length | Cell dimensions. |
| `DataType` | DataType | Data type used for formatting and export. |
| `CssClass` | String | Static CSS classes. |
| `Fill` | Bind | Dynamic background color. |
| `CssStyle` | Bind | Dynamic CSS rules object. |

### SheetSection / SheetTreeSection

| Property | Type | Description |
|----------|------|-------------|
| `Children` | SheetRow collection | Rows of the section (content property). |
| `ItemsSource` | Array | Always a `Bind`. Items the section repeats over; `SheetTreeSection` recurses into nested arrays. |

`SheetCellGroup` exposes `ItemsSource` (always a `Bind`) and a `Cells` collection — it can be used anywhere a `SheetCell` is allowed and emits one rendering of its cells per item. `SheetGroupCell` has no properties of its own; it renders the expand/collapse control and is only meaningful inside a `SheetTreeSection` (empty elsewhere).

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties.

## Example

A grouping report (see [Grouping Models](https://docs-llm.a2v10.com/sql/grouping.md)) rendered as an indented totals table. `SheetTreeSection` walks the nested `Items`, `SheetGroupCell` draws the expand control, and `GroupIndent` indents the label by depth.

```xml
<Sheet GridLines="Both" Columns="Fit,Auto,Auto">
  <Sheet.Header>
    <SheetRow Style="Header">
      <SheetCell />
      <SheetCell Content="Agent/Date" />
      <SheetCell Content="Amount" />
    </SheetRow>
    <SheetRow Style="Total">
      <SheetCell ColSpan="2" Content="Total" />
      <SheetCell Content="{Bind ReportData.Amount, DataType=Currency}" Align="Right" />
    </SheetRow>
  </Sheet.Header>
  <SheetTreeSection ItemsSource="{Bind ReportData.Items}">
    <SheetRow>
      <SheetGroupCell />
      <SheetCell GroupIndent="True" Content="{Bind $groupName}" />
      <SheetCell Content="{Bind Amount, DataType=Currency}" Align="Right" />
    </SheetRow>
  </SheetTreeSection>
</Sheet>
```

### Cross Report with Dynamic Columns

`SheetCellGroup` turns a bound array into a variable number of cells, which is how cross (pivot) columns are rendered. The header binds to the key list (`$cross`), the body to each row's cross array. See [Cross (Pivot) Models](https://docs-llm.a2v10.com/sql/cross.md).

```xml
<Sheet Margin="1rem" GridLines="Both" Compact="True">
  <Sheet.Header>
    <SheetRow Style="Header">
      <SheetCell RowSpan="2">Id</SheetCell>
      <SheetCell RowSpan="2">S1</SheetCell>
      <SheetCell RowSpan="2">N1</SheetCell>
      <SheetCell ColSpan="{Bind RepData.$Cross1Span}">Cross1</SheetCell>
    </SheetRow>
    <SheetRow Style="Header">
      <SheetCellGroup ItemsSource="{Bind RepData.$cross.Cross1}">
        <SheetCell Content="{Bind}" />
      </SheetCellGroup>
      <SheetCell>Total</SheetCell>
    </SheetRow>
  </Sheet.Header>
  <SheetSection ItemsSource="{Bind RepData}">
    <SheetRow>
      <SheetCell Content="{Bind Id}" />
      <SheetCell Content="{Bind S1}" />
      <SheetCell Content="{Bind N1}" />
      <SheetCellGroup ItemsSource="{Bind Cross1}">
        <SheetCell Content="{Bind Val, DataType=Number, HideZeros=True}" />
      </SheetCellGroup>
      <SheetCell Content="{Bind $Cross1Total}" />
    </SheetRow>
  </SheetSection>
</Sheet>
```

## Notes

- Rows defined directly in the sheet body (outside an explicit `SheetSection`) are placed in an implicit section with no data source — fine for static rows, but bound repetition requires a `SheetSection` with `ItemsSource`.
- Use `SheetTreeSection` (not `SheetSection`) for tree and grouping models — it recurses into the nested `Items` arrays; pair it with `SheetGroupCell` for the collapse control and `GroupIndent="True"` to indent by level.
- `SheetCellGroup` is the only way to emit a data-driven number of cells; its `ItemsSource` is always a `Bind`. It is the building block for cross/pivot columns.
- `ColSpan` can itself be bound (e.g. `ColSpan="{Bind RepData.$Cross1Span}"`) so a header spans a variable number of cross columns.
- `Sheet` is export-oriented; set `DataType` on cells so numbers, currency, and dates format and export correctly. `HideZeros=True` suppresses zero values (common in padded cross arrays).
- `GridLines` defaults to `None`; reports usually set `Both`.
