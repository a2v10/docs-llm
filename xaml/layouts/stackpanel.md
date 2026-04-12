# StackPanel

> A container that arranges child elements in a single horizontal or vertical line.

## Overview

`StackPanel` arranges its children in a row or column. It inherits from `Container → UIElement → UIElementBase`. Unlike `Grid`, it does not define explicit column/row structure — elements flow naturally in the chosen direction.

A `Separator` element inside a `StackPanel` divides the children into two groups: content before the separator aligns to the start (left/top), and content after aligns to the end (right/bottom). This is a common pattern for splitting button groups.

## Syntax

```xml
<StackPanel Orientation="Horizontal">
  <Button Content="Save" Command="{BindCmd Save}" />
  <Button Content="Cancel" Command="{BindCmd Close}" />
</StackPanel>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Orientation` | Orientation | `Horizontal` or `Vertical` (default) — layout direction |
| `AlignItems` | AlignItems | Alignment of children perpendicular to the flow direction |
| `JustifyItems` | JustifyItems | Distribution of children along the flow direction: `Start`, `End`, `Center`, `SpaceAround`, `SpaceEvenly`, `SpaceBetween` |
| `Inline` | Boolean | Render as inline (non-block) element |
| `Gap` | GapSize | Spacing between children |

See [Base Classes](../base-classes.md) for inherited properties from `Container` and `UIElement`.

## Example

```xml
<!-- Horizontal button row with separator-based alignment -->
<StackPanel Orientation="Horizontal">
  <Button Content="Save" Command="{BindCmd Save}" Style="Primary" Icon="Save" />
  <Button Content="Print" Command="{BindCmd Print}" Icon="Print" />
  <Separator />
  <Button Content="Cancel" Command="{BindCmd Close}" />
</StackPanel>

<!-- Vertical navigation menu -->
<StackPanel Orientation="Vertical" Gap="4">
  <Button Content="Overview" Command="{BindCmd Navigate, Url='/dashboard'}" Style="Toolbar" />
  <Button Content="Reports" Command="{BindCmd Navigate, Url='/reports'}" Style="Toolbar" />
  <Button Content="Settings" Command="{BindCmd Navigate, Url='/settings'}" Style="Toolbar" />
</StackPanel>

<!-- Centered items -->
<StackPanel Orientation="Horizontal" JustifyItems="Center" Gap="8">
  <Button Content="Previous" Command="{BindCmd Move}" Icon="ChevronLeft" />
  <Static Value="{Bind CurrentPage}" Width="60px" Align="Center" />
  <Button Content="Next" Command="{BindCmd Move}" Icon="ChevronRight" />
</StackPanel>

<!-- Inline stack for badges/tags -->
<StackPanel Orientation="Horizontal" Inline="True" Gap="4">
  <Badge Content="{Bind Tag1}" />
  <Badge Content="{Bind Tag2}" />
</StackPanel>
```

## Notes

- `Separator` inside a `StackPanel` splits children into start and end groups — elements before it go left/top, after go right/bottom.
- For two-dimensional layout (rows and columns), use `Grid` instead.
- `Gap` accepts one or two values: `Gap="8"` (uniform) or `Gap="4,8"` (row, column).
- `Inline="True"` makes the StackPanel itself inline — useful for embedding within text flow.
- Unlike `Toolbar`, `StackPanel` has no `Align` attached property — use `Separator` for end-alignment.
