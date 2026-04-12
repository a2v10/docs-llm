# Button

> A command button element that triggers a BindCmd action when clicked.

## Overview

`Button` renders a clickable button. It inherits from `CommandControl → ContentControl → Control → UIElement → UIElementBase`, which means it supports a `Command` binding, displays `Content` as its label, and inherits all standard control properties (disabled, label, width, etc.).

The button's visual appearance is controlled by `Style` and `Size`. An optional `Icon` can be added to the left or top of the content. A `DropDown` can attach a dropdown menu to the button.

## Syntax

```xml
<Button Content="Label"
        Command="{BindCmd CommandType}"
        Style="Primary"
        Icon="Save"
        Size="Normal" />
```

## Properties

### Button-Specific

| Property | Type | Description |
|----------|------|-------------|
| `Style` | ButtonStyle | Visual style variant (see values below). If bound, use lowercase value string. |
| `Size` | ControlSize | Button size: `Default`, `Normal`, `Large`, `Small`, `Mini` |
| `Icon` | Icon | Icon displayed on the button (300+ icon constants available) |
| `IconAlign` | IconAlign | Icon placement: `Default`/`Left` (beside content), `Top` (above content) |
| `DropDown` | UIElementBase | Dropdown element attached to the button, typically a `DropDownMenu` |
| `Rounded` | Boolean | Rounded corners. If only an icon is shown (no content), the button becomes circular. |
| `Badge` | String | Badge text displayed on the button (supports `Bind`) |

### From CommandControl

| Property | Type | Description |
|----------|------|-------------|
| `Command` | BindCmd | Command executed on click |
| `Content` | Object | Button label text. Supports `Bind`. |

See [Base Classes](../base-classes.md) for properties inherited from `Control`, `UIElement`, and `UIElementBase`.

### ButtonStyle Values

| Value | Description |
|-------|-------------|
| `Default` | Default appearance (theme-defined) |
| `Primary` | Primary action — typically blue |
| `Danger` / `Red` / `Error` | Destructive action — red |
| `Warning` / `Orange` | Warning action — orange |
| `Success` / `Green` | Positive action — green |
| `Info` / `Cyan` | Informational — cyan |
| `Outline` | Outlined, no fill |
| `Toolbar` | Minimal style for toolbar placement |

## Example

```xml
<!-- Toolbar with common actions -->
<Toolbar>
  <Button Content="Save" Command="{BindCmd Save}" Icon="Save" Style="Primary" />
  <Button Content="Reload" Command="{BindCmd Reload}" Icon="Reload" />
  <Separator />
  <Button Content="Delete"
          Command="{BindCmd DbRemove, Argument={Bind Document},
                             Confirm={Confirm Message='Delete this record?'}}"
          Style="Danger"
          Icon="Delete"
          Toolbar.Align="Right" />
</Toolbar>

<!-- Icon-only circular button -->
<Button Command="{BindCmd Close}" Icon="Close" Rounded="True" />

<!-- Button with dropdown -->
<Button Content="Actions" Icon="Menu">
  <Button.DropDown>
    <DropDownMenu>
      <MenuItem Content="Export to Excel"
                Command="{BindCmd Report, Export=True}" />
      <MenuItem Content="Print"
                Command="{BindCmd Report, Print=True}" />
    </DropDownMenu>
  </Button.DropDown>
</Button>

<!-- Open edit dialog -->
<Button Content="Edit"
        Command="{BindCmd Dialog, Action=Edit, Argument={Bind Row}}"
        Icon="Edit"
        Disabled="{Bind !Row.Id}" />
```

## Notes

- `Style` can be data-bound, but the binding value must be a lowercase string (e.g., `"primary"`, not `"Primary"`).
- When `DropDown` is set, the button renders with a split caret that opens the menu independently.
- `Icon` values are defined in the `Icon` enum in `A2v10.ViewEngine.Xaml` — over 300 named icons are available.
- `Disabled` is inherited from `Control` and accepts `Bind` for dynamic disabling.
- To conditionally hide a button entirely, use `If="{Bind condition}"` rather than `Disabled`.
