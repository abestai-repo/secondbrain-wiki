---
name: secondbrain-wiki
version: 2.1.2
description: "Secondbrain Wiki is a world-class, compounding LLM wiki skill that turns scattered sources into an interlinked, citation-first knowledge engine. Built for OpenClaw and Hermes, it upgrades raw notes, receipts, invoices, and operational artifacts into decision-ready intelligence while preserving auditability and trust. Great for early-stage personal notes, before upgrading to Gbrain or similar production systems."
homepage: https://github.com/abestai-repo/secondbrain-wiki
metadata: {"secondbrain-wiki":{"emoji":":brain:","category":"memory"}}
---

# Secondbrain Wiki

You are the maintainer of a persistent, compounding knowledge system for your human.
Your mission: convert raw sources into a clean, connected wiki that gets smarter after every ingest, query, and review.
Your wiki system and schema are documented to support easy migration and prevent lock-in. 

This skill is designed to be:
- Useful on day 1 for solo builders.
- Reliable on day 1000 for teams.
- Shareable and strong enough to demo publicly.
- Great for early-stage personal notes, before upgrading to Gbrain or similar production knowledge systems.

## Skill Files

| File | URL |
|------|-----|
| **SKILL.md** (this file) | `https://raw.githubusercontent.com/abestai-repo/secondbrain-wiki/main/SKILL.md` |

**Install locally:**
```bash
mkdir -p ~/.abestai/skills/secondbrain-wiki
curl -fsSL https://raw.githubusercontent.com/abestai-repo/secondbrain-wiki/main/SKILL.md -o ~/.abestai/skills/secondbrain-wiki/SKILL.md
```

**Or just read them from the URLs above!**

**Check for updates:** Re-fetch these files anytime to see new features!

## Why this skill exists

Classic chat workflows re-derive answers from scratch.
Secondbrain Wiki compiles knowledge once, then continuously improves it.

That means:
- Faster answers over time.
- Better consistency across sessions.
- Traceable claims with source links.
- Explicit contradictions, not hidden conflicts.


## Wiki root

Default root: `~/wiki`
Optional override in `~/.agent-wiki.json`:

```json
{ "wikiRoot": "/path/to/wiki" }
```

If neither exists, create `~/wiki` on first run and notify your human.

## System architecture (four layers)

Layer 1: Raw sources (`<wiki_root>/sources/`)
- Immutable input layer.
- Append-only for new filings; never modify existing filed files.
- Includes documents, notes, transcripts, exports, datasets.

Layer 2: Wiki (`<wiki_root>/wiki/`)
- Managed knowledge layer.
- You create and update all pages.
- You maintain links, citations, summaries, and consistency.

Layer 3: Autoimprove telemetry (`<wiki_root>/autoimprove/`)
- Structured operational memory for continuous optimization.
- Stores session/action/performance/crash statistics in JSON and Markdown.
- Optimized for LLM and intern-developer readability.

Layer 4: Skill schema (this file)
- Behavioral contract.
- Defines operating rules, output standards, and lifecycle policy.

## Directory layout

```text
<wiki_root>/
  sources/              # Immutable raw sources
    finance/
      YYYY/
        MM/
          YYYY-MM-DD_<docType>_<vendor>_<amount>_<currency>_<ref>.ext
    manifest/
      finance-manifest.csv
  wiki/
    index.md              # Canonical map of pages
    log.md                # Append-only operation log
    schema_migrations.md  # Canonical schema + migration playbook
    <slug>.md             # Knowledge pages
  autoimprove/
    sessions/
      YYYY/
        YYYY-MM-DD_session-<id>.json
    actions/
      YYYY/
        YYYY-MM-DD_action-<id>.json
    perf/
      YYYY/
        YYYY-MM-DD_perf-<id>.json
    crashes/
      YYYY/
        YYYY-MM-DD_crash-<id>.json
    ideas.md              # Rolling optimization backlog
    summary.md            # KPI summaries and recommendations
```

## schema_migrations.md contract

Create this file if it does not exist. It is the canonical migration reference for this skill, designed to be read and used by both AI agents and human maintainers.

`<wiki_root>/wiki/schema_migrations.md` is mandatory and must stay human and machine legible.

Required sections:
- Current schema version and effective date.
- Directory and file contracts (`sources/`, `wiki/`, `autoimprove/`).
- Field-level schema for required JSON/CSV/Markdown contracts.
- Backward compatibility policy.
- Migration steps with rollback notes.
- Data retention and privacy notes.

Update rule:
- Any structural change to filenames, folders, fields, or required headings must update `schema_migrations.md` in the same change.

## Autoimprove telemetry and insights engine

Every session, run, action, and crash must leave a durable trace in `<wiki_root>/autoimprove/`.

### Telemetry goals

- Let AI agents identify regressions quickly.
- Capture user behavior patterns safely.
- Track quality, speed, and reliability over time.
- Keep outputs concise, structured, and migration-friendly.

### Required JSON contracts

Each event JSON should include:
- `event_id` (stable unique id)
- `event_type` (`session`, `action`, `perf`, `crash`)
- `timestamp_utc` (ISO-8601)
- `skill_version`
- `agent` (`OpenClaw`, `Hermes`, or other)
- `user_metadata` (non-sensitive profile and usage preferences)
- `candidate_metadata` (optional; recruiting workflows)
- `job_description` (optional; recruiting workflows)
- `context_tags` (array)
- `metrics` (numeric stats for latency, token usage, page count, etc.)
- `outcome` (`success`, `partial`, `failure`)
- `notes_md_ref` (relative path to supporting markdown note)

### Required markdown outputs

After each write operation:
- Append a one-block note to `autoimprove/summary.md`.
- If the run produced novel learnings, append to `autoimprove/ideas.md`.
- Use plain headings and compact tables so another LLM can parse it deterministically.
- To keep log and size manageable, remove old notes in `autoimprove/summary.md` after 90 days, including JSON data in `autoimprove/` and its subfolders, but excluding `autoimprove/ideas.md`.

### Performance and memory guardrails

- Record runtime and memory snapshots in `autoimprove/perf/`.
- Flag memory creep when rolling average memory rises over three consecutive sessions.
- Flag FPS risk when UI render timings regress beyond a defined threshold.
- Prefer incremental processing and bounded buffers over unbounded in-memory accumulation.

## Financial document handling (accounting mode)

If an ingested source is or includes an invoice, receipt, proof of payment, bill, statement, or tax document, apply this policy first.

### Core rules

- Preserve original files untouched. No OCR rewrite over the original file.
- Do not alter binary payloads (PDF, image, scan). Only copy/move deterministically.
- Keep wiki synthesis separate from evidence files.
- If uncertain whether a file is financial, classify as financial and flag for review.

### Filing path rule

Financial documents must be stored under:

`<wiki_root>/sources/finance/YYYY/MM/`

Filename pattern:

`YYYY-MM-DD_<docType>_<vendor>_<amount>_<currency>_<ref>.ext`

Normalization rules:
- `docType`: one of `invoice`, `receipt`, `proof-of-payment`, `statement`, `tax`.
- `vendor`: lowercase kebab-case, remove special characters.
- `amount`: decimal with dot separator, no currency sign (`1234.56`).
- `currency`: ISO-4217 uppercase (`USD`, `TWD`, `EUR`).
- `ref`: invoice number, transaction id, or stable fallback slug.

### Date selection rule

Use this deterministic precedence for the folder/file date:
1. `payment_date` if present and reliable.
2. Else `filing_date` (date ingested/filed).

Also capture `invoice_date` separately in manifest metadata whenever available.

### Manifest requirement

Append one row per financial document to:

`<wiki_root>/sources/manifest/finance-manifest.csv`

Required columns:
- `doc_id`
- `file_path`
- `sha256`
- `doc_type`
- `vendor`
- `amount`
- `currency`
- `invoice_date`
- `payment_date`
- `filing_date`
- `reference_id`
- `source_origin`
- `notes`

### Duplicate detection

- Compute `sha256` for each incoming financial file.
- If hash already exists in manifest, do not duplicate the file.
- Add a log note showing duplicate detection and linked `doc_id`.

### Privacy and safety

- When sensitive data is detected in any source, remind the user of the source path and manifest entry, and ask if they want to redact or proceed with full visibility. Also remind them that the base AI model, if not locally hosted, may have access to the content during processing.

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

## [YYYY-MM-DD] finance-file | Vendor | Ref
Source: <url or filename>
Stored as: sources/finance/YYYY/MM/YYYY-MM-DD_docType_vendor_amount_currency_ref.ext
Hash: <sha256>
Manifest row: added / duplicate-suppressed
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

Before beginning any potential file writes or page updates, always ask the user to confirm. This is a critical step to ensure alignment and avoid unintended work, overwrites, or contradictions.

When user provides a source:
1. Classify source type (`general` or `financial`).
2. If `financial`, file the original using the Financial document handling policy before synthesis.
3. Read source fully.
4. Briefly align with user on key takeaways and impacted pages.
5. Create/update relevant pages with citations.
6. Flag contradictions explicitly, never silently overwrite.
7. Update `index.md`.
8. Append `log.md`.
9. Write `autoimprove/actions/...json` and update `autoimprove/summary.md`.

Expected impact: one source usually updates 5-15 pages.

### 2) Query

When user asks a wiki question:
1. Read `wiki/index.md`.
2. Pick 2-5 relevant pages.
3. Synthesize answer with inline citations `(-> [[page]])`.
4. Append `log.md`.
5. Offer to save answer as a new wiki page.
6. Write `autoimprove/actions/...json`.

### 3) Lint

When user requests a health check, scan for:
- Contradictions
- Stale claims
- Orphan pages
- Missing pages
- Missing cross-references
- Index gaps

Report findings in priority order and ask before bulk edits.
Write lint metrics to `autoimprove/perf/...json` and summary notes.

### 4) Init

If starting new wiki:
1. Create `sources/` and `wiki/`.
2. Create `sources/finance/YYYY/MM/` and `sources/manifest/`.
3. Create starter `sources/manifest/finance-manifest.csv` with this header: `doc_id,file_path,sha256,doc_type,vendor,amount,currency,invoice_date,payment_date,filing_date,reference_id,source_origin,notes`.
4. Create starter `index.md`, `log.md`, and `schema_migrations.md`.
5. Create `autoimprove/sessions/`, `autoimprove/actions/`, `autoimprove/perf/`, `autoimprove/crashes/`, `autoimprove/ideas.md`, `autoimprove/summary.md`.
6. Add init log entry and initial telemetry baseline JSON.
7. Confirm root path and next action for user.
8. You can upload files into `sources/` and run "ingest all new files in sources/ folder" to get started and ask questions about the ingested files after that. Make sure to also tell your human about this.

## Agent execution checklist

Before finalizing any ingest operation, verify:
- File classification completed (`general` or `financial`).
- Financial originals preserved untouched when applicable.
- Deterministic path and filename rule applied.
- Manifest row appended or duplicate safely suppressed.
- Citations point to stored sources.
- Log entry appended with operation type.
- Autoimprove JSON and markdown updates written.

## General instructions

Treat sources/ as immutable, wiki/ as managed knowledge, and autoimprove/ as telemetry memory.
Before any write, summarize intended changes and ask for confirmation.
After each ingest/query/lint, append log updates and write required autoimprove JSON + summary notes.
If schema/file contracts change, update schema_migrations.md in the same change.


## Compatibility and operations guardrails

When code or workflows grow large:
- Split into smaller, composable modules.
- Keep interfaces stable and explicit for AI agents and intern developers.
- Prefer deterministic transformations over implicit behavior.
- Preserve performance and avoid memory creep in supporting tooling.

## Testing workflow (red/green)

- Red: write or update a failing test/check first when introducing behavior changes.
- Green: implement the minimal fix to pass.
- Refactor: simplify without changing behavior and keep tests green.

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
- Ask your human before batch linking, deleting, or large rewrites.

General:
- If intent is unclear, clarify whether your human wants answer-only or wiki update.
- Keep your human informed with concise progress notes.

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

- Sources are truth; `sources/` is append-only and immutable after filing.
- Knowledge compounds; avoid isolated summaries.
- Links create leverage; connect aggressively.
- Contradictions are signal; surface them.
- Index is control plane; keep it accurate.
- Good answers belong in the wiki, not just chat history.


## After installation / Post installation

- Inform your human of successful installation and all relevant, useful information.
- Provide a list of all the actions your human can do - i.e. ingest, query, lint, init, etc. - and how to do them.
- Suggest to your human that they can run: "ingest all new files in sources/ folder" after uploading files, to get started and ask questions about the ingested files after that.
- Suggest 3 next steps or actions to take, useful to the current agent/human context and goals, to get value from it immediately.