# Documentation Conventions

> How each documentation file in this project should be structured — a reference for consistent authoring.

## File Template

Every documentation file follows this structure:

```markdown
# Title (short noun phrase)

> One-sentence description — what this page covers and why it matters.

## Overview

Plain-English explanation of the concept. No code yet. 2–5 short paragraphs or a numbered list for sequential processes.

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
- Debugging tips if applicable.
```

## Rules

- **H1**: file title only — must match the link label in `llms.txt`
- **Blockquote** after H1: mandatory one-liner summary of the page
- **H2 order**: `Overview` → `Syntax` → `Example` → `Notes`
  - Omit sections that genuinely have nothing to say; do not leave `TODO` placeholders
- **Code blocks**: always specify language tag — `sql`, `json`, `xml`, `ts`, etc.
- **Tables**: use for structured mappings (field names, pattern parts, flags)
- **No frontmatter / YAML** — the platform does not use it
- **No bold labels** inside paragraphs — use tables or lists instead
- **Examples**: realistic, self-contained, based on `a2v10sample` schema
- **Language**: English only

## llms.txt Entry Format

Each file must have a corresponding one-line entry in `llms.txt`:

```
- [Title](path/to/file.md): Short description of what it covers
```

The description should complement the file's blockquote — same idea, different phrasing if possible.

## File Naming

- Use kebab-case: `update-model.md`, `tree.md`, `overview.md`
- Group by topic area in subdirectories: `sql/`, `ui/`, `model/`, etc.
- This file (`CONVENTIONS.md`) lives in the project root and is not listed in `llms.txt`
