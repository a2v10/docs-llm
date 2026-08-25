# Error Messages

> How a stored procedure reports a failure — `throw`, and the `UI:` prefix that decides whether the text is shown to the user or to the developer.

## Overview

A stored procedure reports a failure the ordinary way, with `throw`. Execution stops there, and the message text is delivered to the client. In a procedure that opens an explicit transaction with `set xact_abort on` — as the save procedures do — the transaction is rolled back.

Failures come in two different kinds, and the platform tells them apart by the message text itself. Some are addressed to the user: not enough stock, the document has already been posted, somebody else has changed the record. Others are addressed to the developer: a broken internal assumption, a diagnostic dump. They must not be shown the same way.

A message that starts with `UI:` is meant for the user. The platform strips the prefix and shows the rest as a normal application alert — the same dialog as the [`$alert`](https://docs-llm.a2v10.com/client/controller.md) method of the controller. This is an acceptable path from the UI/UX point of view.

A message without the prefix is delivered as a plain browser alert. It looks raw, which is exactly the point: such a message is addressed to the developer, not to the user.

## Syntax

```sql
throw 60000, N'UI:<message text>', 0;
```

| Message text | What the user gets |
|--------------|--------------------|
| `N'UI:Not enough stock'` | An application alert reading `Not enough stock` |
| `N'Not enough stock'` | A plain browser alert with the raw text |

## Example

A guard at the start of an update procedure — a posted document may not be edited. The user can run into this during normal work, so the message carries the prefix:

```sql
create or alter procedure doc.[Document.Update]
@UserId bigint,
@Document doc.[Document.TableType] readonly
as
begin
    set nocount on;
    set transaction isolation level read committed;

    if exists(
        select * from @Document t
            inner join doc.Documents d on d.Id = t.Id
        where d.Done = 1
    )
        throw 60000, N'UI:The document is already posted and cannot be changed', 0;

    /* update the documents table */
end
```

A diagnostic dump, on the other hand, is left without the prefix — it is for the developer reading it during debugging:

```sql
declare @xml nvarchar(max);
set @xml = (select * from @Document for xml auto);
throw 60000, @xml, 0;
```

## Notes

- Only the prefix is removed; the rest of the text is shown exactly as written, so it should read as a finished sentence for the user.
- Everything the user can run into during normal work should carry `UI:`. Leave the prefix off only for what must never happen in a working application.
- A failed row version check is a user-facing case — see [Change Tracking (rowversion)](https://docs-llm.a2v10.com/sql/rowversion.md).
- The same rule applies to any procedure the platform calls, not only to `.Update` — a [command](https://docs-llm.a2v10.com/model/commands.md) declared in `model.json` reports failures the same way.
- For the shape of the save procedure these guards live in, see [Update Model (TVP + MERGE)](https://docs-llm.a2v10.com/sql/update-model.md).
