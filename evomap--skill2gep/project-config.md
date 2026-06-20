---
trigger: always_on
description: Generic skill distillation tool. Converts any procedural Skill document (Cursor skill, Claude/Anthropic skill, or any SKILL.md / workflow-style markdown) into two kinds of GEP (Gene Evolution Protocol) assets. Gene = compact strategy template (signals + strategy + AVOID + validation). Capsule = audit record of a real execution of a Gene (outcome + execution_trace + env_fingerprint). Capsule must be produced from "Gene + at least one real-world execution"; fabricating a Capsule from the document 
---


# skill2gep

A generic tool that is NOT tied to any specific agent platform. Input: any procedural Skill document (written for humans). Output: two kinds of GEP assets.

The essential distinction (per the GEP protocol definition):

| Asset | What it is | Where it comes from | Can be distilled directly from a document? |
|---|---|---|---|
| **Gene** | Reusable strategy template. Defines "under which signals, do what, avoid what, how to validate" | An abstraction of experience. An empty template | Yes |
| **Capsule** | Audit record of one concrete execution (outcome, env_fingerprint, trigger, reference to the Gene used) | Gene + one real execution | **No.** Must start from an existing Gene and run it once to produce a result |

This is a hard constraint. Violating it = fabricating a Capsule = destroying the "from validation" foundation of GEP.

## Core principles (non-negotiable)

1. **From validation only**. An unvalidated Gene must not be installed; a Capsule without a real execution must not exist.
2. **Genes serve control density**. Target 200-300 tokens, hard ceiling 500 tokens. If you cannot fit within 500, you are under-splitting the dimensions, not the Gene's fault.
3. **Capsules serve auditability**. Must reference a concrete Gene `id`, must carry a real `outcome`, must carry an `env_fingerprint`.
4. **Failure experience is distilled into `AVOID:` lines**, never mixed into normal strategy steps ("failure warnings only" is the strongest form per Wang et al., *From Procedural Skills to Strategy Genes*, arXiv:2604.15097).
5. **No hallucination**. Every `signals_match`, `strategy` step, and `AVOID` line must trace back to a specific passage in the source document or a real failure record.

## Workflow overview

```
Source Skill document
  |
  +-- Phase 1: Read and analyze the source
  +-- Phase 2: Recall / deduplicate (local + community)
  +-- Phase 3: Split by dimension, draft candidate Genes (strategy only)
  +-- Phase 4: Gene local validation (hard gate: schema + dry-run + scenario replay)
  +-- Phase 5: Install into local Gene pool + record_outcome
  |
  +-- [Path A] Gene only: stop here
  |
  +-- [Path B] Capsule needed (executable path + audit record):
  |     +-- Phase 6: Pick a real scenario + inject an accepted Gene + execute for real
  |     +-- Phase 7: Capture outcome / blast_radius / env_fingerprint / diff
  |     +-- Phase 8: Assemble the Capsule (type=Capsule, gene=<id>, outcome=...)
  |     +-- Phase 9: Validate the Capsule locally -> write to local Capsule pool
  |
  +-- [Optional] Phase 10: Publish to EvoMap community via evolver / EvoMap API
```

By default, produce both Gene and Capsule. Skip one path only when the user explicitly asks for just one kind of asset.

---

## Phase 1: Read and analyze the source Skill

Use any file-reading tool to load the Skill document the user points to. Common locations:

- Cursor skill: `~/.cursor/skills/<name>/SKILL.md` or `.cursor/skills/<name>/SKILL.md`
- Claude / Anthropic skill: project-root `SKILL.md` or `skills/<name>/SKILL.md`
- Any procedural document: an explicit path given by the user (a README "Workflow" section counts)

Extract the following material (**each item must trace back to a source line**, otherwise it is hallucination):

| Material | Which section of the document it comes from | Which Gene field it feeds |
|---|---|---|
| Trigger scenarios, keywords, error signals | Frontmatter `description`, body "Use when / Scenario / Trigger" | `signals_match` |
| Ordered executable steps | Workflow / Quick Start / flow diagram | `strategy` |
| Explicit pitfalls, anti-patterns, "do not ..." | Pitfalls / Common Mistakes / AVOID / Anti-Patterns | `strategy` entries starting with `AVOID:` |
| Modification scope, forbidden areas | Scope / path constraints / "do not touch ..." | `constraints.max_files`, `constraints.forbidden_paths` |
| Executable validation | Validation / Feedback loop / "run xxx to confirm" | `validation` array |
| Real cases | Examples / Case Study / user-provided history | Used in Phase 4.3 scenario replay and Phase 6 execution |

**If the source has no validation section**: Phase 4 will block. Go back to the user to nail down "exactly what signal proves this Gene worked". Do not paper over it with "manual check" or "looks reasonable".

**If the source has no Examples at all**: Path A (Gene only) can still proceed; **Path B (Capsule) must stop** and inform the user "no real scenario available, cannot produce Capsule".

---

## Phase 2: Recall / deduplicate (local + community)

Before drafting candidates, query three channels to avoid duplication. Using the `evomap-gep` MCP as the reference implementation:

```
MCP: evomap-gep / gep_list_genes        -> local gene pool
MCP: evomap-gep / gep_recall            -> historical experience

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvoMap/skill2gep](https://github.com/EvoMap/skill2gep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
