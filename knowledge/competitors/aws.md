---
competitor: AWS
last_updated: 2026-05-13
aliases: [Amazon Web Services, Bedrock, Bedrock AgentCore, Amazon Quick, Amazon Connect, Matt Garman, Andy Jassy, Prasad Kalyanaraman, Dave Brown, SageMaker, EC2, Coinbase, Stripe, Anthropic]
---

# AWS — Knowledge File

Structure governed by [`_schema.md`](./_schema.md). Do not hand-edit; all writes go through `updating-competitor-knowledge` and `linting-competitor-knowledge`.

## Rules

> Binding beliefs that shape decisions. Each rule is promoted from corroborated Facts. Act on these directly.

- **R1 — AWS is making the most aggressive multi-model bet of any hyperscaler.** *Promoted 2026-05-13 from F1, F3, F5.* *Reinforced 2026-05-13 by F13 (Anthropic deal).* Act on this when: positioning agent products against AWS — assume "single-vendor lock-in" framing fails; lead with portability, evaluation tooling, or vertical depth instead.
- **R2 — AWS positions diversification of its customer base as a defensive moat.** *Promoted 2026-05-13 from F6.* Act on this when: AWS earnings or capex commentary is the subject — Garman's investor framing will lean on revenue diversity vs. concentration; expect this language in any FY26 disclosure.

## Facts

> Confirmed, dated, sourced events. Grouped by category.

### Product

- **F1 — 2026-05-11** — AWS + OpenAI partnership announced May 4, 2026: GPT-5.5 and GPT-5.4 on Bedrock in preview, Codex on Bedrock, Bedrock Managed Agents powered by OpenAI (limited preview). [source](https://aws.amazon.com/blogs/aws/aws-weekly-roundup-whats-next-with-aws-2026-amazon-quick-openai-partnership-and-more-may-4-2026/) — confidence: high
- **F2 — 2026-05-11** — Amazon Quick GA on May 4, 2026 with Free and Plus consumer plans, no AWS account required; document/presentation/infographic generation; native integrations with Google Workspace, Zoom, Airtable, Dropbox, Microsoft Teams. [source](https://aws.amazon.com/blogs/aws/aws-weekly-roundup-whats-next-with-aws-2026-amazon-quick-openai-partnership-and-more-may-4-2026/) — confidence: high
- **F3 — 2026-05-11** — Amazon Connect expansion into 4 agentic AI solutions (supply chain, hiring, customer experience, plus contact center core) announced May 4, 2026. [source](https://aws.amazon.com/blogs/aws/aws-weekly-roundup-whats-next-with-aws-2026-amazon-quick-openai-partnership-and-more-may-4-2026/) — confidence: high
- **F4 — 2026-05-11** — Bedrock AgentCore payments launched May 11, 2026 — payment primitives for agent-driven commerce inside Bedrock. [source](https://aws.amazon.com/blogs/aws/aws-weekly-roundup-amazon-bedrock-agentcore-payments-agent-toolkit-for-aws-and-more-may-11-2026/) — confidence: high
- **F5 — 2026-05-11** — Agent Toolkit for AWS launched May 11, 2026 — production-ready suite for AI coding agents to build on AWS, no additional charge. [source](https://aws.amazon.com/blogs/aws/aws-weekly-roundup-amazon-bedrock-agentcore-payments-agent-toolkit-for-aws-and-more-may-11-2026/) — confidence: high
- **F9 — 2026-05-13** *(previously: H3, promoted 2026-05-13)* — Bedrock AgentCore Payments uses Coinbase CDP wallets and Stripe Privy wallets as connection partners; agents set session-level spending limits and transact autonomously. Available in us-east-1, us-west-2, eu-central-1, ap-southeast-2. [source](https://aws.amazon.com/about-aws/whats-new/2026/04/amazon-bedrock-agentcore-payments-preview/) — confidence: high
- **F10 — 2026-05-13** — Amazon Bedrock AgentCore now available in AWS GovCloud (US-West) for elevated-compliance workloads. [source](https://aws.amazon.com/about-aws/whats-new/2026/05/bedrock-agentcore-launch-aws-govcloud-us/) — confidence: high
- **F11 — 2026-05-13** — AgentCore Optimization launched: recommendations, batch evaluations, and A/B tests to complete the observe-evaluate-improve loop for production agents. [source](https://aws.amazon.com/blogs/aws/aws-weekly-roundup-amazon-bedrock-agentcore-payments-agent-toolkit-for-aws-and-more-may-11-2026/) — confidence: high

### Hiring

- **F7 — 2026-05-11** — Andy Jassy added Prasad Kalyanaraman (AWS infrastructure: data centers, networking, supply chain) to S-team and promoted Dave Brown (compute services / EC2 / Bedrock / SageMaker) to SVP in late April 2026. [source](https://www.geekwire.com/2026/amazon-names-aws-exec-prasad-kalyanaraman-to-s-team-promotes-dave-brown-to-svp/) — confidence: high

### Commentary

- **F6 — 2026-05-11** — Matt Garman (Fortune interview, 2026-04-29): "huge business opportunity" for AWS in agentic AI / SaaS; expects to be undersupplied for years; AWS demand more diversified than concentrated competitors. [source](https://fortune.com/2026/04/29/aws-ceo-matt-garman-interview-openai-saas/) — confidence: high

### Financials

- **F8 — 2026-05-11** — FY26 capex guidance ~$200B (+50% vs 2025); AWS revenue +20% to $128.7B in 2025; operating income $45.6B. [source](https://fortune.com/2026/04/29/aws-ceo-matt-garman-interview-openai-saas/) — confidence: high
- **F12 — 2026-05-13** — Q1 2026 (reported 2026-04-29): AWS segment revenue $37.6B (+28% YoY, fastest growth in 15 quarters); operating income $14.16B (+23% YoY, beat $12.84B consensus); annualized run rate $150B; AI services >$15B annualized run rate in their first three years; Q1 cash capex $43.2B. [source](https://www.cnbc.com/2026/04/29/aws-earnings-q1-2026.html) — confidence: high
- **F13 — 2026-05-13** — AWS backlog reached $364B as of Q1 2026, *explicitly excluding* a new Anthropic deal worth over $100B. [source](https://finance.yahoo.com/markets/stocks/articles/amazon-com-inc-amzn-q1-071842104.html) — confidence: high

## Hypotheses

> Best-guesses awaiting confirmation. Each carries a confidence level and the Fact (or contradicting Fact) that would promote or kill it.

- **H1 — Quick's Free tier (no AWS account required) is a top-of-funnel consumer-conversion play into paid AWS accounts.** *Confidence: low.* Promote on: published Quick → AWS account conversion metric, or AWS exec commentary tying Quick to AWS Free Tier signups. Kill on: Quick repositioned as standalone consumer product with no AWS-account path.
- **H2 — The OpenAI partnership is inference/hosting-only, not training.** *Confidence: medium.* Promote on: AWS or OpenAI confirmation that training workloads do NOT run on AWS. Kill on: any announcement of OpenAI training on Trainium or any AWS silicon.
- **H4 — AWS will surface AgentCore Optimization metrics (eval scores, A/B results) as a Bedrock differentiator vs. Vertex/Foundry.** *Confidence: low.* Promote on: marketing materials or earnings commentary that explicitly position evaluation tooling as a wedge against Google/Microsoft. Kill on: AgentCore Optimization framed as table-stakes infrastructure with no comparative positioning.

## Conflicts Pending Reconciliation

_No conflicts flagged this run._

## Pending (Overflow)

_No overflow this run._

---

## Change log

- **2026-05-13** — Restructured to Rules / Facts / Hypotheses per `_schema.md`. Source events preserved; reorganized only. Promoted R1 (multi-model bet) and R2 (diversification moat) from prior Positioning prose, with corroborating Fact IDs cited.
- **2026-05-13** — Ingest: 5 new Facts (F9–F13), 1 new Hypothesis (H4), 0 conflicts. R1 reinforced by F13 (Anthropic deal corroborates multi-model thesis).
- **2026-05-13** — Lint: promoted H3 → F9 (AgentCore Payments + Stripe interop confirmed by Coinbase/Stripe partnership announcement). 0 demotions, 0 stale flags.
