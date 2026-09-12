# Report Expressions and Bindings

> How values reach a report document — the scope rule, Bind in markup versus braces in a workbook cell, types and formats, functions, and quoting.

## Overview

Every expression is evaluated against the current scope. At the top that is the root of the model; inside a table, a list or a named range it is the current element of the collection.

There is one rule: a bare path is read from the current scope; everything else is ordinary javascript, in which `this` is the current scope, free names are read from the root of the model, and `Root` is the explicit root from any depth.

The two places an expression can be written are not interchangeable. In markup a value arrives only through a binding in an attribute; braces in element text mean nothing there and are printed as they are. Substitution directly in text is a property of workbook cells.

## Syntax

The same expression, written in a workbook cell and in markup:

```xml
<!-- workbook cell -->
{Document.Agent.Name}

<!-- markup -->
<TableCell Content="{Bind Document.Agent.Name}"/>
```

The parsing rule is the same in both. `Document.Agent.Name` is a bare path, so it is read straight from the data. Anything that does not look like a path — a call, arithmetic, parentheses, an index — is handed to javascript.

### Scope and leaving it

Scope is changed by the `ItemsSource` of a table or a list, and, in a workbook, by a named range. Inside a row, a path is read from its element:

```xml
<Table ItemsSource="{Bind Document.Rows}" Columns="1fr,80pt">
    <Table.Body>
        <TableRow>
            <!-- a field of the row -->
            <TableCell Content="{Bind Item.Name}"/>
            <!-- a field of the document: out to the root -->
            <TableCell Content="{Bind Root.Document.Date, DataType=Date}"/>
        </TableRow>
    </Table.Body>
</Table>
```

Nesting works by itself: a table inside a cell of another table evaluates its own `ItemsSource` from the row it stands in.

### Type and format

In a workbook cell the format is written after a colon; in a binding it is given by the `DataType` and `Format` properties:

```xml
{Document.Sum:#,##0.00}

<TableCell Content="{Bind Document.Sum, DataType=Currency}"/>
```

A colon separates the format only after a path: inside an expression it is the conditional operator, not a format.

| DataType | How it prints |
|----------|---------------|
| `String` | As is (default) |
| `Currency` | Thousands separator and at least two decimals: `1 234,50`. More decimals in the value are printed too, up to four |
| `Number` | Thousands separator, decimals only if there are any: `1 234,5` |
| `Date` | Short date |
| `Time` | Time |
| `DateTime` | Date and time |

The exact appearance depends on the user's language. When a specific appearance is required, give a format rather than a type — `{Bind Document.Date, Format='dd.MM.yyyy'}`, or `{Document.Date:dd.MM.yyyy}` in a workbook cell.

A `DataType` or `Format` that is stated always applies — in a cell and in markup, to a path and to an expression alike. When it is not stated, the type is guessed only in a workbook cell and only from the value that came with the data: a `Decimal` prints as currency, a date as date and time.

### Quoting in markup

Inside the braces of a markup extension a space separates arguments, so an expression containing spaces has to be quoted — both as an argument and as a property value:

```xml
<!-- no spaces: no quotes needed -->
<TableCell Content="{Bind Sum, DataType=Currency}"/>
<TableCell Content="{Bind rowTotal(this), DataType=Currency}"/>

<!-- spaces: single quotes -->
<TableCell Content="{Bind 'Price * Qty', DataType=Currency}"/>
<TableCell Content="{Bind 'Document.Done ? 1 : 2'}"/>

<!-- the same as a separate element, when that reads better -->
<TableCell>
    <TableCell.Content>
        <Bind Expression="Price * Qty" DataType="Currency"/>
    </TableCell.Content>
</TableCell>
```

A comma inside quotes does not separate either: `Format='#,##0.00'` is read whole.

The second kind of quote is the backtick, `` `…` ``. Unlike single quotes, backticks stay in the expression: what is inside them is a javascript template string with `${…}` substitutions. Single quotes inside them do not close the value, which makes this the only way to write an expression in an attribute that itself contains single quotes:

```xml
<Span Content="{Bind `Invoice # ${Document.Number} of ${formatDate(Document.Date, 'dd.MM.yyyy')}`}"/>
```

A workbook cell has none of this — there an expression is written as it is.

### Built-in functions

| Function | Description |
|----------|-------------|
| `spellMoney(value, currency)` | An amount spelled out in words. `currency` is a currency code, `'980'` by default |
| `spellMoneyEn(value, currency)` | The same in English |
| `formatDate(value, format)` | A date in the given format, for example `'dd.MM.yyyy'` |
| `qrCode(value)` | A QR code built from the value. A cell draws it as an image |

### Conditional display

Every element has an `If` property. The expression is read by the same rule, and an exclamation mark in front of it inverts the result:

```xml
<Text If="{Bind !Document.Done}">Not posted</Text>
```

## Example

Custom functions are declared in the `Code` section of the template. The current scope is passed into them as an argument — `this` in the body of an arrow function does not mean what the author expects, so the platform does not rely on it:

```xml
<Page.Code>
    const rowTotal = (row) => row.Price * row.Qty;
    const total = (doc) => doc.Rows.reduce((s, r) => s + rowTotal(r), 0);
</Page.Code>

<TableCell Content="{Bind rowTotal(this), DataType=Currency}"/>
<TableCell Content="{Bind total(Document), DataType=Currency}"/>
```

Model collections behave like ordinary javascript arrays, so a total can be computed right in a cell with no function at all:

```js
{(Document.Rows.reduce((s, r) => s + r.Sum, 0))}
```

## Notes

- A computed number never becomes currency on its own: it arrives as a plain fraction and guessing yields "number". In a cell, `{Price}` prints `10.00` while `{(Price * 1)}` prints `10`.
- In markup nothing is guessed at all: `{Bind Sum}` without `DataType` prints exactly what came from the database — a `money` field gives `1234.5000`. State the type or the format explicitly in both places.
- Anything that exists only in javascript cannot be reached by a path: `{Rows.length}` prints nothing, because the data has no such field. Write it as an expression — `{(Rows.length)}`.
- An expression with spaces and no quotes stops markup parsing with the error *"Quote a value with spaces: 'text' or \`template\`"*, showing the whole extension. An unclosed quote is a parse error too, not a binding without an expression.
- Every expression runs within a budget: no deeper than 256 calls, a limited number of operations, and no longer than 5 seconds. An endless loop or unbounded recursion in the `Code` section stops printing with an error instead of hanging the server. The budget is counted per expression, so a long document with thousands of cells does not exhaust it.
- Formatting inside a backtick template string bypasses `DataType` and `Format`. When an expression grows into a template string, it is usually simpler to split it into several `Span` elements or to move it into a `Code` function.

## Hints

- Outer parentheses are the switch between the two halves of the rule: `{Path}` reads data, `{(anything)}` is evaluated as javascript.
- Previously an unquoted expression with spaces silently kept the last word, turning `{Bind Price * Qty}` into a binding to `Qty`. If an old template suddenly refuses to load with a quoting error, that is the fix — the expression was wrong all along.
