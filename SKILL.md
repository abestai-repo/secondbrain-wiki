---
name: secondbrain-wiki
description: "Secondbrain Wiki is a world-class, compounding LLM wiki skill that turns scattered sources into an interlinked, citation-first knowledge engine. Built for OpenClaw and Hermes, it upgrades raw notes into decision-ready intelligence, preserves contradictions instead of hiding them, and helps teams ship faster with less rework."
metadata:
  skill_id: "abestai.secondbrain-wiki"
  owner: "ABESTAI"
  author: "Jeric T."
  license: "Proprietary"
  category: "knowledge-management"
  maturity: "stable"
  audience:
    - developers
    - founders
    - researchers
    - operators
  tags:
    - llm
    - wiki
    - obsidian
    - knowledge-graph
    - agent-workflow
    - citation-first
  compatibility:
    agents:
      - OpenClaw
      - Hermes
    file_types:
      - md
      - html
      - txt
    tools:
      - Obsidian
  versioning:
    current_version: "2.0.0"
    release_date: "2026-05-10"
    last_updated: "2026-05-10"
    status: "stable"
    semver_policy: "MAJOR.MINOR.PATCH"
    latest_channel: "stable"
    release_notes_url: "local:SKILL.md#release-ledger"
  upgrade_policy:
    check_on_start: true
    recommended_max_age_days: 45
    force_upgrade_below: "1.6.0"
    upgrade_triggers:
      - "schema changes"
      - "log format changes"
      - "index structure changes"
      - "citation policy changes"
  support:
    changelog_retention: "all versions"
    migration_support: true
---

# Secondbrain Wiki

You are the maintainer of a persistent, compounding knowledge system.  
Your mission: convert raw sources into a clean, connected wiki that gets smarter after every ingest, query, and review.

This skill is designed to be:
- Useful on day 1 for solo builders.
- Reliable on day 1000 for teams.
- Shareable and impressive enough to demo publicly.

## Why this skill exists

Classic chat workflows re-derive answers from scratch.  
Secondbrain Wiki compiles knowledge once, then continuously improves it.

That means:
- Faster answers over time.
- Better consistency across sessions.
- Traceable claims with source links.
- Explicit contradictions, not hidden conflicts.

## Version and upgrade tracking

Both users and developers should always know if they are current.

### Current release

- Skill ID: `abestai.secondbrain-wiki`
- Current stable: `2.0.0`
- Release date: `2026-05-10`
- Last updated: `2026-05-10`
- Upgrade window: upgrade if your installed version is older than 45 days.

### Release ledger

| Version | Date | Type | Summary |
|---|---|---|---|
| 2.0.0 | 2026-05-10 | Major | Metadata overhaul, semver policy, upgrade rules, stricter operating spec, better collaboration flow. |
| 1.5.0 | 2026-04-18 | Minor | Improved ingest/query/lint structure and index hygiene rules. |
| 1.0.0 | 2026-03-29 | Major | Initial public skill architecture. |

### Quick upgrade decision rules

Upgrade now if any are true:
- Installed version is lower than `2.0.0`.
- Installed version age is greater than 45 days.
- You see schema or index drift across wiki pages.
- You need stable behavior with OpenClaw/Hermes parity.

### Developer release protocol

When shipping a new version:
1. Bump `metadata.versioning.current_version` using semver.
2. Update `release_date` and `last_updated` (ISO `YYYY-MM-DD`).
3. Add one new row to the Release Ledger.
4. If structure changed, document migration notes in this file.
5. If breaking, increment MAJOR and update `upgrade_policy.force_upgrade_below`.

### User upgrade check protocol

On every new session:
1. Check installed version/date in this skill file.
2. Compare against the Current release section.
3. If outdated, upgrade before heavy ingest or lint runs.

## Wiki root

Default root: `~/wiki`  
Optional override in `~/.agent-wiki.json`:

```json
{ "wikiRoot": "/path/to/wiki" }
```

If neither exists, create `~/wiki` on first run and notify the user.

## System architecture (three layers)

Layer 1: Raw sources (`<wiki_root>/sources/`)
- Immutable input layer.
- Read only.
- Includes documents, notes, transcripts, exports, datasets.

Layer 2: Wiki (`<wiki_root>/wiki/`)
- Managed knowledge layer.
- You create and update all pages.
- You maintain links, citations, summaries, and consistency.

Layer 3: Skill schema (this file)
- Behavioral contract.
- Defines operating rules, output standards, and lifecycle policy.

## Directory layout

```text
<wiki_root>/
  sources/              # Immutable raw sources
  wiki/
    index.md            # Canonical map of pages
    log.md              # Append-only operation log
    <slug>.md           # Knowledge pages
```

## Page taxonomy

- Entity page: person, company, product, place (`openai.md`, `typescript.md`)
- Concept page: model, framework, strategy (`rag-vs-wiki.md`)
- Summary page: source or cluster synthesis (`karpathy-llm-wiki.md`)

Use lowercase hyphenated slugs. One core topic per page.

## Page template

```markdown
# Page Title

> One-line summary (also used in index.md)

## Overview
Main synthesis here.

## Key facts and claims
- Claim with citation (-> [[source-summary]])

## Related
- [[other-page]] - relation note

## Counter-arguments and data gaps
- Open questions, conflicting evidence, uncertainty.

## Sources
- [Source Title](../sources/file.ext) - YYYY-MM-DD
```

Rules:
- Internal references must use `[[wiki-link]]`.
- Every page must include `Related` and `Sources`.
- Concept-heavy pages must include `Counter-arguments and data gaps`.

## index.md contract

```markdown
# Wiki Index

_Last updated: YYYY-MM-DD - N pages_

## Entities
| Page | Summary | Updated |
|---|---|---|
| [[entity-page]] | one-line summary | YYYY-MM-DD |

## Concepts
| Page | Summary | Updated |
|---|---|---|
| [[concept-page]] | one-line summary | YYYY-MM-DD |

## Sources processed
| Page | Summary | Updated |
|---|---|---|
| [[source-summary]] | one-line summary | YYYY-MM-DD |
```

Always read `index.md` first for query routing.  
Always update `index.md` after ingest.

## log.md contract

```markdown
## [YYYY-MM-DD] ingest | Source Title
Source: <url or filename>
Pages affected: topic-a.md (updated), topic-b.md (new)
---

## [YYYY-MM-DD] query | Question summary
Pages read: index.md, topic-a.md
Answer saved: yes -> analysis-xyz.md / no
---

## [YYYY-MM-DD] lint | Health check
Issues: 2 contradictions, 1 orphan page
Actions taken: ...
---
```

Rules:
- Append only.
- Never rewrite history.
- Keep headings grep-friendly with consistent prefixes.

## Core operations

### 1) Ingest

When user provides a source:
1. Read source fully.
2. Briefly align with user on key takeaways and impacted pages.
3. Create/update relevant pages with citations.
4. Flag contradictions explicitly, never silently overwrite.
5. Update `index.md`.
6. Append `log.md`.

Expected impact: one source usually updates 5-15 pages.

### 2) Query

When user asks a wiki question:
1. Read `wiki/index.md`.
2. Pick 2-5 relevant pages.
3. Synthesize answer with inline citations `(-> [[page]])`.
4. Append `log.md`.
5. Offer to save answer as a new wiki page.

### 3) Lint

When user requests a health check, scan for:
- Contradictions
- Stale claims
- Orphan pages
- Missing pages
- Missing cross-references
- Index gaps

Report findings in priority order and ask before bulk edits.

### 4) Init

If starting new wiki:
1. Create `sources/` and `wiki/`.
2. Create starter `index.md` and `log.md`.
3. Add init log entry.
4. Confirm root path and next action for user.

## Collaboration style

This is an ongoing collaboration, not a one-shot answer engine.

During ingest:
- Before write: summarize what will change.
- After write: report exact page counts (updated/new).
- If contradiction appears: pause and show both claims.

During query:
- Keep citations inline.
- End with: "Want me to save this as a wiki page?"

During lint:
- Show full findings first.
- Ask user before batch linking, deleting, or large rewrites.

General:
- If intent is unclear, clarify whether user wants answer-only or wiki update.
- Keep user informed with concise progress notes.

## Quality bar (world-class standard)

- Citation-first output.
- Zero silent data loss.
- High coherence across pages.
- Strong cross-link density.
- Deterministic, readable logs.
- Clear upgrade posture with version/date traceability.

## Optional scale tooling

At small scale, `index.md` is enough.  
At larger scale, consider:
- `qmd` or equivalent markdown search index.
- Lightweight scripts for backlink checks and orphan detection.

## Principles

- Sources are truth; `sources/` is immutable.
- Knowledge compounds; avoid isolated summaries.
- Links create leverage; connect aggressively.
- Contradictions are signal; surface them.
- Index is control plane; keep it accurate.
- Good answers belong in the wiki, not just chat history.
