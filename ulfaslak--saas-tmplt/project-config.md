---
trigger: always_on
description: > ⚠️ **Template placeholder.** Replace this block with a 1–3 sentence description of the project: what it is, who it's for, and where it lives. Be concrete — the description anchors every later decision.
---

# About

> ⚠️ **Template placeholder.** Replace this block with a 1–3 sentence description of the project: what it is, who it's for, and where it lives. Be concrete — the description anchors every later decision.
>
> Example: *"This project implements an AI compliance platform that monitors product updates (GitHub PRs, Linear tickets) and analyzes whether they impact legal documents. It is implemented as a single SvelteKit app (in `app/`), deployed to a VPS via Docker Compose. There is no separate backend."*

This project was bootstrapped from [`saas_tmplt`](https://github.com/ulfaslak/saas_tmplt) — see [TEMPLATE.md](TEMPLATE.md) for the full bootstrap checklist before deleting that file.

# Knowledge base

If this project has a sibling repo (or directory) with non-code context — meeting transcripts, decisions, CRM notes, playbooks, team info — name it here so future agents look there first when the human references "what we discussed in the call" or "the decision from last week." This is read-only context; agents should never modify files in that repo.

> ⚠️ **Template placeholder.** If you have no knowledge base, delete this section entirely.

# 🚨 The API keys in `.env` are not yours to spend

If the repo `.env` holds model-provider keys (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, …), **they bill the human's own metered accounts, uncapped.** Writing or running your own code that calls a model provider with one of them is **strictly prohibited unless the human explicitly asks for it in the current session.**

This covers any route you might take yourself: a throwaway script in the scratchpad, `curl` against a provider API, a new SDK client, or a loop that calls a model once per item. Size is not a defence — a 5-call smoke test is as prohibited as a 500-call sweep. It applies hardest to the shapes that look most legitimate: **benchmarks, evals, model comparisons, LLM-as-judge grading, batch extraction, backfills, and anything framed as "at scale."** Those multiply per-call cost by a number the human never agreed to.

**Allowed without asking:**

- **Subagents, the Task tool, or spawning `claude`** — those run on the human's subscription, not the metered API. This is the right way to do work at volume; reach for it whenever you're tempted to write an API loop.
- **The product's own code paths** exercised through normal development: running the app, a staging session, the test suites. That is the software doing its job.

**If a task appears to require direct API calls, stop before writing any of it** and tell the human what you would call, how many calls, and the estimated cost. Never spend first and report after. If they approve, hold to the agreed scope exactly — no extra retries, no widening the sample.

# DNA: architectural guardrails

`AGENTS/DNA/` contains the project's architectural guardrails — what the project *is*, its decisions, structure, and interface contracts. The DNA grows as the project does, but must never drift from the code. It evolves but doesn't change. Contributions that violate DNA cause cancer and must be avoided.

1. Don't violate DNA.
2. Grow DNA — when your work adds new structure, record it.
3. Don't let it drift — if something in DNA/ no longer matches the code, fix it. If it's unclear whether DNA or code should change, think deeply and resolve it only if you are certain, otherwise ask the human.
4. Do not take DNA changes lightly. If you make changes, you must have applied deep reasoning before doing so. Err on the side of asking the human before changing an existing DNA item.

## Reading DNA files

Read the DNA files relevant to your task — not all of them. All files are in `AGENTS/DNA/`.

**Each file holds one kind of statement.** This is what keeps a fact in exactly one place; a fact in two files drifts, and the copies disagree without anyone noticing.

| File | Holds | Test |
|---|---|---|
| [[DECISIONS]] | A **choice** among alternatives we could have made differently. "Postgres + Drizzle ORM." | Could someone violate it by choosing otherwise? Disagreeing means arguing with the human. |
| [[INVARIANTS]] | What must stay **true at runtime**, and the failure that taught it. | Could you write a test that fails when it stops being true? |
| [[ARCHITECTURE]] | **Where** code and data live — file structure, data models, module boundaries. | Could you verify it with `ls` or by opening the file? |
| [[PRODUCT]] | What the **customer** gets: vision, personas, the feature inventory. | Could a user observe it? |
| [[UI_SPEC]] | How the app is **laid out and behaves on screen** — layout, components, badges, route groups. | Would a designer recognise it as a convention? |
| [[DESIGN]] | Brand: colour, typography, spacing, voice and copy rules. | |
| [[DEVELOPMENT]] | How to **work on** the app: local setup, migrations, testing ideology, deploy and ops procedures. | Is it a thing you *do*, not a thing the app does? |
| [[DEVELOPMENT_SETUP]] | Provisioning the infrastructure from scratch: VPS, CI/CD, disaster recovery. | |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ulfaslak/saas_tmplt](https://github.com/ulfaslak/saas_tmplt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
