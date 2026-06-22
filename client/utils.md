# Standard Utilities (std:utils)

> The std:utils module — helper functions for dates, currency, and text, plus type checks, conversion, and formatting.

## Overview

`std:utils` is a standard module of helper functions for processing dates, currency, and text, along with general utilities for type checking, conversion, JSON, formatting, and template merging. It is loaded with [`require`](https://docs-llm.a2v10.com/client/require.md) using the `std:` prefix.

The module groups its functions into namespaces: `date`, `currency`, and `text`, with the remaining general helpers on the module itself.

## Syntax

```js
const utils     = require('std:utils');
const dateUtils = utils.date;     /* date helpers */
const cyUtils   = utils.currency; /* currency helpers */
const textUtils = utils.text;     /* text helpers */
```

### Namespaces

| Namespace | Access | Purpose |
|-----------|--------|---------|
| `date` | `utils.date` | Date and time manipulation |
| `currency` | `utils.currency` | Currency rounding and formatting |
| `text` | `utils.text` | Text inspection and formatting |
| (main) | `utils` | Type checks, conversion, JSON, formatting, template merge |

### date functions

| Function | Purpose |
|----------|---------|
| `today()` | Current date (no time) |
| `now()` | Current date and time |
| `create()` | Construct a date |
| `add()` | Add an interval to a date |
| `diff()` | Difference between two dates |
| `endOfMonth()` | Last day of the month |
| `format()` | Format a date for display |
| `parse()` | Parse a date from text |
| `int2time()` / `time2int()` | Convert between time and integer |

### text functions

| Function | Purpose |
|----------|---------|
| `contains()` | Whether a string contains a substring |
| `containsText()` | Case-insensitive contains |
| `capitalize()` | Capitalize a string |
| `equalNoCase()` | Case-insensitive equality |
| `maxChars()` | Truncate to a maximum length |

### currency functions

| Function | Purpose |
|----------|---------|
| `round()` | Round a currency value |
| `format()` | Format a currency value |

### Enumerations

| Enum | Members |
|------|---------|
| `DataType` | `Currency`, `Number`, `DateTime`, `Date`, `DateUrl`, `Time`, `Period` |
| `DateTimeUnit` | `year`, `month`, `day`, `hour`, `minute`, `second` |
| `DateUnit` | `year`, `month`, `day`, `minute`, `second` |

## Example

```js
const dateUtils = require('std:utils').date;

const template = {
  defaults: {
    'Document.Date': dateUtils.today() /* seed today's date */
  },
  properties: {
    'TDocument.DueDate'() { return dateUtils.add(this.Date, 30, 'day'); }
  }
};

module.exports = template;
```

## Notes

- Load the whole module once and destructure the namespaces you need (`utils.date`, `utils.currency`, `utils.text`).
- Use these helpers for dynamic [`defaults`](https://docs-llm.a2v10.com/template/defaults.md) (such as today's date) instead of constructing dates inline.
- The general namespace includes `mergeTemplate`, used when one [template](https://docs-llm.a2v10.com/template/overview.md) extends another via its `utils` section.
- `std:utils` is a standard module, so it is referenced by name with the `std:` prefix and not by file path — see [require](https://docs-llm.a2v10.com/client/require.md).
