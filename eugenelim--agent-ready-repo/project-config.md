---
trigger: always_on
description: > **This is the canonical agent context file.** `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

> **This is the canonical agent context file.** `CLAUDE.md` is a symlink to this file.
> Cursor, Codex, and Copilot also read it (via their own discovery rules); Gemini CLI
> reads it via the `context.fileName` bridge the `gemini` adapter writes to `.gemini/settings.json`.
>
> Keep this file under ~200 lines. If you're tempted to add to it, ask first whether
> the content belongs in `docs/`, a skill, or a subdirectory `AGENTS.md`.

## What this repo is

<!-- ONE sentence. Replace this. -->
A monorepo for `<project-name>` — a `<one-line description of what it does and for whom>`.

The detailed map of what lives where is in [`docs/architecture/overview.md`](docs/architecture/overview.md).
**Read it before exploring.** It will save you 20 minutes of grep.

## Keeping changes minimal

Code is a liability, not an asset; the same principle unifies *Add a flag only when a second caller actually needs to differ* (next bullet) and *Dependencies are forever* in [§ Check before acting](#check-before-acting).

Scope each change precisely to the request.

### Non-negotiables

- **Surface assumptions before building.** Name them in PLAN's trio.
  The declined-pattern register in the `work-loop` skill
  names temptations; assumptions are different — call them out separately.
- **Stop and ask when requirements conflict.** Use the Surface verb
  defined in the `work-loop` skill — emit a
  short description and wait.
- **Push back when warranted.** Not a yes-machine. Disagreement goes in
  the PR description, not in silence.
- **Prefer the boring, obvious solution.** Cleverness is expensive; see
  the declined-pattern register in the `work-loop` skill.
- **Touch only what you're asked to touch.** See the rest of this section.

- **Limit the diff to what the request requires — extra changes hide
  the real one from review.** If the request needs it — or would ship
  broken without it — it's in scope, even discoveries you make
  mid-implementation.
- **Add a flag or option only when a second caller actually needs to
  differ.** Today's one caller is enough to define the shape.
- **Add docstrings and types to code the change actually touches.**
  Leave nearby untouched code as it is — except under the
  bundled-fixes carve-out defined in the `work-loop` skill (same-area,
  same-concern, mechanical ride-alongs only; surplus still goes to
  follow-up).
- **Validate at boundaries the request crosses** (user input, external
  APIs). Trust internal callers and framework guarantees.
- **Inline a single-use operation.** Extract a helper once a second
  caller actually appears.

When you defer something out of this PR — unrelated find or same-area
cleanup — note it in the PR description with a one-line reason.

## Source of truth

For each kind of decision, there is exactly one place it lives:

| Question                                  | Where it lives                       |
| ----------------------------------------- | ------------------------------------ |
| What is this project, and what's in/out of scope? | `docs/CHARTER.md`             |
| Why did we choose X over Y?               | `docs/adr/`     (Architecture Decision Records) |
| What should we change, and how?           | `docs/rfc/`     (Request For Comments) |
| What exactly does this feature do?        | `docs/specs/<feature>/spec.md`       |
| How will we build it, step by step?       | `docs/specs/<feature>/plan.md`       |
| How is the code organized today?          | `docs/architecture/`                 |
| What is the product doing today?          | `docs/product/` (roadmap, changelog) |
| How do users use the product?             | `docs/guides/` (Diátaxis: tutorials, how-to, reference, explanation) |
| How do agents do `<repeating task>`?      | A skill file (`SKILL.md` with frontmatter); your IDE handles discovery |

If you can't find the answer in one of these places, **the answer doesn't
exist yet** — ask, or open an RFC. Don't guess. Lifecycle and mechanics
(living vs. frozen, ADR vs. RFC, etc.) live in
[`docs/CONVENTIONS.md`](docs/CONVENTIONS.md).

## How we work

For anything beyond a one-line edit, follow the **plan → execute → verify →
review** loop. The mechanics — verification modes, gate sequence, iteration
cap, capture-learnings, specialist-reviewer pass — live in the
`work-loop` skill. Load it before
non-trivial work; that is the canonical source for *how* the loop runs.
[`docs/CONVENTIONS.md`](docs/CONVENTIONS.md#how-we-do-non-trivial-work)
covers the *why*. Commits follow Conventional Commits — format and footer
rules are in [`CONVENTIONS.md § Commits`](docs/CONVENTIONS.md#commits).

`work-loop` runs in **light mode** by default — a lean inline spec, a
single bounded adversarial pass, no state machine — and escalates to
**full mode** when the work trips a risk trigger:

<!-- risk-triggers:start — canonical wording lives here; copied verbatim
     into AGENTS.md, packs/core/seeds/AGENTS.md, and docs/CONVENTIONS.md.
     Keep all four byte-identical (grep-equality is an acceptance
     criterion of the work-loop-light-mode spec). -->
**Risk triggers — any one routes the work to full mode:**

- **Unfamiliar** — territory you don't know well.
- **Multi-person** — more than one person builds or reviews it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eugenelim/agent-ready-repo](https://github.com/eugenelim/agent-ready-repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
