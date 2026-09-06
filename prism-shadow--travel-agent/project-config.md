---
trigger: always_on
description: Reason from first principles, end to end. When the evidence does not support a conclusion, say
---

# travel-agent — Agent Development Guide

## Core Reminder

Reason from first principles, end to end. When the evidence does not support a conclusion, say
"no basis found" instead of presenting an inference as a finding.

## Product Direction

travel-agent is an **open-source consumer travel application** built on **PenguinHarness** (the
agent engine) and **penguin-browser** (the visible in-app browser, plus an optional connection to
the user's own Chrome). It is the only place those two are joined. Its first-class object is the
**Trip**; the interaction it is judged on is one sentence → a few represented options, each with a
reason → a click that is also authorization → the form filled → **a stop at the payment page**.

The goal, the requirements that keep that sentence true, and the scope table of what this product
adopts and declines — with the reason for each — are the root of the spec graph:
**[`SPEC.md`](SPEC.md)**. Read it before proposing a feature; it is what settles whether the
feature belongs here at all.

## Hard Rules

1. **English is the repository's working language.** Everything you write is English — code,
   comments, commit messages, error and log output, test names and fixtures, package metadata, and
   every document under `docs/` and `tasks/`, and every spec. This is not a preference about style; a
   comment or design note that only some readers can parse is documentation that does not exist for
   the rest of them.

   Chinese appears only where it **is** the content, never where it *describes* the content:

   | Allowed | Because |
   | --- | --- |
   | zh i18n catalogs and fields — `strings.ts`, `i18n.ts`, `titleZh`, `short_description_zh` | The string *is* the product's Chinese UI |
   | `*.zh.md` documents (`README.zh.md`) | A deliberate translation, paired with an English original |
   | Test literals asserting zh output or CJK behavior | The literal is the thing under test |

   Some older documents under `docs/` are still Chinese and are being translated. Do not add to
   them in Chinese, and translate the file you are editing if the change is substantial.
2. **Every change leaves the specs true.** There is no changelog. **What changed** is git history —
   write the commit message as the entry: what changed, why, and what you ran to verify it.
   **What is true** is the spec graph, rooted at [`SPEC.md`](SPEC.md) — a change that alters a
   boundary, a contract or a decision updates the owning spec in the *same commit*, and one that
   settles a trade-off worth remembering adds a note under `docs/decisions/`. A spec left describing
   the old behaviour is part of the defect, not a follow-up.
3. **The engine baseline is pinned.** `packages/core` and `packages/server` are a hard-fork snapshot
   of PenguinHarness 0.2.2 (`d14be6f`). Do not merge upstream. Do not "improve" them opportunistically
   — changes there are a deliberate decision, not a side effect.
4. **The model judges; code only enforces.** Write enforcement code only where the model is itself
   inside the threat model. `@travel-agent/domain` was deleted for violating this: two of its three
   pieces were judgements a model makes better than a rule table, and had sat with no caller through
   six phases. The later `packages/transaction` experiment was retired for the same reason: its
   active interaction contract belongs to the server, browser handover belongs to browser-cli, and
   its payment execution machinery had no reachable production executor.
5. **PenguinHarness must not know penguin-browser exists.** The engine provides the agent runtime;
   penguin-browser provides browser control; travel-agent is the only place that joins them.
   Do not add a dependency in that direction.
6. **No silent fallback between browser backends.** The choice is per conversation and cannot change
   while a task runs. An unavailable persisted choice stays visible as unavailable; showing the other
   backend would be a false state.
7. **Payment stops at the gate.** The agent does not press the button that takes the money. The
   enforced production gate lives in `packages/browser-cli/src/executor/payment-gate.ts` and has no
   enable flag. When adding a surface that can click, wire it through the gate — *enumerate, do not
   sample* (`write-gate.ts` states the rule).
8. **Read the current file before editing it.** Keep changes scoped to the request; prefer existing
   repo patterns over a new abstraction.

## Repo Layout

```
packages/core, server        PenguinHarness engine baseline (pinned snapshot; @prismshadow/*)
packages/web                 The active consumer UI, shared by web and desktop
packages/desktop             Electron shell: in-app browser, vault, packaging
packages/browser-cli         penguin-browser: CLI, CDP relay, Playwright executor (vendored)
packages/browser-extension   Chrome extension bridging the relay to the user's Chrome (vendored)
packages/skills              Built-in skill library, incl. skills/penguin-browser
```

`browser-cli` and `browser-extension` are a snapshot of upstream `penguin-browser` at `ba9e13b`
(2026-08-12): upstream history stays in the upstream repo; post-import changes are in this
repository's git history.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Prism-Shadow/travel-agent](https://github.com/Prism-Shadow/travel-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
