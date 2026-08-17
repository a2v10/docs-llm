# ToolbarAligner

> An empty element that absorbs all free space in a horizontal container, pushing everything after it to the opposite edge.

## Overview

`ToolbarAligner` renders nothing. Its only job is to take up all remaining space in its container, so that every element placed after it in the markup is pushed to the far edge. It is the XAML equivalent of `flex-grow: 1` in CSS flexbox.

It works in any container that lays elements out horizontally — [Toolbar](https://docs-llm.a2v10.com/xaml/layouts/toolbar.md), [StackPanel](https://docs-llm.a2v10.com/xaml/layouts/stackpanel.md), and others.

Unlike `Separator`, which draws a visible vertical divider line, `ToolbarAligner` produces no visual output at all — it only distributes space.

It inherits from `UIElementBase` and has no properties of its own.

## Use When

- Several elements at once must be pushed to the right edge of a toolbar.
- The split point between the left and right groups should be visible explicitly in the markup.

## Do Not Use When

- You need a visible divider between button groups — use `Separator` instead.
- Only one trailing element must be right-aligned — the attached property `Toolbar.Align="Right"` on that element is enough (see [Toolbar](https://docs-llm.a2v10.com/xaml/layouts/toolbar.md)).

## Syntax

```xml
<Toolbar>
  <Button Content="Open" Icon="File" />
  <ToolbarAligner />
  <Button Content="Help" Icon="Help" />
</Toolbar>
```

## Properties

The element has no properties of its own.

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for the properties inherited from `UIElementBase`.

## Example

Two buttons on the left, one pushed to the right edge:

```xml
<Toolbar>
  <Button Content="Open" Icon="File" />
  <Button Content="Save" Icon="Save" />
  <ToolbarAligner />
  <Button Content="Help" Icon="Help" />
</Toolbar>
```

A whole group pushed to the right — this is where `ToolbarAligner` is clearer than repeating the attached property on every element:

```xml
<Page.Toolbar>
  <Toolbar>
    <Button Content="Create"
            Command="{BindCmd Dialog, Action=New, Argument={Bind NewAgent}}"
            Icon="Plus" Style="Primary" />
    <Button Content="Edit"
            Command="{BindCmd Dialog, Action=Edit, Argument={Bind Agents.Selected}}"
            Icon="Edit"
            Disabled="{Bind !Agents.HasSelected}" />
    <ToolbarAligner />
    <Button Content="Reload" Command="{BindCmd Reload}" Icon="Reload" />
    <Button Content="Export" Command="{BindCmd Report, Export=True}" Icon="Export" />
  </Toolbar>
</Page.Toolbar>
```

## Notes

- The same result for a single element can be achieved with `Toolbar.Align="Right"` on that element.
- The element occupies space but paints nothing: no border, no background, no divider line.
