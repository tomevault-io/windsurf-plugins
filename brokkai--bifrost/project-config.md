---
trigger: always_on
description: Always respond using ASD-STE100 Simplified Technical English. It is a controlled writing standard. Aerospace and defense groups made it. It helps people write clear technical text.
---

# ASD-STE100 Simplified Technical English

Always respond using ASD-STE100 Simplified Technical English. It is a controlled writing standard. Aerospace and defense groups made it. It helps people write clear technical text.

Key rules:
- **Use approved words only.** The standard gives a word list. Each word has one meaning.
- **Use one word for one idea.** Do not use two words for the same thing.
- **Write short sentences.** Use 20 words or less for instructions.
- **Use active voice.** Write "Turn the switch", not "The switch must be turned".
- **Write short paragraphs.** Keep one topic in each paragraph.

# ExecPlans

Use an ExecPlan for a complex feature or a significant refactor. Follow `.agents/PLANS.md` from design through implementation.

Use `.agents/` as the only repository namespace for planning and design artifacts that agents own. Do not create `.agent/`.

Store each ExecPlan in `.agents/plans/`.

Keep `.agents/PLANS.md` as the standard for ExecPlans. Do not store individual ExecPlans next to `.agents/PLANS.md`.

Store design notes for LLMs or agents in `.agents/docs/`. These notes can include agent context, publication runbooks, parity notes, and similar internal information. Do not publish these notes as product documentation.

Reserve `docs/` for future documentation for human readers. Do not store ExecPlans, agent runbooks, or LLM-only context in `docs/`.

# Git / version control

Commit directly to the current branch. This rule also applies when the current branch is `master`.

Do not create a branch, change branches, rebase, or open a pull request unless the user gives an explicit instruction.

Do not run `git checkout -b`.

The instruction "commit" means that you must commit on the current branch. It does not mean that you must create a branch first. This rule overrides other default branch procedures.

Stage and commit only the files that you changed. Do not run `git add -A`. Do not include unrelated working-tree changes in the commit.

# Expectations

Continue when there is a clear next step toward the goal. This rule applies inside and outside an ExecPlan. Do not stop to ask for approval.

If you made material progress, first make a multiline checkpoint commit. Explain the work to this point. Give detailed reasons for the changes. The diff shows the changes themselves.

# Release tasks

For release preparation, tagging, publication, recovery, and version policy, follow the canonical [Release Process](CONTRIBUTING.md#release-process) and [Version Policy](CONTRIBUTING.md#version-policy) in `CONTRIBUTING.md`. A release task still requires explicit user authorization for version changes, tags, publication, or deployment.

# Crate dependency boundaries

Do not create a new workspace crate only to reorganize code. Create one only
when a clear dependency, compilation, publication, or ownership boundary
requires it. Record the reason in the change that adds the crate.

When a change adds a publishable crate, update the release crate inventory in
`CONTRIBUTING.md`. Bootstrap the crate on crates.io before the next version
release. Configure its trusted publisher at the same time.

## Do not reintroduce the nlp dependency stack into brokk-bifrost-analysis

Issue #1548 prevents a change to the `nlp` feature from invalidating the largest workspace compilation unit.

`scripts/check-workspace-dependencies.mjs` enforces this rule. It prohibits these dependencies in `brokk-bifrost-analysis` and `brokk-bifrost-core`:

- `hf-hub`
- `tokenizers`
- `fastrq`

If a change needs one of these dependencies, put the code in `brokk-bifrost-nlp`. Do not relax the dependency check.

## Keep brokk-bifrost-core at the bottom of the graph

Issue #1549 prevents the analyzer model layer from being recompiled as part of the largest workspace unit.

To keep this result, `brokk-bifrost-core` must not depend on another Bifrost crate.

`scripts/check-workspace-dependencies.mjs` gives `brokk-bifrost-core` an empty allowed-dependency set. Its unit test rejects a `core -> analysis` dependency.

Put code in `brokk-bifrost-analysis` when the code needs one or more of these items:

- An `IAnalyzer`
- A store
- A grammar
- A language module

Do not move such code to `brokk-bifrost-core`, even when the move appears convenient.

`analyzer/capabilities.rs` no longer illustrates this rule. The file moved to `crates/bifrost-core/src/analyzer/capabilities.rs` when the nine language crates were split out. The move is correct: the capability traits name only core types, and every language crate implements them, so leaving the traits in analysis would have made each language crate depend on the largest compilation unit. The parts that are generic over `IAnalyzer` -- `TypeHierarchyProvider::get_polymorphic_matches` and `build_direct_descendant_index` -- stayed in `brokk-bifrost-analysis`, which is what the rule above actually requires.

Read the rule as stated, not through that example. What must stay in analysis is code that needs an `IAnalyzer`, a store, a grammar, or a language module. A trait definition that needs none of them belongs in core.

# Analyzer Test Guidance


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrokkAi/bifrost](https://github.com/BrokkAi/bifrost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
