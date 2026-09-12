# Taskpad

> The side panel of a page or dialog — a container shown at the right edge, set through the Taskpad property of the root.

## Overview

`Taskpad` is the task panel displayed on the right-hand side of a page or a dialog. It is not placed in the content tree: it is assigned to the `Taskpad` property of [Page](https://docs-llm.a2v10.com/xaml/layouts/page.md) or [Dialog](https://docs-llm.a2v10.com/xaml/layouts/dialog.md).

It inherits from `Container → UIElement → UIElementBase`. Content property: `Children`.

## Use When

- The page needs contextual links, filters or secondary actions beside the main content rather than inside it.

## Do Not Use When

- The content is the page's primary subject — it belongs in the page body.
- The panel is a block inside the content flow — use [Panel](https://docs-llm.a2v10.com/xaml/layouts/panel.md) instead.

## Syntax

```xml
<Page.Taskpad>
  <Taskpad Title="Filters" Width="260px" Collapsible="True">
    <!-- content -->
  </Taskpad>
</Page.Taskpad>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Title` | String | The title of the task panel |
| `Overflow` | Boolean | Whether a scrollbar may appear in the panel |
| `Collapsible` | Boolean | Whether the panel can be collapsed |
| `Collapsed` | Boolean? | Whether the panel starts collapsed |
| `Width` | Length | Panel width |
| `Background` | BackgroundStyle | Background colour |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties from `Container` and `UIElement`.

## Example

```xml
<Page xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.Xaml" Title="Agents">
  <Page.Taskpad>
    <Taskpad Title="Filter" Width="240px" Collapsible="True" Overflow="True">
      <Grid Columns="1*">
        <TextBox Label="Name" Value="{Bind Filter.Fragment}" />
        <ComboBox Label="Category" Value="{Bind Filter.Category}"
                  ItemsSource="{Bind Categories}" DisplayProperty="Name" />
        <Button Content="Apply" Command="{BindCmd Reload}" Style="Primary" />
      </Grid>
    </Taskpad>
  </Page.Taskpad>

  <DataGrid ItemsSource="{Bind Agents}" />
</Page>
```

## Notes

- The panel is positioned by the root element, not by the layout around it: its place is always the right edge of the page or dialog.
- A [Toolbar](https://docs-llm.a2v10.com/xaml/layouts/toolbar.md) can also be assigned to `Dialog.Taskpad` — the property takes any element, `Taskpad` is simply the one built for the job.
- `Overflow` decides what happens when the panel content is taller than the page; without it, long content is not scrollable inside the panel.
