# Reports

> The `reports` object in model.json — configures printed forms and data exports: a Xaml template built by the platform, a Stimulsoft report, or an XML/JSON file.

## Overview

A report is either a printed form the platform builds itself from a [Xaml template](https://docs-llm.a2v10.com/report/overview.md), an external Stimulsoft report, or a data file exported as XML or JSON. All of them are declared the same way and, except for the on-form viewer, all of them arrive as a file the user downloads.

Each report calls a stored procedure to load its data, then passes the result to the report engine. The name of that procedure is always built from the model — `[schema].[Model.Report]` — and cannot be set explicitly. Reports that need different data are given different `model` values.

Reports inherit `source`, `schema`, and `model` from the root of `model.json`.

The `name` property sets the downloaded filename and supports `{{Property.Path}}` macros — the platform substitutes values from the loaded model at runtime.

## Syntax

```json
"reports": {
  "<reportName>": {
    "source":     "",
    "schema":     "",
    "model":      "",
    "parameters": {},
    "type":       "pdf | xlsx | stimulsoft | xml | json",
    "report":     "",
    "name":       "",
    "encoding":   "",
    "xmlSchemas": [],
    "validate":   false,
    "variables":  {}
  }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `source` | string | Overrides the root `source` |
| `schema` | string | Overrides the root `schema` |
| `model` | string | Overrides the root `model`. The name of the data procedure is built from it: `[schema].[Model.Report]` |
| `parameters` | object | Static key-value pairs passed to the stored procedure |
| `type` | string | Report format: `pdf`, `stimulsoft` (default), `xml`, or `json`; `xlsx` is declared but not implemented |
| `report` | string | Template filename without extension, relative to the folder of `model.json`; required for `pdf`, `xlsx` and `stimulsoft` |
| `name` | string | Downloaded filename; supports `{{Property}}` macros |
| `encoding` | string | Output encoding for `xml` reports: `utf-8`, `utf-16`, `windows-1251` |
| `xmlSchemas` | array | Paths to XSD schema files for validation |
| `validate` | boolean | If `true`, validates the output XML against `xmlSchemas` |
| `variables` | object | Variables passed to Stimulsoft to control report behavior |

### Report types

| Type | What it produces |
|------|------------------|
| `pdf` | A printed form the platform builds itself from a [Xaml template](https://docs-llm.a2v10.com/report/overview.md) |
| `xlsx` | The same template, produced as an Excel workbook instead of a PDF. Not implemented yet — do not use |
| `stimulsoft` | Default. An external Stimulsoft report |
| `xml` | An XML data file |
| `json` | A JSON data file |

The default type is `stimulsoft`, so a Xaml template requires `type` to be stated explicitly. The print engine must be registered in the application under the same name as the one given in `type`.

The extension is appended to `report` according to the type: `.mrt` for `stimulsoft`; for `pdf` the engine looks for `.vxaml`, then `.xaml`, then `.json` — the last being a [workbook](https://docs-llm.a2v10.com/report/workbook.md) template.

## Example

### Xaml report

```json
"reports": {
  "invoice": {
    "type":   "pdf",
    "model":  "Document",
    "report": "invoice.report",
    "name":   "Invoice"
  }
}
```

Calls `[a2].[Document.Report]`, builds the document from `invoice.report.vxaml` next to `model.json`, and offers the resulting PDF as a download. The same report can be shown on a form — see [Viewing and Printing Reports](https://docs-llm.a2v10.com/report/view.md).

### Stimulsoft report

```json
"reports": {
  "print": {
    "type":      "stimulsoft",
    "report":    "agent_card",
    "name":      "Agent card",
    "variables": {
      "ShowLogo": true
    }
  }
}
```

Calls `[a2].[Agent.Report]`, loads `agent_card.mrt`, and offers the rendered report as a download.

### XML export with dynamic filename

```json
"reports": {
  "exportXml": {
    "type":     "xml",
    "model":    "AgentExport",
    "name":     "{{Agent.Code}}.xml",
    "encoding": "utf-8"
  }
}
```

Calls `[a2].[AgentExport.Report]` — the export needs a different dataset than the printed card, and a different `model` is the only way to ask for it. The downloaded filename uses the `Code` field from the loaded model — for example, `A-00123.xml`.

### JSON export

```json
"reports": {
  "exportJson": {
    "type": "json",
    "name": "agents.json"
  }
}
```

## Notes

- A report cannot name its own procedure: the name is always `model` plus the `.Report` suffix. When two reports need different data, give them different `model` values.
- `type` defaults to `stimulsoft` when omitted. A Xaml template declared without `"type": "pdf"` is simply never reached.
- For `stimulsoft` reports, `report` is required and must be the filename without path (the platform looks for the file relative to the endpoint directory).
- Instead of a filename, `report` may hold a `{{Property.Path}}` expression — the template text is then taken from a field of the report model, that is from the database rather than from disk.
- `variables` are passed to the Stimulsoft report engine and can control visibility of sections, logos, totals, etc.
- `encoding` is only used for `xml` reports; it has no effect on `stimulsoft` or `json`.
- `xmlSchemas` and `validate` are only meaningful for `xml` reports; validation failure returns an error to the client.
- The `{{Property}}` macro in `name` resolves at the time of the report request, using the model data loaded by the stored procedure.
