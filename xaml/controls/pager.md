# Pager

> Page navigation for a collection — buttons plus a counter, driven by a CollectionView rather than by a collection of its own.

## Overview

`Pager` is the page navigation element. It is normally attached to the whole page through the `Pager` property of [Page](https://docs-llm.a2v10.com/xaml/layouts/page.md).

The element works with no collection of its own: the collection is always handled by the `CollectionView` component, and the pager is bound to it. It inherits `UIElementBase`.

## Use When

- A server-paged collection needs page buttons and an "items N–M of K" counter.

## Do Not Use When

- The grid itself is expected to page the data — server paging lives in the stored procedure, see [Paging](https://docs-llm.a2v10.com/sql/paging.md).

## Syntax

```xml
<Page.Pager>
  <Pager Source="{Bind Parent.Pager}" />
</Page.Pager>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Source` | Object | The source collection. Always a binding. Together with `CollectionView` the binding is `Source="{Bind Parent.Pager}"` |
| `Style` | PagerStyle | `Default` — ordinary buttons, `Rounded` — round buttons |
| `EmptyText` | String | Text shown at the right when the collection is empty. Defaults to a localized "no items" |
| `TemplateText` | String | Text shown at the right, with macros — see below. Defaults to a localized "items #[Start]-#[End] of #[Count]" |
| `CssClass` | String | CSS classes to add to the element |

### TemplateText macros

| Macro | Meaning |
|-------|---------|
| `#[Start]` | Number of the first item on the page |
| `#[End]` | Number of the last item on the page |
| `#[Count]` | Number of items in the collection |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for inherited properties.

## Example

```xml
<Page xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.Xaml" Title="Agents">
  <Page.Pager>
    <Pager Source="{Bind Parent.Pager}" Style="Rounded"
           TemplateText="agents #[Start]-#[End] of #[Count]"
           EmptyText="nothing found" />
  </Page.Pager>

  <DataGrid ItemsSource="{Bind Agents}" />
</Page>
```

## Notes

- The pager renders state, it does not fetch data: the page size, the offset and the total count come from the model — see [Paging](https://docs-llm.a2v10.com/sql/paging.md) for the `@Offset` / `@PageSize` / `!RowCount` side of it.
- `TemplateText` replaces the whole default phrase, so include the macros you still want in it.
- `EmptyText` is shown instead of the counter, not instead of the collection; an empty-state block inside the grid or list is a separate element.
