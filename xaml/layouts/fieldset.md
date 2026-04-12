# FieldSet

> A labeled frame container for grouping related form controls.

## Overview

`FieldSet` renders a bordered frame with an optional title and groups the controls inside it visually. It inherits from `Container → UIElement → UIElementBase`. It is the standard way to create labeled sections in forms.

Children are laid out using the `Orientation` property — either stacked vertically (default) or horizontally.

## Syntax

```xml
<FieldSet Title="Contact Information">
  <TextBox Label="Phone" Value="{Bind Contact.Phone}" />
  <TextBox Label="Email" Value="{Bind Contact.Email}" />
</FieldSet>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Title` | String | Frame title (legend text). Supports `Bind`. |
| `Orientation` | Orientation | `Vertical` (default) or `Horizontal` — layout direction of children |
| `Hint` | Popover | Help tooltip shown as `?` icon after the title |
| `Disabled` | Boolean | Disables all control elements inside the fieldset. Supports `Bind`. |

See [Base Classes](../base-classes.md) for inherited properties from `Container` and `UIElement`.

## Example

```xml
<!-- Grouped form sections -->
<Grid Columns="1*,1*">
  <FieldSet Title="Basic Information">
    <TextBox Label="Name" Value="{Bind Item.Name}" Required="True" />
    <TextBox Label="Code" Value="{Bind Item.Code}" />
    <DatePicker Label="Created" Value="{Bind Item.CreatedAt}" />
  </FieldSet>

  <FieldSet Title="Classification">
    <ComboBox Label="Category"
              Value="{Bind Item.Category}"
              ItemsSource="{Bind Categories}"
              DisplayProperty="Name" />
    <CheckBox Label="Active" Value="{Bind Item.IsActive}" />
    <CheckBox Label="Featured" Value="{Bind Item.IsFeatured}" />
  </FieldSet>
</Grid>

<!-- Checkboxes in a fieldset -->
<FieldSet Title="Permissions">
  <Grid Columns="1*,1*">
    <CheckBox Label="Can View" Value="{Bind Role.CanView}" />
    <CheckBox Label="Can Edit" Value="{Bind Role.CanEdit}" />
    <CheckBox Label="Can Delete" Value="{Bind Role.CanDelete}" />
    <CheckBox Label="Can Create" Value="{Bind Role.CanCreate}" />
  </Grid>
</FieldSet>

<!-- Conditionally disabled fieldset -->
<FieldSet Title="Approval Details"
          Disabled="{Bind !Document.IsApproved}">
  <DatePicker Label="Approved At" Value="{Bind Document.ApprovedAt}" />
  <Selector Label="Approved By" Value="{Bind Document.ApprovedBy}"
            DisplayProperty="Name" ItemsSource="{Bind Users}" />
</FieldSet>
```

## Notes

- `FieldSet` renders as an HTML `<fieldset>` with a `<legend>` for the title.
- `Disabled="True"` on a `FieldSet` disables all nested controls at once — useful for conditionally locking a section.
- Unlike `Grid`, `FieldSet` includes a visible border and title — use it for semantic grouping, not pure layout.
- Nesting `Grid` inside `FieldSet` is the standard pattern for multi-column sections within a group.
- `Orientation="Horizontal"` stacks children side by side — useful for compact checkbox or radio button rows.
