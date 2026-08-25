# UploadFile

> A file selection and upload field with drag-and-drop, sending the file to an operation of the `files` section and reporting the result to a delegate.

## Overview

`UploadFile` renders an area where the user picks a file — by clicking or by dragging it onto the element. The content of the selected file is sent to the server and processed by the operation named in `Url`, which is an entry of the [files](https://docs-llm.a2v10.com/model/files.md) section of `model.json`.

The `Url` is the endpoint path followed by the operation name: `/catalog/product/import` refers to the operation `import` of the `/catalog/product` endpoint. What happens to the file afterwards — parsing into rows, storing as binary, sending to Azure Blob Storage — is decided entirely by that operation, not by the control.

When the upload and the processing are finished, the platform calls the [delegate](https://docs-llm.a2v10.com/template/delegates.md) named in `Delegate`, passing it the result returned by the server. `Delegate` is mandatory — an `UploadFile` element without it raises an error while the markup is being compiled. Errors go to `ErrorDelegate`; if none is set, a standard error message is displayed.

`UploadFile` inherits from `UIElementBase` — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).

## Use When

- The user has to send a file to the server — an import, an attachment, a document scan.
- The result of the processing has to be handled in code: reloading a list, raising an event, showing what was imported.

## Do Not Use When

- The point is to display an existing image rather than send a new file — use [Image](https://docs-llm.a2v10.com/xaml/controls/image.md) or [FileImage](https://docs-llm.a2v10.com/xaml/controls/fileimage.md) instead.
- The user should download a static file shipped with the application — use the `Download` command of [BindCmd](https://docs-llm.a2v10.com/xaml/bind.md) instead.

## Syntax

```xml
<UploadFile Url="/catalog/product/import" Delegate="uploadFile"
            Accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Url` | String | Path to the file upload command — an operation of the [files](https://docs-llm.a2v10.com/model/files.md) section. |
| `Argument` | Object | **Binding only.** Object whose properties become parameters of the upload command. A value written directly in the markup is ignored. |
| `Accept` | String | Filter string for the file selection dialog — MIME types of the accepted files, comma-separated. Corresponds to the `accept` attribute of `<input type="file">`. Support depends on the browser. |
| `Limit` | Int32 | Maximum size of the uploaded file, in kilobytes. If not set, files of any size may be uploaded. |
| `Delegate` | String | Name of the [delegate](https://docs-llm.a2v10.com/template/delegates.md) called after the file has been uploaded. Its argument is the result received from the server. Required — omitting it is a markup compilation error. |
| `ErrorDelegate` | String | Name of the [delegate](https://docs-llm.a2v10.com/template/delegates.md) called when an error occurs. If not set, a standard error message is shown. |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for all inherited properties.

## Delegate

```js
upload(this: IRoot, result: object): void
```

| Argument | Description |
|----------|-------------|
| `this` | The root data object — see [IRoot](https://docs-llm.a2v10.com/client/root.md) |
| `result` | The result returned by the server after the file has been processed |

## Example

Importing products from an Excel file. The dialog holds the upload field and a link to a sample file:

```xml
<Dialog>
  <UploadFile Url="/catalog/product/import" Delegate="uploadFile"
              Accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" />
  <Hyperlink Content="@[File.Sample]" Margin="1rem,0" Block="True"
             Command="{BindCmd Command=Download, Url='/entity_import_sample.xlsx'}" />
</Dialog>
```

The `import` operation in `/catalog/product/model.json` decides how the file is processed:

```json
{
  "files": {
    "import": {
      "type":  "parse",
      "parse": "xlsx",
      "model": "Product.Import"
    }
  }
}
```

The template handles what comes back — here it notifies the calling page that the import is done:

```ts
let indexPage;

const template: Template = {
  events: {
    'Model.load': modelLoad
  },
  delegates: {
    uploadFile
  }
};

export default template;

function modelLoad(root, caller) {
  indexPage = caller;
}

function uploadFile(result) {
  indexPage?.$emit('$product.import.done');
}
```

## Notes

- `Accept` is only a recommendation to the browser. The user cannot be prevented from choosing a file of another type — validate on the server.
- The element supports drag-and-drop as well as click-to-select; both paths send the file the same way.
- `Limit` is measured in kilobytes, and omitting it means no size limit at all.
- The dialog that contains the upload field often needs no data model of its own — set `"model": ""` for it in `model.json`.
- `Argument` only works as a binding. Writing a literal object into the markup has no effect — the value is ignored.
