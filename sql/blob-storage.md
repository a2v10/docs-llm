# Files and Images

> Platform-managed file storage — a file in a record field or any number of attachments to a record; the application holds only a reference (`Id` + access token), never the bytes. .NET Core only.

## Overview

The platform stores files itself, checks them, and serves them to the client. The application sees a file only as a reference: its `Id` and an access token. The bytes are never in the application's tables, its procedures, or the model.

There are two scenarios:

- A file in a record field — one file per record: an agent's logo, a product photo. The reference is a column of the application table.
- Attachments — any number of files on any record. The references live in the platform link table `a2blob.BlobLinks`.

The platform script creates two tables in the default database: `a2blob.Blobs` (the files) and `a2blob.BlobLinks` (attachment links). Where the bytes themselves live is decided by the host configuration, not by the application: the same `model.json` and code work with any storage provider.

## Use When

- A record needs a logo, a photo, a scanned document, or attachments — this is the way to store files; the platform takes over storage, type and size checks, and tokens.

## Do Not Use When

- The file is imported into rows (Excel, CSV, DBF, XML) — use the `parse` operation of [File Upload](https://docs-llm.a2v10.com/model/files.md) instead.

## Syntax

### Storage

The provider is set in `appsettings.json` of the host:

```json
"BlobStorage": {
  "Provider": "FileSystem",
  "Path": "d:/data/blobs"
}
```

| Key | Description |
|-----|-------------|
| `Provider` | Name of a storage registered by the host |
| `Path` | Folder for the `FileSystem` storage |

Storages are registered in `Startup.cs`:

```cs
services.AddBlobStorages(factory =>
{
    factory.RegisterStorage<FileSystemBlobStorage>("FileSystem");
    factory.RegisterStorage<AzureBlobStorage>("AzureStorage");
});
```

### Application key type

The attachment table references application records, so the platform needs the type of their keys. The application declares it once, in `_sql/platformid.sql` of the main module:

```sql
if type_id(N'dbo.platformid') is null
    create type dbo.platformid from bigint;
go
```

Allowed base types: `bigint`, `int`, `uniqueidentifier`. The file goes first in `sql.json` of the host that builds `full.sql`, before the platform scripts:

```json
"inputFiles": [
  "../MainApp/_sql/platformid.sql",
  "_assets/sql/a2v10_platform_simple.sql",
  ...
]
```

### The file object in the model

A file reference is a `!RefId` to a [Map set](https://docs-llm.a2v10.com/sql/object.md) of file objects with four fields:

| Field | Source column | Description |
|-------|---------------|-------------|
| `[Id!!Id]` | `a2blob.Blobs.Id` | File identifier |
| `Name` | `a2blob.Blobs.Name` | File name |
| `Mime` | `a2blob.Blobs.Mime` | MIME type |
| `[Token!!Token]` | `a2blob.Blobs.AccessToken` | Access token — the field must be named exactly `Token` |

### Attachment link table

| Column | Description |
|--------|-------------|
| `Id` | Link identifier |
| `Base` | Endpoint URL of the owner, e.g. `/catalog/agentblob` |
| `LinkId` | `Id` of the owner record |
| `Blob` | Reference to `a2blob.Blobs` |

## Example

### File in a record field

An agent with a logo; endpoint `/catalog/agentblob`. The reference is a `bigint` column with a foreign key to `a2blob.Blobs`:

```sql
create table cat.Agents
(
    Id bigint not null constraint PK_Agents primary key ...,
    [Name] nvarchar(255),
    ...
    Logo bigint null
        constraint FK_Agents_Logo_Blobs foreign key references a2blob.Blobs(Id)
);
```

The [blobs](https://docs-llm.a2v10.com/model/blobs.md) section declares the field a file can be uploaded into — the key `Logo` means `Agent.Logo`:

```json
{
  "schema": "cat",
  "model": "Agent",
  "dialogs": {
    "edit": {
      "view": "edit.dialog"
    }
  },
  "blobs": {
    "Logo": { "accept": "image/png,image/jpeg,image/svg+xml", "maxSize": 512 }
  }
}
```

`.Load` returns the field as a reference and the file object as a separate Map set:

```sql
select [Agent!TAgent!Object] = null, [Id!!Id] = a.Id, [Name!!Name] = a.[Name], a.Memo,
    [Logo!TBlob!RefId] = a.Logo
from cat.Agents a
where a.Id = @Id;

select [!TBlob!Map] = null, [Id!!Id] = b.Id, b.[Name], b.Mime, [Token!!Token] = b.AccessToken
from a2blob.Blobs b
    inner join cat.Agents a on b.Id = a.Logo
where a.Id = @Id;
```

`.Update` gets a `Logo bigint` column in the table type and writes it as an ordinary field — nothing file-specific.

The view shows the image and uploads a new one with [BlobImage](https://docs-llm.a2v10.com/xaml/controls/blobimage.md):

```xml
<BlobImage Url="/catalog/agentblob" Value="{Bind Agent.Logo}" Height="10rem" />
```

### Attachments

One endpoint serves the attachments of the whole application. The owner is identified by two values: `base` — the owner's endpoint URL — and the record `Id`. Endpoint `/attachment`:

```json
{
  "schema": "cat",
  "model": "Attachment",
  "requireBase": true,
  "dialogs": {
    "index": {
      "index": true,
      "template": "index.template",
      "view": "index.dialog"
    }
  },
  "blobs": {
    "Upload": { "maxSize": 20480 }
  },
  "commands": {
    "delete": {
      "type": "sql",
      "procedure": "Attachment.Delete"
    }
  }
}
```

With `requireBase` every call must carry `base`; the platform checks the roles of the `model.json` at that URL before calling the procedure. The key `Upload` is not a model field — just the name uploads go to.

The owner's form opens the dialog. `Argument` gives the owner `Id`; `SaveRequired` saves a new record so that `Id` exists:

```xml
<Button Content="Attachments" Icon="Attach"
  Command="{BindCmd Dialog, Action=Show, Argument={Bind Agent},
    Url='/attachment/index?base=/catalog/agentblob', SaveRequired=True}" />
```

`.Index` receives the owner `@Id` and `@Base` and returns the files the same way `.Load` does above. `Delete` removes a link by all three values — never by the link `Id` alone:

```sql
create or alter procedure cat.[Attachment.Index]
@UserId bigint,
@Id bigint,
@Base nvarchar(255)
as
begin
    set nocount on;
    set transaction isolation level read uncommitted;

    select [Owner!TOwner!Object] = null, [Id!!Id] = @Id, [Base] = @Base;

    select [Attachments!TAttachment!Array] = null, [Id!!Id] = l.Id, [Blob!TBlob!RefId] = l.[Blob]
    from a2blob.BlobLinks l
    where l.[Base] = @Base and l.LinkId = @Id
    order by l.Id;

    select [!TBlob!Map] = null, [Id!!Id] = b.Id, b.[Name], b.Mime, [Token!!Token] = b.AccessToken
    from a2blob.Blobs b
        inner join a2blob.BlobLinks l on b.Id = l.[Blob]
    where l.[Base] = @Base and l.LinkId = @Id;
end
go

create or alter procedure cat.[Attachment.Delete]
@UserId bigint,
@Id bigint,
@Base nvarchar(255),
@Owner bigint
as
begin
    set nocount on;
    set transaction isolation level read committed;

    delete from a2blob.BlobLinks where Id = @Id and [Base] = @Base and LinkId = @Owner;
end
go
```

The dialog lists the files and previews the selected one. [Slot](https://docs-llm.a2v10.com/xaml/slot.md) hides [BlobPreview](https://docs-llm.a2v10.com/xaml/controls/blobimage.md) while nothing is selected, and `Fallback` explains the empty space:

```xml
<Dialog xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.Xaml" Title="Attachments" Width="60rem" Height="30rem">
  <Dialog.Buttons>
    <Button Content="@[Close]" Command="{BindCmd Close}" />
  </Dialog.Buttons>
  <Grid Columns="20rem,1*" Rows="1*" Height="100%" AlignItems="Stretch">
    <Grid Rows="Auto,1*" Height="100%" AlignItems="Stretch">
      <Toolbar>
        <Button Icon="Upload" Content="Upload" Command="{BindCmd Execute, CommandName=upload}" />
        <Button Icon="Delete" Command="{BindCmd Execute, CommandName=remove,
          Argument={Bind Attachments.$selected}, Confirm='@[Confirm.Delete.Element]'}" />
      </Toolbar>
      <List ItemsSource="{Bind Attachments}" AutoSelect="FirstItem">
        <ListItemSimple Content="{Bind Blob.Name}" />
      </List>
    </Grid>
    <Slot Scope="{Bind Attachments.$selected}">
      <BlobPreview Value="{Bind Blob}" Height="100%" />
      <Slot.Fallback>
        <EmptyPanel Content="No attachments for this item yet" />
      </Slot.Fallback>
    </Slot>
  </Grid>
</Dialog>
```

The template uploads with [`$uploadBlob`](https://docs-llm.a2v10.com/client/controller.md), passing the `Owner` object returned by `.Index` as `{ Id, Base }`. The result is the new row `{ Id, Blob: { Id, Token, Name, Mime } }`; what to do with it is up to the application:

```js
const template = {
  commands: {
    async upload() {
      const result = await this.$ctrl.$uploadBlob('/attachment/upload', null, this.Owner);
      if (!result)
        return;
      const row = this.Attachments.$append(result);
      row.$select();
    },
    async remove(row) {
      if (!row)
        return;
      await this.$ctrl.$invoke('delete', { Id: row.Id, Base: this.Owner.Base, Owner: this.Owner.Id });
      row.$remove();
    }
  }
};
```

The `delete` command passes `Base` in its data; the platform checks exactly the `Base` the procedure receives.

### Deleting the owner

`LinkId` has no foreign key to the owner record, so links outlive the owner. The procedure that deletes an owner permanently must remove its links:

```sql
delete from a2blob.BlobLinks where [Base] = N'/catalog/agentblob' and LinkId = @Id;
```

## Notes

- Every database needs `dbo.platformid`, even one whose application never works with files. Without it the platform script stops at its first statement with `dbo.platformid is not declared`.
- `dbo.platformid` is declared once for the life of the database: a column on an alias type cannot be changed.
- An endpoint with a `blobs` section cannot have its own `source` — the `a2blob` tables are in the default database.
- Without a provider, with an unregistered provider name, or without `AddBlobStorages`, uploading and reading a file fail with an error.
- Before `.Update` the platform checks every non-empty reference of the `blobs` section: it must arrive with its own token. An honest reference always has one — from `.Load` or from the upload result. Writing a foreign `Id` from script (`this.Agent.Logo.Id = 1003`) fails the save with `Invalid blob token`. Clearing the reference (`Agent.Logo.$empty()`) is always allowed.
- Only fields of the model root are checked (`Agent.Logo`). A file in a document row (`Document.Rows[].Photo`) or in a nested object (`Agent.Contact.Photo`) is not supported.
- An uploaded file appears on the form at once and reaches the record only when the model is saved. Closing the dialog without saving leaves the record unchanged. A new record can be saved together with its file.
- The roles of `base` stop only a user who has no access to any owner. Every attachment procedure therefore filters by `@Base` and the owner, so a foreign `base` or a foreign link `Id` finds nothing. Checking that this user may not see document 42 is the application procedures' job, as everywhere.
- Marking an owner deleted (`Void`) keeps its links, so the record can be restored.
- The `base` value lives in two places — the dialog URL and the owner's delete procedure. Change one, change the other.
- There is no explicit file deletion. A replaced logo, a file uploaded without saving, a removed link — all stay in the storage.
- A file is immutable: replacing it always yields a new `Id`.
- An image from a file cannot be printed in [reports](https://docs-llm.a2v10.com/report/overview.md) yet.

## Hints

- An attachment needs its own column, such as a comment or a category: the application adds it in its own schema script — `alter table a2blob.BlobLinks add Memo nvarchar(255) null`, guarded by a check that the column does not exist yet — and writes it with a separate call.
- Images inside a text field are attachments too — see [TextEditor and TextView](https://docs-llm.a2v10.com/xaml/controls/texteditor.md).
