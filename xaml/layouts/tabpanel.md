# TabPanel

> A container with a tab bar for switching between multiple content sections.

## Overview

`TabPanel` renders a panel with tabs at the top (or bottom). It inherits from `UIElement → UIElementBase`. Its content is defined by nested `Tab` elements. Each `Tab` has a `Header` and contains its own child elements.

Tabs can be driven by a static list of `Tab` elements or dynamically generated from `ItemsSource`.

## Syntax

```xml
<TabPanel>
  <Tab Header="General">
    <Grid Columns="1*,1*">
      <TextBox Label="Name" Value="{Bind Document.Name}" />
    </Grid>
  </Tab>
  <Tab Header="Details">
    <Grid Columns="1*">
      <TextBox Label="Notes" Value="{Bind Document.Notes}" Multiline="True" />
    </Grid>
  </Tab>
</TabPanel>
```

## TabPanel Properties

| Property | Type | Description |
|----------|------|-------------|
| `Tabs` | TabCollection | **Content property.** Collection of `Tab` elements. |
| `ItemsSource` | Array | **Bind only.** Data source for dynamically generated tabs. |
| `Header` | Object | Panel heading displayed on the right side of the tab bar. Supports `Bind` or `UIElementBase`. |
| `Border` | Boolean | Show border around the panel content area |
| `FullPage` | Boolean | Panel occupies the entire page as the root element |
| `MinHeight` | Length | Minimum panel height |
| `Height` | Length | Panel height |
| `Overflow` | Boolean | Enable scroll bars; required for popups (DatePicker, etc.) inside tabs |
| `TabPosition` | TabPosition | `Top` (default) or `Bottom` — tab bar position |

## Tab Properties

| Property | Type | Description |
|----------|------|-------------|
| `Header` | String | Tab button label. Supports `Bind`. |
| `Children` | UIElementCollection | **Content property.** Tab content elements. |
| `If` | Boolean? | Conditionally include this tab. Supports `Bind`. |

## Example

```xml
<!-- Multi-tab document form -->
<TabPanel Border="True">
  <Tab Header="Main">
    <Grid Columns="1*,1*">
      <TextBox Label="Name" Value="{Bind Document.Name}" Required="True" />
      <TextBox Label="Code" Value="{Bind Document.Code}" />
      <DatePicker Label="Date" Value="{Bind Document.Date}" />
      <ComboBox Label="Type" Value="{Bind Document.DocType}"
                ItemsSource="{Bind DocTypes}" DisplayProperty="Name" />
    </Grid>
  </Tab>

  <Tab Header="Items">
    <DataGrid ItemsSource="{Bind Document.Rows}" Hover="True">
      <DataGridColumn Header="Item" Content="{Bind Item.Name}" />
      <DataGridColumn Header="Qty" Content="{Bind Qty, DataType=Number}"
                      Align="Right" Width="80px" />
      <DataGridColumn Header="Price" Content="{Bind Price, DataType=Currency}"
                      Align="Right" Width="100px" />
    </DataGrid>
  </Tab>

  <Tab Header="Notes">
    <TextBox Value="{Bind Document.Notes}" Multiline="True" Rows="8" />
  </Tab>

  <!-- Conditional tab -->
  <Tab Header="Approval" If="{Bind Document.RequiresApproval}">
    <Grid Columns="1*,1*">
      <Static Label="Approved By" Value="{Bind Document.ApprovedBy.Name}" />
      <Static Label="Approved At" Value="{Bind Document.ApprovedAt, DataType=DateTime}" />
    </Grid>
  </Tab>
</TabPanel>

<!-- Full-page tabpanel (tabs as top-level page navigation) -->
<TabPanel FullPage="True">
  <Tab Header="List">
    <DataGrid ItemsSource="{Bind Items}" Hover="True" />
  </Tab>
  <Tab Header="Statistics">
    <!-- chart/summary content -->
  </Tab>
</TabPanel>
```

## Notes

- `Tab.Header` supports `Bind` — useful for dynamic tab titles (e.g., showing a count badge).
- `Tab.If="{Bind condition}"` removes the tab entirely from the DOM when false — use for permission-gated tabs.
- `Overflow="True"` on `TabPanel` is needed when any tab contains popups (DatePicker, ComboBox dropdown, etc.) that extend beyond the tab boundary.
- `FullPage="True"` is used when the TabPanel is the root content of a page and should fill the available height.
- Dynamically generated tabs via `ItemsSource` repeat the first `Tab` element for each item; bindings inside are relative to each item.
