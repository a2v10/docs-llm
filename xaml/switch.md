# Switch, Case and Else

> Shows one of several blocks depending on a value — the markup equivalent of a switch statement.

## Overview

`Switch` evaluates an expression and renders the one `Case` whose `Value` matches it. A `Case` may be replaced by `Else`, which is rendered when no value matched.

`Switch` inherits `UIElementBase`. Content property: `Cases`. `Case` and `Else` hold their children in `Children`.

The expression is always a binding and must be of type `String`.

## Use When

- Several alternative blocks exist and exactly one is shown — a form that differs by document type, a panel that differs by state.
- The alternatives are driven by a value that lives in the model, for example the one a [TabBar](https://docs-llm.a2v10.com/xaml/layouts/tabbar.md) writes.

## Do Not Use When

- There are only two variants and one of them is "nothing" — `If` or `Show` on a single element is shorter; see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).
- The blocks are tabs the user switches between and nothing else reads the selection — use [TabPanel](https://docs-llm.a2v10.com/xaml/layouts/tabpanel.md) instead.

## Syntax

```xml
<Switch Expression="{Bind Document.Kind}">
  <Case Value="invoice">
    <!-- elements -->
  </Case>
  <Case Value="receipt">
    <!-- elements -->
  </Case>
  <Else>
    <!-- elements -->
  </Else>
</Switch>
```

## Properties

### Switch

| Property | Type | Description |
|----------|------|-------------|
| `Cases` | Collection of `Case` | Content property. The collection of alternatives |
| `Expression` | String | Always a binding. The expression whose value selects the case. Must be of type `String` |
| `Height` | Length | Height of each of the `Case` elements |

### Case

| Property | Type | Description |
|----------|------|-------------|
| `Children` | UIElementCollection | Content property. The child elements |
| `Value` | String | The value at which this element is displayed |

### Else

| Property | Type | Description |
|----------|------|-------------|
| `Children` | UIElementCollection | Content property. The child elements |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for the properties `Switch` inherits.

## Example

```xml
<Switch Expression="{Bind Document.Kind}">
  <Case Value="invoice">
    <Grid Columns="1*,1*">
      <Selector Label="Customer" Value="{Bind Document.Agent}" DisplayProperty="Name" />
      <TextBox Label="Contract" Value="{Bind Document.Contract}" />
    </Grid>
  </Case>
  <Case Value="receipt">
    <Grid Columns="1*,1*">
      <Selector Label="Supplier" Value="{Bind Document.Agent}" DisplayProperty="Name" />
      <DatePicker Label="Received" Value="{Bind Document.ReceivedAt}" />
    </Grid>
  </Case>
  <Else>
    <Panel Style="Warning" Header="Unknown document kind">
      <Text><Span Content="{Bind Document.Kind}" /></Text>
    </Panel>
  </Else>
</Switch>
```

## Notes

- Only one `Else` is meaningful in a `Switch`, and it is the fallback — not a case with an empty value.
- The expression must yield a string, so a numeric or boolean model field is compared as text; keep the `Value` attributes in the same spelling the model produces.
- `Height` is set on the `Switch` and applies to every case, which keeps the page from jumping as the user switches between blocks of different size.
