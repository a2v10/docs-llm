# Arrays

> Array data models in A2v10 — the `!Array` and `!LazyArray` markers, child arrays linked via `!ParentId`, and on-demand loading.

## Overview

An array is a collection of objects in the model. It can be a standalone element at the model root (then it has a name and appears at the top level), or a child property nested inside another object.

An array is declared by the type `Array` (or `LazyArray` for a lazy, on-demand array) in the dataset descriptor — the first field of the SELECT. Each row of the result set becomes one object in the array; the object type is set by the descriptor.

Child arrays are nested inside a parent object and usually have no name. The parent object must carry a placeholder property set to `null` with the child element type. The child result set must be processed after the parent, and each child row must reference its parent through a `!ParentId` marker.

Lazy arrays are not filled while the model is built — they load on first access to the property. This is typical when the main model is itself an array and the child arrays are only needed when displayed (for example, a list of agents, each with its related documents).

## Syntax

| Marker | Description |
|--------|-------------|
| `[Name!TType!Array]` | Standalone array `Name` at the model root, elements of type `TType` |
| `[!TType!Array]` | Child array (no name), elements of type `TType` |
| `[Id!!Id]` | Identifier field of the array element |
| `[Name!!Name]` | Display-name field of the array element |
| `[!TParent.Prop!ParentId]` | Links each child row to its parent (see below) |
| `[Prop!TType!LazyArray]` | Lazy (on-demand) child array placeholder |

The parent-link marker `[Name!ParentType.PropertyName!ParentId]`:

| Part | Meaning |
|------|---------|
| `Name` | Optional field name — may be empty |
| `ParentType.PropertyName` | Type name and property name of the parent object the element is written into |
| `ParentId` | Field modifier |

Linking works as follows: the value of the `!ParentId` field is taken, the model is searched for an object of type `ParentType` with a matching identifier (marked with `!Id`), and the element is appended to the array under `PropertyName` of that object.

## Example

### Standalone array

```sql
select [Agents!TAgent!Array] = null, [Id!!Id] = Id, [Name!!Name] = [Name], Code
from a2.Agents where ...;
```

Produces an array `Agents` at the model root. Each element is of type `TAgent` with the properties `Id`, `Name`, `Code`.

### Child array

```sql
/* main object */
select [Document!TDocument!Object] = null, [Id!!Id] = Id, [Date], [Rows!TRow!Array] = null
from a2.Documents where ...;

/* child array */
select [!TRow!Array] = null, [Id!!Id] = Id, [Qty], [Price], [Sum],
    [!TDocument.Rows!ParentId] = null
from a2.DocDetails where ...;
```

Produces a root object `Document` of type `TDocument` with the properties `Id`, `Date`, `Rows`. The `Rows` property is an array of `TRow` elements with `Id`, `Qty`, `Price`, `Sum`.

### Lazy array

```sql
/* main procedure */
create or alter procedure a2.[Agent.Index.Load]
@UserId bigint
as
begin
    set nocount on;
    /* main result set */
    select [Agents!TAgent!Array] = null, [Id!!Id] = Id, [Name!!Name] = [Name], [Code],
        [Documents!TDocument!LazyArray] = null  /* lazy array Documents */
    from a2.Agents where ...;

    /* structure of TDocument — empty result set */
    select [Documents!TDocument!Array] = null, [Id!!Id] = Id, [Date], [Sum], [Memo]
    from a2.Documents where 0 <> 0;
end
go

/* lazy-array loader. The suffix Documents matches the property name in the main model. */
create or alter procedure a2.[Agent.Documents]
@UserId bigint,
@Id bigint  /* identifier of the agent whose documents are loaded */
as
begin
    set nocount on;
    select [Documents!TDocument!Array] = null, [Id!!Id] = Id, [Date], [Sum], [Memo]
    from a2.Documents where [Agent] = @Id and ...;
end
go
```

## Notes

- Keep client-side arrays small: the recommended size is no more than 100 elements, and about 200 is the maximum before noticeable performance loss. For large tables use server-side sorting, filtering, and paging.
- A child result set must be processed after the parent — its parent object must already exist in the model.
- The parent identifier referenced by `!ParentId` must be marked with `!Id` on the parent object.
- The structure (not the data) of a lazy array must be known when the model is first built — return an empty result set (`where 0 <> 0`) that defines the array shape.
- A lazy-array loader receives the parent identifier as `@Id` and returns a result set matching the array description; its `!ParentId` field is not required.
- Lazy-array loaders may take parameters and use paging.

## Hints

A lazy array can be reloaded from the database on its own, without refetching the main model. Pass the array as the argument to the [Refresh (Reload)](https://docs-llm.a2v10.com/xaml/bind.md) command, or to the `$reload` controller method.
