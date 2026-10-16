# Slot

> Sets the data context for the elements inside it — and shows a fallback element while that context is empty.

## Overview

`Slot` binds its children to one object of the model. Bindings inside it are resolved against `Scope`, not against the page model: inside `<Slot Scope="{Bind Agents.$selected}">`, `{Bind Documents}` means `Agents.$selected.Documents`.

When the context is empty — typically nothing is selected in a list — the children are not rendered, and the `Fallback` element is shown in their place.

`Slot` inherits `UIElementBase`. Content property: `Children`.

## Use When

- A detail area follows the selected item of a list or a grid — the master/detail pattern.
- The area has to explain itself when there is no item to show.

## Do Not Use When

- The block is shown or hidden by a condition, with no change of data context — use `If` on the element, see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).
- One of several blocks is chosen by a value — use [Switch, Case and Else](https://docs-llm.a2v10.com/xaml/switch.md) instead.

## Syntax

```xml
<Slot Scope="{Bind Agents.$selected}">
  <!-- elements bound relative to Scope -->
  <Slot.Fallback>
    <!-- element shown while Scope is empty -->
  </Slot.Fallback>
</Slot>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Children` | UIElementCollection | Content property. The child elements |
| `Scope` | Object | Always a binding. The data context |
| `Fallback` | UIElementBase | The element shown when the data context is empty |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for the properties `Slot` inherits.

## Example

Agents on the left, the documents of the selected agent on the right:

```xml
<Grid Columns="20rem,1*">
  <List ItemsSource="{Bind Agents}" AutoSelect="FirstItem">
    <ListItem Content="{Bind Name}" />
  </List>
  <Slot Scope="{Bind Agents.$selected}">
    <!-- in fact: Agents.$selected.Documents -->
    <DataGrid ItemsSource="{Bind Documents}">
      <DataGridColumn Header="Date" Content="{Bind Date, DataType=Date}" />
      <DataGridColumn Header="Sum" Content="{Bind Sum, DataType=Currency}" />
    </DataGrid>
    <Slot.Fallback>
      <Text>No agent selected</Text>
    </Slot.Fallback>
  </Slot>
</Grid>
```

## Notes

- `Scope` is always a binding; a literal value is not a data context.
- `{Bind Agents.$selected}` is the usual scope — the selected element of an array.
- `Fallback` holds one element; wrap several in a container.
