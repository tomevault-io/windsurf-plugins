---
trigger: always_on
description: > **This is the canonical agent context file.** `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

> **This is the canonical agent context file.** `CLAUDE.md` is a symlink to this file.
> Cursor, Codex, Gemini CLI, and Copilot also read it (via their own discovery rules).
>
> Keep this file under ~200 lines. If you’re tempted to add to it, ask first whether
> the content belongs in `docs/`, `core/files/skills/`, or a subdirectory `AGENTS.md`.

## What this repo is

A Python package and template catalog for `llm-wiki-kit` — a kit for building
LLM-maintained markdown wikis (Karpathy’s LLM Wiki pattern, adapted), composed
from a common core plus a catalog of droppable primitives, configured by recipes.

The kit is **not** a wiki vault — it’s the tool that builds and maintains
vaults. End users pip-install it, then run `wiki init --recipe <name>` against
a folder to get a working Obsidian-compatible vault with skills, schemas, and
operations wired up.

The detailed map of what lives where is in [`docs/architecture/overview.md`](docs/architecture/overview.md). **Read it before exploring.** It will save you 20 minutes of grep.

## Keeping changes minimal

Scope each change precisely to the request.

- **Limit the diff to what the request requires — extra changes hide
  the real one from review.** If the request needs it — or would ship
  broken without it — it's in scope, even discoveries you make
  mid-implementation.
- **Add a flag or option only when a second caller actually needs to
  differ.** Today's one caller is enough to define the shape.
- **Add docstrings and types to code the change actually touches.**
  Leave nearby untouched code as it is.
- **Validate at boundaries the request crosses** (user input, external
  APIs). Trust internal callers and framework guarantees.
- **Inline a single-use operation.** Extract a helper once a second
  caller actually appears.

When you defer something out of this PR — unrelated find or same-area
cleanup — note it in the PR description with a one-line reason.

## Source of truth

For each kind of decision, there is exactly one place it lives:

|Question                                                         |Where it lives                                                                            |
|-----------------------------------------------------------------|------------------------------------------------------------------------------------------|
|What is this project, and what’s in/out of scope?                |`docs/CHARTER.md`                                                                         |
|Why did we choose X over Y?                                      |`docs/adr/` (Architecture Decision Records)                                               |
|What should we change, and how?                                  |`docs/rfc/` (Request For Comments)                                                        |
|What exactly does this primitive / command / skill do?           |`docs/specs/<thing>/spec.md`                                                              |
|How will we build it, step by step?                              |`docs/specs/<thing>/plan.md`                                                              |
|How is the kit’s own code organized today?                       |`docs/architecture/`                                                                      |
|Where is the kit going next?                                     |`docs/ROADMAP.md`                                                                         |
|How do users use the kit?                                        |`docs/guides/{tutorials,how-to,reference,explanation}/` (Diátaxis; some existing user docs still live flat at `docs/guides/*.md` and migrate gradually)|
|How does Claude do `<repeating task>` *inside a vault*?          |The `core/files/skills/<task>/SKILL.md` files we ship (these are vault-side, not kit-side)|
|How does Claude do `<repeating task>` *while developing the kit*?|This file plus `docs/CONVENTIONS.md`                                                      |

If you can’t find the answer in one of these places, **the answer doesn’t
exist yet** — ask, or open an RFC. Don’t guess. Lifecycle and mechanics
(living vs. frozen, ADR vs. RFC, etc.) live in `docs/CONVENTIONS.md`.

## Two scopes, one repo

A frequent source of confusion: this repo contains both **kit-development context**
(which agents read when working on the kit’s Python code, templates, and docs)
and **vault-side skills** (markdown files under `core/files/skills/` and
`templates/*/files/skills/` that get *copied into a user’s vault* and read by
*their* Claude session).

- Reading **this `AGENTS.md`**? You’re an agent working on the kit. Follow the
  workflow below. Don’t touch a user’s vault.
- Reading **`core/files/skills/wiki-conflict/SKILL.md`** in a real wiki? You’re
  helping a user resolve a conflict in their vault. That context is separate.

Never let the two leak into each other.

## Workflow

For anything beyond a one-line edit, follow the **plan → execute → verify →
review** loop. Summary:

1. **Plan before acting.** For anything spec-shaped, read the spec first.
   For architecturally significant work, use Plan Mode and the agent’s
   deepest-thinking setting. Phrase every plan task as a verifiable goal,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eugenelim/llm-wiki-kit](https://github.com/eugenelim/llm-wiki-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
