---
trigger: always_on
description: The durable contract for working in this repository: what DSoR is, the vocabulary,
---

# AGENTS.md

The durable contract for working in this repository: what DSoR is, the vocabulary,
the decisions, the invariants, and how it is built and tested. It is loaded every
session, so it holds **only what stays true**. What is true this week lives in
[`docs/status.md`](docs/status.md). The pitch for humans lives in
[`README.md`](README.md).

> `CLAUDE.md` imports this file with `@AGENTS.md`. One contract for every agent.
> This is an import and not a symlink on purpose: symlinks break when a student
> unzips or clones the repository on Windows (decision 8).

## Critical rules

1. **Never weaken a guarantee to simplify an implementation.** The security
   invariants in [§45](specs/dsor/06-conformance.md#45-security-invariants) are the
   product, not features of it. If a requirement is in your way, change the
   requirement in the open (`change-the-spec` skill), never around it.
2. **The agent is untrusted, and that includes you.** Nothing in DSoR may rely on a
   prompt, a tool description, or an agent's good behavior for enforcement. If a test
   only passes because the caller behaved, the test proves nothing.
3. **Spec and code move together.** A behavior change and the requirement it touches
   land in the same pull request, with `pnpm guard` green. No silent divergence in
   either direction.
4. **Never claim unbuilt behavior.** `docs/status.md` is the only authority on what is
   implemented. No present-tense sentence about code that does not exist, anywhere.
5. **Never push directly to `main`.** Every change lands through a pull request, and a
   human marks it ready.

## What this is, in one line

DSoR is the governed layer between an AI worker and a company's real systems: it
checks who is asking, under whose authority, against which rules and which current
state, gets a human's sign-off when the rules demand one, makes sure nothing happens
twice, and keeps the evidence.

Today this repository holds the **specification** (v1.4.0), its machine-readable half
(JSON Schemas, examples, requirement registry, all tested), and an empty
implementation package. The implementation is built in the five stages of
[`docs/learn/learning-path.md`](docs/learn/learning-path.md).

## What we claim, and to whom

- **DSoR is KSoR's companion.** KSoR (`panaversity/ksor`) is the record for what an
  organization *knows*. DSoR is the governed interface to what is operationally
  *true* and what may be *done*. A Digital FTE is KNOW + REMEMBER + REASON + STATE and
  ACT ([§3](specs/dsor/01-model.md#3-digital-fte-architecture)).
- **DSoR never takes the agent's word for anything.** It reads state itself, evaluates
  rules itself, and treats everything the agent supplies as a request, not a fact.
  Lead with that sentence, not with the machinery.
- **The readers are students and junior developers.** Every document opens in plain
  words before it states a rule (decision 7). Writing that only an expert can follow
  is a defect here, the same as a failing test.
- **Vendor-free.** The rules are normative; PostgreSQL, MCP, Better Auth, Graphiti,
  OpenViking, and KSoR are a replaceable reference profile
  ([§2](specs/dsor/01-model.md#2-normative-architecture-and-reference-profile)).

## Vocabulary

Used precisely; do not repurpose. Full glossary:
[§0.3](specs/dsor/00-conventions.md#03-glossary).

| Term | Means |
| --- | --- |
| **requirement** | One sentence in the spec with an id, a level, and exactly one MUST. The unit of conformance and of testing |
| **level** | L1 Core · L2 Autonomous · L3 Financial/Critical · RP (reference profile) · STACK (rules for software around DSoR) |
| **operation / contract** | A named query or command, and its machine-readable spec sheet |
| **control** | A versioned CEL rule that yields ALLOW, DENY, or REQUIRE\_\* |
| **delegation** | The permission slip from a human to an agent, held in DSoR's own store |
| **proposal** | The record of one command invocation and its state. Approvals attach to it |
| **control-plane store** | DSoR's own database: delegations, controls, proposals, approvals, idempotency records, intent records, counters, holds, audit |
| **intent record** | The durable note written *before* a side effect |
| **decision bundle** | The structured evidence for one decision. Never model reasoning |
| **stage** | One of the five build steps in `docs/learn/learning-path.md` |

## Repository layout

```text
AGENTS.md                 this file; CLAUDE.md imports it
README.md                 the pitch, for humans
specs/dsor/               THE SPECIFICATION (prose). Source of truth for every requirement
  README.md               index, how to read, section → file map
  00-conventions.md … 06-conformance.md, appendix-a-schemas.md, appendix-b-cel.md
packages/spec/            @panaversity/dsor-spec — the machine-readable half
  schemas/                normative JSON Schemas (draft 2020-12, URN ids)
  examples/               one validated instance per schema, from the running example
  requirements.json       GENERATED by `pnpm guard --write`; never edit by hand
  src/                    loaders, an exact-decimal money reference, and the tests
packages/dsor/            @panaversity/dsor — the reference implementation (not started)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panaversity/dsor](https://github.com/panaversity/dsor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
