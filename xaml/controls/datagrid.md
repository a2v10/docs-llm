# DataGrid

> A data table control for displaying and interacting with array data from the model.

## Overview

`DataGrid` renders a table where each row corresponds to one element of an [array in the model](https://docs-llm.a2v10.com/client/array.md). Columns are defined with nested `DataGridColumn` elements. It inherits from `Control → UIElement → UIElementBase`.

DataGrid is the primary list control in A2v10. When combined with a `CollectionView`, it gains sorting, filtering, and pagination. Rows can be grouped by field values on the client (`GroupBy`), marked with colors, expanded for details, double-clicked, or right-clicked for a context menu.

## Syntax

```xml
<DataGrid ItemsSource="{Bind Documents}" Hover="True" Striped="True">
  <DataGridColumn Header="Date" Content="{Bind Date, DataType=Date}" Width="120px" />
  <DataGridColumn Header="Name" Content="{Bind Name}" />
  <DataGridColumn Header="Amount" Content="{Bind Amount, DataType=Currency}" Align="Right" />
</DataGrid>
```

## DataGrid Properties

| Property | Type | Description |
|----------|------|-------------|
| `ItemsSource` | Array | **Required.** Data source — always a `Bind` to an array in the model. |
| `Compact` | Boolean | Compact row height style |
| `Hover` | Boolean | Highlight rows on cursor hover |
| `Striped` | Boolean | Alternate row background colors |
| `Border` | Boolean | Show cell borders |
| `Sort` | Boolean | Enable column sorting (requires `CollectionView`) |
| `FixedHeader` | Boolean | Keep header row fixed when scrolling |
| `Style` | DataGridStyle | `Default` or `Light` |
| `Background` | BackgroundStyle | Table background color |
| `Height` | Length | Fixed table height (triggers scrolling) |
| `Mark` | Object | **Bind only.** Returns a CSS class string to color-mark the row. |
| `MarkerStyle` | RowMarkerStyle | Mark style: `None`, `Row`, `Marker`, `Both` |
| `EmptyPanel` | UIElement | Panel shown when the array is empty |
| `EmptyPanelDelegate` | String | Delegate name for dynamic empty panel |
| `DoubleClick` | BindCmd | Command fired on row double-click |
| `ContextMenu` | DropDownMenu | Right-click context menu |
| `RowDetails` | DataGridRowDetails | Expandable row detail section |
| `GroupBy` | String | Row grouping: comma-separated row fields, each field the next nesting level — see [Row Grouping](#row-grouping) |
| `AutoSelect` | AutoSelectMode | Auto-select after load: `FirstItem`, `LastItem` |
| `HeadersVisibility` | HeadersVisibility | Show/hide header row |
| `RowBold` | Boolean? | Bold font for all rows |
| `GridLines` | GridLinesVisibility | Grid lines: `None` (default), `Horizontal`, `Vertical`, `Both` |

### Row Mark Colors

The `Mark` binding should return one of these strings:

| String | Effect |
|--------|--------|
| `"danger"`, `"red"`, `"error"` | Red row marker |
| `"warning"`, `"yellow"` | Yellow row marker |
| `"success"`, `"green"` | Green row marker |
| `"info"`, `"cyan"` | Cyan row marker |

### Row Grouping

The table groups rows itself, on the client, and only the rows it has already received: with server-side paging, groups are built within the current page.

```xml
<DataGrid ItemsSource="{Bind Documents}" GroupBy="Agent.Name,Date">
  <DataGridColumn Header="Number" Content="{Bind No}" />
  <DataGridColumn Header="Amount" Content="{Bind Sum, DataType=Currency}" />
</DataGrid>
```

- Each group starts with a header row: the field value and the row count. A top-level group's count is the sum over its nested groups.
- Clicking a header row collapses or expands the group.
- A date value is shown in the `Date` format.
- Rows with an empty field value go into a group of their own.
- Sorting by a column rebuilds the groups from the sorted rows.

## DataGridColumn Properties

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Bind | **Required.** Cell content — always a `Bind` to a row property. |
| `Header` | String | Column header text |
| `Width` | Length | Column width |
| `Fit` | Boolean | Auto-fit column width to content |
| `Wrap` | WrapMode | Cell text wrap: `Wrap`, `NoWrap`, `PreWrap` |
| `Align` | TextAlign | Cell horizontal alignment: `Left`, `Right`, `Center` |
| `Bold` | Boolean | Bold cell text |
| `Small` | Boolean | Smaller font size for cells |
| `Mark` | Object | **Bind only.** CSS class string for cell-level color marking |
| `Done` | Object | **Bind only.** "Posted" flag: an icon before the cell content — green when the value is true, yellow when not |
| `CheckAll` | String | **Bind only.** A boolean row property, `CheckAll="{Bind Checked}"`. Replaces the header with a "select all" checkbox: if the property is set in every row, a click clears it in all rows, otherwise sets it in all rows. Shows three states: all, some, none |
| `Icon` | Icon | Icon shown before cell content |
| `MaxChars` | Int32 | Max characters before truncation (rest shown as tooltip) |
| `LineClamp` | Int32 | Max lines before truncation |
| `Editable` | Boolean | Allow inline cell editing |
| `ControlType` | ColumnControlType | Control type for editable cells |
| `Sort` | Boolean? | Override table-level sort permission for this column |
| `SortProperty` | String | Model property to sort by (defaults to `Content` path) |
| `If` | Boolean? | Conditional column rendering — use `Bind` |
| `Command` | BindCmd | Make cell content a clickable hyperlink |
| `Role` | ColumnRole | Preset set of properties — see [ColumnRole Values](#columnrole-values) |
| `MinWidth` | Length | Minimum column width |
| `Background` | ColumnBackgroundStyle | Cell background: `Yellow`, `Green`, `Red`, `Blue`, `Gray` |
| `CssClass` | String | Additional CSS classes on cells |

### ColumnRole Values

| Value | Displays | Same as |
|-------|----------|---------|
| `Default` | — (default) | nothing is set |
| `Id` | An identifier | `Fit="True" Align="Right" Wrap="NoWrap"` |
| `Number` | A number | `Fit="True" Align="Right" Wrap="NoWrap"` |
| `Date` | A date | `Fit="True" Align="Center" Wrap="NoWrap"` |
| `CheckBox` | A checkbox | `Fit="True" Align="Center" Wrap="NoWrap" ControlType="CheckBox"` |
| `Code` | A code | `Fit="True" Wrap="NoWrap"` |
| `Fit` | Any value; alignment unchanged | `Fit="True" Wrap="NoWrap"` |

## Example

```xml
<!-- Standard list page -->
<Page Title="Documents">
  <Page.Toolbar>
    <Toolbar>
      <Button Content="Create" Command="{BindCmd Dialog, Action=New}" Icon="Plus" Style="Primary" />
      <Button Content="Reload" Command="{BindCmd Reload}" Icon="Reload" />
    </Toolbar>
  </Page.Toolbar>

  <DataGrid ItemsSource="{Bind Documents}" Hover="True" Sort="True"
            Mark="{Bind Mark}" MarkerStyle="Marker"
            DoubleClick="{BindCmd Dialog, Action=Edit, Argument={Bind Documents.Selected}}">
    <DataGridColumn Header="Date" Content="{Bind Date, DataType=Date}" Width="100px" Role="Date" />
    <DataGridColumn Header="Number" Content="{Bind Number}" Width="80px" Align="Right" />
    <DataGridColumn Header="Partner" Content="{Bind Agent.Name}" />
    <DataGridColumn Header="Amount" Content="{Bind Sum, DataType=Currency}"
                    Align="Right" Width="120px" />
    <DataGridColumn Header="State" Content="{Bind StateText}" Width="100px" />
  </DataGrid>
</Page>

<!-- Editable grid -->
<DataGrid ItemsSource="{Bind Document.Rows}" Hover="True">
  <DataGridColumn Header="#" Content="{Bind Num}" Width="40px" Align="Center" />
  <DataGridColumn Header="Item" Content="{Bind Item.Name}" />
  <DataGridColumn Header="Qty" Content="{Bind Qty, DataType=Number}"
                  Align="Right" Editable="True" Width="80px" />
  <DataGridColumn Header="Price" Content="{Bind Price, DataType=Currency}"
                  Align="Right" Editable="True" Width="100px" />
  <DataGridColumn Header="Total" Content="{Bind Total, DataType=Currency}"
                  Align="Right" Width="120px" Bold="True" />
</DataGrid>

<!-- With empty panel -->
<DataGrid ItemsSource="{Bind Items}">
  <DataGrid.EmptyPanel>
    <EmptyPanel />
  </DataGrid.EmptyPanel>
  <DataGridColumn Header="Name" Content="{Bind Name}" />
</DataGrid>
```

## Notes

- `ItemsSource` must always be a [`Bind`](https://docs-llm.a2v10.com/xaml/bind.md) — DataGrid only works with model arrays.
- Inside `DataGridColumn`, bindings are relative to the current row element.
- `DoubleClick` is the most common way to open an edit dialog from a list — it fires with the selected row as context.
- `Sort="True"` on the table enables sorting UI; individual columns can opt out with `Sort="False"`.
- `Editable="True"` on a column renders an inline edit field; the `ControlType` property selects which control type to use.
- `Mark` binding is evaluated for each row and should return a string (one of the named color classes) or empty string/null for no mark.
- `Role` is shorthand for a set of column properties — `Role="Id"` is the same as `Fit="True" Align="Right" Wrap="NoWrap"`.
- The bound array is shaped server-side — see [array models](https://docs-llm.a2v10.com/sql/array.md). A hierarchical source is a [tree model](https://docs-llm.a2v10.com/sql/tree.md), which DataGrid renders as an expandable tree.
- At runtime the bound array exposes selection and helpers through the [Array API](https://docs-llm.a2v10.com/client/array.md) (for example `Documents.Selected`, used by `DoubleClick` above); a hierarchical row is a [Tree Element](https://docs-llm.a2v10.com/client/tree-element.md) with `$expand` / `$selectPath`.
