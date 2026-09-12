# MenuItem

> One item of a menu — content, an icon, a command, and an attached separator.

## Overview

`MenuItem` is a menu entry. It normally lives inside a [DropDownMenu](https://docs-llm.a2v10.com/xaml/layouts/dropdownmenu.md), which in turn is the `DropDown` of a [Button](https://docs-llm.a2v10.com/xaml/controls/button.md).

It inherits from `CommandControl → ContentControl → Control → UIElement → UIElementBase`, so its content and its command come from those bases. Content property: `Content`.

## Syntax

```xml
<MenuItem Content="Save" Icon="Save" Command="{BindCmd Save}" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Icon` | Icon | Icon of the item |
| `Separator` | SeparatorMode | An attached separator: `None` (default), `Before` — add a separator before the item, `After` — after it |

`Content` comes from `ContentControl` and `Command` from `CommandControl` — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).

## Example

```xml
<Button Content="File" Style="Primary">
  <Button.DropDown>
    <DropDownMenu Direction="DownRight" Separate="True">
      <MenuItem Content="Open" Icon="File" Command="{BindCmd Open, Argument={Bind Document}}" />
      <MenuItem Content="Save" Icon="Save" Command="{BindCmd Save}" />
      <MenuItem Content="Exit" Icon="Exit" Separator="Before" Command="{BindCmd Close}" />
    </DropDownMenu>
  </Button.DropDown>
</Button>
```

## Notes

- The separator belongs to the item, not to the menu: a group of entries is separated by putting `Separator="Before"` on the first item of the next group, so adding or hiding an item keeps the grouping correct.
- A menu item with no command renders but does nothing; for a label-only entry that is intentional, otherwise it is a missing `Command`.
- `If` and `Show` work as on any element, which is how menus are trimmed by permission or state.
