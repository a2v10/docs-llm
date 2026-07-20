# Objects & References

> The simplest data model — an `Object` dataset with scalar properties, references via `!RefId`, and the `Map` sets that resolve them.

## Overview

The simplest model is a single object with scalar properties. It corresponds to one dataset of type `Object`, which returns exactly one row. The procedure that loads it is named after the model with the suffix `.Load` or `.Index`.

Fields without a modifier become ordinary properties. Two modifiers have a special meaning: `!Id` marks the record identifier, through which the platform links this object with other datasets, and `!Name` marks the display name, used wherever the object has to be shown as a single line — in a reference selection field, for example.

A field with the `!RefId` modifier holds a reference to another object. In the database it is an ordinary foreign key, but in the model such a property becomes an object, not a number. The referenced objects themselves come back in a separate dataset of type `Map`, which normally has no name — it creates no property of its own, it only supplies data for references.

Giving a `Map` set a name makes it appear in the model while still serving references. The same rows then both resolve the references and are available as a standalone array property — convenient for small lookups the user should be able to pick from on the client, without a round trip to the server.

## Use When

- The model is a single record — a document header, a catalogue item, a settings page.
- A property is a foreign key that the form must display by name, not by number — declare it `!RefId`.
- A small lookup list has to be selectable on the client without a server call — return it as a named `Map`.

## Do Not Use When

- The model is a collection of records — use [Arrays](https://docs-llm.a2v10.com/sql/array.md) instead.
- The lookup is large or must be searched on the server — leave it unnamed and load values on demand instead of shipping the whole directory.
- The referenced set is bound by key rather than by identifier — use [Named Sets (MapObject)](https://docs-llm.a2v10.com/sql/map-object.md) instead.

## Syntax

| Marker | Description |
|--------|-------------|
| `[Name!TType!Object]` | Object `Name` at the model root, of type `TType` |
| `[Id!!Id]` | Record identifier — the link target for other datasets |
| `[Name!!Name]` | Display name, shown when the object is rendered as one line |
| `[Prop!TType!RefId]` | Reference to an object of type `TType` |
| `[!TType!Map]` | Unnamed `Map` set supplying objects of type `TType` |
| `[Name!TType!Map]` | Named `Map` set — resolves references and appears in the model as an array |

`!Id` and `!Name` are written without a type name, which is why the field name carries two consecutive exclamation marks. The property name itself is arbitrary — the model gets exactly the name written.

## Example

### A single object

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

The name of the first field describes the set itself: `Agent` is the property at the model root, `TAgent` is the type name, `Object` is the dataset type. The resulting model:

```json
{
    "Agent": {
        "Id": 100,
        "Name": "Agent name",
        "Memo": "Note"
    }
}
```

### A reference

```sql
select [Agent!TAgent!Object] = null,
    [Id!!Id] = a.Id, [Name!!Name] = a.[Name],
    [Company!TCompany!RefId] = a.Company
from cat.Agents a
where a.Id = @Id;

select [!TCompany!Map] = null, [Id!!Id] = c.Id, [Name!!Name] = c.[Name]
from cat.Companies c
where c.Id = (select a.Company from cat.Agents a where a.Id = @Id);
```

`Company` is a foreign key in the table, but an object in the model:

```json
{
    "Agent": {
        "Id": 100,
        "Name": "Agent name",
        "Company": {
            "Id": 5,
            "Name": "Company name"
        }
    }
}
```

### A named Map

```sql
select [Companies!TCompany!Map] = null, [Id!!Id] = c.Id, [Name!!Name] = c.[Name]
from cat.Companies c where c.Void = 0;
```

The model gets a `Companies` array of elements, and the same rows continue to resolve every `TCompany` reference in the model.

## Notes

- A `Map` set must always contain an `!Id` field — that is what linking is done by.
- Linking is done by the pair *type name + identifier value*, which has three consequences: the order of the datasets does not matter, a `Map` set may be returned either before or after the set that references it; the value type must match the `!Id` type in the `Map` set exactly, since `int` 22 and `bigint` 22 are different keys; and if there is no matching record in the `Map` set, the property stays an empty object — no error is raised, the form simply shows an empty value.
- One `Map` set serves every reference of its type in the model, however many there are.
- In the model, a named `Map` property is an array of elements.
- Index models cannot be updated — saving applies only to models loaded by `.Load`.

## Hints

If the record with the supplied identifier does not exist, the dataset returns no rows — but metadata is built from field names, not from values, so it is built for an empty set too. The platform constructs an object from that metadata, and the form receives a fully-formed instance of the correct structure, with all the expected properties and empty values. That is exactly how the "create new record" form is loaded: no separate procedure is needed, the same `.Load` does the job.

To give a new instance its initial values, return the `$Defaults` system set — see [System Datasets](https://docs-llm.a2v10.com/sql/system-datasets.md).

For how a model is written back to the database, see [Update Model (TVP + MERGE)](https://docs-llm.a2v10.com/sql/update-model.md). For building `Map` sets for a page of a list, see [Paging & Filters](https://docs-llm.a2v10.com/sql/paging.md).
