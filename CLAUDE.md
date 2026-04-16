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
  overview.md         ← TODO (stub)
  markers.md          ← TODO (stub)
  procedures.md       ← TODO (stub)
  update-model.md     ← DONE (TVP + MERGE pattern, complete with examples)
  tree.md             ← DONE (Tree/Hierarchy, static + dynamic, complete)
xaml/
  overview.md         ← DONE (Page/Dialog roots, extensions, property syntax)
  bind.md             ← DONE (Bind + BindCmd, all properties and CommandTypes)
  base-classes.md     ← DONE
  controls/           ← DONE (button, checkbox, combobox, datagrid, datepicker,
                              selector, static, textbox)
  layouts/            ← DONE (dialog, fieldset, grid, page, repeater,
                              stackpanel, tabpanel, toolbar)
model/
  overview.md         ← DONE (top-level model.json structure + inheritance)
  actions.md          ← DONE (index/copy/view/template/merge/export)
  dialogs.md          ← DONE (modal dialogs)
  popups.md           ← DONE (popup windows)
  commands.md         ← DONE (sql/clr/callApi/javascript/process/sendMessage)
  reports.md          ← DONE (Stimulsoft/xml/json, macros in filename)
  files.md            ← DONE (parse/sql/azureBlob/clr, imageCompress)
```

**Still TODO**: `sql/overview.md`, `sql/markers.md`, `sql/procedures.md` — these are stub files
with `TODO` placeholders throughout. They need real content.

## Authoring Rules (summary — full rules in CONVENTIONS.md)

Every `.md` file follows this exact structure:

```
# Title (short noun phrase — must match llms.txt link label exactly)

> One-sentence blockquote description.

## Overview   ← plain English, no code, 2–5 paragraphs
## Syntax     ← fenced code block + tables for property lists
## Example    ← realistic, complete, uses a2v10sample schema
## Notes      ← bullet list of edge cases / gotchas
## Hints      ← optional: copy-paste patterns, debugging tips
```

Rules:
- No YAML frontmatter
- No bold labels inside paragraphs — use tables or lists
- All code blocks must have language tags (`sql`, `json`, `xml`, `ts`)
- English only
- Omit sections that have nothing to say — never leave `TODO` placeholders

Every new file needs a one-liner added to `llms.txt` under the appropriate `##` section.

## Sample Schema

All examples use the **a2v10sample** database. Common schemas and patterns:
- Schema `a2` or `cat` for application tables
- Model names follow PascalCase: `Agent`, `Document`, `AgentCategory`
- Stored procedure names: `[schema].[ModelName.Verb]` e.g. `[a2].[Agent.Index]`
- Standard verbs: `Index`, `Load`, `Metadata`, `Update`, `Delete`, `Copy`, `Expand`, `Report`
- Tables use `Id bigint` PK, `[Name] nvarchar(255)`, optional `Memo nvarchar(255)`

## Source Material for New Docs

When writing new documentation, draw from the original A2v10 HTML help:
- **Base URL**: `https://raw.githubusercontent.com/alex-kukhtin/A2v10.Help/master/A2v10.Help/html/`
- **app/** — model.json (already documented)
- **sql/** — SQL conventions and markers (needed for the TODO stubs)
- **xaml/** — XAML controls (already documented)

Fetch files with WebFetch using the raw GitHub URL + path.

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
