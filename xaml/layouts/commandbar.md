# CommandBar

> A row of commands inside a data row — its visibility follows the state of the row it sits in.

## Overview

`CommandBar` is a panel of commands, normally placed inside a [DataGrid](https://docs-llm.a2v10.com/xaml/controls/datagrid.md) column or a list item. What makes it different from an ordinary container is that its visibility is decided by the state of its parent: put it in a grid column with `Visibility="Active"` and the commands appear only for the active row.

It inherits from `Container → UIElement → UIElementBase`.

## Use When

- Row-level actions — edit, delete, open — should appear on the active row or under the mouse instead of on every row at once.

## Do Not Use When

- The actions belong to the whole page rather than to a row — use [Toolbar](https://docs-llm.a2v10.com/xaml/layouts/toolbar.md) instead.
- The buttons must always be visible and unrelated to row state — an ordinary [StackPanel](https://docs-llm.a2v10.com/xaml/layouts/stackpanel.md) of buttons is enough.

## Syntax

```xml
<CommandBar Visibility="Hover" Float="Right">
  <Button Icon="Edit" Command="{BindCmd Dialog, Action=Edit, Argument={Bind}}" />
  <Button Icon="Clear" Command="{BindCmd DbRemove, Argument={Bind}}" />
</CommandBar>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Visibility` | CommandBarVisibility | When the panel is shown: `Default` — always (default), `Active` — only for the active element, `Hover` — for the active element and the one under the mouse pointer |
| `Float` | FloatMode | Float the panel: `None` (default), `Left`, `Right` |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties from `Container` and `UIElement`.

## Example

```xml
<DataGrid ItemsSource="{Bind Agents}">
  <DataGridColumn Header="Name" Content="{Bind Name}" />
  <DataGridColumn Header="Memo" Content="{Bind Memo}" />
  <DataGridColumn>
    <CommandBar Visibility="Hover" Float="Right">
      <Button Icon="Edit" Tip="Edit"
              Command="{BindCmd Dialog, Action=Edit, Argument={Bind}}" Style="Toolbar" />
      <Button Icon="Clear" Tip="Delete"
              Command="{BindCmd DbRemove, Argument={Bind},
                                 Confirm={Confirm Message='Delete the agent?'}}"
              Style="Toolbar" />
    </CommandBar>
  </DataGridColumn>
</DataGrid>
```

## Notes

- `Visibility` is about the parent's state, not about a condition on the data: for a condition use `If` or `Show` as on any other element.
- `Visibility="Hover"` keeps the commands on the active row as well, so the row the user is working with does not lose its buttons when the pointer leaves it.
- With `Float="Right"` the commands stay at the right edge of the cell regardless of the column width.
