# Panel

> A framed container with a header, a colour for its purpose, and optional collapsing.

## Overview

`Panel` is a visible box: a header, a body, and a style that says what the box means — error, warning, success, information. It inherits from `Container → UIElement → UIElementBase`.

Unlike [FieldSet](https://docs-llm.a2v10.com/xaml/layouts/fieldset.md), which frames form fields under a title, `Panel` frames anything and can be collapsed by the user.

## Use When

- A block of content needs a header and a visible frame.
- The colour of the frame carries meaning — a red panel for errors, a yellow one for warnings.
- The user should be able to fold the block away.

## Do Not Use When

- The block is a group of form controls under a caption — use [FieldSet](https://docs-llm.a2v10.com/xaml/layouts/fieldset.md) instead.
- Nothing needs to be visible and the grouping exists only to toggle several elements at once — use [Group](https://docs-llm.a2v10.com/xaml/layouts/group.md) instead.

## Syntax

```xml
<Panel Header="Warning" Style="Warning" Collapsible="True">
  <Text>The document has unposted rows.</Text>
</Panel>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Style` | PaneStyle | Display style, see below |
| `Compact` | Boolean | Compact display |
| `Collapsible` | Boolean | Whether the panel can be collapsed |
| `Collapsed` | Boolean? | Whether the panel starts collapsed |
| `Header` | Object | The header: a string, a `Bind`, or a `UIElementBase` element |
| `Icon` | Icon | Icon shown at the left of the header |
| `Hint` | [Popover](https://docs-llm.a2v10.com/xaml/text.md) | Floating tooltip, rendered as a help icon after the header text |
| `Height` | Length | Panel height |
| `DropShadow` | ShadowStyle | Shadow effect |
| `Background` | BackgroundStyle | Background colour |
| `Gap` | GapSize | Spacing between children along both axes |
| `TestId` | String | Identifier for test automation tools |

### PaneStyle Values

| Value | Meaning |
|-------|---------|
| `Default` | An ordinary panel |
| `Danger`, `Error`, `Red` | An error — red |
| `Warning`, `Yellow` | A warning — yellow |
| `Success`, `Green` | Success — green |
| `Info`, `Cyan` | Information — cyan |
| `Cut` | A collapsible panel whose header is rendered as a link |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties from `Container` and `UIElement`.

## Example

```xml
<!-- a coloured panel driven by model state -->
<Panel Header="{Bind Document.StateName}" Style="Danger" If="{Bind Document.HasErrors}">
  <Text><Span Content="{Bind Document.ErrorText}" /></Text>
</Panel>

<!-- a collapsible block of secondary information, folded away by default -->
<Panel Header="History" Icon="Time" Collapsible="True" Collapsed="True" Compact="True">
  <DataGrid ItemsSource="{Bind Document.History}">
    <DataGridColumn Header="Date" Content="{Bind Date, DataType=DateTime}" />
    <DataGridColumn Header="User" Content="{Bind User.Name}" />
  </DataGrid>
</Panel>

<!-- Cut style: the header is a link, the body folds under it -->
<Panel Header="Advanced settings" Style="Cut" Collapsible="True" Collapsed="True">
  <Grid Columns="1*,1*">
    <CheckBox Label="Skip validation" Value="{Bind Options.SkipValidation}" />
    <CheckBox Label="Verbose log" Value="{Bind Options.VerboseLog}" />
  </Grid>
</Panel>
```

## Notes

- `Collapsed` only has an effect together with `Collapsible` — it is the initial state of a panel the user can fold.
- `Header` accepts an element, not just a string: a header with a badge or an icon inside is written as complex property syntax (`<Panel.Header>`).
- The colour styles come in pairs of names (`Danger` and `Red`, `Info` and `Cyan`) that mean the same thing; prefer the meaning over the colour so a theme change does not make the markup read wrong.
