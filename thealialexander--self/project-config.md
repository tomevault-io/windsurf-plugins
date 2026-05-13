---
trigger: always_on
description: This folder contains the standalone Self Context Standard v0.5 proposal site and starter examples.
---

# Self Context Standard - Agent Instructions

This folder contains the standalone Self Context Standard v0.5 proposal site and starter examples.

## Critical Rule: Self-Updating Documentation

Keep this file current. Whenever an agent introduces a new module, changes an existing module's purpose, adds or removes dependencies, or changes the development workflow, update this file as part of the same work.

## Project Direction

Self is a portable `self/` folder standard for agent runtime context, with progressive disclosure as the first principle. The proposal site must stay usable from disk and avoid framework or bundler assumptions.

The primary deployment target is a downloaded `self/` folder used directly inside Claude or Codex Skills-style environments. Optimize the markdown instructions for those harnesses first. Local scripts and the static proposal site are helpful maintainer tooling, but the portable markdown files must remain sufficient on their own when scripts are unavailable.

## Runtime Worlds

Self has three runtime worlds that must stay distinct:

* **Consumer uploaded Skill world**: The user downloads or uploads `self/` as a Skill. `SKILL.md` is the activation surface, `SELF.md` is the routing entrypoint, and markdown files are the portable runtime truth. Do not assume scripts, `manifest.json`, or repo tooling will run automatically.
* **Shell-enabled advanced Skill world**: The Skill is running in an environment that exposes the `self/` files plus Bash and Node.js. In this world, natural-language maintenance requests such as "make Self notice this next time," "make this easier for Claude or Codex to find," "check Self is up to date," "scan Self after this change," "keep the skill in sync," "update the package map," or "check that this still validates" may route to `node self/scripts/sync-self.js` and `node self/scripts/validate-self.js`. These helpers mirror markdown trigger truth into `manifest.json`; they do not make `manifest.json` the source of truth.
* **Repo maintainer/developer world**: Work inside this proposal repo uses `scripts/sync-self-spec.js` and `scripts/validate-self.js` to keep canonical examples, bridge copies, the homepage starter bundle, and the setup wizard base package aligned.

## Current Modules

* **`index.html`**: Static proposal website shell with anchor sections for the standard, manifest, explorer, disclosure flow, validation, common questions, install flow, developer integration notes, and references, plus a Mintlify-style page contextual menu (`Copy page` split button), a GitHub repo + star badge, and a hero `self/` promo block that can toggle between grid and tree file displays.
* **`styles.css`**: Design tokens, layout, responsive behavior, cards, install/developer sections, the compact common-question carousel with left-side tabs and active/hover/waiting states, tool-call simulation styling, code presentation, contextual-menu/dropdown styling, GitHub badge styling, hero folder view-toggle styling, and hover/focus heading-anchor affordances.
* **`app.js`**: Classic non-module browser script. Embeds starter files, exposes `window.SelfStarter` for static subpages, renders the schema explorer, renders install bridge snippets with an OpenAI/Anthropic toggle, validates examples, powers copy/download actions, simulates progressive-disclosure tool calls for common questions, renders the tabbed question carousel and keeps its active/waiting states in sync, wires the contextual menu actions with working page-copy/view/share behavior, renders inline current-color contextual-menu logos, includes hosted-only MCP/Cursor/VS Code install entries in that menu, fetches live GitHub stars for the header repo tracker, adds heading anchor-link copy behavior on larger screens, and drives the hero `self/` grid/tree toggle behavior.
* **`setup/`**: Static consumer setup wizard subpage. `setup/index.html` loads the shared stylesheet, `app.js`, and `setup/setup.js`; `setup/setup.js` uses `window.SelfStarter` to customize a Skill-ready `self/` ZIP locally in the browser with deterministic markdown generation, ordered final previews, explicit unlock-to-edit guards for generated/router/template files, a first-use prompt for post-upload onboarding, live SKILL.md/manifest validation, template-only `security.md`, and starter-only `chrono/`.
* **`assets/context-menu/`**: Downloaded SVG logo fallbacks retained for local reference; the live contextual menu currently uses inline SVGs in `app.js` so the logos inherit the menu icon color.
* **`LICENSE`**: MIT license for the proposal site and starter materials.
* **`.gitignore`**: Keeps local macOS filesystem metadata out of the repository.
* **`llms.txt`**: Agent-readable project summary with links to the proposal, starter example, manifest, validator, and license.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thealialexander/self](https://github.com/thealialexander/self) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
