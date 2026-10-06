# Agent Memory Repo

Agents work better when they remember things across sessions. With Agent Memory Repo, your agent saves what it learns from you in real time, and that memory is live across sessions. Memories link to each other, so your agent navigates them like a wiki.

The spec is also published at [cognition.com/agent-memory-repo](https://cognition.com/agent-memory-repo).

## Try with your agent

### Give your next session a memory.

Save a trial preference, then ask for it in a new session. Your memory lives in a separate repo, never in the public spec repository.

For this local trial, use the same machine with persistent storage. Cloud sessions or another machine need a private remote or other persistent storage to carry memory across sessions.

### 1. Install the skill

**Devin CLI.** Run this in your terminal, then start a new Devin session. ([Installation guide](https://docs.devin.ai/cli/extensibility/plugins/overview))

```sh
devin plugins install AgentMemoryRepo/agentmemoryrepo
```

**Claude Code, Cursor & others.** Run this from your project and choose your agent in the installer. Requires Node.js and Git. Restart your agent if needed. ([Installation guide](https://github.com/vercel-labs/skills#install-a-skill))

```sh
npx skills add AgentMemoryRepo/agentmemoryrepo --skill agent-memory-repo
```

### 2. Save a memory

Send this to your agent. It will create the repo, save an entry, and commit it.

First session:

```text
Use the agent-memory-repo skill to set up a separate local memory repo for this trial. Save this trial preference: I prefer concise bullet-point summaries. Do not configure a remote. Tell me the full path to the memory repo so I can reuse it next session.
```

### 3. Start a new session

Use the same project and machine, and replace the placeholder with the path your agent gave you. The new session should retrieve the saved preference from the repo.

Next session:

```text
Use the agent-memory-repo skill with the memory repo at <paste the full path from the previous session>. What trial preference did I save?
```

The memory repo stays local; no remote is configured by this trial. Invoke the skill when you want to use memory. Automatic startup and scheduled Dreaming are not included.

## The memory loop

Every session follows the same steps.

1. **Clone** the latest memory.
2. **Grep** for what the task needs, or follow links.
3. **Update** entries as the agent learns, with no human in the loop.
4. **Push** after every edit.

## Dreaming

Dreaming is a dedicated agent that runs periodically. It has two jobs.

- **Add new memory.** Spot patterns across sessions and save them as new entries.
- **Clean up memory.** Merge duplicates, remove outdated entries, and check sources to resolve contradictions.

## Use cases

**Personal memory.** Remember a user's context across sessions, like who they work with or how their projects connect. Each session reads and updates the same memory repo.

**Agent swarms.** Share findings across agents working in parallel. Each agent pushes to the same memory repo, and git flags any edits that conflict.

**Team memory.** Share knowledge about customers, processes, and tools, especially for teams without a code repo. A lesson from one member's support investigation becomes available to everyone's agents.

**Multiplayer.** Memories compose because they are folders. When a new user joins a session, their memory repo can be cloned into the same machine.

## Format

Repository layout, entries, metadata, and cross-links. The format is also in [SPEC.md](SPEC.md).

### Repository layout

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

### MEMORY.md

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

### Entries and metadata

In Markdown notes, each entry is a bullet on one line, with optional metadata at the end. Update or remove entries when the information changes.

```markdown
- John coordinates the billing launch [source: https://example.com/sessions/101]
- Payments and website share a 2026-10-15 launch deadline [source: https://example.com/sessions/102; added: 2026-09-03]
```

Metadata uses `[key: value; key: value]`. Keys are open. Recommended keys:

- `source`: a link to the agent session where the information was learned.
- `added`: when it was saved, as `YYYY-MM-DD`.

### Cross-links

Use `[[path]]` to link between files. Paths start at the memory root. Omit `.md` for Markdown files; keep other extensions, as in `[[metrics/autocomplete_keep_rate.sql]]`. Keep information in one place and link to it elsewhere. Update links when moving or renaming files.

```markdown
- Priya owns pricing for the billing launch; see [[team_structure]].
```

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

<details>
<summary><strong>Example: reusing a saved SQL query</strong></summary>

Suppose John asks: "Did last week's release hurt autocomplete?"

This is a new session. The agent has no chat history with John. It reads `MEMORY.md` first and finds this entry:

`MEMORY.md`:

```markdown
- Autocomplete is called ghost_text in code and events. Searching events for "autocomplete" returns nothing. [[metrics/autocomplete_keep_rate.sql]] gives the share of shown suggestions the user accepted and kept for 30 seconds, which is how the team measures the feature [source: https://example.com/sessions/105]
```

It follows the link to the saved query:

`metrics/autocomplete_keep_rate.sql`:

```sql
SELECT date(created_at) AS day,
  COUNT(*) FILTER (WHERE name = 'ghost_text_kept') * 1.0
  / COUNT(*) FILTER (WHERE name = 'ghost_text_shown')
    AS keep_rate
FROM events
WHERE name IN ('ghost_text_shown', 'ghost_text_kept')
  AND created_at > now() - interval '30 days'
GROUP BY day
ORDER BY day;
```

The agent runs it and answers John: the keep rate fell from 31% to 24% on the day of the release. John did not explain what the feature is called in the code or how the team measures it.

A week earlier, this was harder. John asked how autocomplete was doing, and the agent reported that nobody used it. John had to point it to `ghost_text` in the editor code. Its next answer counted every suggestion shown, and John had to explain that the team counts only the ones a user keeps. The agent saved what it learned.

This time, John asked once.

</details>

<details>
<summary><strong>Example: a swarm solves a slow checkout</strong></summary>

Checkout is fast for most users: the median request takes 80 ms. But 1 request in 100 takes 2.4 seconds, and nobody knows why. John asks an agent to find out. The agent starts four agents, one each for the database, the runtime, the cache, and the load balancer. First it makes a folder for the swarm in John's memory repo:

```text
memory-john/
  MEMORY.md
  team_structure.md
  projects/
    payments.md
    website.md
  swarms/
    slow-checkout/         ← one folder per swarm
      README.md            ← the goal and the rules
      findings.md          ← what each agent has measured
      questions.md         ← agents ask each other
      agents/
        database.md        ← one notes file per agent
        runtime.md
        cache.md
        load_balancer.md
      scripts/
        bench.sh           ← the one benchmark every agent runs
```

The `README.md` tells every agent how to use the folder:

`swarms/slow-checkout/README.md`:

```markdown
# Swarm: why is checkout slow for 1 request in 100?

- Median is 80 ms. The slowest 1% take 2.4 s. The goal is under 500 ms [source: https://example.com/sessions/300]
- Measure only with [[swarms/slow-checkout/scripts/bench.sh]], so numbers compare
- Before each step, pull and read findings and questions
- Write what you measure in findings, including what you rule out
- If another agent's area might explain what you see, ask in questions
```

Each agent clones the repo and digs into its own area. After an hour, `findings.md` holds three facts. None of them explains the problem:

`swarms/slow-checkout/findings.md`:

```markdown
- Database: queries are fast. But in bursts, requests wait up to 2 s for a free connection [source: https://example.com/sessions/301]
- Runtime: garbage collection freezes the server for about 400 ms, once every 10 seconds [source: https://example.com/sessions/302]
- Load balancer: ruled out. The slow requests stay slow with it bypassed [source: https://example.com/sessions/303]
```

The runtime agent cannot see why memory fills up on a 10-second beat. It asks the swarm. So does the database agent:

`swarms/slow-checkout/questions.md`:

```markdown
- Runtime asks: does anything run every 10 seconds and use a lot of memory? [source: https://example.com/sessions/302]
  - Cache answers: yes. The price cache refresh runs every 10 seconds and rebuilds the whole price table, about 2 GB [source: https://example.com/sessions/304]
- Database asks: when exactly are the freezes? My bursts are also 10 seconds apart [source: https://example.com/sessions/301]
  - Runtime answers: at :03, :13, :23 of each minute. Your bursts start within 50 ms of each one [source: https://example.com/sessions/302]
```

Now the three facts connect, and the database agent writes the cause:

`swarms/slow-checkout/findings.md`:

```markdown
- Cause: the price cache refresh rebuilds a 2 GB table every 10 seconds. That triggers a 400 ms freeze. Requests pile up during the freeze, then all ask for a database connection at once, and the pool of 20 runs out [source: https://example.com/sessions/301] [source: https://example.com/sessions/302] [source: https://example.com/sessions/304]
- Fix: refresh only the prices that changed. The slowest 1% drop from 2.4 s to 310 ms on bench.sh [source: https://example.com/sessions/304]
```

No agent could have found this alone. The cache agent saw nothing wrong with its refresh. The database agent saw waits with no cause. The runtime agent saw freezes with no source.

All four agents write to the same two files the whole time. Git merges most of their commits. When two agents edit the same line, git rejects the second push, and that agent reads both versions before writing one.

When the swarm finishes, the first agent adds one line to the index in `MEMORY.md`:

`MEMORY.md`:

```markdown
## Index
- [[team_structure]]
- [[projects/payments]]
- [[projects/website]]
- [[swarms/slow-checkout/README]]
```

The folder stays in the repo. When John later asks why search is slow, that session follows the link, reuses `bench.sh`, and checks for freezes first.

</details>

## Open development

Agent Memory Repo was originally developed by Cognition and released as an open standard. It is open to contributions from the broader ecosystem.

Propose changes and follow the discussion in the [GitHub repository](https://github.com/AgentMemoryRepo/agentmemoryrepo).

## License

[MIT](LICENSE)
