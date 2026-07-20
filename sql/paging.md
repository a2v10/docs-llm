# Paging & Filters

> Server-side paging for index models — the `@Offset`/`@PageSize`/`@Order`/`@Dir` parameters, the `!RowCount` field, and filter values returned through `$System`.

## Overview

Any model — most often an index model — can be used with additional parameters such as the current page and assorted filters. This takes extra fields in the main dataset plus one extra system dataset.

Paging consists of three parts, all of them mandatory. The procedure parameters carry the current page, the sort order, and the filter values; the platform passes them on every load. A field with the `!RowCount` modifier in the main dataset carries the total number of records — without it the number of pages cannot be determined. And the `$System` system set returns those same parameters to the form; without it the form loses them on the next refresh.

Filters themselves are ordinary procedure parameters. The platform knows nothing about them beyond what the procedure reports through `$System`. Their number and names are up to the developer, and all parameters must have default values.

## Use When

- The table is large or unbounded and the form must show it a page at a time.
- The user chooses the sort property or column direction on the form.
- The list has filters whose values must survive a data refresh.

## Do Not Use When

- The collection is small and always used as a whole — load it as a plain array instead, see [Arrays](https://docs-llm.a2v10.com/sql/array.md).
- The model is a single record — use [Objects & References](https://docs-llm.a2v10.com/sql/object.md) instead.

## Syntax

| Parameter | Description |
|-----------|-------------|
| `@Offset` | Offset from the start. The index of the first record on the page |
| `@PageSize` | Number of records per page |
| `@Order` | Name of the property to sort by |
| `@Dir` | Sort direction — `asc` or `desc` |

These are all the parameters related to paging.

| Marker | Description |
|--------|-------------|
| `[!!RowCount]` | Total number of records matching the selection, ignoring paging |
| `[!Element!Offset]` | Current offset, returned through `$System` |
| `[!Element!PageSize]` | Current page size, returned through `$System` |
| `[!Element!SortOrder]` | Current sort property, returned through `$System` |
| `[!Element!SortDir]` | Current sort direction, returned through `$System` |
| `[!Element.Path!Filter]` | Filter value, returned through `$System` |

## Example

### Minimal

The processing itself is simple. The platform requires only two things: a `!RowCount` field in the dataset and a `$System` set with the current parameters. Everything else is an ordinary query.

```sql
create or alter procedure cat.[Agent.Index]
@UserId bigint,
@Offset int = 0,
@PageSize int = 20
as
begin
    set nocount on;

    select [Agents!TAgent!Array] = null,
        [Id!!Id] = a.Id, [Name!!Name] = a.[Name],
        [!!RowCount] = count(*) over()
    from cat.Agents a
    where a.Void = 0
    order by a.[Name]
    offset @Offset rows fetch next @PageSize rows only;

    select [!$System!] = null,
        [!Agents!Offset] = @Offset, [!Agents!PageSize] = @PageSize;
end
```

That is enough for a working paged list.

### How it is done in practice

The example below is more elaborate, but none of that elaboration is a platform requirement. It additionally solves four problems: sorting by a property the user chooses, filters (period, fragment search, reference), building `Map` sets for the references on the current page, and selecting the full set of columns only for the records of the page rather than for the whole filtered set.

The temporary table is declared as a named type rather than a local table variable, so that it can be passed to another procedure. Besides the identifier and the row number, it carries the foreign-key columns.

```sql
create type doc.[Document.Map.TableType] as table(
    id          bigint,
    rowNo       int identity(1, 1),
    agent       bigint,
    [rowCount]  int
);
go
------------------------------------------------
create or alter procedure doc.[Document.Index]
@UserId bigint,
@Offset int = 0,
@PageSize int = 20,
@Order nvarchar(32) = N'date',
@Dir nvarchar(5) = N'desc',
@From date = null,
@To date = null,
@Fragment nvarchar(255) = null,
@Agent bigint = null
as
begin
    set nocount on;
    set transaction isolation level read uncommitted;

    set @Order = lower(@Order);
    set @Dir = lower(@Dir);
    declare @fr nvarchar(255) = N'%' + @Fragment + N'%';

    /* selection, sorting and cutting out the page */
    declare @docs doc.[Document.Map.TableType];

    insert into @docs(id, agent, [rowCount])
    select d.Id, d.Agent, count(*) over()
    from doc.Documents d
        left join cat.Agents a on d.Agent = a.Id
    where d.Void = 0
        and (@Agent is null or d.Agent = @Agent)
        and (@From is null or d.[Date] >= @From)
        and (@To is null or d.[Date] <= @To)
        and (@fr is null or d.[No] like @fr or d.Memo like @fr or a.[Name] like @fr)
    order by
        case when @Dir = N'asc'  then case @Order when N'no'   then d.[No]   end end asc,
        case when @Dir = N'desc' then case @Order when N'no'   then d.[No]   end end desc,
        case when @Dir = N'asc'  then case @Order when N'date' then d.[Date] end end asc,
        case when @Dir = N'desc' then case @Order when N'date' then d.[Date] end end desc,
        case when @Dir = N'asc'  then case @Order when N'sum'  then d.[Sum]  end end asc,
        case when @Dir = N'desc' then case @Order when N'sum'  then d.[Sum]  end end desc,
        d.Id
    offset @Offset rows fetch next @PageSize rows only
    option(recompile);

    /* selecting the data */
    select [Documents!TDocument!Array] = null,
        [Id!!Id] = d.Id, [No!!Name] = d.[No], d.[Date], d.[Sum], d.Memo,
        [Agent!TAgent!RefId] = d.Agent,
        [!!RowCount] = t.[rowCount]
    from doc.Documents d
        inner join @docs t on d.Id = t.Id
    order by t.rowNo;

    /* resolving the references */
    exec doc.[Document.Map] @Map = @docs;

    /* returning the state */
    select [!$System!] = null,
        [!Documents!Offset] = @Offset, [!Documents!PageSize] = @PageSize,
        [!Documents!SortOrder] = @Order, [!Documents!SortDir] = @Dir,
        [!Documents.Period.From!Filter] = @From,
        [!Documents.Period.To!Filter] = @To,
        [!Documents.Fragment!Filter] = @Fragment,
        [!Documents.Agent.TAgent.RefId!Filter] = @Agent;
end
```

### Resolving references

If the dataset contains references (`!RefId`), they need `Map` sets. Returning the whole directory is wrong — only the records referenced by the rows of the *current page* should reach the `Map` set. That is exactly why the temporary table is declared as a named type and carries the foreign-key columns: it is passed to a separate procedure which builds the `Map` sets.

```sql
create or alter procedure doc.[Document.Map]
@Map doc.[Document.Map.TableType] readonly
as
begin
    set nocount on;
    set transaction isolation level read uncommitted;

    with TA as (select agent from @Map where agent is not null group by agent)
    select [!TAgent!Map] = null, [Id!!Id] = a.Id, [Name!!Name] = a.[Name]
    from cat.Agents a
        inner join TA on a.Id = TA.agent;
end
```

Such a procedure is usually called from both `.Index` and `.Load`, so the reference-resolution logic is written once.

### Filter forms

A filter is an ordinary procedure parameter, returned to the form through `$System` with the `!Filter` modifier. The element name is compound: the first part is the model element the filter belongs to, the rest is the path to the filter property. The exact form depends on the kind of filter.

Scalar values — strings, numbers, dates, and booleans. The simplest case: a two-part path. The platform determines the kind of filter from the data type of the value itself.

```sql
[!Documents.Fragment!Filter] = @Fragment,   /* string  */
[!Documents.Count!Filter] = @Count,         /* number  */
[!Documents.DateOpt!Filter] = @DateOpt,     /* date    */
[!Documents.Flag!Filter] = @Flag            /* boolean */
```

A period — an intermediate node whose name starts with `Period`. It always has two bounds, `From` and `To`. There may be several period nodes; it is enough that their names start with `Period`.

```sql
[!Documents.Period.From!Filter] = @From,
[!Documents.Period.To!Filter] = @To
```

A reference — the last element of the path is `RefId`, the one before it is the type name. The object the value points to must be present in a `Map` set, otherwise the form cannot display the name of the selected value.

```sql
[!Documents.Agent.TAgent.RefId!Filter] = @Agent
```

An array of references — when the filter allows several values to be selected, the last element of the path is `Array`. The value is a *string* of comma-separated identifiers, each of which must likewise be present in a `Map` set.

```sql
[!Documents.Agents.TAgent.Array!Filter] = @Agents   /* N'15,20,25' */
```

## Notes

- If a filter value is not returned through `$System`, the form loses it on the very first data refresh — the user sees an empty filter and the full list.
- The `!RowCount` field holds the number of records matching the selection *without* paging applied. The value lands in the array's `$RowCount` property.
- The value of `!RowCount` is the same in every row of the set, so it is usually obtained with the window function `count(*) over()`.
- Within a single `case` construct, columns of different types must not be mixed — SQL Server raises a type-conversion error. Each group of types (strings, dates, numbers) needs its own pair of `case` expressions.
- Sorting by identifier is listed last. This guarantees a deterministic record order even when the values in the sort column coincide.
- The `rowNo` field preserves the order produced by the sort. The second select joins the temporary table and `order by t.rowNo` restores that order — otherwise the join may return records in arbitrary sequence.
- The two-stage selection exists for two reasons: selection and sorting run over a minimal set of columns while full data is read only for the records of the page, and the temporary table carries the foreign-key columns so it can be handed to the procedure that builds the `Map` sets.
- A formal description of every filter form is in [System Datasets](https://docs-llm.a2v10.com/sql/system-datasets.md).
