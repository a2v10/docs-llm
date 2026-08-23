# Graphics

> A drawing surface rendered programmatically by a JavaScript delegate, using the d3.js library.

## Overview

`Graphics` reserves a rectangular area in the view and hands it to application code. The platform draws nothing itself: on render it calls a [delegate](https://docs-llm.a2v10.com/template/delegates.md) named by the `Delegate` property and passes it the DOM node of the element. Everything that appears inside is produced by that function.

Drawing is done with the **d3.js** library, so the delegate receives the node already wrapped in a d3 selection. `Delegate` is mandatory — a `Graphics` element without it raises an error while the markup is being compiled.

The content of the element is cleared completely before every call, so the delegate always draws from scratch. It cannot append to the previous result. By default the delegate is called once; set `Watch` when the bound argument changes and the picture has to follow it.

`Graphics` inherits from `UIElementBase` — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).

## Requirements

The `d3.min.js` file ships with the platform, in the `wwwroot/scripts` folder, but it is attached to the application differently depending on the platform version:

| Version | How d3.js is attached |
|---------|-----------------------|
| .NET Framework | Automatically and always — nothing to do |
| .NET Core | Not attached automatically. Add it yourself in `_scripts.html` of the [_layout](https://docs-llm.a2v10.com/app/layout.md) special folder |

If the library is not attached, the element renders an error message instead of the graphics.

## Syntax

```xml
<Graphics Delegate="DrawChart" Argument="{Bind Chart.Items}"
          Watch="Watch" Height="20rem" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Delegate` | String | Name of the [delegate](https://docs-llm.a2v10.com/template/delegates.md) called to draw the graphics. Required — omitting it is a markup compilation error. |
| `Argument` | Object | Value passed to the delegate, normally a [Bind](https://docs-llm.a2v10.com/xaml/bind.md) expression. |
| `Watch` | WatchMode | Whether to redraw when the argument changes. Default `None`. |
| `CenterContent` | Boolean | If `true`, centres the content both horizontally and vertically. Only meaningful together with an explicit `Height`. |
| `Height` | Length | Height of the element. Default `Auto`. |
| `CssClass` | String | CSS classes added to the element. May be a binding. |

`WatchMode` values:

| Value | Behaviour |
|-------|-----------|
| `None` | Default. The delegate is not called again when the argument changes. |
| `Watch` | Redraw when the argument itself changes. |
| `Deep` | Redraw when the argument or any of its nested properties change, recursively. |

## Delegate

```js
draw(this: IRoot, chart: object, arg: any, node: DOMElement): void
```

| Argument | Description |
|----------|-------------|
| `this` | The root data object — see [IRoot](https://docs-llm.a2v10.com/client/root.md) |
| `chart` | The element's DOM node wrapped in a d3 selection, that is, the result of `d3.select(node)` |
| `arg` | The value of the `Argument` property from the markup |
| `node` | The plain DOM node bound to the element |

## Example

A bar chart over an array from the model.

The view declares the surface and passes the array as the argument:

```xml
<Graphics Delegate="DrawChart" Argument="{Bind Chart.Items}"
          Watch="Watch" Height="20rem" />
```

The template maps `DrawChart` to the drawing function:

```js
const template = {
  delegates: {
    DrawChart: drawChart
  }
};

function drawChart(chart, arg, node) {
  const data = arg || [];
  const width = node.clientWidth;
  const height = node.clientHeight;

  const x = d3.scaleBand()
    .domain(data.map(d => d.Name))
    .range([0, width])
    .padding(0.2);

  const y = d3.scaleLinear()
    .domain([0, d3.max(data, d => d.Value)])
    .range([height, 0]);

  const svg = chart.append('svg')
    .attr('width', width)
    .attr('height', height);

  svg.selectAll('rect')
    .data(data)
    .enter()
    .append('rect')
    .attr('x', d => x(d.Name))
    .attr('y', d => y(d.Value))
    .attr('width', x.bandwidth())
    .attr('height', d => height - y(d.Value));
}

module.exports = template;
```

## Notes

- The drawing size is not passed to the delegate — take it from `node.clientWidth` / `node.clientHeight`, as in the example above. That is also why an explicit `Height` is normally set in the markup.
- Do not set `Watch="Deep"` unless it is really needed — deep watching walks the whole object graph on every change and costs performance. `Watch` is enough whenever the argument is replaced rather than edited in place.
- Because the content is wiped before each call, an incremental or animated update built on top of the previous drawing is not possible.
- `CenterContent` without `Height` has no visible effect: with `Height="Auto"` there is no free vertical space to centre within.

## Hints

- The name in `Delegate` is looked up as a key of the template's `delegates` object, so it must match character for character, case included.
- If an error message appears in place of the drawing on .NET Core, `d3.min.js` is not attached — see [_layout](https://docs-llm.a2v10.com/app/layout.md).
