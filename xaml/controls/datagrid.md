# DataGrid

> A data table control for displaying and interacting with array data from the model.

## Overview

`DataGrid` renders a table where each row corresponds to one element of an array in the model. Columns are defined with nested `DataGridColumn` elements. It inherits from `Control → UIElement → UIElementBase`.

DataGrid is the primary list control in A2v10. When combined with a `CollectionView`, it gains sorting, filtering, pagination, and grouping. Rows can be marked with colors, expanded for details, double-clicked, or right-clicked for a context menu.

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
| `AutoSelect` | AutoSelectMode | Auto-select after load: `FirstItem`, `LastItem` |
| `HeadersVisibility` | HeadersVisibility | Show/hide header row |
| `RowBold` | Boolean? | Bold font for all rows |
| `GridLines` | GridLinesVisibility | Show grid lines: `None`, `Columns`, `Rows`, `Both` |

### Row Mark Colors

The `Mark` binding should return one of these strings:

| String | Effect |
|--------|--------|
| `"danger"`, `"red"`, `"error"` | Red row marker |
| `"warning"`, `"yellow"` | Yellow row marker |
| `"success"`, `"green"` | Green row marker |
| `"info"`, `"cyan"` | Cyan row marker |

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
| `Icon` | Icon | Icon shown before cell content |
| `MaxChars` | Int32 | Max characters before truncation (rest shown as tooltip) |
| `LineClamp` | Int32 | Max lines before truncation |
| `Editable` | Boolean | Allow inline cell editing |
| `ControlType` | ColumnControlType | Control type for editable cells |
| `Sort` | Boolean? | Override table-level sort permission for this column |
| `SortProperty` | String | Model property to sort by (defaults to `Content` path) |
| `If` | Boolean? | Conditional column rendering — use `Bind` |
| `Command` | BindCmd | Make cell content a clickable hyperlink |
| `Role` | ColumnRole | Preset role: `Default`, `Id`, `Number`, `Date`, `CheckBox` |
| `MinWidth` | Length | Minimum column width |
| `Background` | ColumnBackgroundStyle | Cell background: `Yellow`, `Green`, `Red`, `Blue`, `Gray` |
| `CssClass` | String | Additional CSS classes on cells |

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

- `ItemsSource` must always be a `Bind` — DataGrid only works with model arrays.
- Inside `DataGridColumn`, bindings are relative to the current row element.
- `DoubleClick` is the most common way to open an edit dialog from a list — it fires with the selected row as context.
- `Sort="True"` on the table enables sorting UI; individual columns can opt out with `Sort="False"`.
- `Editable="True"` on a column renders an inline edit field; the `ControlType` property selects which control type to use.
- `Mark` binding is evaluated for each row and should return a string (one of the named color classes) or empty string/null for no mark.
- `Role="Id"` on a column applies `Id`-specific styles (e.g., monospace, no wrap).
