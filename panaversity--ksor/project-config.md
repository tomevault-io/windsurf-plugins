---
trigger: always_on
description: The durable contract for working in this repository: what ksor is, the
---

# AGENTS.md

The durable contract for working in this repository: what ksor is, the
vocabulary, the decisions, the invariants, and how it is built and tested.
Loaded every session, so it holds **only what stays true** — what is true this
week lives in [`docs/status.md`](docs/status.md); the product pitch lives in
[`README.md`](README.md), its only home.

> CLAUDE.md is a symlink to this file. They are the same file: one contract for
> every agent — human-readable and agent-readable, like everything else here.

## Critical rules

1. **Never weaken provenance, citation, abstention, or governance guarantees to
   simplify an implementation.** They are the product, not features of it.
2. **Never push directly to `main`.** Every change lands through a pull request.
3. **Never break the agent-discoverable surfaces**: docs bundled in the npm
   package (`packages/ksor/docs/`), and — once the site ships — its `llms.txt`
   and `/.well-known/mcp/server.json`. Agents finding ksor is how ksor gets used.

## What this is, in one line

A CLI (`ksor` — the npm package is `@panaversity/ksor`) that compiles a folder
of governed markdown into two surfaces — a static website for people and an MCP
server for AI agents — with cited answers and honest abstention. It is not an
agent framework; it is the knowledge layer agent frameworks read from.

**Which verbs are implemented is not recorded here.** This file describes what
ksor _is_; `docs/status.md` holds what is built this week. One rule keeps the
CLI itself the current answer: an unimplemented verb says so and exits `2`, an
unknown word is refused with exit `1` — so no document has to be kept in step
with the binary.

A Python-era predecessor (vsor, `panaversity/zia-vsor-sdk`) proved much of the
design. Its work may be taken and converted to TypeScript (decision 6), but it
is a source to mine, not an authority to follow: nothing crosses without asking
what it was for, and converted code re-earns its place with tests here.

## What we claim, and to whom

Positioning, recorded because a session that re-derives it tends to describe
the machinery instead of the value:

- **A system of record is where the official version lives.** When the ledger
  and a spreadsheet disagree, the ledger wins. Businesses have had them for
  decades; **AI never did** — it answers from everything it has ever read,
  which is exactly why it cannot tell you which of its sentences were checked.
  KSoR is that record, for institutional knowledge.
- **Vendor-free is the ownership argument.** The agent surface speaks MCP, an
  open standard: one corpus will answer in any assistant, agent framework, or
  worker the owner writes. What a customer owns is the source; runtimes are
  interchangeable. Never position ksor as an integration with one assistant.
- **The interesting problem is not retrieval.** Chunking, embedding, and
  hybrid search are commodity. Whether an agent can be _trusted_ is decided by
  the governance of what it reads — provenance, something citable, and a
  measured floor under which it declines. Lead with that, not the pipeline.
- **Agents are the operator, not the audience for a manual.** The owner tells
  the coding agent they already use; scaffolded projects will therefore ship
  skills and rules as a product surface, not documentation.
- **Out of the box the owner is meant to touch knowledge only** — plain
  markdown, in any language they write in.

## Vocabulary

Used precisely; do not repurpose.

| Term                | Means                                                                                        |
| ------------------- | -------------------------------------------------------------------------------------------- |
| **corpus**          | the governed markdown under `knowledge/` — the source of truth                               |
| **instance**        | one deployment configured (`instance.md`): corpus, floors, budgets. **Not governance**       |
| **build**           | one execution of `ksor build`, identified by a `build_id`                                    |
| **generation**      | the monotonic version of published content — what a citation pins                            |
| **build.lock.json** | the committed record of a build: what was published, from which commit, with which toolchain |
| **surface**         | something that serves the corpus — the website and the MCP server                            |
| **scaffold**        | what `ksor init` writes into an adopter's repo — owned by the adopter (decision 4)           |
| **level**           | how much governance a project has climbed to, 0–4 — a ladder, not a gate                     |
| **abstain**         | the corpus does not cover this — a correct answer, never an error                            |

## Repository layout

| Path                        | What it is                                                 |
| --------------------------- | ---------------------------------------------------------- |
| `packages/ksor/`            | the published package: CLI + SDK (MCP surface lands here)  |
| `packages/ksor/docs/`       | user docs, shipped inside the npm tarball                  |
| `workbench/example-corpus/` | living KSoR fixture: dev target, test + eval surface       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panaversity/ksor](https://github.com/panaversity/ksor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
