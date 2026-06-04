# Documentation Conventions

> How each documentation file in this project should be structured — a reference for consistent authoring.

## File Template

Every documentation file follows this structure:

```markdown
# Title (short noun phrase)

> One-sentence description — what this page covers and why it matters.

## Overview

Plain-English explanation of the concept. No code yet. 2–5 short paragraphs or a numbered list for sequential processes.

## Use When

Optional. A short bullet list of the situations where this feature is the right choice.
Selection criteria, not description — what problem it solves and when to reach for it.

## Do Not Use When

Optional. A short bullet list of cases where this feature is the wrong tool.
Every bullet must name the alternative explicitly — phrase it as "use X instead"
and link to X when a doc for it exists. The counterpart to Use When; a dead end
("don't use this here") is not enough — always point to the way out.

## Syntax

Formal pattern or signature. Use a fenced code block for any formal syntax.
Use a table for multi-part patterns (e.g., column alias components).

## Example

One complete, realistic, runnable SQL example.
Use fenced ```sql blocks.
A brief intro sentence before each block.
Use H3 subheadings (###) to separate multiple related snippets.

## Notes

- Bullet list of edge cases, gotchas, and non-obvious behavior.

## Hints

- Optional section.
- Practical tips: debugging snippets, shortcuts, copy-paste patterns.
- Not for edge cases or gotchas — those belong in Notes.
```

## Rules

- **H1**: file title only — must match the link label in `llms.txt`
- **Blockquote** after H1: mandatory one-liner summary of the page
- **H2 order**: `Overview` → `Use When` → `Do Not Use When` → `Syntax` → `Example` → `Notes` → `Hints`
  - Omit sections that genuinely have nothing to say; do not leave `TODO` placeholders
  - `Use When` / `Do Not Use When` are optional decision-criteria blocks — include them when a
    reader could plausibly reach for the wrong feature; they tell the model *when to choose this*,
    not what it is. Place them right after `Overview`, before `Syntax`.
- **Code blocks**: always specify language tag — `sql`, `json`, `xml`, `ts`, etc.
- **Tables**: use for structured mappings (field names, pattern parts, flags)
- **No frontmatter / YAML** — the platform does not use it
- **No bold labels** inside paragraphs — use tables or lists instead
- **Examples**: realistic, self-contained, based on `a2v10sample` schema
- **Language**: English only
- **Links**: every cross-reference to another doc uses a full absolute URL — `https://docs-llm.a2v10.com/...` — never a relative (`../`, `sql/x.md`) or root-relative (`/sql/x.md`) path (see Links below)

## Links

Every link to another documentation page — both the entries in the index files
(`llms.txt`, `sql.md`, `xaml.md`, `model.md`) and inline cross-references inside a doc — must be
a full absolute URL rooted at the live site:

```
https://docs-llm.a2v10.com/sql/array.md
https://docs-llm.a2v10.com/xaml/base-classes.md
```

Never use a relative path (`../xaml/bind.md`, `sql/array.md`) or a root-relative path
(`/sql/array.md`).

### Why we decided this

The consumer of these docs is an LLM, not a browser. When a model fetches a page, the tool
usually hands it back the **markdown text alone**, detached from the URL it came from. To follow a
link, the model then has to reconstruct an absolute URL itself — and that is exactly where it goes
wrong:

- A relative path (`../`, or a bare sibling like `paging.md`) forces the model to remember the
  current file's exact location and correctly unwind `../` segments. It frequently doesn't, and
  resolves the link against the site root instead — a silent 404.
- A root-relative path (`/sql/array.md`) is safer but still assumes the model reliably knows the
  host.
- A full absolute URL requires the model to know nothing. There is no resolution step, so there is
  nothing to get wrong.

We optimise for the consumer that has the least context: we remove the possibility of error
entirely rather than rely on the model resolving paths correctly. This is also what the llms.txt
convention recommends — give consumers unambiguous links.

The trade-off (verbosity, and links that must be updated if the domain changes) is deliberate and
accepted.

## llms.txt Entry Format

Each file must have a corresponding one-line entry in its section index
(`sql.md` / `xaml.md` / `model.md`):

```
- [Title](https://docs-llm.a2v10.com/path/to/file.md): Short description of what it covers
```

The description should complement the file's blockquote — same idea, different phrasing if possible.

## File Naming

- Use kebab-case: `update-model.md`, `tree.md`, `overview.md`
- Group by topic area in subdirectories: `sql/`, `ui/`, `model/`, etc.
- This file (`CONVENTIONS.md`) lives in the project root and is not listed in `llms.txt`
