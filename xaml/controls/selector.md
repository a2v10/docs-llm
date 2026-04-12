# Selector

> A universal selection control with search support, optionally delegating lookup to the server.

## Overview

`Selector` is the primary lookup/search control. It inherits from `ValuedControl → Control → UIElement → UIElementBase`. It operates in two modes:

- **List mode** — `ItemsSource` bound to a model array; the user picks from a dropdown list, optionally filtered by typing.
- **Delegate mode** — `Delegate` points to a JavaScript function that fetches matches from the server as the user types.

When an item is selected, the whole object is stored in the bound `Value`. A `SetDelegate` can intercept the selection to perform custom assignments.

Selector supports creating new records inline via `NewPane` and `CreateNewCommand`.

## Syntax

```xml
<!-- Simple list -->
<Selector Label="Partner" Value="{Bind Document.Agent}"
          ItemsSource="{Bind Agents}" DisplayProperty="Name" />

<!-- Server-side search delegate -->
<Selector Label="Item" Value="{Bind Row.Item}"
          Delegate="fetchItems" DisplayProperty="Name"
          Placeholder="Search items..." />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `ItemsSource` | Array | **Bind only.** Data source for list mode. |
| `DisplayProperty` | String | Property of items to display in the selector field and list. |
| `Delegate` | String | JavaScript delegate name called when user types (for server search). |
| `SetDelegate` | String | JavaScript delegate called when user selects an item. Prevents automatic assignment if specified. |
| `TextValue` | Object | **Bind only.** Pass typed text (not selected object) to the model. |
| `Align` | TextAlign | Text alignment: `Left`, `Right`, `Center` |
| `Placeholder` | String | Hint text shown when field is empty. |
| `Style` | SelectorStyle | `Default`, `ComboBox` (dropdown style), or `Hyperlink` |
| `Size` | ControlSize | `Default`/`Normal` or `Large` |
| `PanelPlacement` | DropDownPlacement | Dropdown direction: `BottomLeft`, `BottomRight`, `TopLeft`, `TopRight` |
| `ListSize` | Size | Dropdown panel dimensions |
| `ItemsPanel` | UIElementBase | Custom panel for the dropdown (DataGrid supported) |
| `NewPane` | UIElementBase | Panel shown when no results found — for creating new records |
| `CreateNewCommand` | BindCmd | Command fired when creating a new record from `NewPane` |
| `ShowCaret` | Boolean? | Show dropdown caret icon |
| `ShowClear` | Boolean | Show × button to clear the selection |
| `UseAll` | Boolean | Distinguish "nothing selected" from "all selected" |
| `Highlight` | Boolean | Visual emphasis when a value is selected |
| `MaxChars` | Int32 | Max characters before truncation with ellipsis |
| `LineClamp` | Int32 | Max lines before truncation |
| `FetchData` | Object | **Bind only.** Extra data passed to the delegate function |

### From ValuedControl

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | **Binding only.** The selected object from the list. |

See [Base Classes](../base-classes.md) for `Label`, `Required`, `Disabled`, and other inherited properties.

### SelectorStyle Values

| Value | Description |
|-------|-------------|
| `Default` | Standard input with dropdown |
| `ComboBox` | Dropdown-button style |
| `Hyperlink` | Link-styled, no border |

## Delegate Format

The `Delegate` JavaScript function is called as the user types:

```javascript
// Returns array of matching items or a Promise
function fetchItems(this: IRoot, text: string, data?: any): Array | Promise<Array>
```

The `SetDelegate` function is called when the user selects an item:

```javascript
// this — root model, item — selected element
function onItemSet(this: IRoot, item: any): void
```

## Example

```xml
<!-- Lookup from pre-loaded model array -->
<Selector Label="Warehouse"
          Value="{Bind Document.Warehouse}"
          ItemsSource="{Bind Warehouses}"
          DisplayProperty="Name"
          ShowClear="True" />

<!-- Server-side search with delegate -->
<Selector Label="Customer"
          Value="{Bind Document.Customer}"
          Delegate="fetchCustomers"
          DisplayProperty="Name"
          Placeholder="Type to search..."
          ShowClear="True"
          Required="True" />

<!-- With custom dropdown panel (DataGrid) -->
<Selector Label="Product"
          Value="{Bind Row.Product}"
          Delegate="fetchProducts"
          DisplayProperty="Name">
  <Selector.ItemsPanel>
    <DataGrid>
      <DataGridColumn Header="Code" Content="{Bind Code}" Width="80px" />
      <DataGridColumn Header="Name" Content="{Bind Name}" />
      <DataGridColumn Header="Price" Content="{Bind Price, DataType=Currency}"
                      Align="Right" Width="100px" />
    </DataGrid>
  </Selector.ItemsPanel>
</Selector>

<!-- With create-new option -->
<Selector Label="Tag"
          Value="{Bind Document.Tag}"
          Delegate="fetchTags"
          DisplayProperty="Name"
          CreateNewCommand="{BindCmd Dialog, Action=New, Argument={Bind NewTag}}">
  <Selector.NewPane>
    <Grid Columns="1*">
      <TextBox Label="New tag name" Value="{Bind NewTag.Name}" />
    </Grid>
  </Selector.NewPane>
</Selector>
```

## Notes

- When `Delegate` is set, the function is called with the typed text; the result array populates the dropdown.
- When `SetDelegate` is set, automatic assignment of the selected object to `Value` is skipped — the delegate must do all assignments manually.
- `FetchData` passes additional model data to the delegate — useful for context-dependent searches (e.g., filter by document type).
- `ItemsPanel` with a `DataGrid` lets you display multiple columns in the dropdown — useful for product lookups showing code, name, and price.
- `UseAll` is for filter selectors where an empty selection means "all" — it changes the internal null semantics.
- `Style="ComboBox"` makes the selector look like a regular combobox but retains search functionality.
