# TextEditor and TextView

> A text editor with bold and images, and its read-only counterpart; pasted images are stored as attachments of the record. .NET Core and the Tabbed layout only.

## Overview

`TextEditor` edits text with bold and images: a task description, a comment. Images are pasted from the clipboard or dropped onto the editor and are stored as [attachments](https://docs-llm.a2v10.com/sql/blob-storage.md) of the record. `TextView` shows text written in `TextEditor` and changes nothing: line breaks, bold, images; `http(s)://` links open in a new tab, and an image is a link to the original.

The text is a plain string with three markup elements:

- a line break — `\n`;
- bold — `**text**` (Ctrl+B);
- an image — `![name](address)`, the address is returned by `UploadImage`.

There is nothing else: Ctrl+I and Ctrl+U do nothing, pasted HTML goes in as plain text. An `http(s)://` link is text: the editor highlights it and opens it on Ctrl+click (⌘+click); `TextView` makes it clickable. An `nvarchar(max)` column is enough to store the text.

Images and the editor's other abilities come from a delegates object the template provides. No function — no ability.

Both inherit `UIElementBase` — see [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md).

## Use When

- Free text that needs bold or inline screenshots — a task description, a comment thread.

## Do Not Use When

- Plain text with no markup — use [TextBox](https://docs-llm.a2v10.com/xaml/controls/textbox.md) with `Multiline` instead.
- One image per record, not inside text — use [BlobImage](https://docs-llm.a2v10.com/xaml/controls/blobimage.md) instead.

## Syntax

```xml
<TextEditor Value="{Bind Task.Text}" Placeholder="Task description" Delegates="{Bind Root.$TextDelegates}" />

<TextView Value="{Bind Text}" Delegates="{Bind Root.$TextDelegates}" />
```

## Properties

### TextEditor

| Property | Type | Description |
|----------|------|-------------|
| `Value` | [Bind](https://docs-llm.a2v10.com/xaml/bind.md) | Required, always a binding. The string property the editor writes the text into |
| `Placeholder` | String | Hint text in an empty editor |
| `Disabled` | Boolean | Forbid editing. A read-only model disables the editor by itself, as it does any control |
| `Delegates` | Object | A binding to the object with the functions `UploadImage`, `ImageUrl`, `Submit` |

### TextView

| Property | Type | Description |
|----------|------|-------------|
| `Value` | [Bind](https://docs-llm.a2v10.com/xaml/bind.md) | Required, always a binding. The text to show |
| `Delegates` | Object | A binding to the same delegates object as `TextEditor`. Only `ImageUrl` is used; without it images are shown as text |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for all inherited properties.

### Delegates

The editor calls the functions the way it calls template functions: `this` is the page model.

| Function | Description |
|----------|-------------|
| `UploadImage(file: File): Promise<string \| null>` | Uploads a pasted image (PNG, JPEG, GIF, WebP). Returns the string written into the text as the image address, or `null` — the image is then removed. Without this function images cannot be pasted |
| `ImageUrl(src: string): string \| null` | Returns the address the browser shows the image `src` from the text at, or `null` — the image stays text. Called while rendering, so it must only read the model. Without this function images are shown as text |
| `Submit()` | Called on Ctrl+Enter (⌘+Enter) — for example, to send a comment |

## Example

A task with a description; endpoint `/task`. Attachments go through the `/attachment` endpoint of [Files and Images](https://docs-llm.a2v10.com/sql/blob-storage.md), with `base` = `/task`.

An image is an attachment of the text's owner from the moment it is pasted, not from when the text is saved. The text stores an address without a token, such as `_blob/1234`: a token depends on the user, and an address with a token would open only for its author. Every reader gets an address of their own from `ImageUrl`, which looks the file up among the attachments in the model.

### Loading

The task is loaded with its attachments and their tokens. Tokens are issued only for the task's own attachments, so a foreign `Id` typed into the text by hand stays text.

```sql
select [Task!TTask!Object] = null, [Id!!Id] = t.Id, [Name!!Name] = t.[Name], t.[Text]
from cat.Tasks t
where t.Id = @Id;

select [Attachments!TAttachment!Array] = null, [Id!!Id] = l.Id, [Blob!TBlob!RefId] = l.[Blob]
from a2blob.BlobLinks l
where l.[Base] = N'/task' and l.LinkId = @Id
order by l.Id;

select [!TBlob!Map] = null, [Id!!Id] = b.Id, b.[Name], b.Mime, [Token!!Token] = b.AccessToken
from a2blob.Blobs b
    inner join a2blob.BlobLinks l on b.Id = l.[Blob]
where l.[Base] = N'/task' and l.LinkId = @Id;
```

### Template

```js
const BASE = '/task';
const urltools = require('std:url');

const template = {
  options: {
    // attachments are saved by the upload, not by the page's Save button
    skipDirty: ['Attachments']
  },
  properties: {
    'TRoot.$TextDelegates'() { return textDelegates; }
  }
};

// created once: a new object on every read would re-render every text
const textDelegates = {
  async UploadImage(file) {
    // an attachment needs an owner: a new task is saved first
    if (!await this.$ctrl.$ensureSaved())
      return null;
    const row = await this.$ctrl.$uploadBlobFile('/attachment/upload', file, { Id: this.Task.Id, Base: BASE });
    if (!row)
      return null;
    // before returning: the editor calls ImageUrl right away
    this.Attachments.$append(row);
    return `_blob/${row.Blob.Id}`;
  },
  ImageUrl(src) {
    const m = /^_blob\/(\d+)$/.exec(src);
    const a = m && this.Attachments.find(x => x.Blob.Id == +m[1]);
    return a ? urltools.blobUrl(a.Blob) : null;
  }
};

module.exports = template;
```

`require('std:url').blobUrl(blob)` returns the file address for a file object of the model (`Id` and `Token`), or `undefined` for an empty reference. [`$ensureSaved`](https://docs-llm.a2v10.com/client/controller.md) and [`$uploadBlobFile`](https://docs-llm.a2v10.com/client/controller.md) are controller methods.

### View

```xml
<TextEditor Value="{Bind Task.Text}" Placeholder="Task description" Delegates="{Bind Root.$TextDelegates}" />
```

### A comment list

Each comment is shown as text and, by a button, edited in `TextEditor`:

```xml
<Repeater ItemsSource="{Bind Comments}">
  <StackPanel Gap=".5rem">
    <TextView Value="{Bind Text}" Delegates="{Bind Root.$TextDelegates}" If="{Bind !$$Edit}" />
    <TextEditor Value="{Bind $$Draft}" Delegates="{Bind Root.$TextDelegates}" If="{Bind $$Edit}" />
  </StackPanel>
</Repeater>
```

## Notes

- Without a `Value` binding the view does not load and the system reports a markup error.
- Create the delegates object once in the template and return it from a property by reference.
- While an image uploads it is semi-transparent. The editor does not report an "uploading" state: the `UploadImage` promise belongs to the template, so the template knows the upload is in progress and can disable its own buttons meanwhile.
- An image removed from the text stays an attachment of the record.
- An attachment removed from the list but still mentioned in the text is shown as the text `![name](_blob/…)`.
