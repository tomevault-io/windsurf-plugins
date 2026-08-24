---
trigger: always_on
description: A skill that documents last week's contract is worse than no skill: it is trusted.
---

# Reskinnable Demo — architecture

One Next.js app whose **entire** experience — brand, theme, layout, pages,
tools, and agent — is reskinnable at runtime. A skin-agnostic **shell** hosts
one **skin** per route segment `/[skin]/...`. It ships five skins — `banking`,
`airline`, `logistics`, `keel` and `people` — switchable from a dropdown at the top of the
assistant column, plus a repo-local **reskin skill**
(`.claude/skills/reskin/`) for authoring new ones.

The point of the app is the `Skin` contract: a single interface that swaps a
whole product without the shell knowing anything domain-specific. The skins
deliberately sit on **two different data substrates** — banking, logistics and
people are REST-backed, airline and keel are in-memory — to prove the contract
is substrate-agnostic.

**Each skin is also a live sales demo.** It exists to prove CopilotKit and
Intelligence top to bottom to an enterprise buyer, through a fixed set of demo
**beats**: lead with generative UI, show that threads store AG-UI streams rather
than text, manipulate the app four ways (drive it, read the screen, navigate via
real levers, ingest a document into a durable artifact), recall long-term memory,
replay a stored procedure, and learn a new one on stage. `banking` is the original
reference implementation; `people` is the second skin built to hit every beat. The
beats, and what each one must prove, are specified in
[`.claude/skills/reskin/demo-beats.md`](.claude/skills/reskin/demo-beats.md) —
read it before adding or changing a skin's tools, prompt or suggestion pills,
because a skin that wires the contract perfectly and hits no beats is a failed
skin.

## ⚠️ Changing existing code? Review whether the reskin skill went stale

**Every change to existing code in this app ends with one explicit question,
answered out loud before the work is called done:**

> Does this change make anything in `.claude/skills/reskin/` wrong, incomplete,
> or misleading for the next person authoring a skin?

Answer it in the PR description or the commit body — "checked, no skill impact" is
a fine answer. An UNANSWERED question is the failure; a considered "no" is not.

**Why this is a standing rule rather than a nice-to-have.** The skill is the only
instruction a new skin's author reads, and it goes stale SILENTLY — nothing
type-checks it, no test imports it, and a skin built from a stale template still
compiles, lints and renders. Every one of these actually happened while shipping
the LOCK_SKIN root-serving change:

- `templates.md` handed every new skin the exact two patterns that change had just
  removed (a hardcoded `/${skin.id}/…` href and a fixed `pathname.split("/").slice(2)`).
  Both fail **silently** under a lock — the page still renders, the URL is just wrong.
- SKILL.md's verification steps pointed at `pnpm test:unit` and a drift test that the
  same PR **deleted**. Caught by a reviewer, not by any tooling.
- The authoring half of the skill was updated and the VERIFICATION half was not; the
  gap survived until someone asked about it specifically.

**Changes that implicate the skill** — treat these as automatic triggers, not a
judgement call:

| You changed                                                                  | Check                                                                |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| The `Skin` contract (`src/shell/skin-contract.ts`)                           | SKILL.md's field-by-field table, templates.md's scaffolds            |
| Anything a skin must call or must not call (link builders, hooks, providers) | SKILL.md's contract sections, every template that shows the old form |
| A lint rule, test or gate a skin has to pass                                 | SKILL.md § Verification — does it name the right command?            |
| Registration, routing, or the client/server boundary                         | SKILL.md § Registration + the boundary section                       |
| A demo beat's mechanism, or what a beat must prove                           | demo-beats.md                                                        |
| A skin's brand, id or identity                                               | the skin lists in SKILL.md, CLAUDE.md and README.md                  |
| Deleting or renaming a file the skill references                             | grep the skill for the old path                                      |

**The cheap check, ~2 minutes:**

```bash
# 1. Does the skill still reference anything you deleted or renamed?
grep -rn "<old-symbol-or-path>" .claude/skills/reskin/

# 2. Do the templates still teach the pattern you just replaced?
grep -rn "<the-old-pattern>" .claude/skills/reskin/templates.md

# 3. Does SKILL.md § Verification still name commands that exist and gates that run?
```

If the change is load-bearing for skin authors, update the skill **in the same PR**.
A skill that documents last week's contract is worse than no skill: it is trusted.

## Shell vs skins

- **Shell** (`src/shell/`) — skin-agnostic host. Owns the `Skin` contract, the
  client + server registries, routing/provider composition, the inset frame

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CopilotKit/CopilotKit](https://github.com/CopilotKit/CopilotKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
