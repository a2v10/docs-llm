# Viewing and Printing Reports

> Showing a finished report on a form — the endpoint, the PdfReportViewer element, and the print and download buttons.

## Overview

A report can be shown to the user right on a form instead of only being handed over as a file. Three things are needed for that: an endpoint that declares the report, a template next to it, and a view carrying a [PdfReportViewer](https://docs-llm.a2v10.com/xaml/controls/pdfreportviewer.md).

The report is built on the server and displayed by the browser's own built-in PDF viewer.

Showing a report on a form requires no separate template: it is the same report as the one behind the buttons, and the same file. Only that file ever needs changing.

This is NET Core only.

## Syntax

Three values describe which report is meant, and they are the same for the viewer and for both commands:

| Value | What it is |
|-------|------------|
| `Url` | The path to the folder of the endpoint — the one holding the `model.json` with the `reports` section. Not to an action |
| `Report` | The name of the report, that is the key in the `reports` section |
| `Argument` | The object the identifier (`$id`) is taken from; that is what ends up in the report's data query |

## Example

### Endpoint

The endpoint here is the folder `/document/print`; the report is called `invoice` and its template lies next to the `model.json` as `invoice.report.vxaml`. The `index` action serves the view the report will be shown on.

```json
{
  "$schema": "../../@schemas/model-json-schema.json#",
  "actions": {
    "index": {
      "schema": "dbo",
      "model": "Document",
      "view": "index.view"
    }
  },
  "reports": {
    "invoice": {
      "type": "pdf",
      "schema": "dbo",
      "model": "Document",
      "report": "invoice.report"
    }
  }
}
```

### View

`/document/print/index.view.vxaml` — the viewer and two toolbar buttons:

```xml
<Page xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.Xaml" Title="Invoice">
    <Page.Toolbar>
        <Toolbar>
            <Button Content="Print" Icon="Print"
                    Command="{BindCmd Report, Report=invoice, Url='/document/print', Argument={Bind Document}, Print=True}"/>
            <Button Content="Download" Icon="Download"
                    Command="{BindCmd Report, Report=invoice, Url='/document/print', Argument={Bind Document}, Export=True}"/>
        </Toolbar>
    </Page.Toolbar>
    <PdfReportViewer Url="/document/print" Report="invoice" Argument="{Bind Document}"/>
</Page>
```

## Notes

- The buttons show the same report, only not on the form: [BindCmd Report](https://docs-llm.a2v10.com/xaml/bind.md) with `Print=True` opens the print dialog, with `Export=True` downloads the file. Its remaining arguments — `Format` and `Data` — are described with the command.
- `Url` points at the endpoint folder, not at an action. Pointing it at an action is the usual reason a viewer finds no report.
- `Url` and `Report` are mandatory on the viewer: without either of them the view does not load and the system reports a markup error.
- The template itself is written as described in [Reports Overview](https://docs-llm.a2v10.com/report/overview.md) — flow markup or a [workbook](https://docs-llm.a2v10.com/report/workbook.md), both work the same here.
