---
trigger: always_on
description: **Nx monorepo** for building **framework-agnostic component libraries** using **pure TypeScript web components** (custom elements with `tbw-` prefix). Components work natively in vanilla JS, React, Vue, Angular without wrappers.
---

# Copilot Instructions for Toolbox Web

## Project Overview

**Nx monorepo** for building **framework-agnostic component libraries** using **pure TypeScript web components** (custom elements with `tbw-` prefix). Components work natively in vanilla JS, React, Vue, Angular without wrappers.

**Toolchain:** Bun (package manager/runtime) · Nx (task orchestration) · Vite (build) · Vitest (test) · Astro/Starlight (docs)

**Flagship library:** `@toolbox-web/grid` (`<tbw-grid>`)

### Monorepo Structure

| Path                         | Description                                                      |
| ---------------------------- | ---------------------------------------------------------------- |
| `libs/grid/`                 | Core grid component with features, plugins, and internal modules |
| `libs/grid-angular/`         | Angular adapter (`@toolbox-web/grid-angular`)                    |
| `libs/grid-react/`           | React adapter (`@toolbox-web/grid-react`)                        |
| `libs/grid-vue/`             | Vue adapter (`@toolbox-web/grid-vue`)                            |
| `libs/themes/`               | Shared CSS theme system                                          |
| `apps/docs/`                 | Astro/Starlight documentation site (https://toolboxjs.com)       |
| `demos/employee-management/` | Demo apps: `vanilla/`, `angular/`, `react/`, `vue/`, `shared/`   |

## Knowledge Base Architecture

This project's AI knowledge is organized in four tiers to minimize context window usage:

1. **This file** (always loaded) — Project overview, navigation hub, core constraints
2. **Instruction files** (auto-loaded by file path) — Conventions and rules for specific file types (prescriptive: _how to work_)
3. **Skill files** (loaded on demand) — Multi-step workflows and procedures (procedural: _how to do X_)
4. **Knowledge files** (loaded on demand at task start) — Living mental model of the system (descriptive: _how it works and why_)

> **Knowledge files — read before editing, write after learning:**
>
> - **Read gate:** Before editing any file under `libs/grid/**`, `libs/grid-{angular,react,vue}/**`, or making a non-trivial change anywhere else, you MUST first read the knowledge files that cover the affected domain (see [Knowledge Reference](#knowledge-reference)). This rebuilds the mental model — state ownership, invariants, design rationale — so you can spot when a proposed change contradicts an earlier `DECIDED` entry and push back rather than silently regress it. Trivial edits (typos, comments, formatting) are exempt.
> - **Write gate:** During or after any task, if you discover a new invariant, state-ownership fact, data-flow edge, design decision, or tension that is not already in a knowledge file, you MUST add it to the correct file in `.github/knowledge/` using the structured notation (`OWNS / READS FROM / WRITES TO / INVARIANT / FLOW / TENSION / DECIDED`). These files are your externalized mental model — if you don't write it down, the next session will rediscover it from scratch.
> - **Knowledge vs. memory — do not confuse them:** Anything that is true _about this repository_ (architecture facts, design decisions, gotchas, build/test recipes, release plans, deprecation inventories) belongs in `.github/knowledge/*.md` (or, for prescriptive rules, `.github/instructions/*.md`) so it is **committable, reviewable, and shared with every contributor and future agent session**. The `/memories/repo/` scope is for **agent-private, machine-local scratch only** — e.g. notes about an in-flight investigation that the user has not yet decided to formalize. If the fact would help a human contributor or another agent on a different machine, it goes in the knowledge base, not in repo memory. When in doubt, choose the knowledge base.
> - **Rule of thumb:** If the user ever argues for a change that contradicts a `DECIDED` entry, cite the entry and ask them to justify overriding it before implementing. Past decisions have context; don't silently reverse them.

> **Knowledge file style — keep them dense or they stop working:**
> Knowledge files are an _agent_ tool. Their value is being scannable enough that you can rebuild a mental model in seconds without burning context window. They lose that value the moment they drift into changelog prose. Apply these rules every time you touch a `.github/knowledge/*.md` file:
>
> - **`DECIDED` is one bullet, not a paragraph.** Format: `DECIDED (date/PR): <conclusion>. WHY: <one-line rationale>. <File / test reference>.` Aim for 1-4 lines. If you have more to say, the next bullet is `INVARIANT:` or `TENSION:`, not a continuation paragraph.
> - **No history-as-DECIDED.** "We tried X, then Y, then settled on Z because A then B then C" is a commit message, not knowledge. Write the conclusion ("Use Z; X breaks because A") and let `git log` carry the story. The exception is a "RULED OUT:" line listing alternatives that look obvious but don't work — one line each, max.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OysteinAmundsen/toolbox](https://github.com/OysteinAmundsen/toolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
