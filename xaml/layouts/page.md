# Page

> The root element for standalone views — pages with their own URL and browser navigation entry.

## Overview

`Page` is always the root element of a page XAML file. It defines a full-page view that participates in browser navigation and has its own URL managed by the router. It inherits from `RootContainer → Container → UIElement → UIElementBase`.

A page typically contains a `Toolbar` (action buttons at the top), a `Taskpad` (side panel with contextual links), and its main content. A `CollectionView` and `Pager` can be attached to the page for server-side collection management outside of the main content tree.

## Syntax

```xml
<Page
    xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
    Title="Documents">
  <Page.Toolbar>
    <Toolbar> ... </Toolbar>
  </Page.Toolbar>
  <Page.Taskpad>
    <Taskpad> ... </Taskpad>
  </Page.Taskpad>

  <!-- Main content -->
  <DataGrid ItemsSource="{Bind Documents}" ... />
</Page>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Title` | String | Page title displayed in the browser tab and breadcrumb. Supports `Bind`. |
| `Toolbar` | UIElementBase | Top toolbar — typically a `Toolbar` element. |
| `Taskpad` | UIElementBase | Side task panel — typically a `Taskpad` element. |
| `CollectionView` | CollectionView | Server-side collection management (sorting, filtering, paging) attached at page level. |
| `Pager` | Pager | Pagination control attached at page level, works with `CollectionView`. |
| `Background` | BackgroundStyle | Page background color: `LightGray`, `WhiteSmoke`, `White`, `LightYellow` |

### Inherited Content Property

| Property | Type | Description |
|----------|------|-------------|
| `Children` | UIElementCollection | Main page content elements. |

See [Base Classes](../base-classes.md) for inherited properties from `UIElement` and `UIElementBase`.

## Example

```xml
<Page
    xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
    Title="{Bind @@Page.Title}">

  <Page.Toolbar>
    <Toolbar>
      <Button Content="Create"
              Command="{BindCmd Dialog, Action=New, Argument={Bind NewItem}}"
              Icon="Plus" Style="Primary" />
      <Button Content="Reload" Command="{BindCmd Reload}" Icon="Reload" />
      <Button Content="Delete"
              Command="{BindCmd DbRemoveSelected, Argument={Bind Items.Selected}}"
              Icon="Delete" Style="Danger"
              If="{Bind Items.HasSelected}"
              Toolbar.Align="Right" />
    </Toolbar>
  </Page.Toolbar>

  <DataGrid ItemsSource="{Bind Items}" Hover="True" Sort="True"
            DoubleClick="{BindCmd Dialog, Action=Edit, Argument={Bind Items.Selected}}">
    <DataGridColumn Header="Date" Content="{Bind Date, DataType=Date}" Width="100px" />
    <DataGridColumn Header="Name" Content="{Bind Name}" />
    <DataGridColumn Header="Amount" Content="{Bind Amount, DataType=Currency}"
                    Align="Right" Width="120px" />
  </DataGrid>
</Page>
```

### Page with Pager and CollectionView

```xml
<Page xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
      Title="Customers">

  <Page.CollectionView>
    <CollectionView RunAt="Server" Filter="Client">
      <CollectionView.SortDescriptions>
        <SortDescription Property="Name" />
      </CollectionView.SortDescriptions>
    </CollectionView>
  </Page.CollectionView>

  <Page.Pager>
    <Pager />
  </Page.Pager>

  <Page.Toolbar>
    <Toolbar>
      <Button Content="Create" Command="{BindCmd Dialog, Action=New}" Icon="Plus" Style="Primary" />
    </Toolbar>
  </Page.Toolbar>

  <DataGrid ItemsSource="{Bind Customers}" Sort="True" Hover="True">
    <DataGridColumn Header="Name" Content="{Bind Name}" />
    <DataGridColumn Header="Phone" Content="{Bind Phone}" Width="140px" />
  </DataGrid>
</Page>
```

## Notes

- `Page` must be the XML root element; the `xmlns` attribute is required.
- Complex property syntax (`<Page.Toolbar>`, `<Page.Taskpad>`) is used for non-children sub-elements.
- `CollectionView` and `Pager` at page level allow managing collection state (filter, sort, page) independently of where the DataGrid is nested.
- `Title="{Bind @@Page.Title}"` binds to the server-provided page title from the model metadata.
- Background is `LightGray` by default for list pages; form pages typically use `White` or no background.
