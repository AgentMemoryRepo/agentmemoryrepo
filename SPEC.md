# Agent Memory Repo: File Structure Spec

This document specifies the file structure of an Agent Memory Repo. See the [README](README.md) for the motivation, the memory loop, and examples.

## Repository

A memory repo is a git repository. The root of the repository is the **memory root**.

Organize files and folders however you want. The repo can contain Markdown notes, SQL queries, scripts, and other files.

```text
memory-joe/              ← memory root
  MEMORY.md              ← entry point (required)
  team_structure.md
  using_datadog_mcp.md
  projects/
    payments.md
    website.md
  billing/
    count_paying_customers.sql
```

## MEMORY.md

`MEMORY.md` sits at the memory root and is the entry point to the repo. Agents load it at the start of every session.

- Keep it short: include only what every session needs, plus links to everything else.
- Entries that every session needs go at the top.
- Links to other files go under an `## Index` heading.

```markdown
# Memory: Joe

- Joe leads the product team [source: https://example.com/sessions/100]

## Index
- [[team_structure]]
- [[projects/payments]]
- [[projects/website]]
```

## Entries

In Markdown notes, each entry is a bullet on one line, with optional metadata at the end.

```markdown
- Joe coordinates the billing launch [source: https://example.com/sessions/101]
- Payments and website share a 2026-10-15 launch deadline [source: https://example.com/sessions/102; added: 2026-09-03]
```

Update or remove entries when the information changes.

## Metadata

Metadata is a bracketed list of `key: value` pairs, separated by `;`, at the end of an entry:

```text
[key: value; key: value]
```

Keys are open. Recommended keys:

| Key      | Value                                                        |
| -------- | ------------------------------------------------------------ |
| `source` | A link to the agent session where the information was learned. |
| `added`  | When it was saved, as `YYYY-MM-DD`.                          |

## Cross-links

Use `[[path]]` to link between files.

- Paths start at the memory root.
- Omit `.md` for Markdown files: `[[projects/payments]]`.
- Keep other extensions: `[[billing/count_paying_customers.sql]]`.
- Keep information in one place and link to it elsewhere.
- Update links when moving or renaming files.

```markdown
- Priya owns pricing for the billing launch; see [[team_structure]].
```

## Multiple memory repos

A session can load several memory repos at once. Each repo is cloned into its own folder and keeps its own `MEMORY.md`, ownership, permissions, and history.

```text
vm/
├── memory-alice/
│   ├── MEMORY.md
│   └── …
└── memory-bob/
    ├── MEMORY.md
    └── …
```

A `[[path]]` link starts at the root of the repo that contains it.
