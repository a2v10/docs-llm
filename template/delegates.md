# Delegates

> The `delegates` section of a template — callback functions attached to controls, called when the control needs to perform an action.

## Overview

A delegate is a callback function. It is called at the moment a control needs to perform some action — for example, a [selector](https://docs-llm.a2v10.com/xaml/controls/selector.md) calls its delegate when the user types a text fragment, to fetch the matching options.

The descriptor is an ordinary JavaScript object where each property name is a delegate identifier and each value is a function. Delegates are attached to a control in the view through its `Delegate` property, naming the template delegate to call.

The first argument is always the root object (`this` is also the root), and the remaining arguments and the return value depend on the specific delegate — a fetch delegate, for instance, receives the bound element and the entered text and returns the list of matches.

## Syntax

```js
delegates: {
  Delegate1: Function,
  Delegate2: Function
}
```

## Example

### Selector fetch

A selector in the view names the delegate that fetches its options:

```xml
<Selector Delegate="FetchAgent" Value="{Bind Document.Agent}"
          DisplayProperty="Name"
          Label="Choose a counterparty">
</Selector>
```

The template maps that name to a function that calls the server:

```js
const template = {
  delegates: {
    FetchAgent: fetchAgent
  }
};

function fetchAgent(agent, text) {
  const ctrl = this.$ctrl;
  return ctrl.$invoke('fetchCustomer', { Text: text, Kind: 'Customer' });
}

module.exports = template;
```

### Graphics drawing

The same section serves a control with a completely different signature. A [Graphics](https://docs-llm.a2v10.com/xaml/controls/graphics.md) element calls its delegate to draw itself, passing the DOM node instead of a search string, and returns nothing:

```xml
<Graphics Delegate="DrawChart" Argument="{Bind Chart.Items}" Height="20rem" />
```

```js
const template = {
  delegates: {
    DrawChart: drawChart
  }
};

function drawChart(chart, arg, node) {
  chart.append('svg')
    .attr('width', node.clientWidth)
    .attr('height', node.clientHeight);
  // ... draw with d3.js
}
```

## Notes

- A delegate is referenced by name from a control's `Delegate` property; the control decides when to call it.
- `this` inside a delegate is the root object (`TRoot`); the first argument is also the root.
- The remaining arguments and the return value are defined by the control that invokes the delegate — a selector fetch delegate receives the element and the entered text and returns the matching options, while a [Graphics](https://docs-llm.a2v10.com/xaml/controls/graphics.md) delegate receives the d3 selection, the argument, and the DOM node, and returns nothing.
- Use the controller (`this.$ctrl`) to reach platform methods such as `$invoke` for server calls.
