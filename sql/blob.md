# Binary Objects (blob)

> Images and attachments as separate entities — the `!Token` access marker and the dedicated `.Load` / `.Update` procedure pair that reads and writes the byte stream.

## Overview

A binary object — an image, an attachment — is a separate entity with a name (`Name`), a MIME type (`Mime`), and the data itself (`Stream`). It can be stored in a table of its own for each kind of element, or in one common attachment table with an additional link table.

To prevent unauthorized access to attachments, the platform uses access tokens. A token is a hash of a string built from the current session identifier, the user identifier, and an additional access key — a guid tied to that particular attachment. The simplest way to provide the key is a `uniqueidentifier` column with `newid()` as its default. Because the session takes part in the hash, the same attachment yields different token values in different browser sessions.

Working with binary objects consists of three almost independent parts:

1. Loading the object itself and its access token. The ordinary means of the platform are enough here, but the data stream cannot be loaded this way — separate tooling is needed for the bytes. To build the token, include a field with the special `!Token` modifier in the dataset: the platform takes its value, a guid, and puts an access token in its place.
2. Fetching the bytes from the server with that token — either to display an image, or simply to download the attachment.
3. Uploading data to the server. The data may be stored anywhere — in the database, in Azure Storage. The attachment gets an identifier, and the content can later be fetched by it. The identifier reaches the model right away but is persisted only when the model itself is saved. Along with the identifier, saving produces an access token, which also has to be written into the model — there is no need to store it, it is built anew on every load.

The bytes themselves are read and written by two dedicated procedures, one with the `.Load` suffix and one with `.Update`. These procedures are specific to binary data and do not follow the rules for building models — they take fixed parameters and return plain columns with no markers.

## Use When

- An image or an attachment is a separate entity whose bytes are served on demand rather than loaded with the model.
- The content must be protected from unauthorized access — the token is checked before any procedure is called.
- Data is kept in external storage and only its name is held in the database.

## Do Not Use When

- The upload is handled by an endpoint action — parsing a file into rows, storing it as raw binary, or sending it to Azure Blob Storage — configure the `files` section of `model.json` instead, see [File Upload](https://docs-llm.a2v10.com/model/files.md).

## Syntax

### Procedure names

The property that holds the binary object determines the procedure name suffix.

| Procedure | Purpose |
|-----------|---------|
| `[schema].[Model.Property.Load]` | Returns the bytes and the metadata of the object |
| `[schema].[Model.Property.Update]` | Writes the bytes and returns the new identifier and token |

For a `Product` model with an `Image` property in schema `a2`, the procedures are `a2.[Product.Image.Load]` and `a2.[Product.Image.Update]`.

### The `!Token` marker

| Marker | Description |
|--------|-------------|
| `[Token!!Token]` | Field holding the access key — its guid value is replaced by an access token |

See [SQL Markers](https://docs-llm.a2v10.com/sql/markers.md) for the full list of field modifiers.

### Parameters of `.Load`

Names are fixed, order does not matter.

| Parameter | Description |
|-----------|-------------|
| `@TenantId int` | Tenant identifier — in a multi-tenant environment only |
| `@UserId bigint` | Current user identifier |
| `@Id bigint` | Identifier of the data — the image or the attachment |

### Result of `.Load`

A single dataset. Column names matter, order does not.

| Column | Description |
|--------|-------------|
| `Id bigint` | Identifier of the data |
| `Mime nvarchar(255)` | MIME type of the data |
| `Name nvarchar(255)` | Name of the dataset |
| `Stream varbinary(max)` | The data stream itself. May be null |
| `BlobName nvarchar(max)` | Name of the set in external storage, Azure Storage for example. May be null |
| `Token uniqueidentifier` | The field the access token is built from |

The procedure must return either `BlobName` or `Stream`. If `BlobName` is set, the data is taken from external storage and the value of `Stream` is ignored.

### Parameters of `.Update`

| Parameter | Description |
|-----------|-------------|
| `@TenantId int` | Tenant identifier — in a multi-tenant environment only |
| `@UserId bigint` | Current user identifier |
| `@Name nvarchar(255)` | Name of the dataset |
| `@Mime nvarchar(255)` | MIME type of the data |
| `@Stream varbinary(max)` | The data stream, or null if the data is kept in external storage |
| `@BlobName nvarchar(max)` | Name of the set in external storage, or null |

### Result of `.Update`

A single dataset with two columns.

| Column | Description |
|--------|-------------|
| `Id bigint` | Identifier of the data |
| `Token uniqueidentifier` | The field the access token is built from |

The data returned by this procedure goes back to the client, which adds it to its own data model.

## Example

### The table

The access key is an ordinary `uniqueidentifier` column filled by default.

```sql
create table cat.ProductImages
(
    Id bigint identity(100, 1) not null
        constraint PK_ProductImages primary key,
    [Name] nvarchar(255) null,
    Mime nvarchar(255) null,
    [Stream] varbinary(max) null,
    BlobName nvarchar(max) null,
    [Token] uniqueidentifier not null
        constraint DF_ProductImages_Token default(newid())
);
```

### Loading the model

The model returns the identifier of the image and the field the token is built from — not the bytes.

```sql
create or alter procedure a2.[Product.Load]
@UserId bigint,
@Id bigint = null
as
begin
    set nocount on;
    set transaction isolation level read uncommitted;

    select [Product!TProduct!Object] = null,
        [Id!!Id] = p.Id, [Name!!Name] = p.[Name],
        [Image!TImage!RefId] = p.Image
    from cat.Products p
    where p.Id = @Id;

    select [!TImage!Map] = null, [Id!!Id] = i.Id, [Token!!Token] = i.[Token]
    from cat.ProductImages i
        inner join cat.Products p on p.Image = i.Id
    where p.Id = @Id;
end
```

The `Image` property of the model becomes an object of two fields — the value of `Token` is already an access token, not the guid stored in the table:

```json
{
    "Product": {
        "Id": 42,
        "Name": "Product name",
        "Image": {
            "Id": 112233,
            "Token": "hXFVVdnZRnZ5HKwNStO2qSSip_ziQ-IEnjVVQzyKzC8"
        }
    }
}
```

### Reading the bytes

The procedure name is built from the model name and the property name.

```sql
create or alter procedure a2.[Product.Image.Load]
@UserId bigint,
@Id bigint = null
as
begin
    set nocount on;
    set transaction isolation level read uncommitted;

    select i.Id, i.Mime, i.[Name], i.[Stream], i.BlobName, i.[Token]
    from cat.ProductImages i
    where i.Id = @Id;
end
```

### Writing the bytes

```sql
create or alter procedure a2.[Product.Image.Update]
@UserId bigint,
@Name nvarchar(255) = null,
@Mime nvarchar(255) = null,
@Stream varbinary(max) = null,
@BlobName nvarchar(max) = null
as
begin
    set nocount on;
    set transaction isolation level read committed;

    declare @rtable table (Id bigint, [Token] uniqueidentifier);

    insert into cat.ProductImages ([Name], Mime, [Stream], BlobName)
    output inserted.Id, inserted.[Token] into @rtable (Id, [Token])
    values (@Name, @Mime, @Stream, @BlobName);

    select Id, [Token] from @rtable;
end
```

The identifier and the token returned here are written into the model on the client. The link from `cat.Products` to the image is stored when the model itself is saved — see [Update Model (TVP + MERGE)](https://docs-llm.a2v10.com/sql/update-model.md).

### Binding the object in the view

The [Image](https://docs-llm.a2v10.com/xaml/controls/image.md) control displays and uploads the picture in one step: `Source` points at the model property, `Base` at the endpoint of the full model.

```xml
<Image Base="/catalog/product" Source="{Bind Product.Image}" Height="200px" />
```

An attachment can also be shown, downloaded, or printed by the `File` command — see [Bind & BindCmd](https://docs-llm.a2v10.com/xaml/bind.md). To let the user send a file to the server, use [UploadFile](https://docs-llm.a2v10.com/xaml/controls/uploadfile.md).

## Notes

- The access token is checked on the server and is never passed to the procedure. If it does not follow the rules, the procedure is not called at all.
- The token value differs between browser sessions, because the session identifier takes part in the hash. It is built anew on every load and does not need to be stored.
- The field marked `!Token` in the model must correspond to what the blob `.Load` procedure returns in its `Token` column.
- `.Update` takes no identifier — its parameter list consists of the tenant, the user, and the file itself.
- The name of the property holding the binary object defines the procedure name suffix; renaming the property renames the procedures.
- The blob procedures return plain columns without markers — the rules for building models do not apply to them.
- The attachment identifier appears in the model as soon as the upload finishes, but reaches the database only when the model is saved.

## Hints

Storing every kind of attachment in one common table works the same way — the link table then resolves which record an attachment belongs to, while the `.Load` and `.Update` procedures keep the same fixed signature.

When the bytes live in Azure Storage, keep `Stream` null and return `BlobName` alone: the platform reads the content from external storage and ignores `Stream` entirely.
