# Image

> Deprecated — use [BlobImage](https://docs-llm.a2v10.com/xaml/controls/blobimage.md) instead. The old data-bound image backed by a `.Load` / `.Update` procedure pair; kept only to read existing code.

## Overview

`Image` shows a picture stored as a [binary object](https://docs-llm.a2v10.com/sql/blob.md) and lets the user replace it. Displaying and uploading are handled by the one element — this is the simplified approach to working with images.

`Source` is bound to a model property that is an object of two fields, `Id!!Id` and `Token!!Token`. The name of that property determines the suffix of the stored procedures the platform calls to read and write the actual bytes: a property `Image` of a `Product` model means `a2.[Product.Image.Load]` and `a2.[Product.Image.Update]`.

`Base` points at the full model — the endpoint named in the root of `model.json`, not at a files operation.

To display the image the platform calls the `.Load` procedure, passing the identifier of the object bound to `Source` as `@Id`. The access token is verified on the server and is not passed to the procedure; if it does not follow the rules, the procedure is not called at all. When a new image is uploaded, the `.Update` procedure returns the identifier and a new access token, and both are updated in the model.

`Image` inherits from `UIElementBase` — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).

## Do Not Use When

- Any new code — use [BlobImage](https://docs-llm.a2v10.com/xaml/controls/blobimage.md) instead; to view a file read-only, BlobPreview on the same page.

## Syntax

```xml
<Image Base="/catalog/product" Source="{Bind Product.Image}" Height="200px" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Base` | String | URL of the model the image is taken from — the endpoint named in the root of `model.json`. |
| `Source` | [Bind](https://docs-llm.a2v10.com/xaml/bind.md) | Always a binding. Bound to the field that defines the image identifier. |
| `Width` | Length | Width of the image. |
| `Height` | Length | Height of the image. |
| `ReadOnly` | Boolean | Forbid uploading an image. |
| `Limit` | Int32 | Maximum size of the uploaded image, in KB. |
| `Placeholder` | String | Hint shown when no image is selected. |
| `Icon` | Icon | Icon shown together with `Placeholder` when no image is selected. |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for all inherited properties.

## Example

The view binds the picture and the endpoint it comes from:

```xml
<Image Base="/catalog/product" Source="{Bind Product.Image}" Height="200px" />
```

The endpoint is an ordinary model:

```json
{
  "schema": "a2",
  "model": "Product"
}
```

The loading procedure returns the identifier and the token — never the bytes:

```json
{
    "Product": {
        "Id": 42,
        "Image": {
            "Id": 112233,
            "Token": "hXFVVdnZRnZ5HKwNStO2qSSip_ziQ-IEnjVVQzyKzC8"
        }
    }
}
```

The bytes are served by `a2.[Product.Image.Load]` and written by `a2.[Product.Image.Update]` — both procedures are described in [Binary Objects (blob)](https://docs-llm.a2v10.com/sql/blob.md).

## Notes

- Usually only one of `Width` and `Height` is set. The image then keeps its proportions.
- The name `Image` in the procedure names comes from the property name, not from the element — renaming the model property renames the procedures.
- `Limit` is measured in kilobytes.
- The token in `Source` is built anew in every browser session, so it cannot be cached or stored on the client.
