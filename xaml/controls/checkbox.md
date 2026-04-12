# CheckBox

> A checkbox control bound to a boolean value in the model.

## Overview

`CheckBox` renders a checkbox input. It inherits from `CheckBoxBase → ValuedControl → Control → UIElement → UIElementBase`. The checked state is bound to a boolean model property via `Value`.

`CheckBox` has no unique properties beyond what it inherits. The label appears to the right of the checkbox by default.

## Syntax

```xml
<CheckBox Label="Active" Value="{Bind Document.IsActive}" />
```

## Properties

All properties are inherited. The most commonly used:

### From ValuedControl

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | **Binding only.** Bound to a boolean model property. |

### From Control

| Property | Type | Description |
|----------|------|-------------|
| `Label` | String | Text displayed beside the checkbox. Supports `Bind`. |
| `Disabled` | Boolean | Make the checkbox non-interactive. Supports `Bind`. |
| `Required` | Boolean | Visual required indicator. |

See [Base Classes](../base-classes.md) for all inherited properties.

## Example

```xml
<!-- Simple boolean flag -->
<CheckBox Label="Is Active" Value="{Bind Document.IsActive}" />

<!-- In a grid layout -->
<Grid Columns="1*,1*">
  <CheckBox Label="Option 1" Value="{Bind Settings.Option1}" />
  <CheckBox Label="Option 2" Value="{Bind Settings.Option2}" />
  <CheckBox Label="Option 3" Value="{Bind Settings.Option3}" />
  <CheckBox Label="Option 4" Value="{Bind Settings.Option4}" />
</Grid>

<!-- Conditionally disabled -->
<CheckBox Label="Allow Editing"
          Value="{Bind Document.AllowEdit}"
          Disabled="{Bind Document.IsReadOnly}" />

<!-- In a FieldSet -->
<FieldSet Title="Permissions">
  <CheckBox Label="Can View" Value="{Bind Role.CanView}" />
  <CheckBox Label="Can Edit" Value="{Bind Role.CanEdit}" />
  <CheckBox Label="Can Delete" Value="{Bind Role.CanDelete}" />
</FieldSet>
```

## Notes

- `Value` must always be a `Bind` — it connects to a boolean property in the model.
- The `Label` appears to the right of the checkbox (HTML checkbox convention).
- For a toggle/switch style, use `CheckBoxSwitch` which has the same API but renders as a toggle.
- `Disabled="{Bind ...}"` is the standard way to make a checkbox read-only based on a model state.
