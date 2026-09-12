# Radio

> A radio button — several of them bound to one model value, one `CheckedValue` each.

## Overview

`Radio` is a radio button. A set of them is bound to a single value: each button declares the `CheckedValue` that makes it the checked one. Such a set is usually placed inside a [FieldSet](https://docs-llm.a2v10.com/xaml/layouts/fieldset.md), though that is not required.

It inherits from `CheckBoxBase → ValuedControl → Control → UIElement → UIElementBase`.

## Use When

- One value out of a few fixed options has to be visible all at once, with every option readable on screen.

## Do Not Use When

- The options come from data or there are many of them — use [ComboBox](https://docs-llm.a2v10.com/xaml/controls/combobox.md) instead.
- The value is a yes/no flag — use [CheckBox](https://docs-llm.a2v10.com/xaml/controls/checkbox.md) instead.
- The choice switches blocks of the page rather than storing a field — use [TabBar](https://docs-llm.a2v10.com/xaml/layouts/tabbar.md) with [Switch](https://docs-llm.a2v10.com/xaml/switch.md) instead.

## Syntax

```xml
<Radio Label="Option 1" Value="{Bind Element.Option}" CheckedValue="1" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `CheckedValue` | Scalar (String, Number, …) | Required. The value at which this button is checked. May be a binding |
| `Style` | RadioButtonStyle | The look of the button: a standard radio button, or a check box. The check box look is used mostly in printed forms |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for the properties inherited from `ValuedControl` and `Control` — `Value`, `Label`, `Disabled` and the rest.

## Example

All three buttons are bound to the same `Option` field; the one whose `CheckedValue` equals it is checked.

```xml
<Grid Columns="1*, 1*, 1*" AlignItems="Top">
  <FieldSet>
    <Radio Label="Option 1" Value="{Bind Element.Option}" CheckedValue="1" />
    <Radio Label="Option 2" Value="{Bind Element.Option}" CheckedValue="2" />
    <Radio Label="Option 3" Value="{Bind Element.Option}" CheckedValue="3" />
  </FieldSet>
  <FieldSet>
    <Radio Label="Option 1" Value="{Bind Element.Option}" Style="CheckBox" CheckedValue="1" />
    <Radio Label="Option 2" Value="{Bind Element.Option}" Style="CheckBox" CheckedValue="2" />
    <Radio Label="Option 3" Value="{Bind Element.Option}" Style="CheckBox" CheckedValue="3" />
  </FieldSet>
  <Text>Selected: <Span Bold="True">Option 1</Span></Text>
</Grid>
```

## Notes

- `CheckedValue` is mandatory: a radio button without it has nothing to compare against the bound value.
- Grouping is by the binding, not by the container: buttons bound to the same value belong to the same group wherever they are placed.
- `Style="CheckBox"` changes the look only — the behaviour stays exclusive, one choice out of the set.
