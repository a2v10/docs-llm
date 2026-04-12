# XAML Base Classes

> Inherited properties available on all or most XAML elements — the common foundation of the element hierarchy.

## Overview

Every XAML element inherits from one or more abstract base classes. Understanding what each base class provides means you know which properties are available on any concrete element without looking up each one individually.

The inheritance chain from most general to most specific:

```
UIElementBase   — visibility, spacing, tooltip, render mode
  └── UIElement — font styling, CSS classes
        ├── Container — children collection, items source
        │     └── RootContainer — Page, Dialog roots
        └── Control   — label, disabled, width, popover
              └── ValuedControl  — Value, ValidateValue
                    └── ContentControl — content display
                          └── CommandControl — Command binding
```

## UIElementBase

Base for all XAML elements.

| Property | Type | Description |
|----------|------|-------------|
| `If` | Boolean? | Include element in render flow. When `False`, element is not emitted to HTML at all. Use `Bind` for dynamic control. |
| `Show` | Boolean? | Show the element (inverse of `Hide`). CSS-level toggle — element is still in the DOM. |
| `Hide` | Boolean? | Hide the element. CSS-level toggle. |
| `Margin` | Thickness | Outer spacing around the element. |
| `Padding` | Thickness | Inner spacing inside the element. |
| `Wrap` | WrapMode | Text wrapping: `Wrap`, `NoWrap`, `PreWrap` |
| `Absolute` | Thickness | Absolute CSS positioning. |
| `Tip` | String | Tooltip shown on hover. |
| `Render` | RenderMode? | Server-side render control (unlike `If`, processed before sending to client). |
| `HtmlId` | String | HTML `id` attribute for testing or focus navigation. |
| `XamlStyle` | StyleDescriptor | Apply a named style (must reference `StyleResource`). |

## UIElement

Extends `UIElementBase` with font and CSS styling.

| Property | Type | Description |
|----------|------|-------------|
| `Bold` | Boolean? | Bold font. Supports `Bind`. |
| `Italic` | Boolean? | Italic font. Supports `Bind`. |
| `CssClass` | String | Additional CSS class names. Supports `Bind` (binding value must be a Vue class object or string). |
| `UserSelect` | Boolean? | Allow text selection with mouse. |
| `Print` | Boolean? | Include element in print output. |

## Container

Extends `UIElement`. Used for layout elements that hold child elements.

| Property | Type | Description |
|----------|------|-------------|
| `Children` | UIElementCollection | **Content property.** Collection of child UI elements. |
| `ItemsSource` | Array | Data source for generating children dynamically. Always a `Bind`. |

Derived containers: `Grid`, `StackPanel`, `Toolbar`, `FieldSet`, `Flex`, `Group`, `Panel`, `Card`, `CommandBar`, `FlexList`, `FullHeightPanel`, `DropDownMenu`, `TabPanel`, `Splitter`, and more.

`RootContainer` adds nothing to the API surface but is the direct parent of `Page` and `Dialog`.

## Control

Extends `UIElement`. Base for all interactive controls.

| Property | Type | Description |
|----------|------|-------------|
| `Label` | String | Text label displayed above or beside the control. Supports `Bind`. |
| `Description` | String | Helper text displayed below the control. Supports `Bind`. |
| `Required` | Boolean | Visual required indicator (red asterisk). Does not validate — use `Validator` for that. |
| `Disabled` | Boolean | Makes the control unavailable for input. Supports `Bind`. |
| `Width` | Length | Control width. Default: 100% of parent. |
| `TabIndex` | Int32 | Tab navigation order. `-1` excludes from tab chain. |
| `Block` | Boolean | Render as block element (not inline). |
| `TestId` | String | Identifier for test automation tools. |
| `Popover` | Popover | Floating help tooltip shown as `?` icon on the right side. |
| `Hint` | Popover | Floating tooltip shown as icon after the label. |
| `Validator` | Validator | Configures validator display (position, width). |
| `AddOns` | UIElementCollection | Extra controls appended after the field, typically hyperlinks. |
| `Link` | UIElement | Supplementary link displayed in the upper-right corner. |

## ValuedControl

Extends `Control`. Base for all controls that hold a value.

| Property | Type | Description |
|----------|------|-------------|
| `Value` | Object | **Binding only.** The control's data value, bound to model property. |
| `ValidateValue` | Object | **Binding only.** Alternative binding for validation when the validated field differs from the displayed value. |

Derived: `TextBox`, `DatePicker`, `TimePicker`, `ComboBox`, `Selector`, `CheckBox`, `Radio`, `Static`, `MultiSelect`, `PeriodPicker`, `Pager`, `UploadFile`, and more.

## ContentControl

Extends `Control`. Base for controls that display content (not value-bound).

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Object | The displayed content. Supports `Bind` or literal string. |

## CommandControl

Extends `ContentControl`. Base for controls that trigger commands.

| Property | Type | Description |
|----------|------|-------------|
| `Command` | BindCmd | The command executed when the control is activated. |

Derived: `Button`, `MenuItem`.

## Thickness

Used for `Margin`, `Padding`, `Absolute` properties.

```xml
Margin="8"             <!-- all sides -->
Margin="8,4"           <!-- vertical, horizontal -->
Margin="8,4,8,4"       <!-- top, right, bottom, left -->
```

## Length

Used for `Width`, `Height`, and similar properties.

```xml
Width="200px"
Width="50%"
Width="auto"
Columns="1*,2*,auto"   <!-- CSS grid fr units in Grid element -->
```

## Notes

- `If="{Bind condition}"` and `Show="{Bind condition}"` look similar but behave differently: `If=False` removes the element from the DOM entirely; `Show=False` hides it with CSS while keeping it in the DOM.
- `Disabled` on `Control` affects a single control; `Disabled` on `FieldSet` affects all controls inside it.
- `Label` and `Description` both accept `Bind` — useful for dynamic labels based on model data.
- `Required` is cosmetic only. Actual validation requires a `Validator` element or model-level validators.
