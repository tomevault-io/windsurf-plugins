---
trigger: always_on
description: Cross-compatible Claude Code and Codex skills that automate Beancount workflows for [Beancount.io](https://beancount.io/) users (see root `CLAUDE.md` for repo-wide rules).
---

# Beancount Agent Skills

Cross-compatible Claude Code and Codex skills that automate Beancount workflows for [Beancount.io](https://beancount.io/) users (see root `CLAUDE.md` for repo-wide rules).

A skill is a `SKILL.md` instruction package with optional references, evals, scripts, and agent metadata. The canonical implementations live here under `.claude/skills/`; the repository-root `.claude/skills` and `.agents/skills` symlinks expose the same tree to Claude Code and Codex respectively. Do not create platform-specific copies.

## Layout

```
skills/
  .claude/
    skills/
      beancount-ask/          Answer ledger questions with shown, re-runnable BQL (read-only)
      beancount-close/        Month-end close ritual: reconcile all, assert, report, commit
      beancount-import/       Import a bank/card export (CSV/OFX/QIF) as categorized, deduplicated entries
      beancount-importer-author/  Write/repair tested beangulp importers from a sample file
      beancount-init/         Scaffold a new beancount + fava ledger repo
        SKILL.md
      beancount-migrate/      Migrate Mint/Monarch/QBO export history into a fresh ledger
      beancount-options/      Convert natural-language options trades into beancount transactions
        SKILL.md
        references/           Per-strategy guidance loaded on demand
        evals/                Test prompts + fixtures for skill-creator iteration
      beancount-reconcile/    Reconcile one account against a bank/broker statement
        SKILL.md
        references/           Statement-format + matching guidance loaded on demand
        evals/                Statement+ledger fixtures per mismatch class
      mermaid/                Draw syntax-verified Mermaid architecture diagrams
        SKILL.md
      pm/                     /pm — arrange the public .pm adoption board (the only writer to .pm/)
      pm-brainstorm/          /pm-brainstorm — propose .pm milestones and tasks as text
      loop-worker/            /loop-worker — drain a .pm workstream milestone by milestone
      routine-shared/         Shared contract for the routine-* maintenance suite (not a skill)
        contract.md
      routine-logic-simplifier/        /routine-logic-simplifier — simplify convoluted logic, behavior-preserving
      routine-logic-bugfixer/          /routine-logic-bugfixer — model tricky logic, fix provable bugs
      routine-dup-unifier/             /routine-dup-unifier — merge duplicated implementations within a package
      routine-dead-code-removal/       /routine-dead-code-removal — delete provably unreachable code
      routine-useless-test-pruner/     /routine-useless-test-pruner — delete tests that cannot fail
      routine-shipped-feature-inliner/ /routine-shipped-feature-inliner — remove gates for fully shipped features
      routine-flaky-test-fixer/        /routine-flaky-test-fixer — root-cause flaky CI tests
      routine-abstraction-improver/    /routine-abstraction-improver — flatten over-engineered indirection
      routine-abstraction-police/      /routine-abstraction-police — fix stated-boundary import violations
      ship/                   /ship — pull --rebase, commit, push main
        agents/               Codex interface metadata (openai.yaml)
  tmp/                        Scratch space — gitignored, safe for experiments
```

Most stateful `beancount-*` skills have `references/` and `evals/`; small skills such as `beancount-init` may be self-contained. Mutating ledger workflows share the applicable trust rails: propose-then-confirm before writes, categorization restricted to existing accounts, `bean-check` as the verification gate, and the `import-id` convention from `.claude/skills/beancount-import/references/dedup.md` for externally sourced entries. Read-only skills do not pretend to have a write/confirmation phase. Every `routine-*` skill reads `.claude/skills/routine-shared/contract.md` as its first step — the suite's shared preconditions, verify gates, ship protocol, and STOPs live there, not in the individual skills.

## Skills

| Skill                       | Purpose                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stargately/beancount-io](https://github.com/stargately/beancount-io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
