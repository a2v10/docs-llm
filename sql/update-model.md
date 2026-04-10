# Update Model (TVP + MERGE)

> How A2v10 saves form data: the platform calls `.Metadata` to learn the shape of the data, constructs typed table variables, and passes them to `.Update`.

## Overview

When the user submits a form, the client sends the complete model object to the server.
The platform first calls the `.Metadata` procedure, which returns empty result sets describing
how to map model properties to SQL table types (TVPs). The platform then constructs those TVPs
from the model data and passes them as parameters to `.Update`.

The `.Update` procedure uses `MERGE` to insert or update rows, then calls `.Load` at the end
to return the saved model to the client.

When a record has child rows (a detail table), two TVPs are used: one for the parent, one for
the children. Because the parent may not yet have a database `Id` at save time, the parent TVP
includes a client-generated `GUID` column, and the child TVP includes `ParentGUID` to reference it.

## Syntax

The `.Metadata` procedure returns one empty result set per TVP, using this marker format:

| Part          | Meaning                                                        |
|---------------|----------------------------------------------------------------|
| `paramName`   | Parameter name in `.Update` — `@Sample` becomes `Sample`      |
| `modelPath`   | Dot-notation path in the client model — `Sample`, `Sample.Items` |
| `Metadata`    | Fixed keyword — identifies this result set as a metadata descriptor |

```
[paramName!modelPath!Metadata]
```

When there are no child rows, `paramName` and `modelPath` are identical.
When child rows are present, they differ: `[Items!Sample.Items!Metadata]`.

> Drop order before recreation: `Metadata` → `Update` → child `TableType` → parent `TableType`.

## Example

### Simple catalog

A catalog without child rows. One TVP, one `MERGE`.

```sql
drop procedure if exists cat.[Sample.Metadata];
drop procedure if exists cat.[Sample.Update];
drop type if exists cat.[Sample.TableType];
go

------------------------------------------------
create type cat.[Sample.TableType] as table(
    Id     bigint null,
    [Name] nvarchar(255),
    [Memo] nvarchar(255)
);
go

------------------------------------------------
create or alter procedure cat.[Sample.Metadata]
as begin
    set nocount on;
    set transaction isolation level read uncommitted;
    declare @Sample cat.[Sample.TableType];
    select [Sample!Sample!Metadata] = null, * from @Sample;
end
go

------------------------------------------------
create or alter procedure cat.[Sample.Update]
@UserId bigint,
@Sample cat.[Sample.TableType] readonly
as begin
    set nocount on;
    set transaction isolation level read committed;

    declare @rtable table(id bigint);
    declare @id bigint;

    merge cat.Samples as t
    using @Sample as s on t.Id = s.Id
    when matched then update set
        t.[Name] = s.[Name],
        t.[Memo]  = s.[Memo]
    when not matched by target then insert
        ([Name], Memo) values (s.[Name], s.Memo)
    output inserted.Id into @rtable(id);

    select top(1) @id = id from @rtable;
    exec cat.[Sample.Load] @UserId = @UserId, @Id = @id;
end
go
```

### Catalog with child rows

Two TVPs, two MERGEs, wrapped in a transaction.

```sql
------------------------------------------------
drop procedure if exists cat.[Sample.Metadata];
drop procedure if exists cat.[Sample.Update];
drop type if exists cat.[Sample.Item.TableType];
drop type if exists cat.[Sample.TableType];
go

------------------------------------------------
create type cat.[Sample.TableType] as table(
    [GUID] uniqueidentifier,
    Id     bigint null,
    [Name] nvarchar(255),
    [Memo] nvarchar(255)
);
go

------------------------------------------------
create type cat.[Sample.Item.TableType] as table(
    ParentGUID uniqueidentifier,
    Id         bigint null,
    RowNo      int,
    [Name]     nvarchar(255),
    [Memo]     nvarchar(255)
);
go

------------------------------------------------
create or alter procedure cat.[Sample.Metadata]
as begin
    set nocount on;
    set transaction isolation level read uncommitted;
    declare @Sample cat.[Sample.TableType];
    declare @Items  cat.[Sample.Item.TableType];
    select [Sample!Sample!Metadata]      = null, * from @Sample;
    select [Items!Sample.Items!Metadata] = null, * from @Items;
end
go

------------------------------------------------
create or alter procedure cat.[Sample.Update]
@UserId bigint,
@Sample cat.[Sample.TableType] readonly,
@Items  cat.[Sample.Item.TableType] readonly
as begin
    set nocount on;
    set transaction isolation level read committed;
    set xact_abort on;

    declare @rtable table(id bigint, [guid] uniqueidentifier);
    declare @id bigint;

    begin tran;

    merge cat.Samples as t
    using @Sample as s on t.Id = s.Id
    when matched then update set
        t.[Name] = s.[Name],
        t.[Memo]  = s.[Memo]
    when not matched by target then insert
        ([Name], Memo) values (s.[Name], s.Memo)
    output inserted.Id, s.[GUID] into @rtable(id, [guid]);

    select top(1) @id = id from @rtable;

    with T as (
        select i.Id, i.[Name], i.Memo, [Sample] = t.Id, i.RowNo
        from @Items i
            inner join @rtable t on t.[guid] = i.ParentGUID
    )
    merge cat.SampleItems as t
    using T as s on t.Id = s.Id and t.[Sample] = @id
    when matched then update set
        t.RowNo  = s.RowNo,
        t.[Name] = s.[Name],
        t.[Memo]  = s.[Memo]
    when not matched by target then insert
        ([Sample], RowNo, [Name], Memo) values (s.[Sample], s.RowNo, s.[Name], s.Memo)
    when not matched by source and t.[Sample] = @id then delete;

    commit tran;

    exec cat.[Sample.Load] @UserId = @UserId, @Id = @id;
end
go
```

## Notes

- `.Update` must always end with a call to `.Load` — the platform expects the saved model in the response.
- `set xact_abort on` is required when using explicit transactions — it ensures automatic rollback on any error.
- The `when not matched by source ... then delete` clause is what removes child rows deleted on the client.
- `GUID` / `ParentGUID` are needed because the parent `Id` does not exist yet when inserting a new record with children.
- The `output` clause on the parent `MERGE` captures both `Id` and `GUID` so the child MERGE can resolve the parent reference.

## Hints

To inspect a TVP's contents during debugging, dump it to XML and raise as an error:

```sql
declare @xml nvarchar(max);
set @xml = (select * from @Rows for xml auto);
throw 60000, @xml, 0;
```