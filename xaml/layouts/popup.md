# Popup

> The root element of a popup window loaded from the server — the third view root beside Page and Dialog.

## Overview

`Popup` is a popup window whose markup is loaded from the server. Like [Page](https://docs-llm.a2v10.com/xaml/layouts/page.md) and [Dialog](https://docs-llm.a2v10.com/xaml/layouts/dialog.md) it is always the root element of a view, never something nested inside another view.

It inherits from `RootContainer → Container → UIElement → UIElementBase`. Content property: `Children`.

The server side — which endpoint serves the popup and which procedure loads its data — is the `popups` section of `model.json`, see [Popups](https://docs-llm.a2v10.com/model/popups.md).

## Use When

- A small amount of content is shown over the current page and comes from its own endpoint and model.

## Do Not Use When

- The window needs a title bar, buttons and a result — use [Dialog](https://docs-llm.a2v10.com/xaml/layouts/dialog.md) instead.
- The content is already in the current model and needs no server call — a floating hint is [Popover](https://docs-llm.a2v10.com/xaml/text.md), not a popup view.

## Syntax

```xml
<Popup xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.Xaml" MinWidth="320px">
  <Grid Columns="1*">
    <Text><Span Content="{Bind Agent.Name}" Bold="True" /></Text>
    <Static Label="Tax ID" Value="{Bind Agent.TaxId}" />
  </Grid>
</Popup>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Width` | Length | Window width. Determined by the content when not set |
| `MinWidth` | Length | Minimum window width |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties from `Container` and `UIElement`.

## Notes

- The element carries almost no properties of its own: everything about where the window appears and what data it gets is decided by the `popups` section of `model.json` and by the element that opens it.
- Since `Popup` is a root, a view file contains exactly one of them, and it cannot be placed inside a page.
