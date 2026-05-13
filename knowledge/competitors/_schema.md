---
purpose: Schema and operating rules for the competitor knowledge base
applies_to: knowledge/competitors/*.md
owner: James Gray
last_reviewed: 2026-05-13
---

# Competitor Knowledge Base — Schema

This file tells Claude how to read, update, and reason about every file under `knowledge/competitors/`. The competitor files themselves hold **data**; this file holds the **rules**. Edit this file when you change how the knowledge base works. Edit competitor files only through the `updating-competitor-knowledge` skill.

Adapted from Andrej Karpathy's "LLM Wiki" pattern: a structured, AI-maintained knowledge base where confidence is explicit and a learning loop (ingest / query / lint) keeps the contents coherent.

---

## File structure (canonical)

Every `knowledge/competitors/<kebab-name>.md` MUST use this shape:

```markdown
---
competitor: <Competitor Name>
last_updated: <YYYY-MM-DD>
aliases: [<alias1>, <alias2>]
---

# <Competitor Name> — Knowledge File

## Rules
<Binding beliefs that shape decisions. Each rule has a date promoted, the
Facts that triggered the promotion, and a short "act on this when…" note.>

## Facts
<Confirmed, dated, sourced events. Grouped under sub-headings: Product,
Hiring, Commentary, Financials.>

## Hypotheses
<Best-guesses awaiting confirmation. Each carries a confidence level
(low | medium) and what would promote or kill it.>

## Conflicts Pending Reconciliation
<Lint output: new findings that contradict prior claims. Human-reviewed.>

## Pending (Overflow)
<Findings that exceeded the per-run cap. Surfaced for the next run.>
```

---

## What lives where (the binary-vs-text rule)

A claim belongs in this knowledge base only if it is:

1. **Textual** — a sentence Claude can read, not a chart, image, or PDF blob.
2. **Decision-relevant** — would change how we'd respond, position, or act.
3. **Sourced** — has a real URL backing it. If it doesn't, drop it; never invent.

Binary artifacts (PDFs, screenshots, earnings decks) belong in a separate `assets/<competitor>/` folder. The knowledge file links to them but never embeds them.

---

## The three operations (learning loop)

Every workflow run executes these three operations in order. The `competitive-intelligence-brief` agent invokes them; the operator can also invoke them standalone.

### 1. Ingest
**Goal:** Read new sources, append to the right section, tag confidence.

- Fetch recent items inside the cadence window (default: last 7 days).
- Each finding gets a category (product / hiring / commentary / financial), a confidence level (high | low), and a source URL.
- New findings go to **Facts** by default — unless the finding *contradicts* an existing claim (→ Conflicts) or is *speculative* (→ Hypotheses).
- Bump `last_updated` to today.

### 2. Query
**Goal:** Answer specific questions from the knowledge base without re-researching.

- Standing queries are defined per competitor below.
- The agent reads the file, scopes to the relevant sections (Rules first, then Facts), and answers from what's there.
- If the answer requires a Fact that doesn't exist, the agent records the gap as a Hypothesis and notes it in the digest — it does NOT silently web-search to fill the gap during a query operation.

### 3. Lint
**Goal:** Find contradictions, stale claims, and promotion/demotion candidates.

Three sub-checks on every run:

- **Contradictions** — new Fact contradicts an existing Rule, Fact, or active Hypothesis → flag to **Conflicts Pending Reconciliation**.
- **Promotion candidates** — a Hypothesis that has been corroborated by ≥2 independent Facts within 30 days → propose promotion to Fact (or Fact → Rule if it's been acted on repeatedly).
- **Demotion candidates** — a Fact >180 days old with no reaffirming Facts in the last 90 days → propose demotion to Hypothesis. A Rule contradicted by a recent Fact → propose demotion to Hypothesis.

Lint output is **proposals only** — the human approves promotions/demotions during the daily digest review.

---

## Promotion / demotion logic

```
Hypothesis ──(corroborated ≥2x in 30d)──► Fact
Fact ──(acted on repeatedly)──► Rule
Rule ──(contradicted by recent Fact)──► Hypothesis
Fact ──(stale >180d, no reaffirmation)──► Hypothesis
```

Every promotion/demotion is logged inline in the file with the date and the triggering Fact IDs (or finding URLs).

---

## Standing research topics

Each scheduled run iterates these competitors and runs the full ingest/query/lint loop for each. Add or edit competitors here, not in the agent.

- **AWS** — Bedrock / AgentCore / Quick / Connect; OpenAI partnership terms; capex commentary from Garman or Jassy.
- **Google** — Gemini Enterprise / Agentspace; Vertex AI agent platform moves; Workspace agent rollouts.
- **Microsoft** — Copilot / Foundry / Azure AI; M365 Copilot agent surface area; Mustafa Suleyman product strategy.
- **Cross-cutting** — Hyperscaler capex announcements; agent platform pricing changes; agent commerce / payments primitives.

## Standing queries

Questions the agent answers on every run by reading the knowledge file. Answers populate the brief; gaps go to the digest as research opportunities. Edit per competitor or keep generic; the agent loops these for each competitor.

1. **Positioning shift** — *Has the competitor's strategic positioning materially changed in the last 30 days?* Scope: `rules`, `facts`. Look for new Facts that contradict prior Rules, or Rules that have been promoted/demoted since last run.
2. **Capability surface area** — *What agent / AI product capabilities did the competitor ship this period?* Scope: `facts` (Product sub-section, last 7 days).
3. **Leadership signal** — *Any leadership or organizational moves that change the strategic direction?* Scope: `facts` (Hiring sub-section, last 30 days).
4. **Open contradictions** — *Which Hypotheses now have enough corroborating evidence to be reviewed for promotion?* Scope: `hypotheses` cross-referenced against new Facts. (Lint will also flag these — Query reports them up to the brief.)
5. **Stale beliefs** — *Which Facts have not been reaffirmed in 90+ days and may no longer be true?* Scope: `facts`, filtered by age.

---

## What goes in this file vs. the competitor file

| Belongs in `_schema.md` (this file)       | Belongs in `<competitor>.md`                 |
|-------------------------------------------|----------------------------------------------|
| File structure & section definitions      | Actual claims, facts, hypotheses             |
| Operations (ingest / query / lint)        | Findings, sources, dates                     |
| Promotion / demotion logic                | Promotion / demotion *events* with their date|
| Binary-vs-text rule                       | Links to binary artifacts (never the binary) |
| Standing research topics                  | The findings those topics produced           |

If you find yourself editing competitor files to change *how the system works*, stop — that change belongs here.

---

## Non-negotiables

- **No claim without a source URL.** If a finding lacks a real URL, drop it.
- **Never overwrite a prior claim.** Conflicts are appended + flagged.
- **The `updating-competitor-knowledge` skill owns all writes.** Hand-editing competitor files breaks the schema contract.
- **Lint proposes; humans dispose.** Promotions and demotions are surfaced in the daily digest for review, not applied silently.
