# Dialog

> The root element for modal views opened by BindCmd Dialog commands.

## Overview

`Dialog` is the root element of a dialog XAML file. Dialogs are opened programmatically via `{BindCmd Dialog, Action=...}` commands and are not standalone routes. They inherit from `RootContainer → Container → UIElement → UIElementBase`.

A dialog has a title bar, a content area, and a button panel (`Buttons`). The `SaveAndClose` and `Close` / `CloseOk` commands are used to close dialogs. Dialogs support various sizes and placements including side panels and full-screen modes.

## Syntax

```xml
<Dialog
    xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
    Title="Edit Document"
    Size="Medium">

  <Grid Columns="1*">
    <TextBox Label="Name" Value="{Bind Document.Name}" Required="True" />
  </Grid>

  <Dialog.Buttons>
    <Button Content="Save" Command="{BindCmd SaveAndClose}" Style="Primary" />
    <Button Content="Cancel" Command="{BindCmd Close}" />
  </Dialog.Buttons>
</Dialog>
```

## Properties

| Property | Type | Description |
|----------|------|-------------|
| `Title` | String | Dialog title bar text. Supports `Bind`. |
| `HelpUrl` | String | Help documentation path — creates a `?` hyperlink in the title bar. |
| `TitleInfo` | UIElementBase | Additional element rendered after the title text. |
| `Buttons` | UIElementCollection | Button panel content (bottom or top based on `ButtonOnTop`). |
| `Taskpad` | UIElementBase | Side task panel, typically a `Taskpad` element. |
| `Size` | DialogSize | Predefined size (see values below). Ignored if `Width` is set explicitly. |
| `Width` | Length | Explicit dialog width. |
| `Height` | Length | Explicit dialog height. |
| `MinWidth` | Length | Minimum dialog width. |
| `MinHeight` | Length | Minimum dialog height. |
| `Placement` | DialogPlacement | Screen position: `Default`, `SideBarRight`, `SideBarLeft`, `FullScreen` |
| `Maximize` | Boolean | Responsive sizing based on browser window dimensions. |
| `ButtonOnTop` | Boolean | Place button panel at the top of the dialog. |
| `ShowWaitCursor` | Boolean | Show loading indicator and dim dialog during server requests. |
| `AlwaysOk` | Boolean | Treat any close action as returning `true`. |
| `CanCloseDelegate` | String | JavaScript delegate called before close — can cancel or modify close behavior. |
| `Background` | BackgroundStyle | Dialog background color. |
| `Overflow` | Boolean | Allow scrollbars; prevents popup elements (DatePicker, etc.) from being clipped. |
| `SaveEvent` | String | Event name triggered when saving objects within the dialog. |
| `TestId` | String | Identifier for test automation. |
| `CollectionView` | CollectionView | Collection management at dialog level. |
| `HeaderMenu` | DropDownMenu | Dropdown menu in the title bar. |

### DialogSize Values

| Value | Description |
|-------|-------------|
| `Default` | Content-determined size |
| `Small` | Small dialog |
| `Medium` | Medium dialog |
| `Large` | Large dialog |
| `Max` | Maximum size |

### DialogPlacement Values

| Value | Description |
|-------|-------------|
| `Default` | Centered on screen |
| `SideBarRight` | Right side panel |
| `SideBarLeft` | Left side panel |
| `FullScreen` | Full browser window |

## Example

```xml
<!-- Standard edit dialog -->
<Dialog xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
        Title="Edit Customer"
        Size="Medium">

  <Grid Columns="1*,1*">
    <TextBox Label="Name" Value="{Bind Customer.Name}" Required="True" />
    <TextBox Label="Code" Value="{Bind Customer.Code}" />
    <TextBox Label="Phone" Value="{Bind Customer.Phone}" />
    <TextBox Label="Email" Value="{Bind Customer.Email}" />
    <TextBox Label="Notes" Value="{Bind Customer.Notes}"
             Multiline="True" Rows="3" Grid.ColSpan="2" />
  </Grid>

  <Dialog.Buttons>
    <Button Content="Save" Command="{BindCmd SaveAndClose}"
            Style="Primary" Icon="Save" />
    <Button Content="Cancel" Command="{BindCmd Close}" />
  </Dialog.Buttons>
</Dialog>
```

```xml
<!-- View-only dialog (no save) -->
<Dialog xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
        Title="Document Details"
        Size="Large">

  <Grid Columns="1*,1*">
    <Static Label="Number" Value="{Bind Document.Number}" />
    <Static Label="Date" Value="{Bind Document.Date, DataType=Date}" />
  </Grid>

  <Dialog.Buttons>
    <Button Content="Close" Command="{BindCmd Close}" />
  </Dialog.Buttons>
</Dialog>
```

```xml
<!-- Side panel dialog -->
<Dialog xmlns="clr-namespace:A2v10.Xaml;assembly=A2v10.ViewEngine.Xaml"
        Title="Filters"
        Placement="SideBarRight"
        Width="320px">

  <FieldSet Title="Date Range">
    <DatePicker Label="From" Value="{Bind Filter.DateFrom}" />
    <DatePicker Label="To" Value="{Bind Filter.DateTo}" />
  </FieldSet>

  <Dialog.Buttons>
    <Button Content="Apply" Command="{BindCmd CloseOk}" Style="Primary" />
    <Button Content="Reset" Command="{BindCmd Close}" />
  </Dialog.Buttons>
</Dialog>
```

## Notes

- `Dialog` must be the XML root element with the `xmlns` attribute.
- `SaveAndClose` saves the model and closes with `true`; `CloseOk` closes with `true` without saving; `Close` cancels.
- `Overflow="True"` is required when dialog content includes DatePicker, ComboBox, or other popups that render outside their container.
- `CanCloseDelegate` receives the root model as `this`; return `false` or a rejected Promise to prevent closing.
- `CollectionView` at dialog level manages collections (like a list inside a dialog) independently from the main content.
- `AlwaysOk=True` is useful for read-only "show" dialogs where any close action should be treated as acceptance.
