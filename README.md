# Agent Memory Repo

Agents need memory that lasts across sessions, and a single `MEMORY.md` file isn't enough. Agent Memory Repo is an open spec that treats agent memory as a git repo.

Git gives memory a history, a way to merge, and permissions. Agents already know how to use it.

The spec is also published at [cognition.ai/agent-memory-repo](https://cognition.ai/agent-memory-repo).

## The memory loop

Every session follows the same steps.

1. **Clone** the latest memory.
2. **Grep** for what the task needs, or follow links.
3. **Update** entries as the agent learns, with no human in the loop.
4. **Commit** after every edit.

## Dreaming

Dreaming is a dedicated agent that runs periodically. It has two jobs.

- **Add new memory.** Spot patterns across sessions and save them as new entries.
- **Clean up memory.** Merge duplicates, remove outdated entries, and check sources to resolve contradictions.

## Use cases

**Personal memory.** Remember a user's context across sessions, like who they work with or how their projects connect. Each session reads and updates the same memory repo.

**Agent swarms.** Share findings across agents working in parallel. Git merges their changes and surfaces conflicts.

**Team memory.** Share knowledge about customers, processes, and tools, especially for teams without a code repo. A lesson from one member's support investigation becomes available to everyone's agents.

**Multiplayer.** Memories compose because they are folders. When a new user joins a session, their memory repo can be cloned into the same machine.

<details>
<summary><strong>Format</strong>: repository layout, entries, metadata, and cross-links</summary>

The full file structure spec is in [SPEC.md](SPEC.md).

### Repository layout

Organize files and folders however you want. The repo can contain Markdown notes, SQL queries, scripts, and other files.

```text
memory-joe/
  MEMORY.md
  team_structure.md
  using_datadog_mcp.md
  projects/
    payments.md
    website.md
  billing/
    count_paying_customers.sql
```

### MEMORY.md

`MEMORY.md` is the entry point to the repo. Agents load it at the start of every session, so keep it short: include only what every session needs, plus links to everything else.

`MEMORY.md`:

```markdown
# Memory: Joe

- Joe leads the product team [source: https://example.com/sessions/100]

## Index
- [[team_structure]]
- [[projects/payments]]
- [[projects/website]]
```

### Entries and metadata

In Markdown notes, each entry is a bullet on one line, with optional metadata at the end. Update or remove entries when the information changes.

```markdown
- Joe coordinates the billing launch [source: https://example.com/sessions/101]
- Payments and website share a 2026-10-15 launch deadline [source: https://example.com/sessions/102; added: 2026-09-03]
```

Metadata uses `[key: value; key: value]`. Keys are open. Recommended keys:

- `source`: a link to the agent session where the information was learned.
- `added`: when it was saved, as `YYYY-MM-DD`.

### Cross-links

Use `[[path]]` to link between files. Paths start at the memory root. Omit `.md` for Markdown files; keep other extensions, as in `[[billing/count_paying_customers.sql]]`. Keep information in one place and link to it elsewhere. Update links when moving or renaming files.

```markdown
- Priya owns pricing for the billing launch; see [[team_structure]].
```

</details>

## Composability

A session can load several memory repos at once. For example, Alice starts a session with her memory. When Bob joins, the agent clones his memory into the same machine.

```text
1. Alice starts a session

vm/
└── memory-alice/        ← cloned at session start
    ├── MEMORY.md
    └── …

2. Bob joins the session

vm/
├── memory-alice/
│   ├── MEMORY.md
│   └── …
└── memory-bob/          ← cloned when Bob joins
    ├── MEMORY.md
    └── …
```

Once Bob's repo is in the machine, the agent reads both `MEMORY.md` files and follows links as needed. Bob's memory is cloned only if he chooses to share it with the session.

The repos stay separate. The agent tracks who said what and writes each memory to the right repo: Alice's preferences go to `memory-alice/`, and Bob's go to `memory-bob/`. When the destination is unclear, the agent asks.

Because each repo keeps its own ownership, permissions, and history, future sessions can combine them in any mix.

## Worked example: learning a SQL query from code

Suppose Joe asks: "How many paying customers do we have?"

The agent reads the billing code and discovers that a paying customer is an organization with an active, paid subscription. Test organizations are excluded, and an organization can have multiple subscriptions, so the query must count distinct organizations.

After checking the query, the agent saves it:

`billing/count_paying_customers.sql`:

```sql
SELECT COUNT(DISTINCT s.organization_id)
FROM subscriptions s
JOIN organizations o ON o.id = s.organization_id
WHERE s.status = 'active'
  AND s.plan = 'paid'
  AND o.is_test = false;
```

An entry in `MEMORY.md` says what the query does and links to it:

`MEMORY.md`:

```markdown
- [[billing/count_paying_customers.sql]] counts organizations with active, paid subscriptions. Excludes test organizations and counts each organization once [source: https://example.com/sessions/105]
```

A later session follows the link and runs the saved SQL for the current count, without rediscovering the joins and filters.

## Try with your agent

Install the `agent-memory-repo` skill. It teaches an agent to keep memory in a separate local git repo that follows this spec.

```sh
npx skills add AgentMemoryRepo/agentmemoryrepo --skill agent-memory-repo
```

The installer asks which agents to install it for. In Devin, install it as a plugin:

```sh
devin plugins install AgentMemoryRepo/agentmemoryrepo
```

Then try it in two sessions. The trial is local, so run both sessions on the same machine and storage, in the same project.

Session 1:

```text
Use the agent-memory-repo skill to set up a separate local memory repo for this trial. Save this trial preference: I prefer concise bullet-point summaries. Do not configure a remote. Tell me the full path to the memory repo so I can reuse it next session.
```

Session 2, in a new session:

```text
Use the agent-memory-repo skill with the memory repo at <paste the full path from the previous session>. What trial preference did I save?
```

Memory stays local unless you connect a private repo you own. On a new cloud machine or another computer, the local memory repo isn't there. Ask the agent to clone your private memory repo first.

## Open development

Agent Memory Repo was originally developed by Cognition and released as an open standard. It is open to contributions from the broader ecosystem.

Propose changes and follow the discussion in the [GitHub repository](https://github.com/AgentMemoryRepo/agentmemoryrepo).

## License

[MIT](LICENSE)
