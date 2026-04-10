# Tree & Hierarchy

> Hierarchical catalogs in A2v10 — the `!Tree` model type, recursive CTE queries, and lazy-loading via `.Expand` procedures.

## Overview

A tree (hierarchical) data model represents records with parent-child relationships of unlimited nesting depth. The model type `!Tree` requires three special markers in the SELECT:

| Marker | Description |
|--------|-------------|
| `[Root!TType!Tree]` | Root array — marks the result as a tree |
| `[!TType.Items!ParentId]` | Links each row to its parent by this column value |
| `[Items!TType!Items]` | Placeholder column for the nested `Items` array |

Each record must expose `[Id!!Id]` and optionally `[Name!!Name]`.

Two loading strategies are supported: **static** (full tree in one query via CTE) and **dynamic** (lazy load via `.Expand`).

## Static Tree

The entire tree is loaded in a single query using a recursive CTE. The platform assembles the flat result into a nested structure using `!ParentId` markers.

### SQL Pattern

```sql
create or alter procedure dbo.[Agent.Index.Load]
@UserId bigint
as
begin
    set nocount on;

    with T(Id, Parent, [Level]) as (
        select Id, Parent, 0 from dbo.Agents a where a.Parent is null
        union all
        select a.Id, a.Parent, T.[Level] + 1
        from dbo.Agents a inner join T on T.Id = a.Parent
    )
    select [Agents!TAgent!Tree] = null,
        [Id!!Id]                  = a.Id,
        [Name!!Name]              = a.[Name],
        [!TAgent.Items!ParentId]  = T.Parent,
        [Items!TAgent!Items]      = null,
        [Level]                   = T.[Level]
    from dbo.Agents a inner join T on a.Id = T.Id
    order by [Level], [Id!!Id];
end
go
```

### Resulting JSON Model

```json
{
  "Agents": [
    {
      "Id": 10,
      "Name": "Agent1",
      "Level": 0,
      "Items": [
        { "Id": 100, "Name": "Subagent 1.1", "Level": 1, "Items": [] },
        { "Id": 101, "Name": "Subagent 1.2", "Level": 1, "Items": [] }
      ]
    },
    {
      "Id": 20,
      "Name": "Agent2",
      "Level": 0,
      "Items": [
        { "Id": 200, "Name": "Subagent 2.1", "Level": 1, "Items": [] },
        { "Id": 201, "Name": "Subagent 2.2", "Level": 1, "Items": [] }
      ]
    }
  ]
}
```

## Dynamic Tree (Lazy Loading)

Only the top level is loaded initially. When the user expands a node, the platform calls the `.Expand` procedure to fetch its children.

The `[HasChildren!!HasChildren]` marker tells the UI whether to show an expand arrow on a node.

### Load Procedure (top level only)

```sql
create or alter procedure dbo.[Agent.Index.Load]
@UserId bigint
as
begin
    set nocount on;

    select [Agents!TAgent!Tree]    = null,
        [Id!!Id]                   = a.Id,
        [Name!!Name]               = a.[Name],
        [Items!TAgent!Items]       = null,
        [HasChildren!!HasChildren] = case
            when exists(select * from dbo.Agents c where c.Parent = a.Id) then 1
            else 0
        end
    from dbo.Agents a
    where Parent is null
    order by [Id!!Id];
end
go
```

### Expand Procedure (children of a node)

The procedure name must follow the `.Expand` suffix convention. It receives `@Id` — the ID of the node being expanded.

```sql
create or alter procedure dbo.[Agent.Index.Expand]
@UserId bigint,
@Id bigint
as
begin
    set nocount on;

    select [Items!TAgent!Tree]     = null,
        [Id!!Id]                   = a.Id,
        [Name!!Name]               = a.[Name],
        [Items!TAgent!Items]       = null,
        [HasChildren!!HasChildren] = case
            when exists(select * from dbo.Agents c where c.Parent = a.Id) then 1
            else 0
        end
    from dbo.Agents a
    where Parent = @Id
    order by [Id!!Id];
end
go
```

## XAML Binding

```xml
<TreeView ItemsSource="{Bind Agents}" IconFolder="Folder" IconItem="File">
    <TreeViewItem ItemsSource="{Bind Items}" Label="{Bind Name}" />
</TreeView>
```

## Notes

- The `[!TType.Items!ParentId]` marker is only needed in the **static** approach — it tells the platform which column is the parent ID when assembling the hierarchy from a flat result set.
- In the **dynamic** approach, the `.Expand` procedure uses `[Items!TType!Tree]` (not `[Root!TType!Tree]`) as its root marker — the result merges into the existing tree.
- `HasChildren` must be computed in SQL; the platform uses it only to decide whether to render an expand arrow. Once expanded, children are loaded on demand.
- The `@Id` parameter in `.Expand` is always `bigint` matching the `[Id!!Id]` column type.
- Elements implement the `ITreeElement` interface, which provides additional service properties.
