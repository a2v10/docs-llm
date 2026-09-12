# PdfReportViewer

> Shows a finished report on the form itself — built on the server, displayed by the browser's own PDF viewer.

## Overview

`PdfReportViewer` displays a [Xaml report](https://docs-llm.a2v10.com/report/overview.md) inside a view. The report is built on the server and rendered by the browser's built-in PDF viewer, so nothing about the document is interactive.

The element names the report the same way the `Report` command does: `Url` is the folder of the endpoint whose `model.json` declares the report, `Report` is the key in its [reports](https://docs-llm.a2v10.com/model/reports.md) section, and `Argument` is the object the report identifier is taken from.

`PdfReportViewer` inherits from `UIElementBase` — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).

This element is NET Core only.

## Use When

- The user has to see the printed form without downloading it first — an invoice preview next to Print and Download buttons.

## Do Not Use When

- The result should be a file or a print dialog rather than something on screen — use `{BindCmd Report}` with `Export=True` or `Print=True`, see [Bind & BindCmd](https://docs-llm.a2v10.com/xaml/bind.md).
- The data has to stay interactive — sortable, expandable — in which case it is a view, not a report: use [Sheet](https://docs-llm.a2v10.com/xaml/layouts/sheet.md).

## Syntax

```xml
<PdfReportViewer Url="/document/print" Report="invoice" Argument="{Bind Document}" Height="40rem" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Url` | String | Required. The path to the endpoint folder — the one holding the `model.json` with the `reports` section. |
| `Report` | String | Required. The name of the report, that is the key in the `reports` section. |
| `Argument` | Object | Usually a binding. The object the identifier (`$id`) for the report's data query is taken from. |
| `Height` | Length | The height of the viewing area. |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for all inherited properties.

## Notes

- `Url` and `Report` are mandatory: without either of them the view does not load and the system reports a markup error.
- `Url` is the endpoint folder, not an action of it.
- The viewer and the print and download buttons next to it describe the same report — same `Url`, `Report` and `Argument`. The full arrangement is in [Viewing and Printing Reports](https://docs-llm.a2v10.com/report/view.md).
