# XAML (UI Markup)

> UI layer reference for A2v10 — XML views compiled server-side to Vue.js. Page/Dialog roots, data and command binding, controls, layout containers, and text elements.

- [XAML Overview](https://docs-llm.a2v10.com/xaml/overview.md): How XAML views work — Page/Dialog roots, markup extensions, property syntax, element hierarchy
- [Bind & BindCmd](https://docs-llm.a2v10.com/xaml/bind.md): Data binding and command binding — Bind properties, DataType values, all CommandType values
- [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md): Inherited properties on all elements — UIElementBase, UIElement, Inline, Control, ValuedControl, Container
- [Text Elements](https://docs-llm.a2v10.com/xaml/text.md): Inlines that compose text — Text, Paragraph, Span, Hyperlink, Popover, Badge, TagLabel, SpanIcon, SpanSum, StaticImage, Html, Break, Line, and the TextColor values

## Controls

- [Button](https://docs-llm.a2v10.com/xaml/controls/button.md): Command button — Style, Icon, DropDown, ButtonStyle values
- [TextBox](https://docs-llm.a2v10.com/xaml/controls/textbox.md): Text input field — Multiline, Password, Number, UpdateTrigger, EnterCommand
- [DataGrid](https://docs-llm.a2v10.com/xaml/controls/datagrid.md): Data table — ItemsSource, columns, sorting, row marking, DoubleClick, ContextMenu
- [DataGridColumn](https://docs-llm.a2v10.com/xaml/controls/datagrid.md): Column definition — Content, Header, Width, Align, Editable, Command, Role
- [ComboBox](https://docs-llm.a2v10.com/xaml/controls/combobox.md): Dropdown list — static items, bound list, DisplayProperty, ComboBoxItem
- [Selector](https://docs-llm.a2v10.com/xaml/controls/selector.md): Search/lookup control — Delegate, SetDelegate, ItemsPanel, NewPane, server-side search
- [SelectorSimple](https://docs-llm.a2v10.com/xaml/controls/selectorsimple.md): Shorthand selector configured by a single Url — fetch, browse dialog and DisplayProperty by convention
- [CheckBox](https://docs-llm.a2v10.com/xaml/controls/checkbox.md): Boolean checkbox bound to model value
- [DatePicker](https://docs-llm.a2v10.com/xaml/controls/datepicker.md): Date selector with calendar popup — View (Day/Month), Placement
- [Static](https://docs-llm.a2v10.com/xaml/controls/static.md): Read-only display field styled like a disabled TextBox
- [ToolbarAligner](https://docs-llm.a2v10.com/xaml/controls/toolbaraligner.md): Invisible spacer that pushes following elements to the opposite edge — flex-grow for toolbars
- [Graphics](https://docs-llm.a2v10.com/xaml/controls/graphics.md): Drawing surface filled by a d3.js delegate — Delegate, Argument, Watch modes, attaching d3

## Layouts

- [Page](https://docs-llm.a2v10.com/xaml/layouts/page.md): Root element for standalone pages — Title, Toolbar, Taskpad, CollectionView, Pager
- [Dialog](https://docs-llm.a2v10.com/xaml/layouts/dialog.md): Root element for modal dialogs — Size, Placement, Buttons, CanCloseDelegate
- [Grid](https://docs-llm.a2v10.com/xaml/layouts/grid.md): CSS-grid layout container — Columns, Rows, Gap, attached Grid.Col/Row/ColSpan/RowSpan
- [Toolbar](https://docs-llm.a2v10.com/xaml/layouts/toolbar.md): Horizontal action bar — Toolbar.Align for left/right groups, Separator
- [FieldSet](https://docs-llm.a2v10.com/xaml/layouts/fieldset.md): Labeled frame for grouping form controls — Title, Disabled
- [StackPanel](https://docs-llm.a2v10.com/xaml/layouts/stackpanel.md): Single-axis container — Orientation, Gap, Separator for end-alignment
- [TabPanel](https://docs-llm.a2v10.com/xaml/layouts/tabpanel.md): Tabbed content panel — Tab elements, Border, FullPage, dynamic tabs via ItemsSource
- [Repeater](https://docs-llm.a2v10.com/xaml/layouts/repeater.md): Transparent repeating container — renders Content for each item in ItemsSource
- [Sheet](https://docs-llm.a2v10.com/xaml/layouts/sheet.md): Spreadsheet-style report table — header/footer, sections, tree groups, cross columns, Excel export
