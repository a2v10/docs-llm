# Change Tracking (rowversion)

> Detecting a concurrent edit — an `rv` column of type `rowversion`, carried through the table type and compared inside `.Update`.

## Overview

SQL Server supports a special column type whose value is updated every time the record is modified. The update happens regardless of how the record was changed, and even when the same value is written back. A column of this kind has the type `rowversion`.

A `rowversion` column can be used to track changes to an object. If, at the moment the model is saved, the value of the column differs from the one read from the database, the record has been modified some other way in the meantime and the update must be refused.

The platform supports reading and writing such a column, but its name must always be `rv`, in lowercase. The convention works by name alone: in the dataset it is an ordinary field with no marker, and the platform recognises it by its name both when the model is loaded and when it is saved.

All of the change tracking is done at the SQL level — the comparison and the refusal live in the `.Update` procedure, not in the client or in the markup.

## Use When

- Several users may open and save the same record, and silently overwriting somebody else's edit is unacceptable.
- A record is edited both by people and by background processing, and the form must not win by default.

## Syntax

Three pieces are involved.

| Where | What to add |
|-------|-------------|
| Table | `rv rowversion` — the tracked column |
| `.Load` | `d.rv` in the dataset — an ordinary field, no marker |
| Table type | `rv varbinary(8)` — `rowversion` is not supported inside a table type, so its equivalent is used |
| `.Update` | A comparison of the incoming `rv` with the stored one, and a `throw` when they differ |

## Example

### Reading the version

The version comes back with the model as a plain field — it carries no marker, the name is the whole convention.

```sql
select [Document!TDocument!Object] = null, [Id!!Id] = d.Id, d.rv,
    [Date] = d.[Date], d.[No], d.Memo, d.[Sum]
from doc.Documents d
where d.Id = @Id;
```

### Tracking the change

Change tracking for documents in the `doc.Documents` table.

```sql
/* first of all, add the rv column */
alter table doc.Documents add rv rowversion;
go

/* add the tracking column to the table type */
create type doc.[Document.TableType] as table
(
    Id bigint,
    rv varbinary(8), /* rowversion is not supported here — its equivalent is used */
    /* all the other columns */
);
go

/* the model update procedure */
create or alter procedure doc.[Document.Update]
@Document doc.[Document.TableType] readonly
as
begin
    set nocount on;
    set transaction isolation level read committed;

    /* check that the row versions match */
    if exists(
        select * from @Document t
        inner join doc.Documents d on d.Id = t.Id
        where t.rv is not null and t.rv <> d.rv
    )
        throw 60000, N'UI:Invalid row version', 0;

    /* update the documents table */
end
go
```

## Notes

- The column name is fixed: `rv`, lowercase. It is never marked up — the name is what the platform matches on, in the table, in the dataset, and in the table type alike.
- `rowversion` cannot be declared inside a table type — use `varbinary(8)`, which is the same eight bytes.
- The `t.rv is not null` part of the condition skips the check for rows that arrive without a version, so an insert is not rejected for having nothing to compare.
- The value changes on every modification of the record, even when a column is written with the value it already had. A save that changes nothing still advances the version.
- The check belongs at the very start of `.Update`, before any `MERGE` — see [Update Model (TVP + MERGE)](https://docs-llm.a2v10.com/sql/update-model.md) for the shape of the procedure it goes into.
- A failed version check is a normal thing for a user to run into, which is why the message carries the `UI:` prefix — the platform strips it and shows the rest as an application alert. See [Error Messages](https://docs-llm.a2v10.com/sql/errors.md).
