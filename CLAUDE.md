# CLAUDE.md — Project Context

## What this project is

LLM-friendly reference documentation for the **A2v10** platform — a full-stack framework for
building business applications on SQL Server, ASP.NET Core, and Vue.js.

- **Live site**: https://docs-llm.a2v10.com
- **LLM entry point**: https://docs-llm.a2v10.com/llms.txt
- **Source repo**: https://github.com/alex-kukhtin/A2v10.Help (original HTML docs to draw from)

The site serves `llms.txt` as the directory index (`.htaccess`: `DirectoryIndex llms.txt`).
Everything in `llms.txt` is what an LLM sees when it fetches the site root.

## Deployment

Push to `main` → GitHub Actions → FTP upload to the web host. No build step.
All files are served as static content.

## Content Structure

```
llms.txt              ← LLM entry point, one-liner index of all docs
CONVENTIONS.md        ← Authoring rules (read this before writing any doc)
sql/
  procedures.md       ← DONE (name composition, suffix table, standard parameters)
  markers.md          ← DONE (name composition, dataset types + field modifiers hub)
  object.md           ← DONE (Object set, !Id/!Name, !RefId + Map, named Map, new instance)
  update-model.md     ← DONE (TVP + MERGE pattern, complete with examples)
  array.md            ← DONE (Array/LazyArray, !ParentId)
  paging.md           ← DONE (@Offset/@PageSize/@Order/@Dir, !RowCount, filter forms)
  tree.md             ← DONE (Tree/Hierarchy, static + dynamic, complete)
  grouping.md         ← DONE (Group type, GroupMarker)
  cross.md            ← DONE (CrossArray/CrossObject, $cross)
  map-object.md       ← DONE (MapObject, !Key, key list, dynamic keys)
  system-datasets.md  ← DONE ($System/$Aliases/$Defaults, modifiers, set order)
  blob.md             ← DONE (binary objects, !Token, .Load/.Update byte procedures)
  rowversion.md       ← DONE (rv column, varbinary(8) in TVP, version check in .Update)
  errors.md           ← DONE (throw, UI: prefix — user alert vs developer alert)
xaml/
  overview.md         ← DONE (Page/Dialog roots, extensions, property syntax)
  bind.md             ← DONE (Bind + BindCmd, all properties and CommandTypes)
  base-classes.md     ← DONE (UIElementBase/UIElement/Inline/Container/Control/…)
  text.md             ← DONE (all 13 inlines in one file + TextColor values)
  switch.md           ← DONE (Switch/Case/Else — one block of several by a bound value)
  controls/           ← DONE (button, checkbox, combobox, datagrid, datepicker,
                              fileimage, graphics, image, list, menuitem, pager,
                              pdfreportviewer, periodpicker, radio, selector,
                              selectorsimple, static, table, textbox, toolbaraligner,
                              uploadfile)
                         ListItem lives in list.md; TableRow/TableCell/TableColumn
                         in table.md
  layouts/            ← DONE (block, commandbar, dialog, dropdownmenu, fieldset, grid,
                              group, page, panel, partial, popup, repeater, sheet,
                              stackpanel, tabbar, tabpanel, taskpad, toolbar)
                         GridGroup lives in grid.md; TabButton in tabbar.md;
                         Partial + PartialBlock + Include in partial.md
app/                  ← application-wide config (source html/app/)
  menu.md             ← DONE (menu.json navigation tree + icon set)
  layout.md           ← DONE (_layout folder, _scripts.html/_styles.html, Core only)
model/
  overview.md         ← DONE (top-level model.json structure + inheritance)
  actions.md          ← DONE (index/copy/view/template/merge/export)
  dialogs.md          ← DONE (modal dialogs)
  popups.md           ← DONE (popup windows)
  commands.md         ← DONE (sql/clr/callApi/javascript/process/sendMessage)
  reports.md          ← DONE (Stimulsoft/xml/json, macros in filename)
  files.md            ← DONE (parse/sql/azureBlob/clr, imageCompress)
template/             ← DONE (client-side JS model behavior; source html/template/)
  overview.md         ← DONE (exported template object + sections)
  options.md          ← DONE (noDirty/bindOnce/persistSelect/skipDirty)
  defaults.md         ← DONE (initial values for new models)
  properties.md       ← DONE (scalar + computed properties)
  validators.md       ← DONE (data-bound validation, std/func/obj/async)
  events.md           ← DONE (model/object/array lifecycle handlers)
  commands.md         ← DONE (view commands via BindCmd, confirm/guards)
  delegates.md        ← DONE (control callbacks — selector fetch, Graphics draw)
client/               ← DONE (client object model / runtime API; source html/client/)
  overview.md         ← DONE (the five object shapes + index)
  element.md          ← DONE (IElement base members)
  root.md             ← DONE (IRoot — model-wide state, validation, dirty)
  array.md            ← DONE (IElementArray — selection, lazy, insert/find/sum)
  array-element.md    ← DONE (IArrayElement — select/check/remove/move)
  tree-element.md     ← DONE (ITreeElement — expand, selectPath)
  controller.md       ← DONE (IController via $ctrl — invoke/reload/msg/upload)
  utils.md            ← DONE (std:utils — date/currency/text)
  require.md          ← DONE (require + std: modules)
  (source has empty stubs: index/module/global/localization — skipped)
report/               ← DONE (Xaml printed forms, Core only; source html/report/)
  overview.md         ← DONE (two template forms, wiring via reports, data, Code section)
  bind.md             ← DONE (scope rule, Bind vs braces, DataType/Format, quoting, budget)
  elements.md         ← DONE (base element, Length/Thickness, Page/Column/Inlined/
                              Text+Span/Space/Break, List, Line, Checkbox)
  table.md            ← DONE (Table/TableRow/TableCell/TableColumn, TableStyle, nesting)
  images.md           ← DONE (Image/QrCode/Barcode, watermark, bytes vs file name, SVG)
  workbook.md         ← DONE (Workbook/Cell/Range/Row/Column/PageFooter, Excel conversion)
  view.md             ← DONE (PdfReportViewer on a form + BindCmd Report Print/Export)
```

The 28 source pages of `html/report/` are folded into these seven: the element enums
(`report/enums.html`) are placed where they are used — `DataType` in `report/bind.md`,
`TableStyle` in `report/table.md`, `BarcodeType` in `report/images.md` — and `Length` /
`Thickness` live in `report/elements.md`.

No stub files remain. `sql/overview.md` was deleted rather than written: its promised scope
(schemas, standard columns, DDL idempotence) has no source in the help, and the one part that
does — name composition and dataset types, from `models/general.html` — is already
`sql/markers.md`.

## Coverage Policy

The help is not a checklist. These docs cover what gets written into real endpoints; a help page
that describes a superseded element is not a gap, and mirroring all of `html/xaml/` 1:1 is not
the goal. `DONE` in the content map above means that file is written — never that the matching
help folder is exhausted.

Part of the help is older than the platform, and which pages those are is not derivable from the
HTML. Any element added from the lists below is first confirmed as current with the project
owner. Pages that are empty stubs in the help (a few hundred bytes, no properties) are out of
scope until the help itself has content.

Measured state as of 2026-09-12 — help pages versus ours:

| Help folder | Pages | Ours | Where ours live |
|-------------|-------|------|-----------------|
| `xaml/controls/` | 50 | 15 | `xaml/controls/` |
| `xaml/containers/` | 33 | 9 | `xaml/layouts/` |
| `xaml/abstract/` | 11 | 1 | `xaml/base-classes.md` (folded) |
| `xaml/text/` | 14 | 1 | `xaml/text.md` (folded) |
| `xaml/sheet/` | 12 | 1 | `xaml/layouts/sheet.md` (folded) |
| `xaml/bind/` | 8 | 1 | `xaml/bind.md` (folded) |
| `report/` | 28 | 7 | `report/` (folded) |
| `xaml/enums/` | 23 | 0 | — |
| `xaml/components/` | 11 | 0 | — |
| `xaml/simple/` | 13 | 0 | — |

Folding several help pages into one doc is the norm here, not an exception.

## Roadmap — Next Tasks (prioritized)

Ordered by leverage. Cross-references must be full absolute URLs (see CONVENTIONS.md → Links).

1. ~~**Containers**~~ — done on 2026-09-12, triaged by the project owner: `Panel`, `Group`,
   `Block`, `GridGroup` (in `grid.md`), `Taskpad`, `CommandBar`, `DropDownMenu` +
   `DropDownMegaMenu`, `TabBar` (+ `TabButton`), `Popup`, `Partial` + `PartialBlock` + `Include`,
   and `Switch` + `Case` + `Else` from `html/xaml/components/` — the `containers/case.html` copy
   is a stale duplicate, ignore it.
   - Deferred, not now: `Card` + `CardBody`, `FullHeightPanel`, `Flex` + `FlexList`,
     `InlineDialog`, `IFrame`.
   - Obsolete, do not document: `Splitter`, `Wizard` + `WizardPage`.
2. **Controls** — triaged by the project owner on 2026-09-12.
   - Done: `List` + `ListItem`, `Pager`, `PeriodPicker`, `Radio`, `MenuItem`, the `Table` family
     (`Table`, `TableRow`, `TableCell`, `TableColumn`), `TabButton` (in `tabbar.md`).
   - Deferred, not now: `TreeView` + `TreeViewItem`, `TreeGrid` + `TreeGridColumn`, `TimePicker`,
     `PropertyGrid` + `PropertyGridItem`, `Alert`, `MultiSelect`, `EmptyPanel`, `Separator`,
     `GridDivider`, `Header`, `Label`, `Code`, `HtmlElement` + `HtmlAttribute`.
   - Stubs in the help, nothing to write yet: `StateCard`, `ChatItem`, `TableMarkCell` — all
     three also sit in the TODO list of `whatsnew/w202609.html`.
   - `ComboBoxItem`, `DataGridColumn` and `DataGridRowDetails` were already covered inside
     `combobox.md` and `datagrid.md`.
3. **Horizontal links** — nine pairs verified missing:
   - `xaml/controls/datagrid.md` → `xaml/base-classes.md` (the other three links are in place)
   - `xaml/controls/selector.md` → `model/commands.md` (the server-side `fetch` command),
     `sql/array.md`
   - `xaml/controls/combobox.md` → `sql/array.md` (a bound list is an array)
   - `sql/tree.md` → `xaml/layouts/sheet.md`, `xaml/controls/datagrid.md`
     (sheet links to tree, but not back — one-directional)
   - `model/reports.md` → `xaml/layouts/sheet.md` (Excel export)
   - `sql/array.md` → `xaml/controls/datagrid.md` (what renders it)
   - `sql/paging.md` → `xaml/controls/datagrid.md` (sorting/paging UI)
4. **XAML enums** (not started): `html/xaml/enums/` has 23 pages — `Icon`, `TextColor`,
   `ColumnControlType`, `ControlSize`, `WrapMode`, … Today enum values are inlined in the control
   that uses them, which is the right default; a shared page is worth it only for a set several
   controls repeat. The `Icon` list in `app/menu.md` is the *menu.json* set, a different naming —
   do not merge the two.
5. **Anchor-text hygiene pass** (low priority): replace any bare `See [X]` / generic anchors with
   descriptive noun phrases. Not a RAG optimization — just readability for the direct-fetch model.

Closed: SQL stubs (`sql/procedures.md` written, `sql/overview.md` deleted — note that `.Fetch`
and `.Delete` are **not** platform procedure suffixes; the help knows only `.Index`, `.Load`,
`.Metadata`, `.Update`, `.Expand`). All `models/` source pages (`blob` → `sql/blob.md`,
`rowversion` → `sql/rowversion.md`). The `report/` section.

## Authoring Rules (summary — full rules in CONVENTIONS.md)

Every `.md` file follows this exact structure:

```
# Title (short noun phrase — must match llms.txt link label exactly)

> One-sentence blockquote description.

## Overview          ← plain English, no code, 2–5 paragraphs
## Use When          ← optional: when this is the right choice (selection criteria)
## Do Not Use When   ← optional: when it's the wrong tool + the alternative
## Syntax            ← fenced code block + tables for property lists
## Example           ← realistic, complete, uses a2v10sample schema
## Notes             ← bullet list of edge cases / gotchas
## Hints             ← optional: copy-paste patterns, debugging tips
```

Rules:
- No YAML frontmatter
- No bold labels inside paragraphs — use tables or lists
- All code blocks must have language tags (`sql`, `json`, `xml`, `ts`)
- English only
- Omit sections that have nothing to say — never leave `TODO` placeholders

The root `llms.txt` is a flat, one-level index with seven entries — SQL, XAML, MODEL, REPORT,
TEMPLATE, CLIENT, APP — each pointing to a section index page (`sql.md`, `xaml.md`, `model.md`,
`report.md`, `template.md`, `client.md`, `app.md`). Those section pages hold the per-file
one-liners. Every new file needs its one-liner added to the matching section page, not to
`llms.txt`.

## Sample Schema

All examples use the **a2v10sample** database. Common schemas and patterns:
- Schema `a2` or `cat` for application tables
- Model names follow PascalCase: `Agent`, `Document`, `AgentCategory`
- Stored procedure names: `[schema].[ModelName.Verb]` e.g. `[a2].[Agent.Index]`
- Platform suffixes (the only ones the help confirms): `Index`, `Load`, `Metadata`, `Update`,
  `Expand`. Anything else (`Delete`, `Copy`, `Report`, …) is an ordinary procedure invoked by a
  command or an action from `model.json`, not a suffix the platform resolves by convention
- Tables use `Id bigint` PK, `[Name] nvarchar(255)`, optional `Memo nvarchar(255)`

## Source Material for New Docs

When writing new documentation, draw from the original A2v10 HTML help:
- **Base URL**: `https://raw.githubusercontent.com/alex-kukhtin/A2v10.Help/master/A2v10.Help/html/`
- **app/** — model.json (already documented)
- **sql/** — SQL conventions and markers (needed for the TODO stubs)
- **xaml/** — XAML controls (already documented)
- **report/** — Xaml printed forms (already documented)
- **whatsnew/** — release notes; the fastest way to find what the help gained since last time

Fetch files with WebFetch using the raw GitHub URL + path.

A local checkout of the same HTML (usually more current than GitHub) is at
`C:\A2v10_Net48\A2v10.Uk.Help\A2v10.Help\html\` — read it directly instead of fetching.
The source is Ukrainian; docs here are written in English from scratch, not translated.

**The help HTML is the only source of content.** The platform implementation
(`C:\A2v10_Net6`, `C:\A2v10_Net48`) may be consulted solely to settle a contradiction — never
to enrich a page. Nothing found in the code goes into a `.md` file: no properties, no enum
values, no type names, no notes. A finding is logged in `DISCREPANCIES.md` and decided by the
project owner. Missing from the help means missing from the docs; say so instead of digging.

## Key Platform Concepts

- **Endpoint** = a subdirectory with a `model.json` file; maps to a URL
- **Action** = loads data (calls `.Index` or `.Load` SP) and renders a XAML view
- **Dialog/Popup** = modal/inline overlay, same data-loading pattern as actions
- **Command** = server-side operation (sql/clr/callApi/…) with no view rendering
- **SQL markers** = column aliases in `SELECT` that tell the platform how to shape the model
  e.g. `[Root!TAgent!Array]`, `[Id!!Id]`, `[Name!!Name]`, `[!TAgent.Items!ParentId]`
- **TVP + MERGE** = save pattern: `.Metadata` describes shape → `.Update` receives TVP → MERGEs → calls `.Load`
- **XAML** = XML views compiled server-side to Vue.js; root is `<Page>` or `<Dialog>`
- **Bind / BindCmd** = markup extensions for data and command binding

## What Not to Do

- Do not add YAML frontmatter — the platform doesn't use it
- Do not leave `TODO` placeholders — omit the section instead
- Do not invent property names or procedure conventions — verify against source HTML
- Do not create new files without adding the entry to `llms.txt`
