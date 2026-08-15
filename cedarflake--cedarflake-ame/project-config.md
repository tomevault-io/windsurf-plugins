---
trigger: always_on
description: Status: binding repository instructions
---

# Cedarflake Ame Agent Contract

Status: binding repository instructions

## 1. Purpose and document boundaries

This file defines durable project rules for agents and contributors. It owns:

- project-wide engineering standards;
- architecture and dependency boundaries;
- data, filesystem, and media safety rules;
- testing, verification, documentation, and Git discipline;
- constraints that must survive session changes and context compaction.

This file must not contain a product roadmap. Do not record milestones, feature order, schedules,
temporary priorities, completion percentages, active experiments, or current implementation status
here. Delivery plans belong in a separate roadmap document. Accepted technical decisions belong in
architecture decision records. Current implementation status is established from the working tree
and verification evidence.

The canonical active delivery plan is the repository-owned `docs/roadmap.md`.
This relative path is a discovery pointer, not roadmap content. Before planning a stage, reporting roadmap
status, resuming material product work after compaction, starting a new project session, or
delegating product work, read that file completely after recovering the latest relevant original
conversation. Do not create a competing roadmap copy. If the file is unavailable, report the exact
continuity gap before changing product scope or stage order. The roadmap remains lower authority
than the user's current instruction, this contract, accepted ADRs, and verified live implementation.

Tracked files refer to the two real large-library roots only as `local-primary` and
`cloud-primary`. Their machine-specific paths belong in the Git-ignored
`.agents/local-context.toml`, whose tracked shape is documented by
`.agents/local-context.example.toml`. When a task requires real-root discovery, read the local
mapping if it exists, but never copy its paths, account labels, or machine identity into tracked
files, test snapshots, logs intended for commit, commit messages, or user-facing documentation.
The mapping is discovery data only: its presence does not grant source mutation, cloud hydration,
or authorization for a new real-library acceptance run. If it is absent, retain the two logical
roots in planning and report that exact local execution is unavailable.

Do not turn a temporary implementation choice into a permanent rule in this file. Amend this
contract only when a project-wide constraint has genuinely changed.

## 2. Project context

Cedarflake Ame is a local-first desktop application for understanding and organizing very large
personal image libraries. It is intended to work safely with multiple local and cloud-backed
directories without requiring a second full copy of the source collection.

The project owns its product workflow, domain model, catalog, task orchestration, user decisions,
and presentation. Mature external libraries may provide specialized capabilities through adapters,
but Ame must remain maintainable when an engine or UI technology is replaced.

Original media is irreplaceable user data. Cataloging, browsing, and analysis must not modify it.
Move, copy, rename, recycle-bin, and delete capabilities remain in product scope as later, separate
workflows with explicit authorization, current-state revalidation, operation history, and recovery
safeguards where applicable. Convenience never justifies silently changing or downloading source
files.

## 3. Instruction and decision precedence

Apply instructions in this order:

1. the user's current explicit instruction;
2. this repository contract;
3. accepted architecture decision records under `docs/architecture/`;
4. repository-owned tool, formatter, linter, and language configuration;
5. the current task plan or roadmap;
6. general conventions.

Before changing code:

1. read this file completely;
2. inspect the working tree and preserve unrelated or user-owned changes;
3. read the architecture records that own the affected area;
4. inspect the live implementation instead of trusting an old status description;
5. state the smallest complete user-visible outcome being changed;
6. identify safety, migration, licensing, and performance risks.

If the user instruction, this contract, an architecture record, and the implementation disagree,
do not resolve a material conflict silently. Report the conflict before changing product scope,
data safety, licensing, or a stable architecture boundary.

### 3.1 Context compaction and continuity

After context compaction, a resumed task, or a new session, do not continue solely from memory, a
compressed summary, an old handoff, or an agent's previous narration. These sources are discovery
hints, not authoritative evidence of the user's latest intent or the implementation state.

Before resuming material work:

1. query the current task's most recent available conversation history using the provided task or
   thread-history tools;
2. identify the user's latest explicit decisions, corrections, rejected approaches, and unresolved
   questions from the original messages rather than relying on a paraphrased recollection;
3. inspect the live working tree, relevant architecture records, and verification results;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cedarflake/Cedarflake-Ame](https://github.com/Cedarflake/Cedarflake-Ame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
