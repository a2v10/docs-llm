# Reports

> The `reports` object in model.json — configures downloadable reports and data exports in Stimulsoft, XML, or JSON format.

## Overview

Reports generate files that the user can download directly from the browser. A2v10 supports three report types: Stimulsoft designer reports (`.mrt` files), XML exports, and JSON exports.

Each report calls a stored procedure to load its data, then passes the result to the report engine. The procedure name defaults to `[model].Report` but can be overridden with the `procedure` property.

Reports inherit `source`, `schema`, and `model` from the root of `model.json`.

The `name` property sets the downloaded filename and supports `{{Property.Path}}` macros — the platform substitutes values from the loaded model at runtime.

## Syntax

```json
"reports": {
  "<reportName>": {
    "source":     "",
    "schema":     "",
    "model":      "",
    "procedure":  "",
    "parameters": {},
    "type":       "stimulsoft | xml | json",
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
| `model` | string | Overrides the root `model` |
| `procedure` | string | Stored procedure name; defaults to `[model].Report` |
| `parameters` | object | Static key-value pairs passed to the stored procedure |
| `type` | string | Report format: `stimulsoft` (default), `xml`, or `json` |
| `report` | string | Stimulsoft report filename (`.mrt`); required for `stimulsoft` type |
| `name` | string | Downloaded filename; supports `{{Property}}` macros |
| `encoding` | string | Output encoding for `xml` reports: `utf-8`, `utf-16`, `windows-1251` |
| `xmlSchemas` | array | Paths to XSD schema files for validation |
| `validate` | boolean | If `true`, validates the output XML against `xmlSchemas` |
| `variables` | object | Variables passed to Stimulsoft to control report behavior |

## Example

### Stimulsoft report

```json
"reports": {
  "print": {
    "type":      "stimulsoft",
    "report":    "agent_card",
    "name":      "Agent.mrt",
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
    "type":      "xml",
    "procedure": "Agent.ExportXml",
    "name":      "{{Agent.Code}}.xml",
    "encoding":  "utf-8"
  }
}
```

The downloaded filename uses the `Code` field from the loaded model — for example, `A-00123.xml`.

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

- `type` defaults to `stimulsoft` when omitted.
- For `stimulsoft` reports, `report` is required and must be the filename without path (the platform looks for the file relative to the endpoint directory).
- `variables` are passed to the Stimulsoft report engine and can control visibility of sections, logos, totals, etc.
- `encoding` is only used for `xml` reports; it has no effect on `stimulsoft` or `json`.
- `xmlSchemas` and `validate` are only meaningful for `xml` reports; validation failure returns an error to the client.
- The `{{Property}}` macro in `name` resolves at the time of the report request, using the model data loaded by the stored procedure.
