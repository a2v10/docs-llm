# Update Model (TVP + MERGE)

> How data is saved in A2v10: table-valued parameters carry form data into the database, MERGE handles insert/update.

## Overview

The entire model is saved via a single stored procedure call. The platform uses a two-phase process:

1. The client sends the complete model to the server.
2. The platform calls the `.Metadata` procedure for the model to determine how to transform the data.
3. The `.Metadata` procedure returns empty result sets whose column aliases encode transformation rules.
4. The platform converts client data into typed table variables using the metadata schema.
5. The `.Update` procedure is called with the resulting table-valued parameters.

## Metadata Column Alias Format

Each column in the `.Metadata` result sets follows the pattern:

```
[ParameterName!DataPath!Metadata]
```

| Part | Description |
|------|-------------|
| `ParameterName` | The SQL parameter name in the `.Update` procedure |
| `DataPath` | Dot-notation path into the model (e.g., `Document.Rows`) |
| `Metadata` | Literal — marks this as a metadata descriptor |

## Service Fields

The platform automatically populates the following fields if they exist in the table type:

| Field | Description |
|-------|-------------|
| `GUID` | Client-generated GUID for the record |
| `RowNumber` | 1-based row index within the parent |
| `CurrentKey` | Current record identifier |
| `ParentId` | Database ID of the parent record |
| `ParentGUID` | GUID of the parent record (used when parent is new) |
| `ParentKey` | Key of the parent |
| `ParentRowNumber` | Row number of the parent |

`ParentId` / `ParentGUID` are essential for child rows when the parent record is new (no database ID yet).

## Example

### Table Types

```sql
create type a2v10sample.[Document.TableType] as table(
	Id bigint,
	Kind nvarchar(32),
	[Date] date,
	[No] nvarchar(255),
	[Sum] money,
	[Agent] bigint,
	Memo nvarchar(255)
)
go

create type a2v10sample.[DocDetails.TableType] as table(
	Id bigint null,
	ParentId bigint null,
	RowNumber int,
	[Qty] float,
	[Price] float,
	[Sum] money,
	Product bigint,
	[Memo] nvarchar(255)
)
go
```

### Metadata Procedure

```sql
create or alter procedure a2v10sample.[Document.Metadata]
as
begin
	set nocount on;

	declare @Document a2v10sample.[Document.TableType];
	declare @Rows a2v10sample.[DocDetails.TableType];

	select [Document!Document!Metadata] = null, * from @Document;
	select [Rows!Document.Rows!Metadata] = null, * from @Rows;
end
go
```

### Update Procedure

```sql
create or alter procedure a2v10sample.[Document.Update]
	@UserId bigint,
	@Document a2v10sample.[Document.TableType] readonly,
	@Rows a2v10sample.[DocDetails.TableType] readonly,
	@Kind nvarchar(32)
as
begin
	set nocount on;

	declare @RetId bigint;
	declare @output table(op sysname, id bigint);

	merge a2v10sample.Documents as target
	using @Document as source
	on (target.Id = source.Id)
	when matched then update set
		target.[Date]  = source.[Date],
		target.[No]    = source.[No],
		target.Agent   = source.Agent,
		target.[Sum]   = source.[Sum],
		target.Memo    = source.Memo,
		target.DateModified = getdate()
	when not matched by target then
		insert (Kind, [Date], [No], Agent, [Sum], Memo)
		values (@Kind, [Date], [No], Agent, [Sum], Memo)
	output $action op, inserted.Id id
	into @output(op, id);

	select top(1) @RetId = id from @output;

	merge a2v10sample.DocDetails as target
	using @Rows as source
	on (target.Id = source.Id and target.Document = @RetId)
	when matched then update set
		target.RowNo   = source.RowNumber,
		target.Product = source.Product,
		target.Qty     = source.Qty,
		target.Price   = source.Price,
		target.[Sum]   = source.[Sum],
		target.Memo    = source.Memo
	when not matched by target then
		insert (Document, RowNo, Qty, Price, [Sum], Product, Memo)
		values (@RetId, RowNumber, Qty, Price, [Sum], Product, Memo)
	when not matched by source and target.Document = @RetId then delete;

	exec a2v10sample.[Document.Load] @UserId, @RetId;
end
go
```

## Notes

- The `.Update` procedure must call the corresponding `.Load` procedure at the end to return the updated model to the client.
- The `.Metadata` procedure only defines the schema — its result sets must be **empty** (select from empty table variables).
- For debugging, dump a TVP to XML inside the procedure:

```sql
declare @xml nvarchar(max);
set @xml = (select * from @Rows for xml auto);
throw 60000, @xml, 0;
```
