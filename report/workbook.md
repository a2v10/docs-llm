# Report Workbook

> The second way to make a printed form — drawn in Excel and converted into a template: cells, named ranges, row heights and column widths, and the page footer.

## Overview

A workbook template is the same report as a hand-written one — the same model, the same expressions, the same engine — but instead of flow elements it describes cells: their content, column widths, merging, borders and formats.

The form is drawn in Excel and converted: the platform has a handler that takes an uploaded `.xlsx` file and returns a finished template in json. The application wires that handler up with a command of its own, and the result is stored as a template file or in the database.

Only the first sheet of the workbook is converted. The remaining sheets are not transferred at all.

The template itself is still a page: orientation, margins, font, title, the `Code` section and the watermark are set the same way as on [Page](https://docs-llm.a2v10.com/report/elements.md#page).

## Use When

- The form is defined cell by cell, or its layout is maintained by someone who does not write markup.
- An existing Excel blank has to be reproduced exactly.

## Do Not Use When

- The form is simple and lives next to the code — hand-written [flow elements](https://docs-llm.a2v10.com/report/elements.md) are quicker to write and to edit.

## Syntax

What the converter carries over from the workbook:

- cell content — as text, together with the expressions in braces;
- column widths and row heights;
- merged cells — they become `ColSpan` and `RowSpan`;
- styling: font, weight, borders, alignment, text rotation, grey background;
- numeric cell formats — they become the format of the value;
- page margins and orientation;
- named ranges — [Range](#range);
- the page footer — [PageFooter](#pagefooter).

A template looks like this:

```json
{
  "Workbook": {
    "RowCount": 15,
    "ColumnCount": 8,
    "Columns": { "A": { "Width": 21.75 }, "B": { "Width": 61.5 } },
    "Rows": { "1": { "Height": 31.5 } },
    "Cells": {
      "A1": { "Value": "Invoice # {Document.Number} of {Document.Date:dd-MM-yyyy}",
              "ColSpan": 8, "Style": "S32" },
      "C3": { "Value": "{Document.Company.Name}", "ColSpan": 3, "Style": "S30" }
    },
    "Ranges": [ { "Value": "{Document.Rows}", "Start": 10, "End": 10 } ],
    "Styles": { "S30": { }, "S32": { } }
  },
  "FontFamily": "Arial",
  "FontSize": 9,
  "Margin": "10mm,10mm"
}
```

### Workbook

| Property | Type | Description |
|----------|------|-------------|
| `RowCount` | UInt32 | The number of rows in the workbook |
| `ColumnCount` | UInt32 | The number of columns in the workbook |
| `Cells` | CellCollection | Cells by address: `"A1"`, `"C3"` — see [Cell](#cell). Empty cells are not stored |
| `Rows` | RowCollection | Row heights by number — see [Row and Column](#row-and-column) |
| `Columns` | ColumnCollection | Column widths by letter — see [Row and Column](#row-and-column) |
| `Ranges` | RangeCollection | Repeating ranges — see [Range](#range) |
| `Header` | [Range](#range) | Rows printed at the start of every page |
| `Footer` | [Range](#range) | Rows printed at the end of every page |
| `TableHeader` | [Range](#range) | The header of a long table: repeated on every page the table continues onto |
| `TableFooter` | [Range](#range) | The footer of a long table: printed once, after the rows |
| `PageFooter` | [PageFooter](#pagefooter) | The page footer, usually with a number |
| `Styles` | StyleCollection | The set of styling the cells refer to. Created by the converter from the styling of the workbook; not written by hand |
| `ColumnWidth` | Single? | Default column width in points, for columns absent from `Columns` |
| `RowHeight` | Single? | Default row height in points |

### Cell

A cell of the workbook. Stored under its address (`"A1"`) and almost always created by the converter from an Excel cell — that is, written in the workbook rather than here.

| Property | Type | Description |
|----------|------|-------------|
| `Value` | String | The content of the cell: text in which values from the model are substituted in braces — see [Expressions and Bindings](https://docs-llm.a2v10.com/report/bind.md) |
| `ColSpan` | UInt32 | How many columns the cell occupies (from merged Excel cells) |
| `RowSpan` | UInt32 | How many rows the cell occupies |
| `Style` | String | A reference to a styling set of the workbook (`"S30"`) |
| `Format` | String | The format of the value. Usually arrives from the numeric format of the Excel cell through the styling, so it is rarely given separately |
| `DataType` | DataType | The type of the value: `String`, `Number`, `Currency`, `Date`, `Time`, `DateTime`. When not specified, it is determined from the value itself |

Plus the properties of [the base element](https://docs-llm.a2v10.com/report/elements.md#base-element).

What can be written in a cell:

```js
Invoice # {Document.Number} of {Document.Date:dd-MM-yyyy}
{Document.Company.Name}
{Item.Name}
{Sum:#,##0.00}
{(spellMoney(Document.Sum))}
{Company.Logo}
```

Text and substitutions mix freely; a format is written after a colon. If the value turns out to be bytes, the cell draws a picture — that is how a logo gets into a form, see [Images and Codes](https://docs-llm.a2v10.com/report/images.md).

### Range

A range is several rows of the workbook with a special purpose: they repeat for every element of a collection, print on every page, or serve as the header of a long table. In Excel it is defined as a named range of whole rows.

| Property | Type | Description |
|----------|------|-------------|
| `Value` | String | The name of the range in braces — for repeating ranges, the expression of the collection |
| `Start` | UInt32 | The number of the first row, inclusive |
| `End` | UInt32 | The number of the last row, inclusive |

Select whole rows — rows, not cells — and give the selection a name. Excel does not allow a dot in names, so a path is written with an underscore and the converter turns it into a dot:

| Name in Excel | What it means |
|---------------|---------------|
| `Document_Rows` | Becomes `{Document.Rows}`: the rows of the range repeat for every element of the collection, and inside them expressions are read from that element (`{Item.Name}`, `{Sum}`) |
| `Header` | The rows print at the start of every page |
| `Footer` | The rows print at the end of every page |
| `TableHeader` | The header of a long table: repeated on every page the table continues onto. This, not `Header`, is what "table header on every sheet" needs |
| `TableFooter` | The total rows of a table: printed once, after all the rows |
| `Fit` | The exception: it names one column, not rows. That column stretches over all the free width of the page, while the rest keep their given widths |

A range that starts inside another one is treated as nested: for every element of the outer collection it walks its own, taken from that element. That is how, for example, serial numbers inside a line of a delivery note are printed.

### Row and Column

Sizes of rows and columns. Both are stored only for those whose size differs from the default — the rest take the default from [Workbook](#workbook).

| Element | Property | Type | Description |
|---------|----------|------|-------------|
| `Row` | `Height` | Single | Row height in points |
| `Column` | `Width` | Single | Column width in points. A negative value means the column stretches over the free space — that is how the converter marks the column named [Fit](#range) |

Rows are stored by number, columns by letter:

```json
"Rows":    { "1": { "Height": 31.5 } },
"Columns": { "A": { "Width": 21.75 } }
```

### PageFooter

The page footer — the one set in the Excel page setup. Printed on every page, centred, below the content.

| Property | Type | Description |
|----------|------|-------------|
| `Center` | String | The centre text of the footer |
| `Left` | String | The left part |
| `Right` | String | The right part |

Excel codes become service marks that are substituted while printing:

| In Excel | In the template | What prints |
|----------|-----------------|-------------|
| `&P` | `&(Page)` | The current page number |
| `&N` | `&(Pages)` | The total number of pages |

A footer typed in Excel as `&C&P of &N` therefore prints as "1 of 3".

## Example

A delivery note form — which rows of the sheet get which names:

```js
rows 1-9     document header (ordinary cells)
row  9       named range TableHeader - "Item | Qty | Price | Sum"
row  10      named range Document_Rows - one item row
rows 11-12   named range TableFooter - "Total"
rows 13-15   signatures
```

In the template that single repeating row looks like this:

```json
"Ranges": [ { "Value": "{Document.Rows}", "Start": 10, "End": 10 } ]
```

## Notes

- A range must consist of whole rows (in Excel the reference looks like `$10:$10`). A named range of cells is not treated as a range and is silently skipped — except for the single case of `Fit`, which on the contrary must be a whole column.
- The nesting depth of ranges is two levels. A third range inside the second one is not processed.
- Only `Center` of the page footer actually works: the converter carries over the centre part alone, and only that part is printed. The left and right parts can be set in Excel but will not reach the report.
- Do not confuse the page footer with the `Footer` range: the range is made of ordinary workbook cells, also prints on every page, but its content is anything at all, with expressions and styling.
- Column width in Excel is measured in characters of the font rather than in points, and the converter recalculates it. The numbers in the template do not match what Excel shows, and that is normal.
- A `Model` section that can be seen next to `Workbook` in forms of the metadata layer has nothing to do with the print engine: it describes which data to select and is read on the other side. The engine receives a finished model and ignores that section.
- Editing the converted file by hand is possible but pointless: the converter recreates it from scratch every time. The exception is the expressions in cells — and even those are easier to fix in the workbook and convert again.

## Hints

- A table header that has to repeat on every sheet is the `TableHeader` range, not `Header`. `Header` is the page header, which prints above everything including the first page.
- A format is usually easier to set in the workbook itself: the numeric format of the cell is carried over, so the text can stay plain `{Sum}`.
