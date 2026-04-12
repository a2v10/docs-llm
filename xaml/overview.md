# XAML Overview

> A2v10's UI markup language — XML-based view definitions that the platform compiles into Vue.js components.

## Overview

A2v10 views are written in a custom XAML dialect — XML files with `.xaml` extension. Each file defines either a **Page** (standalone route with a URL) or a **Dialog** (modal opened by a command). The platform parses XAML server-side and renders it into an HTML/Vue.js component tree; developers never write Vue templates by hand.

A XAML file always has a single root element — either `<Page>` or `<Dialog>`. All other elements are nested inside it. Elements map directly to C# classes in `A2v10.ViewEngine.Xaml`, and every XML attribute corresponds to a class property.

Property values are either **literal strings** (e.g., `Label="Name"`) or **markup extensions** enclosed in curly braces (e.g., `Value="{Bind Document.Name}"`). The two built-in markup extensions are `Bind` (data binding) and `BindCmd` (command binding).

XAML views are model-aware: they bind to the data model loaded by `model.json`. Bindings are reactive — when the model changes, the UI updates automatically, and vice versa.

## Syntax

```xml
<!-- Page root -->
<Page
    xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
    Title="My Page">
  <Page.Toolbar>
    <Toolbar>
      <Button Content="Save" Command="{BindCmd Save}" Icon="Save" />
    </Toolbar>
  </Page.Toolbar>
  <Grid Columns="1*,1*">
    <TextBox Label="Name" Value="{Bind Document.Name}" />
    <DatePicker Label="Date" Value="{Bind Document.Date}" />
  </Grid>
</Page>
```

```xml
<!-- Dialog root -->
<Dialog
    xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
    Title="Edit Item" Size="Medium">
  <Grid Columns="1*">
    <TextBox Label="Name" Value="{Bind Element.Name}" Required="True" />
  </Grid>
  <Dialog.Buttons>
    <Button Content="OK" Command="{BindCmd SaveAndClose}" Style="Primary" />
    <Button Content="Cancel" Command="{BindCmd Close}" />
  </Dialog.Buttons>
</Dialog>
```

## Markup Extensions

Curly-brace syntax `{ExtensionName prop=value, ...}` is used inline in attribute values.

| Extension | Purpose | Example |
|-----------|---------|---------|
| `Bind` | Bind to model data (reactive, bidirectional) | `Value="{Bind Doc.Name}"` |
| `BindCmd` | Bind to a command | `Command="{BindCmd Save}"` |
| `StaticResource` | Reference a named resource | `Style="{StaticResource myStyle}"` |

To use a literal `{` character in an attribute value, prefix with `{}`:

```xml
Label="{}{value}"   <!-- renders as: {value} -->
```

## Property Syntax

**Simple properties** — literal values:
```xml
<TextBox Label="Name" Multiline="True" Rows="3" />
```

**Markup extensions** — dynamic bindings:
```xml
<TextBox Value="{Bind Document.Name}" Disabled="{Bind Document.IsReadOnly}" />
```

**Complex/nested properties** — property element syntax:
```xml
<Button Content="Open">
  <Button.DropDown>
    <DropDownMenu>
      <MenuItem Content="Option 1" Command="{BindCmd Execute, CommandName=cmd1}" />
    </DropDownMenu>
  </Button.DropDown>
</Button>
```

## Element Inheritance

All XAML elements inherit from a base class chain. Understanding the chain tells you which properties are available on any element:

```
UIElementBase
  └── UIElement
        ├── Container (layout containers with Children)
        │     └── RootContainer → Page, Dialog
        └── Control (interactive controls)
              ├── ValuedControl (controls with Value binding)
              │     ├── TextBox, DatePicker, ComboBox, Selector, CheckBox ...
              └── ContentControl → CommandControl → Button, MenuItem
```

See [Base Classes](base-classes.md) for all inherited properties.

## Notes

- The `xmlns` declaration is required on the root element.
- Binding paths are relative to the model root unless inside a `Repeater` or `DataGrid`, where they are relative to the current item.
- Boolean properties accept `True`/`False` (case-insensitive).
- `Size` (control size) and `Style` (visual variant) are distinct properties — do not confuse them.
- `If` removes the element from the DOM; `Show`/`Hide` toggle CSS visibility only.
