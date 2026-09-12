# PeriodPicker

> A period selector — its value is always a TPeriod, shown as a field or as a hyperlink.

## Overview

`PeriodPicker` lets the user pick a reporting period. Its `Value` must always be bound to an instance of `TPeriod` — not to a pair of dates.

It inherits from `ValuedControl → Control → UIElement → UIElementBase`.

## Use When

- A report or a list is filtered by a period the user chooses — a month, a quarter, an arbitrary range.

## Do Not Use When

- A single date is needed — use [DatePicker](https://docs-llm.a2v10.com/xaml/controls/datepicker.md) instead.
- The two ends of the range are independent model fields with their own meaning — two date pickers are then closer to the data.

## Syntax

```xml
<PeriodPicker Label="Period" Value="{Bind Filter.Period}" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Align` | TextAlign | Text alignment |
| `Placement` | DropDownPlacement | Where the selector drops: `BottomLeft` (default), `BottomRight`, `TopLeft`, `TopRight` — the first word is the direction, the second the anchored edge |
| `Style` | PeriodPickerStyle | `Default` — an ordinary selector, `Hyperlink` — displayed as a hyperlink |
| `Display` | DisplayMode | What the selector's caption shows: `Date` (default) — only the dates, `Name` — the name of the period, `NameDate` — both |
| `ShowAllData` | Boolean | Whether to offer the "all time" menu item |
| `Size` | ControlSize | Element size. Currently supported for the `Hyperlink` style only: `Default` or `Large`; the other `ControlSize` values are not supported |

See [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for the properties inherited from `ValuedControl` and `Control` — `Value`, `Label`, `Disabled`, `Width` and the rest.

## Example

```xml
<!-- in a taskpad filter -->
<Taskpad Title="Filter" Width="260px">
  <Grid Columns="1*">
    <PeriodPicker Label="Period" Value="{Bind Filter.Period}"
                  Display="NameDate" ShowAllData="True" />
    <Button Content="Apply" Command="{BindCmd Reload}" Style="Primary" />
  </Grid>
</Taskpad>

<!-- as a hyperlink in a report header -->
<PeriodPicker Value="{Bind Filter.Period}" Style="Hyperlink" Size="Large"
              Display="Name" Placement="BottomRight" />
```

## Notes

- `Display` chooses between the period's name and its dates in the caption; `NameDate` is the one that shows both.
- `ShowAllData` adds an "all time" option; without it the user must always pick a bounded range.
- `Size` is ignored outside the `Hyperlink` style.
