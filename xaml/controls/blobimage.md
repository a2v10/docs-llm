# BlobImage and BlobPreview

> BlobImage shows, uploads and clears the image in a record field; BlobPreview shows any platform-stored file read-only. .NET Core only.

## Overview

Both elements work with files kept by the platform — see [Files and Images](https://docs-llm.a2v10.com/sql/blob-storage.md). The model holds a file object with the fields `Id`, `Token`, `Name`, `Mime`.

`BlobImage` is the image in a record field: an agent's logo, a product photo. It shows the image, uploads a new one, and clears the field. An empty field shows a file drop area. After an upload the element writes `Id`, `Token`, and also `Name` and `Mime` if the object has them, into the bound field; the model becomes dirty, as after editing any field. The file reaches the record only when the model is saved — this works for a new record too.

`BlobPreview` shows a file and changes nothing: an image is displayed, a PDF opens in the browser's built-in viewer, any other file is shown as its name — a link that downloads it. It has no `Url`: the file is read by its own `Id` and token.

Both inherit `UIElementBase` — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).

## Use When

- `BlobImage` — one image per record, declared in the [blobs](https://docs-llm.a2v10.com/model/blobs.md) section of the endpoint.
- `BlobPreview` — viewing the selected attachment of a list, or any file object loaded with its token.

## Do Not Use When

- The file is imported into rows — use [UploadFile](https://docs-llm.a2v10.com/xaml/controls/uploadfile.md) instead.

## Syntax

```xml
<BlobImage Url="/catalog/agentblob" Value="{Bind Agent.Logo}" Height="10rem" />

<BlobPreview Value="{Bind Blob}" Height="100%" />
```

## Properties

### BlobImage

| Property | Type | Description |
|----------|------|-------------|
| `Url` | String | The endpoint — the folder whose `model.json` has the [blobs](https://docs-llm.a2v10.com/model/blobs.md) section. The key in `blobs` is not written here: it is taken from the field name in `Value` |
| `Value` | [Bind](https://docs-llm.a2v10.com/xaml/bind.md) | Required, always a binding. A field of the model root, `<model>.<Key>`, e.g. `Agent.Logo`. The element writes the upload result into it |
| `Width` | Length | Width of the image |
| `Height` | Length | Height of the image |

### BlobPreview

| Property | Type | Description |
|----------|------|-------------|
| `Value` | [Bind](https://docs-llm.a2v10.com/xaml/bind.md) | Required, always a binding. A file object with the fields `Id`, `Token`, `Name`, `Mime` |
| `Width` | Length | Width of the preview area |
| `Height` | Length | Height of the preview area |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for all inherited properties.

## Example

### An agent's logo

```xml
<BlobImage Url="/catalog/agentblob" Value="{Bind Agent.Logo}" Height="10rem" />
```

The matching `/catalog/agentblob/model.json`:

```json
{
  "schema": "cat",
  "model": "Agent",
  "blobs": {
    "Logo": { "accept": "image/png,image/jpeg", "maxSize": 512 }
  }
}
```

### Preview of the selected attachment

[Slot](https://docs-llm.a2v10.com/xaml/slot.md) renders nothing while no item is selected, and `Fallback` explains why the space is empty:

```xml
<Slot Scope="{Bind Attachments.$selected}">
  <BlobPreview Value="{Bind Blob}" Height="100%" />
  <Slot.Fallback>
    <EmptyPanel Content="No attachments for this item yet" />
  </Slot.Fallback>
</Slot>
```

## Notes

- Without a `Value` binding the view does not load and the system reports a markup error — for both elements.
- The ✕ button of `BlobImage` clears the field (`$empty()`).
- Usually only one of `Width` and `Height` is set on `BlobImage`. The image then keeps its proportions.
- `BlobImage` works only with a field of the model root; a file in an array row or a nested object is not supported.
