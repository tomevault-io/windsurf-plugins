---
trigger: always_on
description: > Scope: `libs/naas-abi-marketplace/naas_abi_marketplace/domains/`.
---

# AGENT.md — How to file a module in the staff framework

> Scope: `libs/naas-abi-marketplace/naas_abi_marketplace/domains/`.
> This file explains the **decision logic**. For the structure itself, read [`README.md`](README.md).
> For the module index, read [`AGENTS.md`](AGENTS.md).

Filing a module takes two decisions, in this order:

1. **Which bucket?** — determined by the *organizational function* the module serves.
2. **Which component folder?** — determined by what the module *primarily is*.

Result: `domains/<bucket>/<component>/<module>/`.

---

## Decision 1 — Which bucket?

### The question to ask

> **What organizational function does this serve?**

Not *what technology does it use*, not *which department would own it*, not *who asked for it*.

A Postgres integration is not "signals because it is a database". A Postgres integration used to
run payroll queries serves **personnel**; used to serve the data platform it serves **signals**.
Technology is not function. If you find yourself filing by tech stack, you are answering the
wrong question.

### The nine questions

Each bucket answers exactly one question. Find the one your module answers:

| Bucket | The question it answers |
|---|---|
| `personnel` | *Who is in the organization, and what is their status?* |
| `intelligence` | *What is true about the world outside us?* |
| `operations` | *How do we execute the mission we are on right now?* |
| `logistics` | *What physical or contracted resources do we need, and where are they?* |
| `plans` | *What should we do next, and how will we get there?* |
| `signals` | *How does information move and get stored inside the organization?* |
| `training` | *How do our people become capable of their work?* |
| `finance` | *Where does the money go, and what is it worth?* |
| `external` | *How do we relate to people and bodies outside the organization?* |

### Tie-breakers

These are the ambiguities that actually come up.

**`operations` vs `plans` — the timeline test.**
Operations is *now*; plans is *next*. Executing this quarter's campaign is `operations`;
designing next year's is `plans`. If the module acts on committed work, it is operations. If it
produces a decision about work not yet committed, it is plans.

**`intelligence` vs `signals` — inside or outside.**
Intelligence is about the world *outside* the organization. Signals is about information moving
*inside* it. A competitor-monitoring module is intelligence; a document-ingestion pipeline that
serves every internal team is signals — even though both end up producing searchable text.

**`signals` vs `logistics` — information or materiel.**
S4 in the staff system is explicitly physical: materiel, transport, facilities, medical. If the
thing being moved or stored is *information*, it is signals. If it is *goods, money-as-assets, or
physical space*, it is logistics. This is why `document` is signals: it moves bytes, not boxes.

**`external` vs `operations` — audience or customer.**
If the counterparty is a paying customer in a commercial relationship, it is operations
(sales, success, support). If the counterparty is a broader audience, community, partner body or
public, it is external.

**`intelligence` vs `plans` — analysis or decision.**
Intelligence tells you what is true. Plans decides what to do about it. A module that measures
content performance is intelligence; a module that decides the next content calendar is plans.

**Cross-cutting modules.** If a module genuinely serves every bucket equally — an LLM provider, a
generic storage adapter — it does not belong in `domains/` at all. Those live at the marketplace
top level (`ai/`, `applications/`).

**Still stuck?** A module that plausibly fits two buckets is usually doing two jobs. Prefer
splitting it over guessing. If it cannot be split, file it under the bucket that owns its
*output*, not its input.

---

## Decision 2 — Which component folder?

### The rule: primary component

File the module under the component folder matching **what the module principally is**. Look at
what the module actually contains and pick the dominant one.

```
<bucket>/agents/        module whose deliverable is a conversational agent
<bucket>/apps/          module whose deliverable is a launchable web app
<bucket>/workflows/     module whose deliverable is multi-step automations
<bucket>/pipelines/     module whose deliverable is data processing
<bucket>/integrations/  module whose deliverable is a third-party API wrapper
<bucket>/ontologies/    module whose deliverable is vocabulary / RDF schema
```

**The module keeps its own internal structure.** Filing a module under `agents/` does not flatten
it. `finance/agents/accountant/` still contains `accountant/agents/`, `accountant/workflows/`,
`accountant/ontologies/` and `accountant/models/`. The outer folder says *what kind of module
this is*; the inner folders are the module's own shape, unchanged.

### Worked examples

**Single-component modules — unambiguous.**

`financial_cockpit` is an app-only module (P&L and treasury dashboard under `web/`).
→ `finance/apps/financial_cockpit/`

`organizations` contains only `ontologies/`. Nothing else.
→ `intelligence/ontologies/organizations/`

**Multi-component modules — pick the dominant one.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jupyter-naas/abi](https://github.com/jupyter-naas/abi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
