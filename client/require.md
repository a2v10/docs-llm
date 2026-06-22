# require

> Loads a module and returns its exports — application modules by path, standard modules by their `std:` name.

## Overview

`require` loads a module and returns a reference to its exports. Application modules are addressed by file path, relative to the application root. Standard modules supplied by the platform are addressed by a name beginning with the `std:` prefix and have no file path.

This is the same mechanism a [template](https://docs-llm.a2v10.com/template/overview.md) uses to pull in helpers — most commonly [`std:utils`](https://docs-llm.a2v10.com/client/utils.md).

## Syntax

```js
require(path)
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `path` | string | Path to the module file, relative to the application root; or a `std:`-prefixed name for a standard module |

The return value is a reference to the module's exports.

## Example

```js
const utils     = require('std:utils');        /* standard module */
const dateUtils = require('std:utils').date;   /* a namespace of it */
const helpers   = require('document/helpers');  /* application module by path */

const template = {
  defaults: {
    'Document.Date': dateUtils.today()
  }
};

module.exports = template;
```

## Notes

- Standard modules use the `std:` prefix and no path; for example `std:utils`. Application modules use a path relative to the application root.
- A module exposes its public surface through `module.exports`; `require` returns exactly that object.
- A [template](https://docs-llm.a2v10.com/template/overview.md) file is itself a module — it ends with `module.exports = template;`.
