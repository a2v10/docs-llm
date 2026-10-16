# Blobs

> The `blobs` object in model.json — declares where in this endpoint a file can be uploaded, with optional type and size limits. .NET Core only.

## Overview

The `blobs` object declares the upload targets of an endpoint for the platform-managed [Files and Images](https://docs-llm.a2v10.com/sql/blob-storage.md) storage.

In an ordinary endpoint each key is the name of a field of the model root: with `"model": "Agent"` the key `Logo` means `Agent.Logo`. On save the platform checks the reference in that field. Case does not matter; PascalCase is customary, as in SQL and XAML.

In an endpoint with `requireBase` the key is just the name uploads go to (`$uploadBlob('/attachment/upload', null, owner)`) — not a model field. Such an endpoint serves attachments: every call carries `base`, the owner's endpoint URL, and the platform checks the roles of the `model.json` at that URL before calling the procedure.

The right to upload is given by the roles of the endpoint.

## Do Not Use When

- The file is parsed into rows or handled by .NET code — use [File Upload](https://docs-llm.a2v10.com/model/files.md) instead.

## Syntax

```json
"blobs": {
  "<Key>": { "accept": "", "maxSize": 0 }
}
```

| Property | Type | Description |
|----------|------|-------------|
| `accept` | string | Optional. Allowed MIME types, comma-separated; `image/*` is allowed. Omitted — any type |
| `maxSize` | number | Optional. Maximum file size in KB. Omitted — no limit |

The root property that makes an endpoint an attachment endpoint, see [model.json Overview](https://docs-llm.a2v10.com/model/overview.md):

| Property | Type | Description |
|----------|------|-------------|
| `requireBase` | boolean | Every call must carry the `base` parameter — the owner's endpoint URL (`/attachment/index?base=/catalog/agentblob`); otherwise an error. The roles of `model.json` at `base` are checked before the procedure is called; no `model.json` there is an error. An upload to `blobs` links the file to the owner |

## Example

### A file in a record field

```json
{
  "schema": "cat",
  "model": "Agent",
  "blobs": {
    "Logo": { "accept": "image/png,image/jpeg", "maxSize": 512 }
  }
}
```

### Attachments

```json
{
  "schema": "cat",
  "model": "Attachment",
  "requireBase": true,
  "blobs": {
    "Upload": { "maxSize": 20480 }
  }
}
```

## Notes

- Both limits are checked by the server. A file that violates them is not stored, and the user sees an error message.
- For images (PNG, JPEG, GIF, WebP) and PDF the type is determined from the file content; for other files, from the extension of the name. What the browser reports is ignored.
- An endpoint with a `blobs` section cannot have its own `source`.
- The `accept` argument of [`$uploadBlob`](https://docs-llm.a2v10.com/client/controller.md) only filters the file picker; the server still checks `accept` from `model.json`.
