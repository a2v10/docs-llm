# System Datasets

> The `$System`, `$Aliases`, and `$Defaults` datasets — sets that create no model properties but control how the platform processes the others.

## Overview

Besides the datasets that form the model itself, a stored procedure may return system datasets. They do not create model properties directly — they govern how the platform processes the other sets: paging parameters, filters, default values, and so on.

A system dataset is recognised by the type name in its first field. Every system type name begins with `$`. As with ordinary sets, the value of the first field must always be `null`.

| Type | Description |
|------|-------------|
| `$System` | Paging, sorting, filter values, and model flags |
| `$Aliases` | Substitution of field names computed while the procedure runs |
| `$Defaults` | Default values for a new (not loaded) instance of the model. NET.Core only |

## $System

The most frequently used system set. Its main purpose is to return to the form the state it was loaded with: the current page, the sort order, and the filter values. Without this set the form loses those values on every data refresh.

The set consists of a single row. The first field is `[!$System!]` with the value `null`; all the other fields describe individual parameters.

```sql
select [Documents!TDocument!Array] = null, [Id!!Id] = d.Id, [Date] = d.[Date],
    [!!RowCount] = (select count(*) from doc.Documents)
from doc.Documents d
order by d.[Date]
offset @Offset rows fetch next @PageSize rows only;

select [!$System!] = null,
    /* paging */
    [!Documents!Offset] = @Offset, [!Documents!PageSize] = @PageSize,
    [!Documents!SortOrder] = @Order, [!Documents!SortDir] = @Dir,
    /* filters */
    [!Documents.Period.From!Filter] = @From, [!Documents.Period.To!Filter] = @To,
    [!Documents.Fragment!Filter] = @Fragment;
```

The values are *procedure parameters*, not constants. That is precisely why this set returns to the form what the form was called with.

### $System modifiers

The second element of the field name says which model element the parameter applies to — in the example above, the `Documents` array. The third element is the modifier itself.

| Modifier | Element name | Value type | Description |
|----------|--------------|-----------|-------------|
| `PageSize` | required | int | Number of elements per page |
| `Offset` | required | int | Offset from the start — the index of the first element on the page |
| `SortOrder` | required | string | Name of the property the set is sorted by |
| `SortDir` | required | string | Sort direction, `asc` or `desc` |
| `GroupBy` | required | string | Property the set is grouped by |
| `HasRows` | required | int or bit | Whether rows exist. Used for lazy collections whose contents are not loaded yet: the count is unknown, but their presence is known |
| `Filter` | required | any | Filter value. The element name is compound — see below |
| `ReadOnly` | not specified | bit | If `1`, the platform locks every control on the form and forbids saving the model |
| `Copy` | not specified | bit | Marks the model as loaded in copy mode |
| `Permissions` | required | int | Bit mask of access rights to the element |

If no element name is given for `PageSize`, `Offset`, `SortOrder`, `SortDir`, `GroupBy`, `HasRows`, `Filter`, or `Permissions`, the model fails to load.

A field with an unknown modifier is not an error — its value is added to the model's system properties as is, under its own name.

### Filters

The element name for the `Filter` modifier must consist of at least two parts separated by a dot: `ModelElement.FilterProperty`.

| Form | Description |
|------|-------------|
| `Documents.Fragment` | Simple filter. A value of any scalar type |
| `Documents.Period.From`<br>`Documents.Period.To` | Period. An intermediate node whose name starts with `Period` is treated as a period with two bounds |
| `Documents.Agent.TAgent.RefId` | Reference. The last element is `RefId`, the one before it is the type name. The object the value points to must be present in the corresponding `Map` set |
| `Documents.Agents.TAgent.Array` | Array of references. The value is a string of comma-separated identifiers, each of which must be present in a `Map` set |

The filter value must be read by the form under the same name. For how filters are used in views, see [Paging & Filters](https://docs-llm.a2v10.com/sql/paging.md).

## $Aliases

The structure of a model is described by field names, and a field name in SQL is a literal — it cannot be computed at run time without resorting to dynamic SQL. The `$Aliases` set removes that limitation: the real field name is passed as a *value*, and SQL may compute a value any way it likes.

Each field of the set is an alias, and its value is the full name that alias stands for. Subsequent sets write the alias in place of the full name.

```sql
select [!$Aliases!] = null, [~Document] = N'Document!TDocument!Object',
    [~TRow] = N'!TRow!Array', [~EntRef] = N'Entity!TEntity!RefId';

select [~Document] = null, [Id!!Id] = @Id, [Rows!TRow!Array] = null;

select [~TRow] = null, [Id!!Id] = r.Id, [!TDocument.Rows!ParentId] = @Id,
    [~EntRef] = r.Entity
from doc.DocRows r where r.Document = @Id;

select [!TEntity!Map] = null, [Id!!Id] = Id, [Name] = [Name]
from cat.Entities;
```

The `$Aliases` set must be returned before aliases are first used. Aliases apply only to the sets that follow it. Names without a defined alias are used unchanged.

Substitution applies to *every* field name in the following sets. An alias must therefore never coincide with the real name of some field, or that field would be silently replaced. This is why aliases start with `~`: such a name cannot occur among ordinary fields.

### Computed field names

The main purpose of aliases is names that cannot be written as a literal because they depend on the data. The most typical case is a [MapObject](https://docs-llm.a2v10.com/sql/map-object.md) set, where the list of keys is part of the *name* of the placeholder field.

When the list of keys is known in advance, it is written directly in the name:

```sql
select [Card!TCard!Object] = null, [Id!!Id] = @Id,
    [Attrs!TAttr!MapObject!Attr1:Attr2] = null;
```

Usually, though, the key list is stored in the database and unknown when the procedure is written. The field name is then built at run time and substituted through `$Aliases`:

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

## $Defaults

NET.Core only. The `$Defaults` set assigns default values to model properties. The values are applied *only to those elements of the model root that were not loaded*. Because of that the set can be returned unconditionally, without checking whether a new instance is being created or an existing one loaded.

```sql
select [Document!TDocument!Object] = null, [Id!!Id] = d.Id, [Date] = d.[Date],
    [Agent!TAgent!RefId] = d.Agent
from doc.Documents d where d.Id = @Id;

/* applied only if the document was not loaded */
select [!$Defaults!] = null,
    [Document.Date] = getdate(),
    [Document.Name] = N'New document',
    [Document.Agent!TAgent!RefId] = @DefaultAgent;

select [!TAgent!Map] = null, [Id!!Id] = Id, [Name] = [Name]
from cat.Agents;
```

The field name is a path to a model property, its elements separated by dots. Intermediate objects are created automatically.

- A `null` value is ignored — setting an empty default makes no sense.
- The `RefId` modifier is supported. The object the value points to must be present in a `Map` set, and the value type must match the identifier type in that set exactly (`int` 22 does not equal `bigint` 22). The `Map` set itself may be returned later.
- String values are localized.
- The `Utc` modifier is supported, converting a date to local time.

## Notes

Datasets are processed sequentially, one after another, which imposes requirements on their order:

- `$Aliases` must be returned before the first set that uses the aliases it defines.
- `$System` must be returned after every model element it refers to has been processed. In practice it is usually placed last.
- `$Defaults` may be placed anywhere — the references (`RefId`) inside it are resolved after all the sets have been processed.
