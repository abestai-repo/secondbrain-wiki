# secondbrain-wiki

ABESTAI Secondbrain LLM Wiki designed by Jeric T.
A Karpathy-style wiki skill for OpenClaw and Hermes, powered by Markdown with a migration-safe schema and autoimprove telemetry.

## Obsidian + AI Agent Setup

### 1) Best vault location

Set your Obsidian vault to the **wiki root** folder (`<wiki_root>`), not to `<wiki_root>/wiki`.

Reason: opening `<wiki_root>` exposes all required layers together:
- `sources/` (immutable evidence)
- `wiki/` (knowledge pages and index)
- `autoimprove/` (telemetry and insights for recursive improvement)

Expected structure:

```text
<wiki_root>/
  sources/
  wiki/
  autoimprove/
```

### 2) Recommended Obsidian practices

- Keep wikilinks enabled (`[[page]]`) so internal page linking matches `SKILL.md`.
- Keep `wiki/index.md` pinned as your routing map.
- Keep `wiki/log.md` visible during ingest/query/lint to track append-only operations.
- Keep `autoimprove/summary.md` and `autoimprove/ideas.md` visible for iterative improvement loops.

## AI Agent Starter Pack

When handing this system to an AI agent, provide these files first:

1. `SKILL.md`

### Agent kickoff instruction template

Use this as the first prompt to a new agent:

```text
You are operating the secondbrain-wiki skill.
Read SKILL.md first.
Treat sources/ as immutable, wiki/ as managed knowledge, and autoimprove/ as telemetry memory.
Before any write, summarize intended changes and ask for confirmation.
After each ingest/query/lint, append log updates and write required autoimprove JSON + summary notes.
If schema/file contracts change, update schema_migrations.md in the same change.
```
