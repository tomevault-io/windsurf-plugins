---
trigger: always_on
description: This project uses a spec-driven development workflow. Follow these rules in all interactions.
---

# Spec-Driven Development (SDD)

This project uses a spec-driven development workflow. Follow these rules in all interactions.

> **Command files**: `.cursor/commands/` is the canonical source for all slash commands. `.claude/commands/` is generated from it by `./scripts/sync-commands.sh` (real copies, not symlinks — Claude Code's command discovery doesn't reliably follow symlinks). To change a command, edit it in `.cursor/commands/` and run the sync script. Never edit `.claude/commands/` directly.

## Core Principle

**Spec before code.** Define behavior before implementing it.

```
Project setup (once):
  /strategy → /vision → /personas → /constitution → /design-tokens

GTM pipeline (from strategy):
  /strategy (Phase 5: GTM sketch) → /gtm (full playbook) → /find-early-users (prospect list)

Finding product-market fit (iterate before building):
  /strategy → /gtm → /find-early-users → talk to people → /strategy (update) → repeat
      ↓ (once stable)
  /vision → /personas → /roadmap → /build-next

Per feature (Red-Green-Refactor TDD):
  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
  │    SPEC      │ ──▶ │  RED (test)  │ ──▶ │ GREEN (impl) │ ──▶ │  REFACTOR    │
  │ (Gherkin +   │     │  (failing)   │     │ (until tests │     │ (clean up,   │
  │  mockup +    │     │              │     │  pass)       │     │  tests must  │
  │  persona     │     │              │     │              │     │  still pass) │
  │  revision)   │     │              │     │              │     │              │
  └──────┬───────┘     └──────────────┘     └──────┬───────┘     └──────┬───────┘
         │                                         │                     │
      [PAUSE]                                      ▼                     ▼
    user approves                           ┌──────────────┐     ┌──────────────┐
    then /tdd                               │ DRIFT CHECK  │     │ DRIFT CHECK  │
                                            │ (layer 1)    │     │ (layer 1b)   │
                                            └──────────────┘     └──────┬───────┘
                                                                        │
                                                                        ▼
                                                                 ┌──────────────┐
                                                                 │  COMPOUND    │
                                                                 │ (learnings)  │
                                                                 └──────────────┘
```

---

## Command Triggers

When the user says any of these phrases, **automatically invoke `/spec-first`**:

| User says | Action |
|-----------|--------|
| "spec first" | Run `/spec-first {feature}` |
| "spec-first" | Run `/spec-first {feature}` |
| "write a spec for" | Run `/spec-first {feature}` |
| "create a spec" | Run `/spec-first {feature}` |
| "spec this out" | Run `/spec-first {feature}` |
| "spec out" | Run `/spec-first {feature}` |
| "plan this feature" | Run `/spec-first {feature}` |
| "write the spec" | Run `/spec-first {feature}` |
| "create spec" | Run `/spec-first {feature}` |
| "update the spec for" | Run `/spec-first {feature}` (update mode) |
| "update spec" | Run `/spec-first {feature}` (update mode) |

When the user says any of these after a spec is shown, **invoke `/tdd`**:

| User says | Action |
|-----------|--------|
| "tdd" | Run `/tdd {feature}` |
| "go ahead" | Run `/tdd` with current spec |
| "build it" | Run `/tdd` with current spec |
| "implement it" | Run `/tdd` with current spec |
| "ship it" | Run `/tdd` with current spec |

Extract the feature description from the rest of their message.

When the user says any of these, **invoke the corresponding ralph/utility command**:

| User says | Action |
|-----------|--------|
| "ralph setup", "set up ralph", "configure ralph" | Run `/ralph-setup` |
| "ralph run", "run ralph", "start the loop", "run the loop" | Run `/ralph-run` |
| "clean slate", "kill everything", "restart everything", "nuke localhost" | Run `/clean-slate` |
| "generate guide", "update guide", "how to use guide" | Run `/guide` |

When the user says any of these, **invoke `/strategy`**:

| User says | Action |
|-----------|--------|
| "strategy" | Run `/strategy` |
| "product strategy" | Run `/strategy` |
| "business strategy" | Run `/strategy` |
| "shape this" | Run `/strategy` |
| "who are we selling to" | Run `/strategy` |
| "business model" | Run `/strategy` |

When the user says any of these, **invoke `/gtm`**:

| User says | Action |
|-----------|--------|
| "gtm playbook" | Run `/gtm` |
| "marketing plan" | Run `/gtm` |
| "how do we get users" | Run `/gtm` |
| "distribution plan" | Run `/gtm` |
| "growth plan" | Run `/gtm` |
| "channel strategy" | Run `/gtm` |
| "launch plan" | Run `/gtm` |
| "outreach plan" | Run `/gtm` |

When the user says any of these, **invoke `/find-early-users`**:

| User says | Action |
|-----------|--------|
| "find early users" | Run `/find-early-users` |
| "find users" | Run `/find-early-users` |
| "find prospects" | Run `/find-early-users` |
| "find people" | Run `/find-early-users` |
| "who should I talk to" | Run `/find-early-users` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdrianRogowski/auto-sdd](https://github.com/AdrianRogowski/auto-sdd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
