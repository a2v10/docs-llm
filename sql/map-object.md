# Named Sets (MapObject)

> The `MapObject` dataset — binding by `!Key` instead of identifier, so every key value becomes a named property of the parent object.

## Overview

A `MapObject` dataset resembles a `Map`, but linking is done by key rather than by identifier. Every distinct key value becomes a named property of the parent object.

Like any other child set, it needs a placeholder field in the parent set, and the set itself must contain a key (`!Key`) and a reference to the parent element (`!ParentId`).

Keys that are actually present in the data reach the model automatically. That alone would make the object structure depend on the data — with no row present, there would be no corresponding property either. Since the structure of an object has to be stable, the list of keys is written into the *name* of the placeholder field, as a fourth element separated by colons, and the platform creates those properties while building metadata whether or not there is data for them.

Most often the list of keys is stored in the database and is unknown when the procedure is written. A field name in SQL is a literal and cannot be computed, so in that case the name is assembled at run time and substituted through the `$Aliases` system set.

## Use When

- The set of properties is stored as data — key/value rows in a table — and has to appear as named properties of the object.
- The list of keys is configurable and must not require the procedure to be rewritten.

## Do Not Use When

- The list of properties is known in advance — describe them as ordinary fields instead; that is simpler and this mechanism is rarely needed.
- Linking is by identifier rather than key — use a `Map` set instead, see [Objects & References](https://docs-llm.a2v10.com/sql/object.md).
- The variable columns belong to an array rather than an object — use [Cross (Pivot) Models](https://docs-llm.a2v10.com/sql/cross.md) instead.

## Syntax

| Marker | Description |
|--------|-------------|
| `[Prop!TType!MapObject]` | Placeholder field in the parent set |
| `[Prop!TType!MapObject!Key1:Key2]` | The same, with an explicit list of keys as the fourth element |
| `[!TType!MapObject]` | Descriptor of the child set |
| `[!!Key]` | The key — its value becomes the property name |
| `[!TParent.Prop!ParentId]` | Link to the parent element |

## Example

### Keys taken from the data

```sql
select [Card!TCard!Object] = null, [Id!!Id] = c.Id, [Name] = c.[Name],
    [Attrs!TAttr!MapObject] = null
from cat.Cards c where c.Id = @Id;

select [!TAttr!MapObject] = null, [!!Key] = a.Prop, [Value] = a.[Value],
    [!TCard.Attrs!ParentId] = @Id
from cat.CardAttributes a where a.Card = @Id;
```

If `CardAttributes` holds rows with the keys `Color` and `Weight`, the model becomes:

```json
{
    "Card": {
        "Id": 1,
        "Name": "Name",
        "Attrs": {
            "Color":  { "Value": "red" },
            "Weight": { "Value": "10 kg" }
        }
    }
}
```

### An explicit list of keys

```sql
select [Card!TCard!Object] = null, [Id!!Id] = c.Id,
    [Attrs!TAttr!MapObject!Color:Weight] = null
from cat.Cards c where c.Id = @Id;
```

The properties `Color` and `Weight` are now always present in the model, even when the child set is empty.

### Keys computed from the data

```sql
/* the list of keys depends on the data */
declare @keys nvarchar(max);
select @keys = string_agg([Prop], N':') from cat.CardProps;

select [!$Aliases!] = null, [~Attrs] = N'Attrs!TAttr!MapObject!' + @keys;

select [Card!TCard!Object] = null, [Id!!Id] = c.Id, [Name] = c.[Name],
    [~Attrs] = null
from cat.Cards c where c.Id = @Id;

select [!TAttr!MapObject] = null, [!!Key] = a.Prop, [Value] = a.[Value],
    [!TCard.Attrs!ParentId] = @Id
from cat.CardAttributes a where a.Card = @Id;
```

## Notes

- If the set has no field with the `!Key` modifier, it is built as an ordinary array.
- The list of keys in the field name is separated by colons and comes after the dataset type — it is the fourth element of the name.
- The `$Aliases` set must be returned before the first set that uses the alias. See [System Datasets](https://docs-llm.a2v10.com/sql/system-datasets.md).
- The mechanism is used rarely. When the property list is fixed, ordinary fields are the better choice.
