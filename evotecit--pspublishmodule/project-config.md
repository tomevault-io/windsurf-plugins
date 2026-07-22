---
trigger: always_on
description: Last updated: 2026-03-29
---

# Agent Guide (PSPublishModule / PowerForge.Web + Websites)

Last updated: 2026-03-29

This file is the "start here" context for any agent working on the PowerForge.Web engine and the tracked sibling repos that build on it.

## Repos / Local Paths

These are the maintainer's default Windows paths (safe to assume in this workspace):

- Engine (this repo): `C:\Support\GitHub\PSPublishModule`
  - Web engine: `PowerForge.Web\`
  - Web CLI: `PowerForge.Web.Cli\`
  - Web docs: `Docs\PowerForge.Web.*.md`
  - PowerShell module: `PSPublishModule\` (+ packaging under `Module\`)
  - Core .NET libs/CLI: `PowerForge\`, `PowerForge.Cli\`

- HtmlForgeX website: `C:\Support\GitHub\HtmlForgeX\Website`
  - Remote: `https://github.com/EvotecIT/HtmlForgeX.git` (website lives under `Website\`)
  - Use this location for all HtmlForgeX website work.

- IntelligenceX website: `C:\Support\GitHub\IntelligenceX\Website`
  - Remote: `https://github.com/EvotecIT/IntelligenceX.git`

- CodeGlyphX website: `C:\Support\GitHub\CodeMatrix\Website`
  - Remote: `https://github.com/EvotecIT/CodeGlyphX.git`

- OfficeIMO website/docs: `C:\Support\GitHub\OfficeIMO\Website`
  - Remote: `https://github.com/EvotecIT/OfficeIMO.git`
  - Track this repo when standardizing website workflows, docs generation, and PowerForge-based housekeeping.

- DomainDetective repo: `C:\Support\GitHub\DomainDetective`
  - Remote: `https://github.com/EvotecIT/DomainDetective.git`
  - Track this repo when standardizing PowerForge-adjacent workflows, module usage, and release/build consistency.

- TestimoX website/docs: `C:\Support\GitHub\TestimoX\Website`
  - Remote: `https://github.com/EvotecIT/TestimoX.git`
  - Track this repo when standardizing PowerForge-based website workflows, docs generation, and quality gates.

## Portable Path Discovery (WSL/macOS/Linux)

If you're not on Windows or you don't have `C:\Support\GitHub`, use this layout heuristic:

- Common layout: the repos are siblings under one parent folder:
  - `<root>/PSPublishModule`
  - `<root>/HtmlForgeX/Website`
  - `<root>/IntelligenceX/Website`
  - `<root>/CodeMatrix/Website`
  - `<root>/OfficeIMO/Website`
  - `<root>/DomainDetective`
  - `<root>/TestimoX/Website`

Practical search strategy:

1. Start at the current repo root (where this `AGENTS.md` lives).
2. Check `..` (parent) and `../..` (grandparent) for sibling repo folders above.
3. If you're in WSL, Windows drives usually live under `/mnt/c`:
   - Example: `/mnt/c/Support/GitHub/PSPublishModule`

Recommended environment variable (makes site scripts deterministic):

- Set `POWERFORGE_ROOT` to the engine repo root (path to `PSPublishModule`).
  - Website `build.ps1` scripts prefer `POWERFORGE_ROOT` when resolving `PowerForge.Web.Cli`.

## What To Read First (Canonical)

1. `Docs\PowerForge.Web.Roadmap.md` (inventory: Have/Partial/Missing + milestones)
2. `Docs\PowerForge.Web.AgentHandoff.md` (high-signal handoff + commands)
3. `Docs\PowerForge.Web.QualityGates.md` (CI/dev contract, baselines, budgets)
4. `Docs\PowerForge.Web.WebsiteStarter.md` (golden path for building new sites without surprises)
5. `Docs\PowerForge.Web.Parity.md` (rough parity notes vs DocFX/Hugo/Astro/etc.)

Reference docs (as needed):
- `Docs\PowerForge.Web.ContentSpec.md` (content model + navigation)
- `Docs\PowerForge.Web.Theme.md` (theme anatomy + shortcodes)
- `Docs\PowerForge.Web.Pipeline.md` (pipeline tasks)
- `Docs\PowerForge.Web.ApiDocs.md` (API generator)
- `Docs\PSPublishModule.ProjectBuild.md` (PowerShell module build/publish pipeline)

## Repo Skills (.agents/skills)

This repo ships agent skills under `.agents/skills` so new contributors/agents don't
need per-user global skill installs.

- Website scaffolding skill: `.agents/skills/powerforge-website-starter`
- Module pipeline skill: `.agents/skills/powerforge-module-builder`
- Library/release pipeline skill: `.agents/skills/powerforge-library-builder`

## Working Agreements (Best Practices)

- Prefer engine fixes over theme hacks when the same issue can recur across sites.
- Keep `PSPublishModule` cmdlets thin:
  - parameter binding
  - `ShouldProcess` / prompting / PowerShell UX
  - output mapping back to PowerShell-facing contract types
- Move reusable logic into shared services first:
  - `PowerForge` for host-agnostic logic
  - `PowerForge.PowerShell` for logic that still needs PowerShell-host/runtime concepts
- Do not add new business logic to `PSPublishModule\Cmdlets\` when the same behavior could be reused by `PowerForge.Cli`, PowerForge Studio, tests, or another C# host.
- If a public PSPublishModule result type must stay stable, keep the reusable internal model in `PowerForge`/`PowerForge.PowerShell` and map it back in `PSPublishModule` instead of forcing cmdlet-specific types into shared layers.
- CI/release should fail on regressions; dev should warn and summarize:
  - Verify: use baselines + `failOnNewWarnings:true` in CI.
  - Audit: use baselines + `failOnNewIssues:true` in CI.
- Prefer stable theme helpers over ad-hoc rendering:
  - Scriban: use `pf.nav_links` / `pf.nav_actions` / `pf.menu_tree` (avoid `navigation.menus[0]`).
- Commit frequently. Avoid "big bang" diffs that mix unrelated changes.

## Module Layering

When touching the PowerShell module stack, prefer this boundary:

- `PSPublishModule\Cmdlets\`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EvotecIT/PSPublishModule](https://github.com/EvotecIT/PSPublishModule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
