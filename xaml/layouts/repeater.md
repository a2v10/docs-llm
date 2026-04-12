# Repeater

> An invisible container that repeats its content for each item in an array.

## Overview

`Repeater` renders its `Content` element once for each item in `ItemsSource`. It inherits from `UIElement → UIElementBase`. Unlike `DataGrid` or `Table`, it produces no wrapper HTML — it is a transparent repeating mechanism.

Inside the repeated `Content`, bindings are relative to the current array item (not the page root).

## Syntax

```xml
<Repeater ItemsSource="{Bind Items}">
  <Repeater.Content>
    <Card>
      <TextBox Label="Name" Value="{Bind Name}" />
    </Card>
  </Repeater.Content>
</Repeater>
```

Or using the content property shorthand:

```xml
<Repeater ItemsSource="{Bind Items}">
  <Card>
    <TextBox Label="Name" Value="{Bind Name}" />
  </Card>
</Repeater>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Content` | UIElementBase | **Content property.** The element repeated for each item in `ItemsSource`. |
| `ItemsSource` | Array | **Bind only.** The array to iterate over. |

See [Base Classes](../base-classes.md) for inherited properties from `UIElement` and `UIElementBase`.

## Example

```xml
<!-- Repeat a card for each document -->
<Repeater ItemsSource="{Bind Documents}">
  <Card>
    <Grid Columns="1*,120px">
      <Static Label="Name" Value="{Bind Name}" />
      <Static Label="Amount" Value="{Bind Amount, DataType=Currency}" Align="Right" />
    </Grid>
  </Card>
</Repeater>

<!-- Repeat form rows for detail items -->
<Repeater ItemsSource="{Bind Document.Rows}">
  <Grid Columns="1*,80px,100px">
    <Selector Label="Item" Value="{Bind Item}" DisplayProperty="Name"
              ItemsSource="{Bind $root.Items}" />
    <TextBox Label="Qty" Value="{Bind Qty, DataType=Number}" Align="Right" />
    <TextBox Label="Price" Value="{Bind Price, DataType=Currency}" Align="Right" />
  </Grid>
</Repeater>
```

## Notes

- `Repeater` emits no wrapper element — child elements are rendered directly into the parent container.
- Bindings inside `Content` are relative to the current item from `ItemsSource`. To access the page root, use `$root.` prefix.
- For tabular data, prefer `DataGrid` — it provides sorting, selection, and header row. Use `Repeater` for card layouts or custom repetitive structures.
- `If` and `Show` on the `Repeater` itself controls whether the entire repeated section renders.
