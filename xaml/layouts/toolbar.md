# Toolbar

> A horizontal panel for action buttons, typically placed at the top of a Page or Dialog.

## Overview

`Toolbar` renders a horizontal bar of controls. It inherits from `Container → UIElement → UIElementBase`. It typically contains `Button` elements, `Separator` elements, and occasionally `TextBox` or `ComboBox` for inline filters.

The `Toolbar.Align="Right"` attached property splits toolbar items into a left group and a right group — elements before a right-aligned item are in the left group; elements with `Toolbar.Align="Right"` appear in a right-aligned group.

## Syntax

```xml
<Toolbar>
  <Button Content="Save" Command="{BindCmd Save}" Icon="Save" Style="Primary" />
  <Button Content="Reload" Command="{BindCmd Reload}" Icon="Reload" />
  <Separator />
  <Button Content="Help" Icon="Help" Toolbar.Align="Right" />
</Toolbar>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Style` | ToolbarStyle | `Default`, `Transparent`, `Light` — appearance depends on UI theme |
| `Border` | ToolbarBorderStyle | `None` (default), `Bottom`, `BottomShadow` |
| `AlignItems` | AlignItems | Vertical alignment of items within the toolbar |
| `Orientation` | Orientation | `Horizontal` (default) or `Vertical` |

### Attached Properties

| Property | Type | Description |
|----------|------|-------------|
| `Toolbar.Align` | ToolbarAlign | `Left` (default) or `Right` — places item in the right group |

See [Base Classes](../base-classes.md) for inherited properties from `Container` and `UIElement`.

## Example

```xml
<!-- Page toolbar -->
<Page.Toolbar>
  <Toolbar>
    <Button Content="Create"
            Command="{BindCmd Dialog, Action=New, Argument={Bind NewDoc}}"
            Icon="Plus" Style="Primary" />
    <Button Content="Edit"
            Command="{BindCmd Dialog, Action=Edit, Argument={Bind Docs.Selected}}"
            Icon="Edit"
            Disabled="{Bind !Docs.HasSelected}" />
    <Button Content="Delete"
            Command="{BindCmd DbRemoveSelected, Argument={Bind Docs.Selected},
                               Confirm={Confirm Message='Delete selected records?'}}"
            Icon="Delete" Style="Danger"
            Disabled="{Bind !Docs.HasSelected}" />
    <Separator />
    <Button Content="Reload" Command="{BindCmd Reload}" Icon="Reload" />
    <Button Content="Export"
            Command="{BindCmd Report, Export=True}"
            Icon="Export"
            Toolbar.Align="Right" />
  </Toolbar>
</Page.Toolbar>

<!-- Dialog toolbar (less common, usually just buttons) -->
<Dialog.Buttons>
  <Button Content="Save" Command="{BindCmd SaveAndClose}" Style="Primary" Icon="Save" />
  <Button Content="Cancel" Command="{BindCmd Close}" />
</Dialog.Buttons>

<!-- Toolbar inside a card with filter controls -->
<Toolbar>
  <ComboBox Value="{Bind Filter.Status}"
            ItemsSource="{Bind Statuses}"
            DisplayProperty="Name"
            Placeholder="All statuses" />
  <TextBox Value="{Bind Filter.Search}"
           Placeholder="Search..."
           ShowClear="True"
           EnterCommand="{BindCmd Reload}" />
  <Button Content="Search" Command="{BindCmd Reload}" Icon="Search" Style="Primary" />
</Toolbar>
```

## Notes

- `Toolbar` is typically used as a value for `Page.Toolbar` or `Dialog.Taskpad` — it can also appear inline inside other containers.
- `Separator` creates a visual divider between button groups.
- `Toolbar.Align="Right"` on any element shifts it (and subsequent right-aligned elements) to the right side.
- Buttons in toolbars often use `Disabled="{Bind !Items.HasSelected}"` to disable actions when nothing is selected.
- For button-only toolbars, prefer `Style="Toolbar"` on individual `Button` elements over `Style="Primary"` for visual consistency.
