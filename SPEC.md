# Agent Memory Repo: Format

Repository layout, entries, metadata, and cross-links. This mirrors the Format section of [cognition.com/agent-memory-repo](https://cognition.com/agent-memory-repo#format). See the [README](README.md) for the memory loop, use cases, and examples.

## Repository layout

Organize files and folders however you want. The repo can contain Markdown notes, SQL queries, scripts, and other files.

```text
memory-john/
  MEMORY.md
  team_structure.md
  using_datadog_mcp.md
  projects/
    payments.md
    website.md
  metrics/
    autocomplete_keep_rate.sql
```

## MEMORY.md

`MEMORY.md` is the entry point to the repo. Agents load it at the start of every session, so keep it short: include only what every session needs, plus links to everything else.

`MEMORY.md`:

```markdown
# Memory: John

- John leads the product team [source: https://example.com/sessions/100]

## Index
- [[team_structure]]
- [[projects/payments]]
- [[projects/website]]
```

## Entries and metadata

In Markdown notes, each entry is a bullet on one line, with optional metadata at the end. Update or remove entries when the information changes.

```markdown
- John coordinates the billing launch [source: https://example.com/sessions/101]
- Payments and website share a 2026-10-15 launch deadline [source: https://example.com/sessions/102; added: 2026-09-03]
```

Metadata uses `[key: value; key: value]`. Keys are open. Recommended keys:

- `source`: a link to the agent session where the information was learned.
- `added`: when it was saved, as `YYYY-MM-DD`.

A key ends at the first `:`, so a value can contain colons, as URLs do. A value cannot contain `;`, `[`, or `]`, which delimit the metadata. In a URL, write them as `%3B`, `%5B`, and `%5D`.

## Cross-links

Use `[[path]]` to link between files. Paths start at the memory root. Omit `.md` for Markdown files; keep other extensions, as in `[[metrics/autocomplete_keep_rate.sql]]`. Keep information in one place and link to it elsewhere. Update links when moving or renaming files.

```markdown
- Priya owns pricing for the billing launch; see [[team_structure]].
```
