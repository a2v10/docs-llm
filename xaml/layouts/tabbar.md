# TabBar

> A bar of buttons bound to one value — the pressed button sets the value, the value selects the button.

## Overview

`TabBar` is a container of `TabButton` elements whose selection is bound to a model value. Pressing a button writes its `ActiveValue` into `Value`; changing `Value` from anywhere selects the matching button. The bar itself only selects — it holds no content.

To show a different block for each selection, pair it with [Switch](https://docs-llm.a2v10.com/xaml/switch.md) bound to the same value.

`TabBar` inherits from `UIElement → UIElementBase`. Content property: `Buttons`.

## Use When

- The chosen variant must live in the model — it has to be saved, validated, sent to the server, or read by several parts of the view.
- The same value drives more than one thing on the page.
- The bar is a main menu, a wizard step indicator or a button group — those are its styles.

## Do Not Use When

- Tabs simply hold content and nothing else needs the selection — use [TabPanel](https://docs-llm.a2v10.com/xaml/layouts/tabpanel.md) instead, which owns its tabs and their content together.

## Syntax

```xml
<TabBar Value="{Bind Root.Value}">
  <TabBar.Description>
    Selected: <Span Content="{Bind Root.Value}"/>
  </TabBar.Description>
  <TabButton ActiveValue="1" Content="First button"/>
  <TabButton ActiveValue="2" Content="Second button"/>
</TabBar>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Buttons` | TabButtonCollection | Content property. The collection of `TabButton` elements |
| `ItemsSource` | Array | Binding only. Data source for the buttons |
| `Value` | Object | The value that decides which button is active |
| `Style` | TabBarStyle | Display style, see below |
| `DropShadow` | ShadowStyle | Shadow effect |
| `Description` | String \| UIElementBase | Text or element shown to the right of the buttons. Some styles do not display it |

### TabBarStyle Values

| Value | Meaning |
|-------|---------|
| `Default` | Standard display (default) |
| `MainMenu` | As the main menu of the system |
| `Tab` | As a bar of tabs |
| `Wizard` | As a wizard |
| `ButtonGroup` | As a group of buttons |

### TabButton

One button of the bar. Inherits `UIElementBase`. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | String \| UIElementBase | The text of the button, or an element |
| `Description` | String \| UIElementBase | A description for the button |
| `ActiveValue` | String | The value that makes this button active: the button is selected while the bar's `Value` equals it |
| `Badge` | String | A text badge on the button. May be a binding |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties.

## Example

A bar that selects a section, and a `Switch` that shows it:

```xml
<Grid Columns="1*" Rows="auto,1*">
  <TabBar Value="{Bind Root.Mode}" Style="Tab">
    <TabButton ActiveValue="list" Content="List" />
    <TabButton ActiveValue="chart" Content="Chart" />
    <TabButton ActiveValue="log" Content="Log" Badge="{Bind Log.Count}" />
  </TabBar>

  <Switch Expression="{Bind Root.Mode}">
    <Case Value="list">
      <DataGrid ItemsSource="{Bind Documents}" />
    </Case>
    <Case Value="chart">
      <Graphics Delegate="drawChart" Argument="{Bind Documents}" />
    </Case>
    <Else>
      <Text>Nothing selected</Text>
    </Else>
  </Switch>
</Grid>
```

## Notes

- `ActiveValue` is a string, and the comparison with `Value` is by value — a numeric model field works, but the markup still writes the variants as strings.
- The bar stores the selection in the model, which is what makes it survive a reload and lets a command or a validator read it. `TabPanel` keeps its current tab to itself.
- With `ItemsSource` the buttons are generated from data; the bound variant and a static list of `TabButton` elements are alternatives, not a combination.
