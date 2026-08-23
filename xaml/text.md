# Text Elements

> Inline elements used to compose text — Text, Paragraph, Span, Hyperlink, Popover, Badge, TagLabel, SpanIcon, SpanSum, StaticImage, Html, Break, Line.

## Overview

Text elements (also called *inlines*) display text and small images. Most of them are not used on their own: they are placed inside an element whose content property is an `InlineCollection` — `Text` or `Paragraph` — and they flow together as one line of text.

A collection of inlines may hold both child elements and plain text strings. A plain string is transferred into the markup as is, with no extra tags added around it. This is what makes mixed content natural: a sentence with a highlighted fragment, an icon, or a link inside it is written as text with elements interleaved.

All of them except `Text` and `Paragraph` derive from the abstract `Inline` class, which adds `Block`, `Float`, and `Color` on top of [UIElement](https://docs-llm.a2v10.com/xaml/base-classes.md). `Text` and `Paragraph` are containers of inlines rather than inlines themselves, and derive from `UIElement` directly.

Containers of inlines:

- [Text](#text) — a fragment of text; the usual host for other inlines
- [Paragraph](#paragraph) — a paragraph of text

Text pieces and decorations:

- [Span](#span) — a fragment of text, the basic building block
- [Badge](#badge) — a small badge with auxiliary text, such as a count
- [TagLabel](#taglabel) — a coloured tag-style marker, usually an entity state
- [SpanIcon](#spanicon) — an icon inside text
- [SpanSum](#spansum) — a monetary amount coloured by direction
- [StaticImage](#staticimage) — an image inside text
- [Html](#html) — raw HTML markup
- [Break](#break) — a line break
- [Line](#line) — a horizontal rule

Interactive:

- [Hyperlink](#hyperlink) — a link that runs a command
- [Popover](#popover) — a link that opens a popup window

## Inline

The abstract base of every text element. See also [Base Classes](https://docs-llm.a2v10.com/xaml/base-classes.md) for what `UIElement` and `UIElementBase` add — `Bold`, `Italic`, `CssClass`, `Tip`, `If`/`Show`/`Hide`, and the rest.

| Property | Type | Description |
|----------|------|-------------|
| `Block` | Boolean | Render the element as a block rather than inline |
| `Float` | FloatMode | Float the element: `None` (default), `Left`, `Right` |
| `Color` | [TextColor](#textcolor) | Text colour |

## Text

A fragment of text that can contain other inline elements. Content property: `Inlines`.

Inherits `UIElement`.

| Property | Type | Description |
|----------|------|-------------|
| `Inlines` | InlineCollection | Content property. The text, which may contain other inline elements |
| `Align` | TextAlign | Text alignment |
| `Block` | Boolean | Render as a block rather than an inline element |
| `Small` | Boolean | Smaller font. The exact size depends on the UI theme |
| `Big` | Boolean | Larger font. The exact size depends on the UI theme |
| `Size` | TextSize | Text size: `Normal` (default), `Big` — same as `Big="True"`, `Small` — same as `Small="True"` |
| `Gray` | Boolean | Grey text. The exact colour depends on the UI theme |
| `Color` | [TextColor](#textcolor) | Text colour |

## Paragraph

A paragraph of text. Content property: `Inlines`.

Inherits `UIElement`.

| Property | Type | Description |
|----------|------|-------------|
| `Inlines` | InlineCollection | Content property. A collection of strings or `Inline` elements |
| `Small` | Boolean | Smaller font. The exact size depends on the UI theme |
| `Color` | [TextColor](#textcolor) | Text colour |

## Span

A fragment of text — the element reached for most often. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | String | Content property. The text |
| `Small` | Boolean | Smaller font |
| `Big` | Boolean | Larger font |
| `Space` | SpaceMode | Add a space around the element: `None` (default), `Before`, `After`, `Both` |
| `MaxChars` | Int32 | Maximum number of characters. Works for bound values only: longer text is cut and an ellipsis is appended |

## Hyperlink

A link that executes a command on click. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | String | The link text |
| `Command` | [BindCmd](https://docs-llm.a2v10.com/xaml/bind.md) | The command executed on click |
| `Icon` | Icon | Icon shown in the link |
| `Size` | ControlSize | Link size: `Default`, `Small`, `Normal`, `Large` |
| `Style` | HyperlinkStyle | `Default` — an ordinary link; `Popover` — a link for a popup hint, underlined with a dashed line |
| `DropDown` | UIElementBase | Drop-down element attached to the link, normally a `DropDownMenu` |
| `HideCaret` | Boolean | Hide the drop-down caret |
| `Highlight` | Boolean | Highlight the icon with the active-link colour on hover. Only meaningful for some icons, such as `Send` |
| `TestId` | String | Identifier for test automation tools |
| `Hint` | [Popover](#popover) | Popup hint for the link, shown as the icon given in the `Popover`. With no icon, a help icon is used |

## Popover

A link that opens a popup window on click, or on hover. The window can hold plain text or any element, or load its content from a URL. Content property: `Content`.

A popover also converts from a plain string and from any element deriving from `UIElement`, so a property typed as `Popover` — for example `Hint` on a control — accepts those directly.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Object | Content property. Plain text or any element |
| `Url` | String | URL the content is fetched from. The popup then has its own data model, template, and markup |
| `Text` | String | The link text |
| `Icon` | Icon | Icon for the link |
| `ShowOnHover` | Boolean | Open on hover instead of click. Incompatible with external content |
| `Badge` | String | Text badge on the element, usually shown on the icon |
| `Width` | Length | Width of the popup window |
| `Placement` | PopupPlacement | Position of the popup: `TopRight` (default), `TopLeft`, `RightBottom`, `RightTop`, `BottomRight`, `BottomLeft`, `LeftBottom`, `LeftTop`. The first word is the side, the second the direction it unfolds |
| `Underline` | PopoverUnderlineMode | Link underline: `Enable` (default), `Disable`, `ShowOnHover` |
| `Background` | PopoverBackgroundStyle | Popup background: `Default`, `Yellow`, `Cyan`, `Green`, `Red`, `Blue`, `White`. The exact colours depend on the UI theme |
| `OffsetX` | Length | Horizontal offset of the popup. May be negative |
| `MaxChars` | Int32 | Maximum characters in `Text`; longer text is cut with an ellipsis and shown in full as a tooltip |
| `LineClamp` | Int32 | Maximum lines in `Text`; the rest is cut with an ellipsis and shown in full as a tooltip |

## Badge

A small badge carrying auxiliary text, such as an item count. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | String | Content property. The badge text |
| `Small` | Boolean | Smaller font |

## TagLabel

A tag-style marker, most often used to display the state of an entity. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | String | Content property. The label text |
| `Outline` | Boolean | Draw the outline only |
| `Style` | Enum | Colour of the label — see below |

`Style` values: `Default`, `Green`, `Success`, `Warning`, `Orange`, `Info`, `Cyan`, `Error`, `Danger`, `Red`, `Purple`, `Pink`, `Gold`, `Blue`, `Salmon`, `Seagreen`, `Tan`, `Magenta`, `LightGray`, `Olive`, `Teal`.

## SpanIcon

An icon inside text.

| Property | Type | Description |
|----------|------|-------------|
| `Icon` | Icon | The icon to display |
| `Size` | Length | Font size of the icon |
| `Gray` | Boolean | Draw the icon in grey. The exact colour depends on the UI theme |

## SpanSum

An inline element for a monetary amount. Displays the amount in green or red and adds an income/expense icon. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | Object | The amount |
| `Dir` | Int32 | Direction: greater than zero — income, less than zero — expense, zero — both |

## StaticImage

An image inside text.

| Property | Type | Description |
|----------|------|-------------|
| `Url` | String | Path to the image file, relative to the application folder. May be a [Bind](https://docs-llm.a2v10.com/xaml/bind.md) |
| `Height` | Length | Image height |

## Html

A fragment of text rendered as HTML markup. Content property: `Content`.

| Property | Type | Description |
|----------|------|-------------|
| `Content` | String | Content property. Text in which HTML tags are allowed |

Its normal use is showing HTML text that comes from elsewhere — from the database, for example. Writing HTML by hand in XAML is awkward, because every tag has to be escaped:

```xml
<Html>This is &lt;b&gt;bold&lt;/b&gt; text</Html>
```

Use XAML elements instead:

```xml
<Text>This is <Span Bold="True">bold</Span> text</Text>
```

Or, when the markup really is HTML, wrap it in CDATA:

```xml
<Html>
  <![CDATA[
  This is <b>bold</b> text
  ]]>
</Html>
```

## Break

A line break, normally placed inside a [Text](#text) element. The element has no properties of its own.

## Line

A plain horizontal line spanning the full width of its container. The colour of the line depends on the container it sits in. The element has no properties of its own.

## TextColor

Sets the colour of text. Used by `Inline`, `Text`, `Paragraph`, `Block`, and `CheckBoxBase`.

| Value | Colour |
|-------|--------|
| `Default` | Not set — the text inherits the colour of the parent element |
| `Gray` | `#999999` |
| `Label` | `#999999` — the label colour, same as `Gray` |
| `LightGray` | `#cccccc` |
| `Green` | `#3EA055` |
| `Red` | `#a94442` |
| `Danger` | `#a94442` — the error colour, same as `Red` |

## Example

A line of text mixing plain strings with inline elements:

```xml
<Text>Total items: <Badge>25</Badge></Text>
```

A document header — state as a tag, an amount, and a link that opens the counterparty:

```xml
<Text>
  <TagLabel Style="Success" Content="{Bind Document.State}" />
  <Span Space="Both" Content="{Bind Document.No}" />
  <SpanSum Content="{Bind Document.Sum}" Dir="1" />
  <Hyperlink Content="{Bind Document.Agent.Name}"
             Icon="Link"
             Command="{BindCmd Dialog, Action=Edit, Url='/catalog/agent',
                      Argument={Bind Document.Agent}}" />
</Text>
```

A paragraph with a hint that opens on hover:

```xml
<Paragraph Color="Gray">
  The rate is taken from the National Bank
  <Popover Text="on the document date" Icon="Help" ShowOnHover="True"
           Background="Cyan" Placement="TopRight">
    If no rate is set for that date, the closest earlier one is used.
  </Popover>
</Paragraph>
```

## Notes

- A plain text string inside an `Inlines` collection is emitted as is — no wrapping tag is added. Elements and strings can therefore be mixed freely inside `Text` and `Paragraph`.
- Never display HTML that a user entered or that came from an external source through `Html`. The platform strips clearly dangerous fragments such as scripts and frames, but the surrounding markup can still be broken irreparably.
- `Outline` on `TagLabel` only has an effect in some UI themes.
- `ShowOnHover` on `Popover` makes interactive content inside the popup useless: the window disappears as soon as the cursor leaves the text. It is also incompatible with `Url`, deliberately — otherwise the server would be called on every mouse move.
- `MaxChars` on `Span` truncates the display only. Show the user the full text some other way, for example through `Tip`.
- Several elements have a `Badge` property of their own — `Header` and `Tab` among them — which inserts a `Badge` element at the right place instead of it being written out.
- `Icon` values in XAML are PascalCase (`Help`, `HelpOutline`, `Search`), unlike the lower-case names used in [menu.json](https://docs-llm.a2v10.com/app/menu.md).
