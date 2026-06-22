# Template Overview

> The client-side JavaScript template object that defines the behavior of a data model — initial values, computed properties, validation, events, commands, and delegates.

## Overview

A template is a JavaScript object that describes the behavior of a model on the client. SQL shapes the data and XAML renders the view; the template is where the model gains logic — default values for new records, computed properties, validation rules, lifecycle event handlers, UI commands, and callback delegates.

The template is an ordinary module that exports a single object. The platform reads each named section and applies it to the model after the data is loaded and the view is built. Every section is optional — include only what the model needs.

Sections fall into two groups. Some describe data: `defaults` seeds new models, `properties` adds computed and scalar attributes, `validators` checks correctness. Others describe behavior wired to the view by name: `commands` are invoked from buttons via `BindCmd`, `delegates` are callbacks attached to controls such as a selector. `events` handle model and element lifecycle, `options` tune model-wide behavior, and `utils` is a free-form bag of helpers.

## Syntax

```js
const template = {
  options:    {}, /* model-wide behavior flags */
  defaults:   {}, /* initial values for new models */
  properties: {}, /* additional scalar and computed properties */
  validators: {}, /* data validation rules */
  events:     {}, /* model and element lifecycle handlers */
  commands:   {}, /* commands invoked from the view */
  delegates:  {}, /* callbacks attached to controls */
  utils:      {}  /* arbitrary helpers (useful for mergeTemplate) */
};

module.exports = template; /* export the template from the current module */
```

### Sections

| Section | Type | Description |
|---------|------|-------------|
| [`options`](https://docs-llm.a2v10.com/template/options.md) | object | Flags that tune behavior of the whole model |
| [`defaults`](https://docs-llm.a2v10.com/template/defaults.md) | object | Initial values applied to a new model |
| [`properties`](https://docs-llm.a2v10.com/template/properties.md) | object | Extra scalar and computed properties added to model elements |
| [`validators`](https://docs-llm.a2v10.com/template/validators.md) | object | Validation rules bound to data, not to the UI |
| [`events`](https://docs-llm.a2v10.com/template/events.md) | object | Handlers for model and element lifecycle events |
| [`commands`](https://docs-llm.a2v10.com/template/commands.md) | object | Commands invoked from the view via `BindCmd` |
| [`delegates`](https://docs-llm.a2v10.com/template/delegates.md) | object | Callback functions attached to controls |
| `utils` | object | Arbitrary object of helpers; useful for `mergeTemplate` |

## Example

A minimal template for a document model — seed today's date, compute a row sum, require an agent, and expose a command:

```js
const dateUtils = require('std:utils').date;

const template = {
  defaults: {
    'Document.Date': dateUtils.today()
  },
  properties: {
    'TRow.Sum'() { return this.Qty * this.Price; }
  },
  validators: {
    'Document.Agent': 'Agent is required'
  },
  commands: {
    apply: {
      exec(doc) { return this.$ctrl.$invoke('apply', { Id: doc.Id }); },
      canExec(doc) { return !doc.$isNew; }
    }
  }
};

module.exports = template;
```

## Notes

- Every section is optional. The platform applies only the sections present in the exported object.
- The template runs on the client. It is bound to the data model, not to individual UI controls — the same logic applies regardless of how the data is rendered.
- Property and validator names are paths into the model. Type-scoped names (for example `TRow.Sum`) use the model's type name; array segments in a path are marked with `[]` (for example `Document.Rows[].Sum`).
- `utils` is not interpreted by the platform; it is a convenient place for shared helpers that survive `mergeTemplate` when one template extends another.
