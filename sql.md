# SQL

> Data layer reference for A2v10 — naming conventions, SELECT markers, stored procedure patterns, and the dataset shapes the platform understands.

- [Conventions & Naming](sql/overview.md): Naming rules, schemas, standard columns, DDL idempotence
- [SQL Markers](sql/markers.md): Column alias markers in SELECT — how the platform maps result sets to models
- [Stored Procedures](sql/procedures.md): Patterns for Index, Load, Metadata, Update, Fetch, Delete
- [Update Model (TVP + MERGE)](sql/update-model.md): How data is saved via table-valued parameters and MERGE
- [Tree & Hierarchy](sql/tree.md): Hierarchical catalogs — Parent column, IsFolder, recursive queries
- [Grouping Models](sql/grouping.md): Aggregated totals hierarchy — !Group type, GROUP BY ROLLUP, GroupMarker columns
- [Cross (Pivot) Models](sql/cross.md): Variable columns from data — !CrossArray / !CrossObject, Key/ParentId markers, $cross
