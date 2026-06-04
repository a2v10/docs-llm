# Cross (Pivot) Models

> Datasets whose columns are determined by the data — the `!CrossArray` and `!CrossObject` model types, the `!Key` / `!ParentId` markers, and the `$cross` service property.

## Overview

A cross model contains one or more fields that are expanded either into a horizontal array of elements (`CrossArray` type) or into an object keyed by value (`CrossObject` type). Such models are mostly used to build cross (pivot) reports where the number of columns is variable and determined by the returned data. This is similar to the SQL `PIVOT` operator, but — unlike `PIVOT` — you do not need to know in advance which columns will appear in the result set.

Cross models only make sense for arrays. Using them for plain objects (not arrays) is pointless (though not forbidden) — use `MapObject` instead.

For cross arrays the differences from ordinary nested arrays are:

- Every cross array in every record has the same size.
- Element order is determined by the cross element's `Key`.
- The main array (the one holding elements with cross arrays) gets an extra `$cross` property containing the arrays of keys for each cross field.

For cross objects the logic is:

- Every cross object in every record has the same structure.
- The main array gets the same extra `$cross` property containing the key arrays for each cross field.

## Syntax

The main set declares a placeholder cross column; a following set fills it and links back by parent id.

| Marker | Description |
|--------|-------------|
| `[Field!TCross!CrossArray]` | Placeholder cross column in the main set — becomes the cross array (`CrossObject` for object form) |
| `[!TCross!CrossArray]` | Root marker of the cross set |
| `[Key!!Key]` | Text key — used internally, determines element order, and is exposed via `$cross` |
| `[!TMain.Field!ParentId]` | Links a cross row to the main element — `Type.Field` names the type and the cross field on it |

Each main element must expose an `[Id!!Id]` field so later sets can reference it.

```sql
/* main set */
select [RepData!TData!Array] = null, [Id!!Id] = Id, S1, N1,
    [Cross1!TCross!CrossArray] = null
from ...;

/* cross set */
select [!TCross!CrossArray] = null, [Key!!Key] = [Key], Val,
    [!TData.Cross1!ParentId] = Id
from ...;
```

## How It Works

Processing the first set creates a plain `RepData` array of `TData` elements. Each element has a `Cross1` field that will later become a cross array of `TCross` elements.

The second set is processed as follows. First the platform finds the reference to the cross field on the main element (the `!ParentId` field). The reference type names the type and field separated by a dot — `!TData.Cross1!ParentId` means the `Cross1` field on a `TData` element. The cross record's contents are stored in an internal buffer on the matched main record.

A cross array must contain a text `!Key` field — it is used for internal processing and is the value placed into the array returned by `$cross`. That property is added to the array itself (not to its elements), so the key list is reached as `RepData.$cross.Cross1`.

After all cross rows are processed, the platform takes the union of all possible keys across every `TData` record and turns the internal buffers into arrays. As a result all cross arrays have the same length, with missing values filled by `0`.

### Example Dataset

Main set:

| RepData!TData!Array | Id!!Id | S1 | N1 | Cross1!TCross!CrossArray |
|---|---|---|---|---|
| null | 10 | Str1 | 100 | null |
| null | 20 | Str2 | 200 | null |

Cross set:

| !TCross!CrossArray | Key!!Key | Val | !TData.Cross1!ParentId |
|---|---|---|---|
| null | K1 | 111 | 10 |
| null | K2 | 222 | 20 |

### Resulting Model

```json
{
  "RepData": [
    {
      "Id": 10,
      "S1": "Str1",
      "N1": 100,
      "Cross1": [
        { "Key": "K1", "Val": 111 },
        { "Key": "K2", "Val": 0 }
      ]
    },
    {
      "Id": 20,
      "S1": "Str2",
      "N1": 200,
      "Cross1": [
        { "Key": "K1", "Val": 0 },
        { "Key": "K2", "Val": 222 }
      ]
    }
  ],
  "$cross": { "Cross1": ["K1", "K2"] }
}
```

## Example

Cross models are most often used to build reports together with [Sheet](../xaml/layouts/sheet.md) tables. To make the example realistic it also outputs totals. Cross arrays are rendered with the `SheetCellGroup` element.

### Template

Computed properties drive the variable columns and totals. Note `$Cross1Total` returns an array.

```ts
const template = {
    properties: {
        /* properties on the whole array */

        /* number of columns to span (cross keys + total) */
        'TDataArray.$Cross1Span'() { return this.$cross.Cross1.length + 1; },

        /* per-key totals across all rows — returns an array */
        'TDataArray.$Cross1Total'() {
            return this.$cross.Cross1.reduce((prevArray, currKey, currIndex) => {
                prevArray.push({ Val: this.reduce((prevTotal, currElem) => prevTotal + currElem.Cross1[currIndex].Val, 0) });
                return prevArray;
            }, []);
        },

        /* grand total of the Cross1 section over all rows */
        'TDataArray.$GrandTotal'() {
            return this.$Cross1Total.reduce((p, c) => p + c.Val, 0);
        },

        /* properties on each array element */

        /* Cross1 total for a single row */
        'TData.$Cross1Total'() { return this.Cross1.reduce((p, c) => p + c.Val, 0); }
    }
};
```

### XAML

```xml
<Sheet Margin="1rem" GridLines="Both" Compact="True">
    <Sheet.Header>
        <SheetRow Style="Header">
            <SheetCell RowSpan="2">Id</SheetCell>
            <SheetCell RowSpan="2">S1</SheetCell>
            <SheetCell RowSpan="2">N1</SheetCell>
            <SheetCell ColSpan="{Bind RepData.$Cross1Span}">Cross1</SheetCell>
        </SheetRow>
        <SheetRow Style="Header">
            <SheetCellGroup ItemsSource="{Bind RepData.$cross.Cross1}">
                <SheetCell Content="{Bind}" />
            </SheetCellGroup>
            <SheetCell>Total</SheetCell>
        </SheetRow>
        <SheetRow Style="Total">
            <SheetCell ColSpan="3">Total</SheetCell>
            <SheetCellGroup ItemsSource="{Bind RepData.$Cross1Total}">
                <SheetCell Content="{Bind Val}" />
            </SheetCellGroup>
            <SheetCell Content="{Bind RepData.$GrandTotal}" />
        </SheetRow>
    </Sheet.Header>
    <SheetSection ItemsSource="{Bind RepData}">
        <SheetRow>
            <SheetCell Content="{Bind Id}" />
            <SheetCell Content="{Bind S1}" />
            <SheetCell Content="{Bind N1}" />
            <SheetCellGroup ItemsSource="{Bind Cross1}">
                <SheetCell Content="{Bind Val, DataType=Number, HideZeros=True}" />
            </SheetCellGroup>
            <SheetCell Content="{Bind $Cross1Total}" />
        </SheetRow>
    </SheetSection>
</Sheet>
```

### Result

| Id | S1 | N1 | K1 | K2 | Total |
|---|---|---|---|---|---|
| Total | | | 111 | 222 | 333 |
| 10 | Str1 | 100 | 111 | | 111 |
| 20 | Str2 | 200 | | 222 | 222 |

## Notes

- Cross arrays are only meaningful for arrays. For an object use `MapObject` instead of a cross model.
- Every main element must expose `[Id!!Id]` so cross sets can link back to it via `!ParentId`.
- A cross set must include a text `[Key!!Key]` field — it determines element order and is what `$cross` returns.
- The `$cross` property lives on the array, not on its elements — reach it as `RepData.$cross.Cross1`.
- After processing, all cross arrays are padded to the union of keys; missing values become `0`. Use `HideZeros=True` on a cell to suppress them in the UI.
- The `!ParentId` reference type uses dot notation — `TData.Cross1` means the `Cross1` field on a `TData` element.
