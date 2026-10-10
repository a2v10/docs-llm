# ColorPicker

> A drop-down for choosing a colour name, and ColorComboBox — a drop-down list whose items are shown as coloured labels.

## Overview

`ColorPicker` lets the user pick one of a fixed set of colours. The list drops down as coloured labels; the value is the colour name, a plain string such as `green` or `salmon`, stored in the bound property. It suits entities that carry a colour of their own — a category, a state, a tag.

`ColorComboBox` is the counterpart for choosing an item, not a colour: a drop-down list built from a model array, where each item is drawn as a label in the item's own colour. The colour names are the same as in `ColorPicker`, so a colour chosen with `ColorPicker` on a catalog form is shown by `ColorComboBox` wherever that catalog is picked from.

Both inherit `ValuedControl → Control → UIElement → UIElementBase`. In both, the arrow keys ↑ and ↓ step through the items without opening the list.

## Syntax

```xml
<ColorPicker Label="Colour" Value="{Bind Category.Color}" />

<ColorComboBox Label="State" Value="{Bind Document.State}" ItemsSource="{Bind States}">
  <ColorComboBoxItem Content="{Bind Name}" Value="{Bind Id}" Color="{Bind Color}" />
</ColorComboBox>
```

## Properties

### ColorPicker

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Bind | The name of the chosen colour (a string) — see [Colour names](#colour-names) |
| `Text` | String | Text shown on the labels |
| `Compact` | Boolean | Compact appearance |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for `Label`, `Required`, `Disabled`, and other inherited properties.

### Colour names

`default`, `green`, `orange`, `cyan`, `red`, `purple`, `pink`, `gold`, `blue`, `salmon`, `seagreen`, `tan`, `magenta`, `lightgray`, `olive`, `teal`.

## ColorComboBox

Content property: `Children`.

| Property | Type | Description |
|----------|------|-------------|
| `ItemsSource` | Array | **Bind only.** The model array the list is built from |
| `Children` | List\<ColorComboBoxItem\> | **Content property.** Exactly one `ColorComboBoxItem` that maps array item properties to the label text, value and colour |
| `Align` | TextAlign | Text alignment |
| `DropUp` | Boolean | Open the list upwards |
| `Size` | ControlSize | `Default` — normal size; `Large` — enlarged list |

### ColorComboBoxItem

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Bind | **Required.** Array item property shown as the label text |
| `Value` | Bind | **Required.** Array item property that becomes the value of the list |
| `Color` | Bind | **Required.** Array item property holding the label colour — one of the [colour names](#colour-names) |
| `Outline` | Boolean | Draw the label outlines only |

If the list's `Value` is bound to an object, selecting an item assigns the whole array item to it. Otherwise it is assigned the item property named by `ColorComboBoxItem.Value`.

## Example

A category catalog (`cat.AgentCategories`, with a `Color nvarchar(32)` column) stores its colour; the colour is chosen on the category form.

```xml
<Dialog Title="Category">
  <Grid>
    <TextBox Label="Name" Value="{Bind Category.Name}" Required="True" />
    <ColorPicker Label="Colour" Value="{Bind Category.Color}" />
  </Grid>
</Dialog>
```

The agent form picks a category; each item is shown in the category's colour, and the selected category object goes into `Agent.Category`.

```sql
select [Categories!TCategory!Array] = null, [Id!!Id] = c.Id, [Name!!Name] = c.[Name], c.Color
from cat.AgentCategories c
order by c.[Name];
```

```xml
<ColorComboBox Label="Category" Value="{Bind Agent.Category}" ItemsSource="{Bind Categories}">
  <ColorComboBoxItem Content="{Bind Name}" Value="{Bind Id}" Color="{Bind Color}" />
</ColorComboBox>
```

## Notes

- The colour is stored as its name, not as a CSS colour: a `nvarchar` column holding `green`, `salmon`, and so on.
- `ColorComboBoxItem` is a mapping, not a list item: write exactly one, with all three bindings — `Content`, `Value`, `Color`.
- Binding `Value` of `ColorComboBox` to an object property (a reference) stores the whole selected item; binding it to a scalar stores the property named in `ColorComboBoxItem.Value`.
