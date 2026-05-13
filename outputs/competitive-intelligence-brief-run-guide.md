# Competitive Intelligence Brief — Run Guide

This guide walks you through running the **Competitive Intelligence Brief** workflow in Claude Code. The workflow takes a competitor name and produces a publish-ready brief plus an updated knowledge file that compounds across runs.

## A. What was built

| Artifact | What it does | Location |
|----------|--------------|----------|
| Sub-agent: `competitive-intelligence-brief` | Orchestrates the workflow end-to-end: loads existing knowledge, runs research, writes the brief, updates the knowledge file, prints a digest. Invoked by name from any Claude Code session in this repo. | [`.claude/agents/competitive-intelligence-brief.md`](../.claude/agents/competitive-intelligence-brief.md) |
| Skill: `researching-competitor-signals` | Gathers product / hiring / public-commentary signals via web search, with strict source-URL discipline. Returns a categorized findings list. | [`.claude/skills/researching-competitor-signals/SKILL.md`](../.claude/skills/researching-competitor-signals/SKILL.md) |
| Skill: `writing-competitor-briefs` | Turns findings into a publish-ready, business-audience-clear brief at `briefs/<competitor>/<date>.md`. Every section has an implication line. | [`.claude/skills/writing-competitor-briefs/SKILL.md`](../.claude/skills/writing-competitor-briefs/SKILL.md) |
| Skill: `updating-competitor-knowledge` | Persists new findings to `knowledge/competitors/<name>.md`. Owns the canonical schema. First-run-vs-update branching, conflict-append-flag, per-run cap (default 20). | [`.claude/skills/updating-competitor-knowledge/SKILL.md`](../.claude/skills/updating-competitor-knowledge/SKILL.md) |

Plus folders created on first runs:
- `briefs/<competitor>/<YYYY-MM-DD>.md` — dated brief outputs
- `knowledge/competitors/<competitor>.md` — the compounding knowledge file (one per competitor)

## B. Setup steps

**Already done.** Files are in place. To make the agent and skills discoverable in a new Claude Code session, all you need is to open Claude Code from this repo (`/Users/jamesgray/Code/demo/framework-v4`).

If you want to verify everything is wired:

1. From the repo root, run `ls .claude/agents/ .claude/skills/` — you should see the 1 agent file and 3 skill directories listed in the table above.
2. Open a Claude Code session in this repo. The agent and skills are auto-loaded — no installation step.
3. (Optional) Type `/agents` in Claude Code to see `competitive-intelligence-brief` in the list.
4. (Optional) Type `/skills` (or browse `.claude/skills/`) to confirm the three skills are visible.

No API keys, no MCP servers, no external services. WebSearch and WebFetch are built into Claude Code and used automatically.

## C. First run

### Sample invocation

In Claude Code, type:

```
Run competitive intel on Anthropic
```

(Anthropic is suggested because none of your existing knowledge files cover it — this exercises the first-run path. To exercise the update path on an existing competitor, try `Update the brief for Microsoft`.)

### What should happen, step by step

1. The main thread will say something like: *"I'll launch the competitive-intelligence-brief agent to handle this."* and dispatch the sub-agent.
2. The sub-agent loads `knowledge/competitors/anthropic.md` (does not exist → first-run path).
3. The sub-agent invokes `researching-competitor-signals` — you'll see 3 parallel `WebSearch` calls (product / hiring / commentary) and some `WebFetch` calls to verify specific sources.
4. The sub-agent invokes `writing-competitor-briefs` — writes `briefs/anthropic/<today>.md`.
5. The sub-agent invokes `updating-competitor-knowledge` — creates `knowledge/competitors/anthropic.md` with the schema header.
6. The sub-agent prints a 30-second digest with top signals and paths to the brief and knowledge file.

Total run time: roughly 2–5 minutes depending on web-search latency.

### What good output looks like

- **Brief** — at `briefs/anthropic/<today>.md`. Reads in 60 seconds. Has the canonical sections (What changed since last run, Positioning, Product Moves, Hiring Signals, Public Commentary, Open Questions). Every finding has an `*Implication:*` line. Every URL is real. Empty sections are marked explicitly, never padded.
- **Knowledge file** — at `knowledge/competitors/anthropic.md`. Has frontmatter (`competitor`, `last_updated`, `aliases`) and the canonical sections. Findings are date-stamped and source-linked.
- **Digest in chat** — a compact summary pointing to the two files above with 3–5 top-signal bullets.

### Common first-run issues and how to fix them

| Symptom | Likely cause | Fix |
|---|---|---|
| Agent says "I can't find the knowledge file" but still proceeds | This is the expected first-run path, not an error | Continue — the update skill will create the file |
| Brief is thin (1–2 findings total) | The competitor genuinely has no fresh signal in the cadence window | Try a wider window: `Run competitive intel on <competitor> for the last 30 days` |
| A cited URL 404s when you click it | A source moved after WebSearch indexed it; rare | Manually verify the claim and remove the bad citation if needed; flag for `/handsonai:improve` |
| Agent invokes only one skill and stops | Skill discovery failed | Make sure you're in the repo root when you start Claude Code; restart the session |
| Two simultaneous runs on the same competitor | Concurrent-write detection (v1 assumes single-operator) | Wait for the first run to finish; re-run if needed |
| Knowledge file frontmatter looks malformed | A human edited the file directly | Open the file, repair the frontmatter, re-run. **Never edit knowledge files outside the update skill.** |

## D. What to do next

### Repeatable trigger

Any of these will work from any Claude Code session in this repo:

- `Run competitive intel on <Competitor>`
- `What's new with <Competitor>?`
- `What's new with <Competitor> since last month?` — sets a 30-day cadence window
- `Update the brief for <Competitor>`
- `Set up competitive intel tracking for <NewCompetitor>` — explicit first-run framing

### Sharing with teammates

This workflow is currently scoped to **individual use** on your local filesystem. To share with a team:

1. Commit `.claude/agents/`, `.claude/skills/`, `briefs/`, and `knowledge/` to a shared git repo.
2. Teammates clone the repo and open Claude Code from it — the agent and skills are immediately available.
3. Treat `knowledge/competitors/*.md` as **append-only**. Never hand-edit; always run the agent. Hand-edits will break the schema and the agent will refuse to write.

A future "Cowork-compatible skill wrapper" (in the spec's Future Enhancements) would expose the workflow to non-Claude-Code surfaces — defer until v1 is validated in real use.

### When to revisit and improve

Run `/handsonai:improve` when you see any of these:

- Source URLs increasingly 404 or don't support the claim (research skill needs a tighter verification step)
- Briefs feel generic or stop landing implications (writing skill template needs voice/template iteration)
- Knowledge files balloon past 500+ findings per competitor (per-run cap or reconciliation pass needed)
- A new finding contradicts existing content but the conflict-flag goes unactioned for weeks (you need the future reconciliation pass)
- You start running the workflow at a regular cadence — that's the trigger to build the scheduled-task wrapper from the spec's Future Enhancements

Two recommended near-term follow-ups from the test pass:

1. **Run the agent against a competitor you haven't tested** (e.g., Anthropic) to validate the live Agent → Skill invocation chain end-to-end inside Claude Code.
2. **Run a second pass on Microsoft** to exercise the update path and the conflict-flag logic. Add a synthetic conflicting finding to the test prompt.

These were flagged as test-coverage gaps in [`outputs/competitive-intelligence-brief-test-results.md`](competitive-intelligence-brief-test-results.md).

### Change management (skip for individual use)

Not applicable — v1 is single-operator. If you decide to share with a team:

- **Training:** 5-minute demo per teammate — show one run start-to-finish.
- **Communication:** A short Notion or Slack post explaining what the briefs are for, where they live, and the cadence you plan to run them at.
- **Rollout:** Pilot with one or two competitors for two weeks. Read the briefs yourself, share with the team after. If briefs land usefully, expand to a tracked list.
