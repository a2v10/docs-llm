# Block

> A plain rectangular container — size, border, background, scrolling, and a positioning context for absolute children.

## Overview

`Block` is the general-purpose box: it holds other controls or containers and gives them a size, a border, a background, and optionally a scrollbar. It inherits from `UIElement → UIElementBase`.

It differs from [Panel](https://docs-llm.a2v10.com/xaml/layouts/panel.md) in having no header and no meaning — it is geometry, not a message. By default a block takes the full width of its parent and exactly the height its content needs.

## Use When

- A fixed-height, scrollable area is needed around content.
- Children have to be positioned absolutely — a block with `Relative="True"` is what they are positioned against.
- Content needs a border, a background or a shadow without a header.

## Do Not Use When

- The box carries a meaning and needs a header — use [Panel](https://docs-llm.a2v10.com/xaml/layouts/panel.md) instead.
- Nothing visual is needed and the container only groups elements — use [Group](https://docs-llm.a2v10.com/xaml/layouts/group.md) instead.
- Children have to be laid out in rows and columns — use [Grid](https://docs-llm.a2v10.com/xaml/layouts/grid.md) instead.

## Syntax

```xml
<Block Height="300px" Scroll="True" Border="True">
  <Text><Span Content="{Bind Document.Memo}" /></Text>
</Block>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Children` | UIElementCollection | Content property. The child elements |
| `Align` | TextAlign | Horizontal alignment of the container's content |
| `Color` | TextColor | Text colour |
| `Background` | BackgroundStyle | Background colour |
| `Height` | Length | Block height |
| `Width` | Length | Block width. By default the block takes 100% of the parent's width |
| `MaxWidth` | Length | Maximum width. Wider content wraps |
| `Border` | Boolean | Draw a border around the block |
| `Scroll` | Boolean? | Whether the block scrolls. Meaningful only when a height is set: `True` adds a scrollbar, `False` clips the overflow. Without a height the block grows to fit its content |
| `DropShadow` | ShadowStyle | Shadow effect |
| `Relative` | Boolean | Relative positioning: every child with `Absolute` set is positioned against this block |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties.

## Example

```xml
<!-- a scrollable log area of fixed height -->
<Block Height="240px" Scroll="True" Border="True" Background="White">
  <Repeater ItemsSource="{Bind Document.Log}">
    <Repeater.Content>
      <Text><Span Content="{Bind Message}" /></Text>
    </Repeater.Content>
  </Repeater>
</Block>

<!-- a positioning context for an absolutely placed badge -->
<Block Relative="True" Height="120px">
  <Image Source="{Bind Product.Photo}" />
  <Badge Content="{Bind Product.StateName}" Absolute="8,8,0,0" />
</Block>
```

## Notes

- `Scroll` without `Height` does nothing: there is no overflow to scroll when the block sizes itself to its content.
- `Scroll="False"` is not the same as leaving it unset — it clips content that does not fit, where the default would have grown the block.
- `Relative` matters only for children that set `Absolute`; on its own it changes nothing visible.
