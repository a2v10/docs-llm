# XAML (UI Markup)

> UI layer reference for A2v10 — XML views compiled server-side to Vue.js. Page/Dialog roots, data and command binding, controls, and layout containers.

- [XAML Overview](xaml/overview.md): How XAML views work — Page/Dialog roots, markup extensions, property syntax, element hierarchy
- [Bind & BindCmd](xaml/bind.md): Data binding and command binding — Bind properties, DataType values, all CommandType values
- [Base Classes](xaml/base-classes.md): Inherited properties on all elements — UIElementBase, UIElement, Control, ValuedControl, Container

## Controls

- [Button](xaml/controls/button.md): Command button — Style, Icon, DropDown, ButtonStyle values
- [TextBox](xaml/controls/textbox.md): Text input field — Multiline, Password, Number, UpdateTrigger, EnterCommand
- [DataGrid](xaml/controls/datagrid.md): Data table — ItemsSource, columns, sorting, row marking, DoubleClick, ContextMenu
- [DataGridColumn](xaml/controls/datagrid.md): Column definition — Content, Header, Width, Align, Editable, Command, Role
- [ComboBox](xaml/controls/combobox.md): Dropdown list — static items, bound list, DisplayProperty, ComboBoxItem
- [Selector](xaml/controls/selector.md): Search/lookup control — Delegate, SetDelegate, ItemsPanel, NewPane, server-side search
- [CheckBox](xaml/controls/checkbox.md): Boolean checkbox bound to model value
- [DatePicker](xaml/controls/datepicker.md): Date selector with calendar popup — View (Day/Month), Placement
- [Static](xaml/controls/static.md): Read-only display field styled like a disabled TextBox

## Layouts

- [Page](xaml/layouts/page.md): Root element for standalone pages — Title, Toolbar, Taskpad, CollectionView, Pager
- [Dialog](xaml/layouts/dialog.md): Root element for modal dialogs — Size, Placement, Buttons, CanCloseDelegate
- [Grid](xaml/layouts/grid.md): CSS-grid layout container — Columns, Rows, Gap, attached Grid.Col/Row/ColSpan/RowSpan
- [Toolbar](xaml/layouts/toolbar.md): Horizontal action bar — Toolbar.Align for left/right groups, Separator
- [FieldSet](xaml/layouts/fieldset.md): Labeled frame for grouping form controls — Title, Disabled
- [StackPanel](xaml/layouts/stackpanel.md): Single-axis container — Orientation, Gap, Separator for end-alignment
- [TabPanel](xaml/layouts/tabpanel.md): Tabbed content panel — Tab elements, Border, FullPage, dynamic tabs via ItemsSource
- [Repeater](xaml/layouts/repeater.md): Transparent repeating container — renders Content for each item in ItemsSource
- [Sheet](xaml/layouts/sheet.md): Spreadsheet-style report table — header/footer, sections, tree groups, cross columns, Excel export
