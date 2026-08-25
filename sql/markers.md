# SQL Markers

> Column alias markers in SELECT statements — how the platform maps SQL result sets to model objects and arrays.

## Overview

An A2v10 stored procedure does not return data in an agreed-upon binary shape — it returns ordinary result sets, and the platform derives the model structure from the *column aliases*. Every name in a result set can carry up to three elements, so a single `SELECT` describes both the values and the shape they are built into.

The first field of every result set is the dataset descriptor. Its value must always be `null`; only its name matters. The name says what the set is: which property of the model it creates, what type its elements have, and what kind of collection it is — an object, an array, a tree, a lookup map.

All remaining fields describe individual properties. A field with no modifier becomes a plain scalar property. A field with a modifier gets a special role instead — record identifier, display name, reference to another object, link to a parent element, and so on.

Because the structure comes from names rather than values, an empty result set still produces complete metadata. That is why the same `.Load` procedure serves both an existing record and a brand-new one: with no rows returned, the platform still knows the full shape and constructs an empty instance of it.

## Syntax

A name consists of one to three elements separated by `!`. Every part is optional, but the separators are not — a name beginning with `!!` means "no property name, no type name, modifier only".

```
[PropertyName!TypeName!Modifier]
```

| Element | Meaning |
|---------|---------|
| 1 — property name | Name of the property in the model. May be compound. Omit it to keep the field out of the model. |
| 2 — type name | Element type name, or a path to a property in the model (for `!ParentId`). |
| 3 — modifier | The field's type or its special purpose. |

Names containing special characters, spaces, or SQL keywords must be wrapped in square brackets — in practice every marker is written in brackets.

### Dataset types

The dataset type is the third element of the descriptor — the first field of the result set.

| Type | Description |
|------|-------------|
| `Object` | A single element. The set returns exactly one row. See [Objects & References](https://docs-llm.a2v10.com/sql/object.md) |
| `Array` | An array of elements. A child array must carry a `!ParentId` field. See [Arrays](https://docs-llm.a2v10.com/sql/array.md) |
| `LazyArray` | A child array loaded on first access to the property. See [Arrays](https://docs-llm.a2v10.com/sql/array.md) |
| `Map` | Referenced objects. Always a child set, usually unnamed. Requires an `!Id` field — linking happens through it. See [Objects & References](https://docs-llm.a2v10.com/sql/object.md) |
| `MapObject` | A named set. Linking is done by `!Key`; each key value becomes a property of the parent object. See [Named Sets (MapObject)](https://docs-llm.a2v10.com/sql/map-object.md) |
| `Tree` | A tree of elements, static or dynamic. Requires `!Id`, `!ParentId`, and `!Items`. See [Tree & Hierarchy](https://docs-llm.a2v10.com/sql/tree.md) |
| `Group` | A tree built by grouping a flat table. Requires `!GroupMarker` and `!Items`. See [Grouping Models](https://docs-llm.a2v10.com/sql/grouping.md) |
| `CrossArray` | A cross (pivot) array. Always a child set; requires `!ParentId` and `!Key`. See [Cross (Pivot) Models](https://docs-llm.a2v10.com/sql/cross.md) |
| `CrossObject` | The object form of a cross set. See [Cross (Pivot) Models](https://docs-llm.a2v10.com/sql/cross.md) |

Besides these, a procedure may return system datasets. They create no model properties — they control how the other sets are processed. Their type names begin with `$`. See [System Datasets](https://docs-llm.a2v10.com/sql/system-datasets.md).

### Field modifiers

| Modifier | Written as | Purpose |
|----------|-----------|---------|
| `Id` | `[Id!!Id]` | Record identifier. Linking with other sets — `Map` sets in particular — happens through it |
| `Name` | `[Name!!Name]` | Display name, used wherever the object is shown as a single line |
| `RefId` | `[Company!TCompany!RefId]` | Reference to another object, resolved from a `Map` set of that type |
| `ParentId` | `[!TDocument.Rows!ParentId]` | Links a child row to its parent — the second element is the parent type and property |
| `Key` | `[!!Key]` | Key of a `MapObject` or cross set element |
| `Items` | `[Items!TItem!Items]` | Child-collection property of a `Tree` or `Group` element |
| `GroupMarker` | `[!!GroupMarker]` | Grouping-level marker in a `Group` set |
| `RowCount` | `[!!RowCount]` | Total number of matching records, ignoring paging. See [Paging & Filters](https://docs-llm.a2v10.com/sql/paging.md) |
| `Token` | `[Token!!Token]` | Access key of a binary object — replaced by an access token. See [Binary Objects (blob)](https://docs-llm.a2v10.com/sql/blob.md) |

## Example

A descriptor plus the three most common field markers:

```sql
select [Agent!TAgent!Object] = null,
    [Id!!Id] = a.Id, [Name!!Name] = a.[Name], a.Memo,
    [Company!TCompany!RefId] = a.Company
from cat.Agents a
where a.Id = @Id;
```

The descriptor `[Agent!TAgent!Object]` reads as: property `Agent` at the model root, elements of type `TAgent`, dataset type `Object`. `Id` and `Name` carry no type name, hence the doubled exclamation marks. `Memo` has no marker at all and becomes a plain scalar property. `Company` is an ordinary foreign key in the database, but in the model it becomes an object, resolved from a `Map` set of type `TCompany`.

## Notes

- The first field of every dataset must be `null`. Only its name is read.
- The property name is arbitrary — `[Code!!Id]` is valid and puts the identifier into a property named `Code`.
- Linking between sets is done by the pair *type name + identifier value*. The order of result sets does not matter for `Map` sets, but a child `Array` must be processed after its parent.
- Identifier types must match exactly: `int` 22 and `bigint` 22 are different keys and will not link.
- Metadata is built from field names, not from values — an empty result set still describes the full structure.
