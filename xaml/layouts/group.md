# Group

> An invisible container — holds several elements where only one is allowed, and switches them all on or off together.

## Overview

`Group` has no appearance of its own. It exists for two reasons: to put several elements in a place where the markup allows a single one, and to control the visibility of everything inside it with one condition — `If`, `Show` or `Hide` on the group instead of on each child.

It inherits from `Container → UIElement → UIElementBase` and adds no properties of its own.

## Use When

- A property accepts one element (a header, a cell, a taskpad slot) and two or three are needed there.
- Several elements appear and disappear together, and repeating the same `If` on each of them is the only alternative.

## Do Not Use When

- The group is inside a [Grid](https://docs-llm.a2v10.com/xaml/layouts/grid.md) and its children carry `Grid.Row` / `Grid.Col` — use `GridGroup` instead, which is transparent to attached properties; see [Grid](https://docs-llm.a2v10.com/xaml/layouts/grid.md#gridgroup).
- The block should be visible — a frame, a header, a background — use [Panel](https://docs-llm.a2v10.com/xaml/layouts/panel.md) or [FieldSet](https://docs-llm.a2v10.com/xaml/layouts/fieldset.md) instead.
- The children need to be laid out in a row or a column with spacing — use [StackPanel](https://docs-llm.a2v10.com/xaml/layouts/stackpanel.md) instead.

## Syntax

```xml
<Group If="{Bind Document.CanBeApproved}">
  <Button Content="Approve" Command="{BindCmd Execute, CommandName=Approve}" />
  <Button Content="Reject" Command="{BindCmd Execute, CommandName=Reject}" />
</Group>
```

## Properties

The element has no properties of its own. See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for what `Container`, `UIElement` and `UIElementBase` provide — `Children`, `ItemsSource`, `If`, `Show`, `Hide` and the rest.

## Example

```xml
<!-- two elements in a property that takes one -->
<Panel>
  <Panel.Header>
    <Group>
      <Span Content="{Bind Document.Name}" Bold="True" />
      <Badge Content="{Bind Document.RowCount}" />
    </Group>
  </Panel.Header>
  <DataGrid ItemsSource="{Bind Document.Rows}" />
</Panel>

<!-- one condition for a whole block of fields -->
<Grid Columns="1*,1*">
  <TextBox Label="Name" Value="{Bind Agent.Name}" />
  <Group Show="{Bind Agent.IsCompany}">
    <TextBox Label="Tax ID" Value="{Bind Agent.TaxId}" />
    <TextBox Label="Bank account" Value="{Bind Agent.Account}" />
  </Group>
</Grid>
```

## Notes

- Since the element renders nothing, only the visibility properties are meaningful on it. Margins, background and the like have nowhere to apply.
- `If` removes the children from the DOM; `Show` and `Hide` keep them and toggle CSS. The choice on a group is the same as on a single element — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).
