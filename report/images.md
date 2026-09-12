# Report Images and Codes

> Pictures in a report — bytes from the model or a file of the application, raster or SVG, the page watermark, and the QrCode and Barcode elements.

## Overview

Wherever a template expects an image, the value may be one of two things, and the platform tells them apart by itself:

- bytes — a `varbinary(max)` field of the model, like any other field;
- a string — a file name, and the file is read from the application.

The kind of image is determined from the first bytes, not from the file extension: content that starts with `<svg` or `<?xml` is a vector image, anything else is a raster (png, jpeg and so on). An SVG stored in the database as `varbinary` is therefore drawn too, and a file extension means nothing.

A string in a workbook cell stays text and is not treated as a file name — otherwise every text cell would become a file reference. A file name is written where the template expects an image proper: `Image` and `Watermark`.

## Syntax

### Image

An image: a logo, a signature, a stamp.

| Property | Type | Description |
|----------|------|-------------|
| `Source` | Object | The image: bytes from the model (a binding) or a file name |
| `FileName` | String | The second spelling of the same thing: accepts a file name as well as bytes. Kept for compatibility with older forms |
| `Width` | [Length](https://docs-llm.a2v10.com/report/elements.md#length) | Image width |
| `Height` | [Length](https://docs-llm.a2v10.com/report/elements.md#length) | Image height |

Plus the properties of [the base element](https://docs-llm.a2v10.com/report/elements.md#base-element).

```xml
<!-- from the model -->
<Image Source="{Bind Company.Logo}" Height="20mm"/>

<!-- from a file of the application -->
<Image FileName="stamp.svg" Width="40mm"/>
<Image FileName="{Bind Document.SignFile}" Width="40mm"/>
```

A file name is counted from the report folder and may contain nested folders.

In a workbook the same thing is an ordinary cell: if the value turns out to be bytes, the cell draws a picture.

```js
{Company.Logo}
```

### Watermark

The `Watermark` property of [Page](https://docs-llm.a2v10.com/report/elements.md#page) takes the same kind of value — bytes or a file name — and draws it under the content of every page. An empty value means there is no watermark, so a watermark can be switched on by a condition:

```xml
<!-- always -->
<Page Watermark="draft.svg">

<!-- by condition: a function in the Code section, a call without spaces -->
<Page Watermark="{Bind watermarkOf(Document)}">
    <Page.Code>
        const watermarkOf = (doc) => doc.Done ? '' : 'draft.svg';
    </Page.Code>
</Page>
```

There are no separate properties for rotation, opacity or size: all of that is set inside the SVG itself, where it is a one-liner anyway.

### QrCode

A QR code built from a value. The value is converted to a string and encoded while printing. Content property: `Value`.

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | Content property: a constant string or a binding |
| `Size` | [Length](https://docs-llm.a2v10.com/report/elements.md#length) | The side of the square. When not specified, the code takes the available space |

```xml
<QrCode Size="25mm" Value="{Bind Document.QrData}"/>
```

### Barcode

A linear barcode of the EAN standard. Content property: `Value`.

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | Content property: the code, constant or from the model |
| `Type` | BarcodeType | `EAN13` (default) or `EAN8` |
| `Width` | [Length](https://docs-llm.a2v10.com/report/elements.md#length) | Width. The code is stretched to fill it |
| `Height` | Int32 | The height of the bars. A number, not a `Length` |
| `PrintDigits` | Boolean | Whether to print the digits under the bars. `True` by default |
| `Color` | String | Bar colour (a base element property) |

```xml
<Barcode Type="EAN13" Width="45mm" Value="{Bind Item.Barcode}"/>
```

## Example

A header with a logo from the model and a stamp from a file, and a page watermark for an unposted document:

```xml
<Page xmlns="clr-namespace:A2v10.Xaml.Report;assembly=A2v10.Xaml.Report"
      Watermark="{Bind watermarkOf(Document)}">
    <Page.Code>
        const watermarkOf = (doc) => doc.Done ? '' : 'draft.svg';
    </Page.Code>

    <Column>
        <Inlined>
            <Image Source="{Bind Company.Logo}" Height="15mm"/>
            <Text><Span Content="{Bind Company.Name}" Bold="True"/></Text>
        </Inlined>

        <Table Columns="1fr,40mm">
            <Table.Body>
                <TableRow>
                    <TableCell Content="{Bind Document.Memo}"/>
                    <TableCell>
                        <QrCode Size="25mm" Value="{Bind Document.QrData}"/>
                    </TableCell>
                </TableRow>
            </Table.Body>
        </Table>

        <Image FileName="stamp.svg" Width="40mm"/>
    </Column>
</Page>
```

## Notes

- In a compiled application files live inside the assembly rather than on disk, and the resource name is built from the path with separators replaced by dots. Folder names must therefore be plain identifiers — no spaces, hyphens or dots. A folder named `print forms` will not be found in a compiled application; `printforms` will.
- When the value is empty, `Image` simply draws nothing — no empty frame is left in its place.
- The watermark is drawn under the content, so a cell with its own background covers it: a solid fill is opaque. Leave header and total cells unfilled when the watermark has to show through.
- A workbook has no QrCode element of its own: there a cell prints a QR code when its expression returns `qrCode(...)` — `{(qrCode(Document.Number))}`.
- A barcode value is digits only: 12 or 13 for `EAN13`, 7 or 8 for `EAN8`. The shorter form means the engine computes the check digit; the longer one means it is already there and will be verified. Any other length, or a wrong check digit, stops printing with a message. No other barcode standards are supported.
- `Source` and `FileName` are two spellings of one property pair: both accept bytes and a file name alike.

## Hints

- An image stored in the database is an ordinary `varbinary(max)` field of the report model — see [Binary Objects](https://docs-llm.a2v10.com/sql/blob.md) for how such a field is loaded.
- To make one template serve both a draft and a final copy, keep the condition in a `Code` function and return an empty string for the final one: an empty watermark means none.
