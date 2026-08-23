# _layout

> The special application folder that attaches external scripts and stylesheets to the application shell — `_scripts.html` and `_styles.html`. .NET Core only.

## Overview

Everything on this page applies to the **.NET Core** version of the platform only.

`_layout` is a special folder in the application root. Special folders are recognised by the leading underscore in the name, and this one exists to attach external modules to the application: third-party JavaScript libraries, the application's own scripts, and additional stylesheets.

The folder may contain two files, `_scripts.html` and `_styles.html`. Both are optional. If a file is present, its content is inserted into the application markup when the application shell is loaded — once, not per endpoint.

This is also where libraries that ship with the platform but are not attached automatically are added. `d3.min.js`, required by the [Graphics](https://docs-llm.a2v10.com/xaml/controls/graphics.md) element, is the usual case.

## Syntax

`_scripts.html` — additional libraries and application scripts:

```xml
<script type="text/javascript" src="/scripts/lib/chartjs/chart.min.js"></script>
```

`_styles.html` — additional stylesheets:

```xml
<link href="/css/app/myapp.min.css?v=20260518" rel="stylesheet" />
```

## Example

An application that draws charts with `Graphics` and carries its own stylesheet.

`_layout/_scripts.html`:

```xml
<script type="text/javascript" src="/scripts/d3.min.js"></script>
<script type="text/javascript" src="/scripts/app/charts.js?v=20260518"></script>
```

`_layout/_styles.html`:

```xml
<link href="/css/app/myapp.min.css?v=20260518" rel="stylesheet" />
```

## Notes

- Browsers cache scripts and stylesheets. Add a version parameter to the link — `?v=20260518` above — and change it whenever the file itself changes, otherwise users keep the old version after an update. The platform does not do this for you.
- Both files contain plain HTML fragments: the tags to be inserted, with no wrapping document, `<head>`, or `<body>`.
- The files are attached application-wide. There is no per-endpoint equivalent — a script needed by a single view is still loaded everywhere.
- Other special folders in the application root follow the same underscore convention: `_home`, `_assets`, `_emails`, `_files`.
