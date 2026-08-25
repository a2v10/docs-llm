# SQL

> Data layer reference for A2v10 — naming conventions, SELECT markers, stored procedure patterns, and the dataset shapes the platform understands.

- [Conventions & Naming](https://docs-llm.a2v10.com/sql/overview.md): Naming rules, schemas, standard columns, DDL idempotence
- [SQL Markers](https://docs-llm.a2v10.com/sql/markers.md): Column alias markers in SELECT — how the platform maps result sets to models
- [Stored Procedures](https://docs-llm.a2v10.com/sql/procedures.md): Patterns for Index, Load, Metadata, Update, Fetch, Delete
- [Update Model (TVP + MERGE)](https://docs-llm.a2v10.com/sql/update-model.md): How data is saved via table-valued parameters and MERGE
- [Objects & References](https://docs-llm.a2v10.com/sql/object.md): Single-object models — !Id / !Name, references via !RefId, Map sets, new instances
- [Arrays](https://docs-llm.a2v10.com/sql/array.md): Array models — !Array / !LazyArray markers, child arrays via !ParentId, on-demand loading
- [Paging & Filters](https://docs-llm.a2v10.com/sql/paging.md): Server-side paging — @Offset / @PageSize / @Order / @Dir, !RowCount, filter forms
- [Tree & Hierarchy](https://docs-llm.a2v10.com/sql/tree.md): Hierarchical catalogs — Parent column, IsFolder, recursive queries
- [Grouping Models](https://docs-llm.a2v10.com/sql/grouping.md): Aggregated totals hierarchy — !Group type, GROUP BY ROLLUP, GroupMarker columns
- [Cross (Pivot) Models](https://docs-llm.a2v10.com/sql/cross.md): Variable columns from data — !CrossArray / !CrossObject, Key/ParentId markers, $cross
- [Named Sets (MapObject)](https://docs-llm.a2v10.com/sql/map-object.md): Binding by !Key — each key value becomes a named property of the parent object
- [System Datasets](https://docs-llm.a2v10.com/sql/system-datasets.md): $System, $Aliases and $Defaults — sets that control processing rather than shape
- [Binary Objects (blob)](https://docs-llm.a2v10.com/sql/blob.md): Images and attachments — !Token access marker, the .Load / .Update byte-stream procedures
- [Change Tracking (rowversion)](https://docs-llm.a2v10.com/sql/rowversion.md): Refusing a save when the record changed meanwhile — the rv column, varbinary(8) in the TVP
- [Error Messages](https://docs-llm.a2v10.com/sql/errors.md): Reporting failures with throw — the UI: prefix and who each message is addressed to
