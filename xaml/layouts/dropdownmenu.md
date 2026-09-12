# DropDownMenu

> The dropdown of a button — a menu of MenuItem elements, and the DropDownMegaMenu variant with sections and columns.

## Overview

`DropDownMenu` is the menu that drops out of a button: it is placed in the `DropDown` property of a [Button](https://docs-llm.a2v10.com/xaml/controls/button.md), and its content is normally a collection of `MenuItem` elements.

`DropDownMegaMenu` is the same menu with sections: items are grouped by a field of the data source and can be laid out in several columns. Grouping works for bound data sources only — static menu items cannot be grouped.

Both inherit from `Container → UIElement → UIElementBase`.

## Syntax

```xml
<Toolbar>
  <Button Content="Open" Icon="File">
    <Button.DropDown>
      <DropDownMenu>
        <MenuItem Content="Text" Icon="FileContent" />
        <MenuItem Content="Image" Icon="FileImage" />
      </DropDownMenu>
    </Button.DropDown>
  </Button>
  <Button Icon="Save" Content="Save" />
</Toolbar>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Direction` | DropDownDirection | Which way the menu opens, see below |
| `Separate` | Boolean | Show the menu detached from its parent button; the background of the element is then white |
| `Background` | BackgroundStyle | Background colour |

### DropDownMegaMenu

Adds to the same set:

| Property | Type | Description |
|----------|------|-------------|
| `GroupBy` | String | The name of the field in the data source to group by. Each group becomes its own section |
| `Columns` | Int32 | Number of columns in the menu. One by default |
| `Width` | Length | Menu width |

### DropDownDirection Values

| Value | Meaning |
|-------|---------|
| `DownRight` | Down, expanding to the right (default). Anchored by its left edge |
| `DownLeft` | Down, expanding to the left. Anchored by its right edge |
| `UpRight` | Up, expanding to the right. Anchored by its left edge |
| `UpLeft` | Up, expanding to the left. Anchored by its right edge |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties from `Container` and `UIElement`.

## Example

```xml
<!-- static menu, opening upwards from a bottom toolbar -->
<Button Content="Actions" Icon="Menu">
  <Button.DropDown>
    <DropDownMenu Direction="UpRight">
      <MenuItem Content="Post" Icon="Apply"
                Command="{BindCmd Execute, CommandName=Apply, Argument={Bind Document}}" />
      <MenuItem Content="Unpost" Icon="Undo"
                Command="{BindCmd Execute, CommandName=Unapply, Argument={Bind Document}}" />
      <MenuItem Content="Print" Icon="Print"
                Command="{BindCmd Report, Report=invoice, Url='/document/print',
                                   Argument={Bind Document}, Print=True}" />
    </DropDownMenu>
  </Button.DropDown>
</Button>

<!-- bound menu grouped into sections, two columns wide -->
<Button Content="Reports" Icon="Report">
  <Button.DropDown>
    <DropDownMegaMenu ItemsSource="{Bind ReportList}" GroupBy="Group"
                      Columns="2" Width="420px" Separate="True">
      <MenuItem Content="{Bind Name}"
                Command="{BindCmd Open, Argument={Bind}}" />
    </DropDownMegaMenu>
  </Button.DropDown>
</Button>
```

## Notes

- Direction is about anchoring as much as about direction: `DownLeft` pins the menu's right edge to the button, which is what keeps a menu at the right edge of a toolbar from running off the page.
- `GroupBy` names a field of the items, so the grouping order is the order the rows arrive in from the server — sort the dataset if the sections must come in a particular order.
- A menu of `MenuItem` elements with commands is the usual content, but the container accepts any elements.
