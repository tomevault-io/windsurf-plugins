---
trigger: always_on
description: > **Status (2026-06-22):** Current. Normative (operating contract for agents). Authoritative section numbering: §0-§4; invariants run §1.0-§1.12 (there is NO §1.13 — the regex/recognizer doctrine is §1.11 + §1.12, expanded in §2.4/§2.5). Other docs that cite "AGENTS.md §1.13" are stale against this file.
---

> **Status (2026-06-22):** Current. Normative (operating contract for agents). Authoritative section numbering: §0-§4; invariants run §1.0-§1.12 (there is NO §1.13 — the regex/recognizer doctrine is §1.11 + §1.12, expanded in §2.4/§2.5). Other docs that cite "AGENTS.md §1.13" are stale against this file.

# LawVM Agent Guide

LawVM treats legislation as an executable state-transition system: amendment acts are legal-language programs that replace, repeal, insert, renumber, move, delay commencement, restrict scope, and otherwise mutate a statute tree. LawVM compiles those instructions into typed operations, replays them over legal text structure, materializes point-in-time text, and emits an auditable account of how that text-state came to be — source facts, repairs, and remaining disagreement or uncertainty.

This is an operating contract for agents working in the repository, not background prose. It is organized as: (§0) the directive, (§1) what you must never do, (§2) what you must always do, (§3) how to verify, (§4) where to look. Each rule appears once; jurisdiction-specific detail lives in the pointed-to specs. Every legal-rule example here is deliberately jurisdiction-neutral — "the active frontend's parser", "a jurisdiction-local drafting idiom" — because the rules are universal even though most live corpora are not (CLI/quick-start examples in §4 are the exception, and name concrete jurisdictions on purpose).

---

## 0. Prime Directive

**Do not silently delete, mutate, reroute, widen, reorder, or invent legal state.**

Any repair that changes legal structure or text must be **owned**:

1. a stable rule/finding name;
2. a typed emission (observation, finding, source-pathology record, migration event, or failed op);
3. strict-mode rejectability;
4. a regression test with a witness;
5. a stated source witness or legal reason that makes it defensible.

A heuristic is allowed. An *invisible* heuristic is forbidden. If the system cannot prove the requested mutation is valid, **preserve the uncertainty** — emit a failure or unresolved finding. Never make the tree "look right" by guessing.

**Generators propose; typed validators authorize; replay consumes only authorized operations, events, and migration records.** A recovery heuristic may be rich and messy, but it only proposes candidates; a small, deterministic, auditable checker accepts them and emits the witness. Never fuse the accept/reject decision into the candidate generator — a bad candidate must remain rejectable. This is the discipline behind strict mode and the proof-carrying certificate.

**Evidence is not authority.** A long grinding loop must not turn local evidence, a candidate, or a report into implicit replay authority. The work object is a promotion chain — `source witness → candidate claim → execution-authorization status → dry-run/replay proof → agreement or adjudication row` — and every change states which link it strengthens. A finding earns the right to mutate replay only by climbing that chain through a deliberate proof boundary, never by accumulation.

**Success = source-faithful text-state, not oracle overlap.** The terminal product is a correct-by-construction consolidation: every output node traces to the operation and source instruction that produced it. The official consolidation is a *fallible* comparison surface, not the objective — a replay-vs-oracle similarity score is a regression guard, and maximizing it rewards deleting oracle-present state to match a possibly-wrong oracle. **Over-retention (failing to delete) is the safe wrong; over-repeal (destroying state) is the forbidden one.** Once a frontend closes the divergences it can deterministically resolve, every remaining one resolves to: a **deterministic gap** (LawVM is wrong — fix it), a **manual-compilation frontier** (the source does not deterministically specify the result — savings/exceptions, contingent commencement, point-in-time selection, cross-act placement, span-vs-enumeration ambiguity — needs an owned claim, not a guessed op), or **oracle-suspect** (LawVM is right, the official text is stale/editorial/wrong — a finding, not a failure). A saturated score is not a stop condition: when a frontend reaches its source-faithful frontier, high-value work *shifts* to invariants, classification, and cross-frontend harmonization — declare a specific question *resolved*, never a jurisdiction *done*.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliask/lawvm](https://github.com/eliask/lawvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
