---
trigger: always_on
description: Sneeze project context — MBE engine (static lib) consumed by a host metaverse browser application
---


# OMBI — Open Metaverse Browser Initiative

> **This file is a lightweight project overview.** Detailed module documentation lives in per-module `*.md` files alongside the source code. When you need to understand a module, read its `.md` file — not this file. This file covers project-wide concerns: what the project is, how to write code for it, and where to find things.

### How to maintain `*.md` documentation

All `*.md` files (this one and per-module docs) describe **what the system is right now** and **how to use it**. They are reference documents, not session logs.

- **Rewrite, don't append.** When code changes affect a module's shape, rewrite the affected section from scratch. Do not add paragraphs explaining what changed or why.
- **Delete stale content.** If a class was renamed, the old name must not appear. If a method was removed, remove it from the doc. If something no longer exists, remove it entirely.
- **No revision history.** Do not record what was deleted, what used to exist, what was refactored, or the history of decisions. The doc describes the current state only.
- **Watch for the drift pattern.** The natural tendency is to append context after each change until the doc becomes an archaeological record. Resist this. After making code changes, re-read the `.md` and ask: "Does every line describe something that exists right now?" Delete everything that doesn't.

### The `docs/` wiki (the public reference manual)

`docs/` is a **separate body of documentation** from the per-module `src/**/*.md` files described above. It is a cohesive, navigable reference manual written for newcomers — evaluators, integrators, and contributors coming in cold — and is intended to be **published to the project wiki**. Do not confuse the two:

- **`src/**/*.md`** — terse, per-module reference notes living beside the code. Treated as *unverified hints* (see below).
- **`docs/**/*.md`** — the curated wiki. Prose-first, organized into five tiers, written from the code itself.

**Authoring is AI-maintained, source-of-truth is the code.** These pages are written and kept current primarily by AI coding agents working directly from the source tree. The **single source of truth is the code** (`include/*.h` + the current `src/` implementation). The `src/**/*.md` notes, `OMB_Architecture_*.md`, and this `project.mdc` are *unverified hints* — when a hint and the code disagree, the code is right. Every page is written by reading the code, not by paraphrasing other docs.

**Structure.** Five tiers, each a folder under `docs/` with an `index.md`:
- **Overview** — what the OMB is, core vocabulary, the open standards Sneeze builds on.
- **Architecture** — engine/host split, lifecycle, fabric loading, threading, trust & isolation, coding conventions.
- **Systems** — one page per subsystem (engine, control, context, container, scene, network, storage, console, viewport, msf, persona, wasm, spirv, compute, xr, ui).
- **API** — one folder per public header in `include/`, one page per class.
- **Guides** — embedding, building, contributing.

`docs/Home.md` is the landing page. `docs/STYLE.md` is the **authoring contract** (tiers, page template, front-matter schema, voice, the rule that the wiki never names a specific browser product or any application that embeds Sneeze — "the engine" / "a host application" only). Read `STYLE.md` before writing or editing any wiki page.

**Front matter links each page to the code.** Every page begins with YAML front matter; two fields make the doc-to-code dependency explicit and checkable:

```yaml
sources:
  - include/Scene.h
  - src/context/scene/Scene.cpp
verified: <commit-sha>
```

- **`sources`** — the repo-relative code files the page documents (hand-maintained).
- **`verified`** — the commit the page was last checked against. Set this to the current `HEAD` when you write or re-verify a page.

**Drift detection — `tools/DocDrift/`.** `tools/DocDrift/docdrift.py` (Python 3 stdlib only, read-only) is the mechanical half of the maintenance loop. For each page it runs `git log <verified>..HEAD -- <sources>` and flags any page whose listed sources changed since it was verified (or whose source was deleted). It never edits docs. Flags: `--strict` (exit non-zero on drift; default is warn-only), `--quiet`, `--docs DIR`. See `tools/DocDrift/README.md`.

**The maintenance loop:** (1) run DocDrift; (2) open each flagged page and compare against current code — code wins; (3) fix the page and bump its `verified` to the current `HEAD`.

**Known limitation:** `sources` is hand-maintained, so DocDrift catches changes to files a page *lists*, not coverage a page *forgot* to list. When you add a source file to a subsystem, add it to the relevant page's `sources`.

**Automation / CI (the next step — for whoever owns publishing).** Two pieces are meant to live in CI:
1. **Drift check** — run `python tools/DocDrift/docdrift.py` as a **warn-only** check (no `--strict`) so drift is surfaced on a pull request without blocking the merge. Promote to `--strict` later if the team wants a hard gate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MetaversalCorp/Sneeze](https://github.com/MetaversalCorp/Sneeze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
