# DatePicker

> A date (or month/year) selection control with a calendar popup.

## Overview

`DatePicker` renders a date input field with a calendar dropdown. It inherits from `ValuedControl → Control → UIElement → UIElementBase`. The bound value is a date property in the model.

Two view modes are supported: `Day` (default, for selecting a specific date) and `Month` (for selecting a month/year period). The control preserves time when only the date is changed — so a date+time value can be set using separate `DatePicker` and `TimePicker` controls.

## Syntax

```xml
<DatePicker Label="Date" Value="{Bind Document.Date}" />
<DatePicker Label="Period" Value="{Bind Document.Period}" View="Month" />
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Align` | TextAlign | Text alignment: `Left`, `Right`, `Center` |
| `View` | DatePickerView | `Day` (default) — date selection; `Month` — month/year selection |
| `Placement` | DropDownPlacement | Calendar popup direction: `BottomLeft` (default), `BottomRight`, `TopLeft`, `TopRight` |
| `Size` | ControlSize | `Default`/`Normal` or `Large` |
| `YearCutOff` | String | Two-digit year interpretation for manual entry. Prefix `+` or `-` for relative to current year. |

### From ValuedControl

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | **Binding only.** The date value. Use `DataType=Date` or `DataType=DateTime` on the Bind. |

See [Base Classes](../base-classes.md) for `Label`, `Required`, `Disabled`, and other inherited properties.

## Example

```xml
<!-- Date-only picker -->
<DatePicker Label="Document Date" Value="{Bind Document.Date}" Required="True" />

<!-- Month/year picker for period selection -->
<DatePicker Label="Report Period"
            Value="{Bind Filter.Month}"
            View="Month" />

<!-- Date + time combo using two controls -->
<Grid Columns="1*,120px">
  <DatePicker Label="Date and Time" Value="{Bind Document.DateTime}" />
  <TimePicker Value="{Bind Document.DateTime}" />
</Grid>

<!-- Calendar opens upward (useful near bottom of page) -->
<DatePicker Label="End Date"
            Value="{Bind Document.EndDate}"
            Placement="TopLeft" />

<!-- Disabled based on model state -->
<DatePicker Label="Approved Date"
            Value="{Bind Document.ApprovedDate}"
            Disabled="{Bind !Document.IsApproved}" />
```

## Notes

- When only the date changes, the time component of the value is preserved unchanged. This enables date+time entry using separate controls.
- `View="Month"` shows a month/year picker — the value is stored as the first day of the selected month.
- `DataType=Date` on the Bind formats the displayed value as date-only; `DataType=DateTime` shows date and time.
- `YearCutOff` applies only to manual keyboard entry of two-digit years, not calendar selection.
- `Placement` is useful when the control is near the bottom of a scrollable area — set to `TopLeft`/`TopRight` to open the calendar upward.
