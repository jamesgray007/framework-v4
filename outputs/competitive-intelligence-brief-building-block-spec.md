# Competitive Intelligence Brief — AI Building Block Spec

## Execution Pattern

**Agent** — A single Claude Code sub-agent orchestrates the workflow, invoking three reusable skills for the bounded-judgment steps (research, synthesis, knowledge update). Chosen over Skill-Powered Prompt because the workflow has tool use (web search), parallelizable sub-steps within research, and meaningful per-step decision logic that benefits from an orchestrator. Chosen over multi-agent because the critical path is sequential and single-owner — no agent handoffs required.

## Architecture Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Lens | Individual | v1 runs locally per operator; team-share via shared repo is a future enhancement |
| Platform | Claude Code | User-confirmed; sub-agent markdown is the chosen primitive |
| Platform Mode | code | Claude Code is a code-mode CLI; artifacts are markdown files under `.claude/` |
| Orchestration | Agent | Single sub-agent invoking 3 skills; sequential critical path with bounded-judgment sub-tasks |
| Involvement | Augmented | Manual-on-demand invocation by the operator; scheduled execution is a future enhancement |
| Trigger | Manual on demand | v1 scope; cadenced runs (weekly/monthly per competitor) deferred to a future scheduled-task wrapper |

## Scenario Summary

| Field | Value |
|---|---|
| **Workflow Name** | Competitive Intelligence Brief |
| **Description** | Given a competitor name, research the competitor's recent moves and produce a structured brief — positioning, product moves, hiring signals, public commentary — then write findings to a structured context file. Run repeatedly, the workflow's outputs accrete into a self-improving knowledge base for that competitor. |
| **Outcome** | A publish-ready structured brief on the competitor's recent moves, plus an updated `knowledge/competitors/{name}.md` file with new findings appended (date-stamped, source-linked). |
| **Trigger** | Manual on demand (v1). Scheduled per-competitor cadence deferred to future. |
| **Type** | Augmented |
| **Business Process** | Competitive intelligence / market research |
| **Owner** | Workflow operator (James Gray) |

## Step-by-Step Decomposition

| Step | Name | Phase | Autonomy | Orchestration | Integration (use/build) | Intelligence | Skill Candidate? | Human Gate? |
|------|------|-------|----------|---------------|------------------------|--------------|-------------------|-------------|
| 1 | Load existing competitor knowledge | Setup | Deterministic | Agent | CLI: Read, Glob (use, built-in) | Model: fast; Context: knowledge file schema; Memory: read prior runs via knowledge file | No (inline) | No |
| 2 | Research competitor's recent moves | Research | Guided | Skill | CLI: WebSearch, WebFetch (use, built-in) | Model: reasoning; Context: Step 1 knowledge + cadence_window | Yes — `researching-competitor-signals` (new) | No |
| 3 | Synthesize structured brief | Synthesis | Guided | Skill | CLI: Write (use, built-in) | Model: reasoning; Context: findings + existing knowledge + brief template | Yes — `writing-competitor-briefs` (new) | No |
| 4 | Update knowledge file | Persistence | Deterministic | Skill | CLI: Read, Write, Edit, Glob (use, built-in) | Model: fast; Context: schema convention; Memory: writes the compounding state | Yes — `updating-competitor-knowledge` (new) | No |
| 5 | Output digest | Output | Deterministic | Agent | — | Model: fast | No (inline) | No |

### Autonomy Spectrum Summary

- **Deterministic (Steps 1, 4, 5):** Fixed-path operations with rule-based decisions. Step 1 is file-read + frontmatter parse + branch on existence. Step 4 is schema-driven persistence with explicit conflict-append rules. Step 5 is template-driven summarization.
- **Guided (Steps 2, 3):** Bounded model judgment. Step 2 decides "what's genuinely new" by cross-referencing existing knowledge; Step 3 decides "what's the implication, not just the fact." Both operate under explicit constraints (source-URL discipline, schema sections) — no free-form re-planning.
- **No Autonomous steps:** The workflow does not self-redirect, backtrack, or re-invoke based on intermediate failure. A failed sub-step surfaces in metadata; the agent proceeds or stops per documented rules.
- **No Human steps:** All five steps execute via the agent. The operator triggers the run and consumes the output; there is no in-loop human handoff.

## Skill Candidates

### researching-competitor-signals (Step 2)

| Dimension | Detail |
|---|---|
| **Purpose** | Bounded competitor research routine — gather product, hiring, and public-commentary signals from the web within a defined cadence window, with strict source-URL discipline. |
| **Covers Steps / Domains** | Step 2 |
| **Inputs** | `competitor_name` (string, required); `existing_knowledge` (markdown, optional — empty on first run); `cadence_window` (string, default: "last 7 days"); `aliases` (list of strings, optional) |
| **Outputs** | Categorized findings list — each record `{category, claim, source_url, confidence, conflicts_with_existing}`. Categories: product, hiring, commentary. |
| **Decision Logic** | Run three search categories (product launches/announcements; hiring signals; public commentary) — instructions allow these to be issued in parallel when supported. Every claim MUST carry a source URL — claims without URLs are dropped. Single-source claims are marked `confidence: low`. Multi-source corroborated claims are `confidence: high`. Cross-reference each finding against `existing_knowledge`; tag as `new`, `restatement`, or `conflict`. Default cadence window is "last 7 days" per evaluation criteria. |
| **Failure Modes** | Web search timeout or rate limit → return partial findings + surface in result metadata. Hallucinated claim (no URL) → drop the claim. Paywalled source → annotate `paywalled: true` and do not invent content. No new findings since last run → return empty findings with `status: no_change`. |
| **Required Tools** | CLI: WebSearch (use), WebFetch (use) |
| **Depends On** | None |

### writing-competitor-briefs (Step 3)

| Dimension | Detail |
|---|---|
| **Purpose** | Transform raw competitor findings into a publish-ready, business-audience-clear structured brief with inline citations and a "what changed since last run" lead. |
| **Covers Steps / Domains** | Step 3 |
| **Inputs** | `competitor_name` (string); `findings` (categorized list from research skill or any compatible source); `existing_knowledge` (markdown, optional); `output_path` (path, default: `briefs/{competitor}/{date}.md`) |
| **Outputs** | Markdown brief saved to `output_path`. Sections: "What changed since last run", Positioning, Product moves, Hiring signals, Public commentary, Open questions. |
| **Decision Logic** | Every section calls out the *implication*, not just the fact. Sections with no findings are marked explicitly (e.g., "No hiring signals found this run") — never padded. All claims are inline-cited with the source URL from the findings record. Brief opens with a 3–5 bullet "what changed" summary derived from `new` and `conflict` findings. If findings volume is thin, produce a partial brief and surface the gap; do not synthesize content. Brand voice: clear and concise for a business audience. |
| **Failure Modes** | Findings too thin to populate any section → produce a "no material change" brief with explicit empty sections. Generic "they announced X" output without implication → internally re-prompt for the implication before saving. Capability/scenario mismatch (brief asserts something the source doesn't support) → drop the claim, log a quality flag. Brief duplicates prior run → flag as a quality alert in the result metadata. |
| **Required Tools** | CLI: Write (use), Read (use, for reading template + existing knowledge) |
| **Depends On** | Brief template (carried inside the skill — section structure, length norms, voice anchors) |

### updating-competitor-knowledge (Step 4)

| Dimension | Detail |
|---|---|
| **Purpose** | Persist new competitor findings to the structured `knowledge/competitors/<name>.md` file. This is the compounding mechanism that turns one-off runs into a self-improving knowledge base. |
| **Covers Steps / Domains** | Step 4 |
| **Inputs** | `competitor_name` (string); `findings` (the same findings list passed to the writing skill); `existing_knowledge` (markdown or None for first run) |
| **Outputs** | Updated `knowledge/competitors/<competitor-name>.md`. Status record `{path, mode: first_run|update, new_count, conflict_count, last_updated}`. |
| **Decision Logic** | **First-run path** (file does not exist): create the file with the schema header (frontmatter: `competitor`, `last_updated`, `aliases`); seed sections from highest-confidence findings; set `last_updated` to today. **Update path** (file exists): append new findings to their categorized sections with date stamps and source links; never overwrite previously confirmed content. **Conflict handling**: when a finding conflicts with existing content, append the new finding AND add a `> ⚠️ conflict-with: ...` flag note adjacent — never silently overwrite. Bump `last_updated` to today. |
| **Failure Modes** | Schema mismatch on read (hand-edited file is malformed) → notify operator, skip the write, do not corrupt the file. Concurrent write detected (v1 assumes single operator; future enhancement to file-lock) → flag and require manual rerun. Findings bloat (excessive low-signal accretion) → enforce a per-run cap (default: 20 findings appended per run; configurable) and log overflow to a `pending/` subsection. |
| **Required Tools** | CLI: Read (use), Write (use), Edit (use), Glob (use) |
| **Depends On** | Schema convention (carried inside the skill — this is the artifact Session 10's self-improvement work will iterate on) |

## Agent Configuration

### competitive-intelligence-brief

| Component | Detail |
|---|---|
| **Name** | `competitive-intelligence-brief` |
| **Purpose** | Invoked when the operator wants a competitive intelligence brief on a named competitor, or wants to update an existing competitor's knowledge base. The agent owns Step 1 (load knowledge), orchestrates Steps 2–4 by invoking the three skills in sequence, and owns Step 5 (digest assembly). |
| **Instructions** | **Mission:** produce a publish-ready competitive brief AND grow the per-competitor knowledge file. **Responsibilities:** (1) parse competitor name and optional inputs (`cadence_window`, `competitor_aliases`, `output_path`); (2) read `knowledge/competitors/<kebab-cased-name>.md` if it exists, parse frontmatter, flag first-run / stale; (3) invoke `researching-competitor-signals` with existing knowledge; (4) invoke `writing-competitor-briefs` with the findings; (5) invoke `updating-competitor-knowledge` with the findings; (6) assemble a 30-second digest pointing to the brief and the updated knowledge file, surfacing any conflict / staleness / schema-fit flags. **Behavior:** never overwrite the knowledge file directly — always go through the update skill. Never produce a claim without a source URL — drop unsourced claims. **Goals:** source accuracy (every URL real and valid), freshness (default cadence window: last 7 days), completeness (all required brief sections present or explicitly marked empty). **Tone & style:** clear and concise for a business audience. **Output format:** at end of run, print the digest inline + paths to the brief and updated knowledge file. |
| **Model** | reasoning-heavy (opus) — Step 2 cross-reference and Step 3 implication-extraction benefit from strong reasoning |
| **Tools** | WebSearch, WebFetch, Read, Write, Edit, Glob |
| **Skills** | `researching-competitor-signals`, `writing-competitor-briefs`, `updating-competitor-knowledge` |
| **Trigger Examples** | (1) User: "Run competitive intel on Microsoft" → agent loads `knowledge/competitors/microsoft.md` if exists, invokes research skill (cadence: last 7 days), then writing skill, then update skill, prints digest. (2) User: "What's new with Google since last month?" → same flow but `cadence_window` = "last 30 days". (3) User: "Set up competitive intel tracking for AWS" → first-run path: no knowledge file exists, agent invokes research with empty existing-knowledge, writing skill produces a seed brief, update skill creates the knowledge file. |

## Step Sequence and Dependencies

```
Step 1 (load knowledge — inline)
        │
        ▼
Step 2 (research — skill)
   ├── product search    ┐
   ├── hiring search     ├── parallelizable
   └── commentary search ┘
   └── cross-reference (sequential after the 3)
        │
        ▼
Step 3 (synthesize brief — skill)
        │
        ▼
Step 4 (update knowledge file — skill)
        │
        ▼
Step 5 (digest — inline)
```

**Parallel:** The three sub-searches inside Step 2 (product, hiring, commentary) have no data dependency on each other and can run concurrently when the platform supports it.

**Sequential:** Step 1 → Step 2 → Step 3 → Step 4 → Step 5. Step 2's cross-reference sub-step depends on all three searches completing.

**Critical path:** Step 1 → Step 2 (gated on slowest sub-search + cross-reference) → Step 3 → Step 4 → Step 5.

## Prerequisites

1. Claude Code installed and configured with WebSearch/WebFetch enabled
2. `.claude/agents/` directory in the working repo (created during Build if absent)
3. `.claude/skills/` directory in the working repo (created during Build if absent)
4. `knowledge/competitors/` directory created on first run (the update skill creates it if missing)
5. `briefs/` directory created on first run (the writing skill creates it if missing)
6. No external API keys, no MCP servers, no third-party integrations required

## Context Inventory

| # | Artifact | Type | Used By | Status | Location | Key Contents |
|---|---|---|---|---|---|---|
| 1 | `knowledge/competitors/<name>.md` | Context | Step 1 (read), Step 4 (write) | Create (on first run, per competitor) | `knowledge/competitors/<name>.md` | Frontmatter (`competitor`, `last_updated`, `aliases`) + structured sections (positioning, product, hiring, commentary, open questions, conflicts pending) |
| 2 | Knowledge file schema convention | Context | Step 1 (read), Step 4 (write) | Create | Inline in `updating-competitor-knowledge` skill | Section conventions, append rules, conflict-flag format, frontmatter spec |
| 3 | Brief template | Context | Step 3 | Create | Inline in `writing-competitor-briefs` skill | Section headings, length norms, citation format, voice anchors |
| 4 | Web search results | External | Step 2 | Exists | Platform capability (WebSearch / WebFetch) | Real-time web search hits |
| 5 | Per-competitor search hints (aliases, product/exec names) | Context | Step 2 | Create (accretes over time) | Inside each `knowledge/competitors/<name>.md` frontmatter (`aliases`) | Search-term hints loaded by Step 1, passed to Step 2 |
| 6 | Output location for briefs | Context | Step 3 | Create (on first run) | `briefs/<competitor>/<date>.md` (default; overridable) | Rendered brief markdown |

## Data Readiness Summary

All context items are AI-Accessible. No data readiness blockers. Two items (schema convention, brief template) are "Create" but lightweight and produced during Build by writing them inline into their respective skills.

## Integration Options

### Web search (Step 2)

**Curated (recommended):**

| Block | Option | Source URL | Trade-off |
|-------|--------|-----------|-----------|
| CLI | WebSearch (Claude Code built-in) | https://docs.claude.com/en/docs/claude-code | Zero setup; first-class tool; ideal for the breadth searches in Step 2 |
| CLI | WebFetch (Claude Code built-in) | https://docs.claude.com/en/docs/claude-code | Zero setup; first-class; ideal for fetching specific source URLs found by WebSearch |

*Recommendation: WebSearch + WebFetch together. WebSearch for discovery; WebFetch for verifying a specific source URL the brief will cite. No external integrations needed.*

### Filesystem read/write (Steps 1, 3, 4)

**Curated (recommended):**

| Block | Option | Source URL | Trade-off |
|-------|--------|-----------|-----------|
| CLI | Read / Write / Edit / Glob (Claude Code built-in) | https://docs.claude.com/en/docs/claude-code | Native filesystem tools; no integration work; full control over knowledge file format |

*Recommendation: built-in filesystem tools. No external storage integration needed for v1.*

## Model Recommendation

**Default:** reasoning-heavy (`opus`) — Step 2's cross-reference logic ("is this genuinely new?") and Step 3's implication extraction ("what does this *mean*?") both benefit from strong reasoning.

**Per-step overrides** (optional, for cost optimization):
- Steps 1, 4, 5: fast model (`haiku`) acceptable — these are deterministic and don't require reasoning. The skill/agent framework can route by step if desired; default is to keep the whole run on `opus` for simplicity.

## Recommended Implementation Order

### Quick Wins (implement first)
1. **`updating-competitor-knowledge` skill** — codifies the compounding mechanism + schema convention. Without it the workflow does not compound, which is the whole point.
2. **Agent file `competitive-intelligence-brief.md`** — even with skills stubbed, the agent runnable proves the orchestration shape end-to-end.

### Core (implement second)
3. **`researching-competitor-signals` skill** — replace any temporary inline research with the dedicated skill once the agent shape is validated.
4. **`writing-competitor-briefs` skill** — replace any temporary inline brief assembly with the dedicated skill; iterate on template based on the three test runs.

### Future Enhancement (optional)
5. **Scheduled-task wrapper** — wrap the agent in a Claude Code scheduled task (`mcp__scheduled-tasks`) for cadenced per-competitor runs (weekly top-tier, monthly second-tier).
6. **Cowork-compatible skill wrapper** — a top-level skill that mirrors the agent so the workflow runs in Cowork as well.
7. **Reconciliation pass** — periodic skill that resolves conflict-flagged entries in the knowledge file and prunes stale claims.
8. **Digest aggregator** — rolls per-competitor digests into a weekly cross-competitor newsletter.
9. **Per-run findings cap configurability + bloat detection** — for long-running compounding files.

## Where to Run

**Claude Code** in the working repo. Manual invocation: ask the agent to run competitive intel on a named competitor (e.g., "Run competitive intel on Microsoft"). The agent triggers the three skills in sequence and prints the digest with paths to the brief and updated knowledge file.

For frequent use across multiple competitors, the future scheduled-task wrapper (Implementation Order #5) is recommended.

## Evaluation Criteria

### What good output looks like
Every URL in the brief is a real, valid source — zero fabricated links. The brief reads clearly and concisely for a business audience: short paragraphs, plain language, every section calls out the implication rather than just stating the fact, and the "what changed since last run" summary at the top gives a 30-second read on the new signal.

### Quality dimensions
1. **Source accuracy** — every cited URL resolves to a real page that supports the claim it's attached to. Zero fabricated or hallucinated URLs. Single-source claims marked low-confidence.
2. **Information freshness** — findings are scoped to the last 7 days by default (overridable via `cadence_window`). Stale findings older than the cadence window are excluded from "what changed" and surfaced separately if relevant.
3. **Completeness** — all required brief sections are present: "What changed since last run", Positioning, Product moves, Hiring signals, Public commentary, Open questions. Empty sections are marked explicitly, never padded.

### Test scenarios

| # | Scenario | Input description | What to look for |
|---|----------|-------------------|------------------|
| 1 | Microsoft | Competitor: Microsoft. First-run path. cadence_window: last 7 days. | High signal volume; cross-reference logic must distinguish noise from material moves; brief stays concise despite abundant input; all cited URLs valid; knowledge file created cleanly. |
| 2 | Google | Competitor: Google. First-run path. cadence_window: last 7 days. | Similar to Microsoft — tests the workflow at high-volume, high-noise competitors. Brief should not duplicate Microsoft's brief in structure or boilerplate. |
| 3 | AWS | Competitor: AWS. First-run path. cadence_window: last 7 days. | Tests product-moves-heavy domain (lots of service launches). Brief should land the *implication* (e.g., what AWS's move means for buyers) not just enumerate releases. |

### Minimum quality bar
A competitive brief is created at `briefs/<competitor>/<date>.md` containing every required section (or an explicit empty marker), with at least one source-cited finding per populated section, and the per-competitor knowledge file at `knowledge/competitors/<name>.md` is created (first run) or updated (subsequent runs) with the new findings appended.
