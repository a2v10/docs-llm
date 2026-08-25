# FileImage

> An image bound to data whose bytes come from an operation of the `files` section of model.json.

## Overview

`FileImage` displays a picture stored as a [binary object](https://docs-llm.a2v10.com/sql/blob.md). It differs from [Image](https://docs-llm.a2v10.com/xaml/controls/image.md) in where the bytes come from: `Url` points at an operation declared in the [files](https://docs-llm.a2v10.com/model/files.md) section of `model.json`, not at a model whose property name resolves to a `.Load` / `.Update` procedure pair.

`Value` is bound to the field that defines the image identifier. That identifier is what the file operation receives to locate the content.

`FileImage` inherits from `UIElementBase` — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).

## Use When

- The image is served by an upload operation already declared in the `files` section — the same endpoint that stored it also returns it.

## Do Not Use When

- The picture has its own `.Load` / `.Update` procedures and should be replaceable in place — use [Image](https://docs-llm.a2v10.com/xaml/controls/image.md) instead.
- The user has to choose and send a file — use [UploadFile](https://docs-llm.a2v10.com/xaml/controls/uploadfile.md) instead.

## Syntax

```xml
<FileImage Url="/catalog/product/photo" Value="{Bind Product.Photo}" Height="200px" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Url` | String | URL of the image. Must refer to the [files](https://docs-llm.a2v10.com/model/files.md) section of the model description file. |
| `Value` | [Bind](https://docs-llm.a2v10.com/xaml/bind.md) | Bound to the field that defines the image identifier. |
| `Width` | Length | Width of the image. |
| `Height` | Length | Height of the image. |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for all inherited properties.

## Notes

- Usually only one of `Width` and `Height` is set. The image then keeps its proportions.
- The `Url` is the endpoint path followed by the operation name from the `files` section — for `/catalog/product` with an operation named `photo`, that is `/catalog/product/photo`.
