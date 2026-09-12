# Reports Overview

> What a Xaml report template is, how it is wired to an endpoint, where its data comes from, and the two ways to write one.

## Overview

A printed form in A2v10 is a markup file with its own set of elements — namespace `A2v10.Xaml.Report` — that the print engine turns into a PDF. It has nothing in common with a view except the word "Xaml": a different element set, a different engine, and a result that is not interactive.

A template can be written in two ways, and they are equal in power:

- Flow — the root element is `Page`, holding columns, tables, lists and text. Written by hand, reads like ordinary markup.
- Workbook — the form is drawn in Excel and converted into a template. Convenient when the form is defined cell by cell and its appearance has to be agreed with someone who does not write markup.

Both use the same engine, the same expressions, the same styling and the same code section. The engine decides which it is from the first significant character of the content: `<` is markup, `{` is a workbook. Neither the file extension nor where the template came from affects this.

Reports are NET Core only.

## Use When

- The printed form is simple and lives next to the code — write flow markup by hand.
- The form is defined cell by cell, or its layout is maintained by someone who does not write markup — draw it in Excel and use a [workbook](https://docs-llm.a2v10.com/report/workbook.md).
- The document must be produced by the platform itself, with no third-party report designer and no extra licence.

## Do Not Use When

- The report is an existing Stimulsoft `.mrt` form — keep `type: stimulsoft`, see [Reports in model.json](https://docs-llm.a2v10.com/model/reports.md).
- The output is a data file rather than a printed document — use the `xml` or `json` [report types](https://docs-llm.a2v10.com/model/reports.md) instead.
- The user has to interact with the result (sort, expand, drill down) — that is a view, not a report; use [Sheet](https://docs-llm.a2v10.com/xaml/layouts/sheet.md) on a page.

## Syntax

The smallest possible template:

```xml
<Page xmlns="clr-namespace:A2v10.Xaml.Report;assembly=A2v10.Xaml.Report">
    <Column>
        <Text>Hello</Text>
    </Column>
</Page>
```

### Wiring

A report is declared in the `reports` section of `model.json`. The type has to be stated explicitly: the default is `stimulsoft`, and a Xaml template is not used in that case.

```json
"reports": {
  "invoice": {
    "model": "Document",
    "type":  "pdf",
    "report": "invoice.report",
    "name":  "Invoice"
  }
}
```

`report` is the template file name without extension, relative to the folder of `model.json`. For the example above the engine looks for `invoice.report.vxaml`, `invoice.report.xaml`, `invoice.report.json` in that order and takes the first one found.

Instead of a file name, `report` may hold an expression in double braces — `"report": "{{Model.FormText}}"`. The template text is then taken from a field of the report model, that is from the database rather than from disk. Either way the content may be markup or a workbook; the format is detected from the content.

The print engine must be registered in the application under the same name as the one given in `type`:

```csharp
services.AddReportEngines(factory =>
{
    factory.RegisterEngine<PdfReportEngine>("pdf");
});
```

### Data

Data for a report is prepared by a stored procedure, exactly as for an ordinary model (`model` and `parameters` in the report description). The name of the procedure is always built from the model — `[schema].[Model.Report]` — and cannot be set to something else. The engine receives an already built model and loads nothing on its own while walking the template: if something is missing from the model, the corresponding place in the document stays empty.

The root of the model is available in expressions under the name `Root`, and its properties simply by name. Details: [Expressions and Bindings](https://docs-llm.a2v10.com/report/bind.md).

## Example

A document header, a table of rows and a total:

```xml
<Page xmlns="clr-namespace:A2v10.Xaml.Report;assembly=A2v10.Xaml.Report"
      Orientation="Portrait" FontFamily="Verdana">
    <Column>
        <Text Style="Title" Align="Center">
            <Span Content="{Bind Document.Name}"/>
        </Text>
        <Text Margin="10,0,0,0">
            <Span Content="{Bind Document.Agent.Name}"/>
        </Text>

        <Table Columns="30pt,1fr,60pt,80pt,80pt" Margin="10,0,0,0"
               ItemsSource="{Bind Document.Rows}">
            <Table.Header>
                <TableRow>
                    <TableCell Content="#"/>
                    <TableCell Content="Item"/>
                    <TableCell Content="Qty"/>
                    <TableCell Content="Price"/>
                    <TableCell Content="Sum"/>
                </TableRow>
            </Table.Header>
            <Table.Body>
                <TableRow>
                    <TableCell Content="{Bind RowNo}"/>
                    <TableCell Content="{Bind Item.Name}"/>
                    <TableCell Content="{Bind Qty, DataType=Number}" Align="Right"/>
                    <TableCell Content="{Bind Price, DataType=Currency}" Align="Right"/>
                    <TableCell Content="{Bind Sum, DataType=Currency}" Align="Right"/>
                </TableRow>
            </Table.Body>
            <Table.Footer>
                <TableRow>
                    <TableCell ColSpan="4" Content="Total" Align="Right"/>
                    <TableCell Content="{Bind Document.Sum, DataType=Currency}" Align="Right"/>
                </TableRow>
            </Table.Footer>
        </Table>
    </Column>
</Page>
```

Table body rows come from the collection named in `ItemsSource`, and inside such a row expressions are read from its element. That is why the body says `{Bind Sum}` while the footer says `{Bind Document.Sum}`.

### Code section

The `Code` property of `Page` is javascript that runs once before the document is built. Functions declared there can be called from expressions:

```xml
<Page.Code>
    const rowTotal = (row) => row.Price * row.Qty;
</Page.Code>
```

The current scope is passed into the function as an argument — `{Bind rowTotal(this)}`. Why it is written that way is explained in [Expressions and Bindings](https://docs-llm.a2v10.com/report/bind.md).

## Notes

- The default report type is `stimulsoft`. A Xaml template declared without `"type": "pdf"` is simply never reached.
- `type: xlsx` is meant to produce an Excel workbook from the same Xaml template, but it is not implemented yet and must not be used.
- The engine does not fetch anything by itself. Everything the template prints must already be in the model returned by the stored procedure.
- Flow and workbook templates differ in one respect only: in a workbook cell the value type is guessed, in markup it is stated explicitly.
- The same template file serves a download, a print dialog and the on-form viewer — see [Viewing and Printing](https://docs-llm.a2v10.com/report/view.md).

## Hints

- A report that renders blank is usually a data problem, not a template problem: run the report procedure and check that the model really contains the object the template reads.
- While debugging the data, declare a second report over the same `model` with `"type": "json"` — the response is then the model itself, exactly as the engine sees it.
