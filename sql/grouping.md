# Grouping Models

> Aggregated parent-child datasets in A2v10 — the `!Group` model type, `GROUP BY ROLLUP`, and grouping markers that build a totals hierarchy from repeating values.

## Overview

A grouping model represents records with parent-child relationships of unlimited nesting depth. Unlike tree (hierarchical) models, where the parent-child relationship is defined by a parent record identifier, in grouping models the relationship is derived from repeating values.

Grouping datasets are normally produced with the T-SQL `GROUP BY ROLLUP` operator, which emits one subtotal row per grouping level plus a grand-total row.

The model is represented by the `!Group` object type. For it to assemble correctly, every record must carry two additional mandatory fields:

| Marker | Description |
|--------|-------------|
| `[Field!!GroupMarker]` | Grouping marker — the value of the T-SQL `grouping()` function for the field. There are as many of these as there are columns in the `GROUP BY ROLLUP` clause. |
| `[Items!TType!Items]` | Placeholder for the child array. In the dataset its value is always `null`; the platform uses it to build the hierarchy. The element type must match the root object type (the one with the `!Group` modifier). |

Because of how the model is processed, the dataset MUST be sorted by the grouping markers in descending order, so that group (subtotal) rows are emitted before their detail rows.

If only one grouping level is defined, the model effectively just computes totals for the aggregated fields (those produced with `sum()`).

## Syntax

```sql
select [Root!TType!Group] = null,
    [Field1]              = ...,
    [Field2]              = ...,
    [Value]              = sum(...),
    [Field1!!GroupMarker] = grouping(field1),
    [Field2!!GroupMarker] = grouping(field2),
    [Items!TType!Items]   = null
from ...
group by rollup (field1, field2)
order by [Field1!!GroupMarker] desc, [Field2!!GroupMarker] desc, field1, field2;
```

## Example

Given a `Documents` table:

| Id | Date | Agent | Amount |
|----|------|-------|--------|
| 10 | 2026-05-01 | 10 | 150.00 |
| 11 | 2026-05-02 | 10 | 300.00 |
| 12 | 2026-05-01 | 20 | 320.00 |
| 13 | 2026-05-02 | 20 | 270.00 |

and an `Agents` table:

| Id | Name |
|----|------|
| 10 | Agent 1 |
| 20 | Agent 2 |

The following procedure builds a dataset grouped by date and agent:

```sql
create or alter procedure dbo.[Report.Documents.Load]
@UserId bigint
as
begin
    set nocount on;
    set transaction isolation level read uncommitted;

    select [ReportData!TData!Group] = null,
        [Agent]               = a.[Name],
        [Date]                = convert(nvarchar, d.[Date], 104),
        Amount                = sum(d.Amount),
        [Agent!!GroupMarker]  = grouping(a.[Name]),
        [Date!!GroupMarker]   = grouping(d.[Date]),
        [Items!TData!Items]   = null
    from dbo.Documents d inner join dbo.Agents a on d.Agent = a.Id
    group by rollup (a.[Name], d.[Date])
    order by [Agent!!GroupMarker] desc, [Date!!GroupMarker] desc, a.[Name], d.[Date];
end
go
```

### Resulting Dataset

| ReportData!TData!Group | Agent | Date | Amount | Agent!!GroupMarker | Date!!GroupMarker | Items!TData!Items |
|---|---|---|---|---|---|---|
| null | null | null | 1040.00 | 1 | 1 | null |
| null | Agent 1 | null | 450.00 | 0 | 1 | null |
| null | Agent 2 | null | 590.00 | 0 | 1 | null |
| null | Agent 1 | 01.05.2026 | 150.00 | 0 | 0 | null |
| null | Agent 1 | 02.05.2026 | 300.00 | 0 | 0 | null |
| null | Agent 2 | 01.05.2026 | 320.00 | 0 | 0 | null |
| null | Agent 2 | 02.05.2026 | 270.00 | 0 | 0 | null |

The first three rows are subtotals — the grand total for the whole report (first row) and per-agent totals (second and third rows). The grouping markers indicate which field each subtotal belongs to.

### Resulting JSON Model

Service properties are omitted for brevity except `$level` and `$groupName`.

```json
{
  "ReportData": {
    "Agent": "",
    "Date": "",
    "Amount": 1040,
    "Items": [
      {
        "Agent": "Agent 1",
        "Date": "",
        "Amount": 450,
        "Items": [
          { "Agent": "Agent 1", "Date": "01.05.2026", "Amount": 150, "Items": [], "$level": 2, "$groupName": "01.05.2026" },
          { "Agent": "Agent 1", "Date": "02.05.2026", "Amount": 300, "Items": [], "$level": 2, "$groupName": "02.05.2026" }
        ],
        "$level": 1,
        "$groupName": "Agent 1"
      },
      {
        "Agent": "Agent 2",
        "Date": "",
        "Amount": 590,
        "Items": [
          { "Agent": "Agent 2", "Date": "01.05.2026", "Amount": 320, "Items": [], "$level": 2, "$groupName": "01.05.2026" },
          { "Agent": "Agent 2", "Date": "02.05.2026", "Amount": 270, "Items": [], "$level": 2, "$groupName": "02.05.2026" }
        ],
        "$level": 1,
        "$groupName": "Agent 2"
      }
    ],
    "$level": 0
  }
}
```

The `$level` property is the depth in the tree (starting from 0); `$groupName` is the group name — the value of the dataset selected according to the level.

### XAML Binding

A grouping model can be displayed with a [Sheet](https://docs-llm.a2v10.com/xaml/layouts/sheet.md) element:

```xml
<Sheet GridLines="Both" Columns="Fit,Auto,Auto">
    <Sheet.Header>
        <SheetRow Style="Header">
            <SheetCell />
            <SheetCell Content="Agent/Date"/>
            <SheetCell Content="Amount"/>
        </SheetRow>
        <SheetRow Style="Total">
            <SheetCell ColSpan="2" Content="Total"/>
            <SheetCell Content="{Bind ReportData.Amount, DataType=Currency}" Align="Right"/>
        </SheetRow>
    </Sheet.Header>
    <SheetTreeSection ItemsSource="{Bind ReportData.Items}">
        <SheetRow>
            <SheetGroupCell/>
            <SheetCell GroupIndent="True" Content="{Bind $groupName}"/>
            <SheetCell Content="{Bind Amount, DataType=Currency}" Align="Right"/>
        </SheetRow>
    </SheetTreeSection>
</Sheet>
```

This renders an indented totals table:

| Agent/Date | Amount |
|---|---|
| Total | 1 040,00 |
| Agent 1 | 450,00 |
| 01.05.2026 | 150,00 |
| 02.05.2026 | 300,00 |
| Agent 2 | 590,00 |
| 01.05.2026 | 320,00 |
| 02.05.2026 | 270,00 |

## Notes

- Sort the dataset by all grouping markers in descending order before the detail columns; otherwise the platform cannot assemble the hierarchy correctly — group rows must precede their children.
- There must be exactly one `[Field!!GroupMarker]` column per column listed in `GROUP BY ROLLUP`.
- The `[Items!TType!Items]` element type must match the root `!Group` object type (`TData` in the example).
- A single grouping level reduces the model to plain totals over the `sum()` fields.
- `$level` and `$groupName` are service properties added by the platform — `$level` is the zero-based depth, `$groupName` is the group label picked from the row according to its level.
- Aggregated value columns use `sum()`; the rollup grand-total row carries `grouping()` value `1` for every grouped field.
