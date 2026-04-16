# File Upload

> The `files` object in model.json — configures server-side handling of uploaded files: parsing, binary storage, Azure Blob upload, or CLR processing.

## Overview

The `files` object declares named upload operations for an endpoint. Each operation specifies how the platform should handle an incoming file: parse it into rows and save to the database, store it as raw binary, upload it to Azure Blob Storage, or pass it to custom .NET code.

The `type` property is required and determines the execution path. For parse operations, the `parse` property further specifies the expected file format.

File upload operations call two stored procedures: a `[model].Metadata` procedure that returns column definitions for validation, and a `[model].Update` procedure that saves the data. Both are resolved using the `source`, `schema`, and `model` values (inherited from root unless overridden).

## Syntax

```json
"files": {
  "<operationName>": {
    "type":           "parse | sql | azureBlob | clr",
    "parse":          "excel | xlsx | csv | dbf | xml | auto",
    "source":         "",
    "schema":         "",
    "model":          "",
    "locale":         "",
    "container":      "",
    "azureSource":    "",
    "clrType":        "",
    "async":          false,
    "parameters":     {},
    "imageCompress":  {}
  }
}
```

### type values

| Value | Description |
|-------|-------------|
| `parse` | Parses the file content (Excel, CSV, DBF, XML) and saves rows to the database |
| `sql` | Stores the raw file as `varbinary(max)` in the database |
| `azureBlob` | Uploads the file to an Azure Blob Storage container |
| `clr` | Passes the file to a .NET class implementing `IInvokeTarget` |

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `type` | string | Required. Upload operation type (see table above) |
| `parse` | string | Required for `parse` type. File format: `excel`, `xlsx`, `csv`, `dbf`, `xml`, or `auto` |
| `source` | string | Overrides the root `source` |
| `schema` | string | Overrides the root `schema` |
| `model` | string | Overrides the root `model`; empty string means no database storage |
| `locale` | string | Locale used when parsing numeric and date values, e.g. `"uk"` or `"uk-UA"` |
| `container` | string | Azure Blob container name — required for `azureBlob` type |
| `azureSource` | string | Azure Storage connection name; defaults to `"AzureStorage"` |
| `clrType` | string | Assembly-qualified .NET type — required for `clr` type |
| `async` | boolean | If `true`, the CLR operation runs asynchronously |
| `parameters` | object | Static key-value pairs passed to the stored procedure |
| `imageCompress` | object | JPEG compression settings (see below) |

### imageCompress object

| Property | Type | Description |
|----------|------|-------------|
| `quality` | integer | JPEG quality (0–100) |
| `threshold` | number | Minimum file size in MB before compression is applied |

## Example

### Parse Excel upload

```json
"files": {
  "import": {
    "type":   "parse",
    "parse":  "xlsx",
    "model":  "Agent",
    "locale": "uk-UA"
  }
}
```

The platform calls `[a2].[Agent.Metadata]` to get column definitions, parses the uploaded `.xlsx` file, and saves each row via `[a2].[Agent.Update]`.

### Azure Blob upload with image compression

```json
"files": {
  "uploadPhoto": {
    "type":      "azureBlob",
    "container": "agent-photos",
    "model":     "AgentPhoto",
    "imageCompress": {
      "quality":   80,
      "threshold": 0.5
    }
  }
}
```

Images larger than 0.5 MB are recompressed to 80% JPEG quality before upload.

### Raw binary SQL storage

```json
"files": {
  "attach": {
    "type":  "sql",
    "model": "AgentDocument"
  }
}
```

Stores the file as `varbinary(max)` via `[a2].[AgentDocument.Update]`.

### CLR handler

```json
"files": {
  "processXml": {
    "type":    "clr",
    "clrType": "MyApp.Import.XmlImportHandler, MyApp",
    "async":   true
  }
}
```

## Notes

- Setting `model` to `""` skips database storage — useful when the file is only uploaded to Azure Blob without any SQL interaction.
- The `parse: "auto"` option detects the format from the file extension; use it when accepting multiple formats through one operation.
- `locale` affects how numbers (decimal separators) and dates are parsed from text-based formats (CSV, DBF). It has no effect on binary formats.
- For `azureBlob`, `azureSource` must match a connection string name in the application configuration that points to an Azure Storage account.
- `imageCompress` only applies to image files (JPEG/PNG); other file types are passed through unchanged.
- CLR handlers receive the raw file stream and are responsible for all processing; the `async` flag offloads execution to a background task.
