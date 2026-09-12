# List

> A selectable list of items — static or bound, with styles from a plain column to a chat, row marking and an empty-state panel.

## Overview

`List` displays a collection of items and lets the user select one. Its content is normally a set of `ListItem` elements; with `ItemsSource` the single `ListItem` inside it becomes the template repeated for every row of the collection.

It inherits from `Control → UIElement → UIElementBase`. Content property: `Content`.

Compared with [DataGrid](https://docs-llm.a2v10.com/xaml/controls/datagrid.md), a list has no columns: each item is a block of markup, which is what makes it the right choice for cards, feeds and anything where a row is not a tuple of fields.

## Use When

- An item is a block — a title, a couple of lines, an icon, a command bar — rather than a row of columns.
- The collection is shown as cards in two or three columns, as a chat, or as an underlined list.
- Selection matters but a tabular presentation does not.

## Do Not Use When

- Items are uniform records compared field by field, or need sorting and paging — use [DataGrid](https://docs-llm.a2v10.com/xaml/controls/datagrid.md) instead.
- The collection is a hierarchy — use a tree control instead.
- Nothing is selectable and the markup merely repeats — use [Repeater](https://docs-llm.a2v10.com/xaml/layouts/repeater.md) instead.

## Syntax

```xml
<List ItemsSource="{Bind Agents}" Style="Underlined" Border="True">
  <ListItem Header="{Bind Name}" Content="{Bind Memo}" Icon="User"
            Command="{BindCmd Open, Argument={Bind}}" />
</List>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Content` | UIElementCollection | Content property. The content of the list |
| `ItemsSource` | Array | Binding only. The data source displayed in the list |
| `Style` | ListStyle | Display style, see below |
| `Background` | BackgroundStyle | Background style for the list |
| `Border` | Boolean | Draw a border around the list |
| `BorderStyle` | BorderStyle | Which borders to draw: `None` (default), `Top`, `TopBottom`, `All` |
| `Striped` | Boolean | Alternate the background of odd and even items |
| `Flush` | Boolean | Remove the line after the last item — `Underlined` style only |
| `AutoSelect` | AutoSelectMode | Select an item automatically after the list is loaded or refreshed: `None` (default), `FirstItem`, `LastItem`, `ItemId` (the identifier is taken from the page URL) |
| `Height` | Length | List height |
| `MaxHeight` | Length | Maximum height. A taller list gets a vertical scrollbar |
| `Select` | Boolean? | Whether items can be selected |
| `Mark` | Object | Binding only. Marking of a list item — a vertical bar at its left, see below |
| `MarkerStyle` | RowMarkerStyle | How a marked item is shown: `None` (default), `Row` — background colour, `Marker` — a bar at the left, `Both` |
| `EmptyPanel` | UIElement | A panel shown when the content is empty, usually an `EmptyPanel` element |
| `DoubleClick` | BindCmd | Command executed on a double click on an item. Remember to pass an argument |

### ListStyle Values

| Value | Meaning |
|-------|---------|
| `List` | Default. Items are placed top to bottom |
| `TwoColumnsGrid` | Items are placed in a table of two equal columns |
| `ThreeColumnsGrid` | Items are placed in a table of three equal columns |
| `Underlined` | Items top to bottom with a horizontal line after each |
| `Chat` | Items are laid out as a chat; the items are then usually `ChatItem` elements |

### ListItem

One item of the list. Inherits `UIElement → UIElementBase`. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Object | Content property. The text of the item, or a control |
| `Header` | Object | The header of the item: text or a control |
| `Footer` | Object | The footer of the item: text or a control |
| `Icon` | Icon | Icon displayed in the item |
| `CommandBar` | [CommandBar](https://docs-llm.a2v10.com/xaml/layouts/commandbar.md) | Command bar attached to the item |
| `Command` | BindCmd | Command executed when the item is clicked |

### Item marking

The binding expression of `Mark` must return a string, which is added to the item as an extra CSS class. The default theme supports `"danger"`, `"red"`, `"error"`; `"warning"`, `"yellow"`; `"success"`, `"green"`; `"info"`, `"cyan"`.

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties.

## Example

```xml
<!-- cards in two columns, marked by state, with row commands -->
<List ItemsSource="{Bind Documents}" Style="TwoColumnsGrid"
      Mark="{Bind StateColor}" MarkerStyle="Both"
      AutoSelect="FirstItem" MaxHeight="600px"
      DoubleClick="{BindCmd Open, Argument={Bind Documents.Selected}}">
  <ListItem Icon="File" Header="{Bind Name}" Content="{Bind Memo}" Footer="{Bind Date}">
    <ListItem.CommandBar>
      <CommandBar Visibility="Hover" Float="Right">
        <Button Icon="Edit" Command="{BindCmd Dialog, Action=Edit, Argument={Bind}}" Style="Toolbar" />
      </CommandBar>
    </ListItem.CommandBar>
  </ListItem>
</List>

<!-- a static list of links, no selection -->
<List Style="Underlined" Flush="True" Select="False">
  <ListItem Content="Agents" Icon="Users" Command="{BindCmd Navigate, Url='/catalog/agent'}" />
  <ListItem Content="Documents" Icon="File" Command="{BindCmd Navigate, Url='/doc/index'}" />
</List>
```

## Notes

- `Command` on a `ListItem` and a `CommandBar` inside it do not combine: with a command on the item, the buttons in the command bar cannot be clicked.
- `Flush` only has an effect with `Style="Underlined"` — there is no trailing line in the other styles to remove.
- `Mark` alone changes nothing visible: `MarkerStyle` decides whether the returned class paints the background, the left bar, or both.
- Since `Mark` returns a CSS class, it is not limited to the theme's names — `"danger bold"` marks the item red and bold.
- `MaxHeight` is what makes a list scroll; `Height` fixes it whether the content fills it or not.
