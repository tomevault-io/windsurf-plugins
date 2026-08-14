---
trigger: always_on
description: This project is a Web/Electron app for browsing Polkadot products. It can run as a standalone web application or as an Electron
---

# Overview

This project is a Web/Electron app for browsing Polkadot products. It can run as a standalone web application or as an Electron
desktop app.

A product is an application or set of applications resolvable through dotNS (Polkadot on-chain naming) names. A product can have multiple presentations: an SPA, a widget, or a separate JS worker.

## Known Claude commands

- `/create-feature <feature-name>` - Create a new feature boilerplate in `features` directory. If a feature already exists —
  review it and adjust according to defined structure
- `/create-domain-module <domain-name> <module-name>` - Create a new domain module boilerplate in `domains` directory.
- `/commit-and-push` - Stage all modified/new files and push to the current branch.
- `/pr-message` - Draft a pull request message from `.github/PULL_REQUEST_TEMPLATE.md`.

## Project skills

These skills (under `.claude/skills/`) carry the project's procedural rules. They are part of the contract — don't reimplement guidance they own.

> **Required plugin dependency:** the `superpowers` plugin (`superpowers@claude-plugins-official`) is declared in `.claude/settings.json` so the whole team/CI gets it.

- `architecture` — **entry point for any decision-driven code change, and orchestrator of the full lifecycle** (frame → place → brainstorm → approve → plan → implement → review → fix). Orchestrates `code-placement`, `domain-development`, `feature-development`; delegates brainstorming + plan-writing + implementation to `superpowers`; gates implementation on plan approval; hard-stops on mid-flight plan deviations; and runs `reviewer` as the mandatory post-implementation gate. Run it first, always.
- `code-placement` — fires first when the target layer is uncertain (domain / aggregate / feature / widget / shared). Entry point for "where does this go?".
- `domain-development` — fires when working in `src/domains/`. Points at `docs/code/domain-development.md`.
- `feature-development` — fires when working in `src/features/` or `src/routes/` files that host features. Points at `docs/code/feature-development.md`.
- `react-best-practices` — fires when writing/reviewing/refactoring React code. Performance and rendering rules in `rules/`.
- `reviewer` — audits a branch diff / PR against this project's checklists (`docs/claude/`) with blocking/major/minor severity. Project-rule-aware; complements the generic `/code-review`. Run after a change, before merge.

## Project Structure

> **MANDATORY — start every code change with the `architecture` skill.** Before writing or changing any non-trivial code in `src/` (new code, new/moved/renamed files, a new abstraction, extending behavior, or a refactor that relocates logic), you MUST invoke the `architecture` skill first. It is the single entry point that decides _what_ the change is and _where_ it goes, orchestrating `code-placement`, `domain-development`, and `feature-development`, and (for non-trivial changes) produces a short plan in `docs/_plans/`. Do not generate a file path, pick a layer, or open the flow docs yourself before running it — this does not depend on the skill auto-activating. Inside `src/domains/`, a leaf/container module may ONLY contain the file kinds enumerated below (`types`, `service`, `resource`, `hooks`, `gateway`, `repository`, `schemas`, `constants`, `index`, `README`, `bootstrap`, `$usecase/`) — the PreToolUse hook blocks new non-canonical files outright. If your logic has no canonical home, STOP and ask — never invent a new filename (e.g. `changes.ts`, `manager.ts`, `helpers.ts`) to resolve the ambiguity. Skip the skill only for trivial in-place edits within one existing file (`docs/code/glossary.md` defines the threshold).

@docs/code/project-structure.md

## Code style

@docs/code/style.md

## Development flows

- Adding or extending business logic → [docs/code/domain-development.md](./docs/code/domain-development.md).
- Building user-facing scenarios → [docs/code/feature-development.md](./docs/code/feature-development.md).
- Finding the seam (when domain/feature boundary is unclear) → [docs/abstract/event-storming.md](docs/abstract/event-storming.md).
- Auditing a change before merge → the `reviewer` skill (checklists in [docs/claude/](docs/claude/); review output to [docs/_reviews/](docs/_reviews/)).
- Thresholds (trivial vs non-trivial, peer file, multi-source) → [docs/code/glossary.md](./docs/code/glossary.md).
- Turning a recurring correction into a durable rule → [docs/code/rule-extraction.md](./docs/code/rule-extraction.md).

Always check whether the change belongs in a domain before touching a feature.

## Stack

- **TypeScript** - Primary language
- **React** - UI framework
- **RxJS** - Reactive programming for data streams
- **Immer** - Immutable cache updates (`produce`)
- **Vite** - Build tool
- **Electron** - Desktop app shell
- **TanStack Router** (`@tanstack/react-router`) - File-based routing from `src/routes/`
- **Valibot** - Schema validation
- **Tailwind CSS 4** - Styling
- **@novasamatech/tr-ui** - Main UI kit
- **polkadot-api** - Polkadot chain interaction
- **Dexie** - IndexedDB persistence
- **Vitest** + **React Testing Library** - Unit tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [paritytech/polkadot-desktop-community](https://github.com/paritytech/polkadot-desktop-community) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
