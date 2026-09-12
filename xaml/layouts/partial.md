# Partial, PartialBlock and Include

> Embedding one view inside another — Include hosts a page chosen at runtime, Partial and PartialBlock are the roots of the page it hosts.

## Overview

These three elements solve one problem: showing a view with its own model and its own template as part of another page.

`Include` is the host. It is a container that displays a complete independent page, most often chosen dynamically from a property of some object. It inherits `UIElementBase`.

`Partial` and `PartialBlock` are roots of the hosted view — the element a view file starts with instead of `Page`. `Partial` is a frame that fills the slot it is given; `PartialBlock` is a block with its own width, height and scrolling. Both inherit from `RootContainer → Container → UIElement → UIElementBase`, and both take `Children` as content property.

The two halves are independent: the hosted models and their views need have nothing in common with the hosting page or with each other.

## Use When

- A master-detail page where the detail view differs by the type of the selected row — the left column lists entities, the right one includes a page chosen by the current row.
- A piece of a page that must load its own data and refresh on its own.

## Do Not Use When

- The content is part of the current model — an ordinary container is enough, and a second model means a second server call for nothing.
- The embedded view is a modal — use [Dialog](https://docs-llm.a2v10.com/xaml/layouts/dialog.md), or [Popup](https://docs-llm.a2v10.com/xaml/layouts/popup.md) for a popup window.

## Syntax

The host, on the outer page:

```xml
<Include Source="{Bind Agents.Selected.DetailUrl}" Argument="{Bind Agents.Selected}" FullHeight="True" />
```

The hosted view, in its own file:

```xml
<Partial xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.Xaml">
  <Grid Columns="1*">
    <TextBox Label="Name" Value="{Bind Agent.Name}" />
  </Grid>
</Partial>
```

## Properties

### Include

| Property | Type | Description |
|----------|------|-------------|
| `Source` | String | URL of the page content. May be a binding |
| `Argument` | Object | Usually a `Bind`. Supplies the object identifier put into the URL |
| `Data` | Object | An object whose properties are passed in the URL as parameters |
| `FullHeight` | Boolean | When `True`, the container takes the full height of the hosting page |

### Partial

| Property | Type | Description |
|----------|------|-------------|
| `Background` | BackgroundStyle | Page background colour |

### PartialBlock

| Property | Type | Description |
|----------|------|-------------|
| `Height` | Length | Block height |
| `Width` | Length | Block width |
| `FullHeight` | Boolean | When `True`, the block takes the full height of its parent container |
| `Overflow` | Boolean | Whether a scrollbar may appear in the block |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties.

## Example

A page of two columns: a list of entities on the left, the properties of the selected one on the right. The right-hand view is chosen by the row itself, so different kinds of entity can show different forms.

```xml
<Page xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.Xaml" Title="Entities">
  <Grid Columns="320px,1*" Height="100%">
    <DataGrid ItemsSource="{Bind Entities}" Hover="True" />

    <Include Source="{Bind Entities.Selected.FormUrl}"
             Argument="{Bind Entities.Selected}"
             FullHeight="True" />
  </Grid>
</Page>
```

`FormUrl` is a property of the row — a plain field or a computed one — holding the URL of the view to show. Each of those URLs serves a view whose root is `Partial`.

## Notes

- `Include` needs a URL, not a file name: it points at an endpoint, which loads its own model through its own `model.json`.
- A view hosted by `Include` has its own model, so its dirty state, validation and commands are its own; the hosting page does not save it.
- Choose `Partial` when the hosted view should fill the slot it is given, and `PartialBlock` when it needs its own size or a scrollbar.
