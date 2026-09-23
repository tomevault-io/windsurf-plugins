---
trigger: always_on
description: Instructions for AI coding agents working on the OpenPersona codebase.
---

# AGENTS.md

Instructions for AI coding agents working on the OpenPersona codebase.

## Project Overview

OpenPersona is a persona lifecycle framework: it declares, generates, enforces, and evolves AI agent personas. It uses an open four-layer model (Soul / Body / Faculty / Skill), compiles `persona.json` into portable SKILL.md skill packs, and works with any agent (Cursor, Claude Code, Codex, ZeroClaw, OpenClaw, and 30+ others via `npx skills add`). CLI management (install / switch / uninstall) defaults to OpenClaw integration.

**Key distinction:** This repo is the *framework itself*, not a persona. The `skills/open-persona/SKILL.md` is a meta-skill for *using* the framework; this file guides *developing* it.

`ROADMAP.md` is listed in `.gitignore` and is not part of the published tree — keep a local copy for maintainer planning if you use it.

## Setup

```bash
npm install        # Install dependencies
node --test tests/ # Run all tests — must pass before any PR
```

- **Node.js ≥ 18** required (see `engines` in package.json)
- No build step — plain CommonJS, no transpilation
- Dependencies: fs-extra, mustache, commander, inquirer, chalk, adm-zip

## Project Structure

```
bin/cli.js              ← CLI entry point (commander-based)
lib/
  generator/            ← Core generation pipeline
    index.js            ← Orchestrator (the heart of the project)
    validate.js         ← Generate Gate (hard-reject constraint checks)
    derived.js          ← Derived template variable computation (returns plain object; caller applies via Object.assign)
    body.js             ← Body layer description builder
    social.js           ← Social aspect: Agent Card + ACN config + Contact Book seed builders
    economy.js          ← Economy aspect: load descriptor + write initial state
  lifecycle/            ← Persona lifecycle management
    installer.js        ← Persona installation to ~/.openpersona (with optional OpenClaw sync)
    uninstaller.js      ← Persona removal
    switcher.js         ← Active persona switching + handoff generation
    forker.js           ← Persona fork (derive child from installed parent)
    refine.js           ← Skill Pack Refinement (behavior-guide bootstrap + compliance scan + skill gate + social auto-sync)
    porter.js           ← Export / import persona packs
    contributor.js      ← Persona Harvest (community contribution)
  social/               ← Social Contact Book (runtime CRUD for contacts.json / contacts.jsonl)
    http.js             ← Thin HTTP client (GET/POST, no external deps) used by acn-client.js
    contacts.js         ← Local contacts CRUD: load / save / add / remove / lookup / list / log
    acn-client.js       ← ACN read client: fetchAgent / searchAgents / syncContacts / autoDiscover
  state/                ← Runtime state management
    runner.js           ← Persona directory resolution + state-sync delegation
    evolution.js        ← Evolution governance (evolve-report CLI + promoteToInstinct Soul-Memory Bridge)
  registry/             ← Local persona registry (~/.openpersona/persona-registry.json)
    index.js            ← loadRegistry / saveRegistry / registryAdd / registryRemove / registrySetActive
  remote/               ← External service calls (outbound network)
    registrar.js        ← ACN registration logic (acn-register CLI command); integrates auto-discover hook
    downloader.js       ← Preset/package downloading from ClawHub
    searcher.js         ← Persona search on ClawHub
    curator.js          ← Pack Curator (openpersona curate CLI command; privileged, requires OPENPERSONA_CURATOR_TOKEN)
  report/               ← Reporting and visualization
    vitality.js         ← Vitality score calculation (AgentBooks wrapper)
    vitality-report.js  ← HTML vitality report rendering
    canvas.js           ← Living Canvas HTML profile page
    helpers.js          ← Shared report utilities (readJsonSafe, formatDate, daysBetween, truncate)
  publisher/            ← Publish persona to OpenPersona public directory
    index.js            ← Validate GitHub repo + report to OpenPersona telemetry
  utils.js              ← Path constants + print helpers + string utilities (no registry logic)
templates/
  skill.template.md          ← Mustache template → generated SKILL.md (four-layer headings)
  soul/                      ← Soul layer molds
    soul-injection.template.md ← Soul layer injection orchestrator (delegates to partials/)
    soul-state.template.json   ← Initial state.json mold (seeded from soul config)
    partials/                  ← Mustache partials for soul-injection (6 files, each ~30–75 lines)
      soul-intro.partial.md              ← persona intro, selfie, personality
      soul-awareness-identity.partial.md ← Self-Awareness > Identity + dormant Capabilities
      soul-awareness-body.partial.md     ← Self-Awareness > Body (Signal Protocol, runtime, interface)
      soul-awareness-growth.partial.md   ← Self-Awareness > Growth (pendingCommands, constraints, evolution sources)
      soul-how-you-grow.partial.md       ← How You Grow (stage criteria, eventLog, self-narrative)
      soul-economy.partial.md            ← Survival Policy
  body/                      ← Body layer molds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acnlabs/OpenPersona](https://github.com/acnlabs/OpenPersona) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
