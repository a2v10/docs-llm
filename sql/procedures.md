# Stored Procedures

> The procedures the platform calls for a model — how their names are built from the model name, what parameters they receive, and which suffix serves which action.

## Overview

The platform knows nothing about tables, column names, or any other technical detail of the database. It calls stored procedures, and they return or receive data in the agreed format. A model is a business entity — a customer, a supplier, a document — and the procedures are the whole of the contract between it and the database.

There are only four things the system can do with a model: load a list of instances, load a single instance, save an instance, and execute a command. Each of the first three is served by a procedure whose name is the model name plus a fixed suffix.

The schema and the model name come from the endpoint description — the `schema` and `model` properties of [model.json](https://docs-llm.a2v10.com/model/overview.md). A model named `Agent` in schema `cat` is loaded by `cat.[Agent.Load]`, and its list by `cat.[Agent.Index]`.

Everything the platform learns about the shape of the data it gets from the names of the returned columns, not from their values — which is why a procedure that returns no rows still describes a complete model. See [SQL Markers](https://docs-llm.a2v10.com/sql/markers.md).

## Syntax

```
[schema].[ModelName.Suffix]
```

| Suffix | Purpose |
|--------|---------|
| `.Index` | Load a list of instances. See [Paging & Filters](https://docs-llm.a2v10.com/sql/paging.md) |
| `.Load` | Load a single instance. See [Objects & References](https://docs-llm.a2v10.com/sql/object.md) |
| `.Metadata` | Return the metadata that describes how to turn the model into table types. See [Update Model (TVP + MERGE)](https://docs-llm.a2v10.com/sql/update-model.md) |
| `.Update` | Save the instance. See [Update Model (TVP + MERGE)](https://docs-llm.a2v10.com/sql/update-model.md) |
| `.Expand` | Return the child elements of a node when the user expands it in a dynamic tree. See [Tree & Hierarchy](https://docs-llm.a2v10.com/sql/tree.md) |

Binary objects are served by their own pair of procedures, named after the model property rather than the model alone — `[schema].[Model.Property.Load]` and `[schema].[Model.Property.Update]`. See [Binary Objects (blob)](https://docs-llm.a2v10.com/sql/blob.md).

### Parameters

Parameter names are fixed; the order does not matter.

| Parameter | Where | Description |
|-----------|-------|-------------|
| `@UserId bigint` | Everywhere | Identifier of the current user |
| `@TenantId int` | Everywhere | Tenant identifier — in a multi-tenant environment only |
| `@Id bigint` | `.Load`, `.Expand` | Identifier of the instance being loaded or expanded |
| `@Offset`, `@PageSize`, `@Order`, `@Dir` | `.Index` | Paging and sorting — see [Paging & Filters](https://docs-llm.a2v10.com/sql/paging.md) |
| The table types | `.Update` | One `readonly` table-valued parameter per set, built by the platform from the model |

## Example

The standard shape of a loading procedure — a header, one or more datasets, nothing else:

```sql
create or alter procedure cat.[Agent.Load]
@UserId bigint,
@Id bigint = null
as
begin
    set nocount on;
    set transaction isolation level read uncommitted;

    select [Agent!TAgent!Object] = null,
        [Id!!Id] = a.Id, [Name!!Name] = a.[Name], a.Memo
    from cat.Agents a
    where a.Id = @Id;
end
```

The saving procedure is the counterpart: it receives table types instead of scalars, works at `read committed`, and ends by calling `.Load` so the client gets the stored model back.

## Notes

- Index models cannot be updated. Saving works only for a model loaded by `.Load`.
- `@Id` is declared with a default of `null` so that the same `.Load` serves the "new record" form: with no rows returned, the platform still builds the structure from the column names and hands the form an empty instance.
- Loading procedures run at `set transaction isolation level read uncommitted`, saving procedures at `read committed`.
- `.Update` must end with a call to `.Load` — the platform expects the saved model in the response.
- The blob procedures are the exception to all of this: they take a fixed parameter list, return plain columns without markers, and do not follow the rules for building models.
- A procedure reports a failure with `throw`; whether the text reaches the user or the developer depends on the `UI:` prefix — see [Error Messages](https://docs-llm.a2v10.com/sql/errors.md).
