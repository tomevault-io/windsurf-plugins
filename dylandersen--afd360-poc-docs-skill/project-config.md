---
trigger: always_on
description: Scaffolds a tailored Fumadocs documentation site for a Salesforce Solutions Engineer delivering a post-sale Agentforce, Salesforce, or Data 360 Proof-of-Concept. Use when the user runs /setup-docs, /update-docs, asks to "spin up customer docs", "create POC docs site", "scaffold Fumadocs for a customer", or "add a section to my POC docs". Generates a Next.js + Fumadocs site with sections for Overview, Architecture, Setup, Data Model, Agents & Flows, Handoff, and Troubleshooting, pre-populated wit
---


# afd360-poc-docs-skill — Customer POC Documentation Site

Scaffolds a production-ready [Fumadocs](https://fumadocs.dev) site tailored for an SE handing off a Salesforce / Agentforce / Data 360 Proof-of-Concept to a customer. Also supports incremental updates to an already-scaffolded site.

## What it produces

A Next.js + Fumadocs site at a directory the SE chooses, with:

- Opinionated sidebar structure tuned for a POC handoff
- Pages pre-filled with customer + POC context and **starter content** (real tables, ASCII system/ERD sketches, example callouts) the SE can edit in place. Mermaid is intentionally **not** used in starter content because Fumadocs ships without a Mermaid renderer — SEs should drop a PNG/SVG into `public/` if they want a richer diagram.
- A single `site.config.ts` file that centralizes every knob an SE typically wants to change (customer, POC, theme colors, fonts, logo, personas, integrations, repo URL)
- **Auto-extracted customer branding.** If the SE provides the customer's public website during intake, the scaffolder runs `scripts/brand-extractor/index.mjs` against that URL and bakes the logo, favicon, OG hero, primary/secondary colors, and detected font family (with a Google-Fonts neighbor suggested for proprietary fonts) into `site.config.ts` + `public/brand/` + `data/brand-snapshot.json`. Respects `robots.txt`; always surfaces what was extracted for SE review; every value is overrideable in `site.config.ts`.
- Static client-side search (Orama), `llms-full.txt`, OG image route
- Tailwind v4, TypeScript strict, works with `pnpm` (preferred), `npm`, or `bun`

The template tracks **Fumadocs 16+** and requires **Node 22+**. The template's `package.json` is the source of truth for versions — do not hardcode versions anywhere in this skill's prose.

> **Status:** All scripts are implemented and the template passes `next build`. If `scripts/preflight.mjs` or `scripts/verify-placeholders.mjs` are ever missing on disk, **degrade gracefully** — for preflight, fall back to a bare `node --version` check (require ≥ 22) and confirm at least one of `pnpm`/`npm`/`bun` is on PATH; for verify-placeholders, do a `grep -rE '__[A-Z][A-Z0-9_]*__'` over the target. Do not bail or refuse intake — the SE shouldn't pay for a tooling gap.

## Invocation

Triggered by:

- `/setup-docs` — scaffold a new site (default)
- `/update-docs` — add or update sections in an existing scaffolded site
- Natural phrases: "spin up customer docs", "scaffold POC docs", "create docs site for \<customer\>", "add a \<section\> page to my POC docs"

## Workflow

Follow these phases **in order**. Phase 0 is non-negotiable — it catches 90% of "why didn't it work" issues before the SE has invested any time. Phase 1.5 is new: between intake and scaffold, offer the optional section library and let the SE skip anything they don't have answers for yet.

### Phase 0 — Preflight (always run first)

Run `node "<skill-dir>/scripts/preflight.mjs"`. It checks, in order:

1. Node version ≥ 22 (Fumadocs 16 minimum) — **hard requirement**
2. A package manager is available (`pnpm` preferred, `npm` fallback, `bun` acceptable) — **hard requirement**
3. `git` is on PATH (needed for Phase 4's `git init`) — **soft warning, do not block**
4. Network reachability to the npm registry (`https://registry.npmjs.org`) — **soft warning, do not block**

The script exits **0** on hard-requirement pass (with or without soft warnings) and **1** on hard-requirement fail. If it exits 1, **stop and print the script's `fix:` block** for the SE — it includes platform-specific install commands. Soft warnings should be surfaced but **do not stop intake** — the SE's network may flake or git may genuinely not be needed yet.

If `scripts/preflight.mjs` is missing for any reason (corrupted install, partial sync), **fall back to a manual check**: confirm `node --version` returns ≥ v22 and that `pnpm --version`, `npm --version`, or `bun --version` succeeds. **Never** stop intake just because the helper script is absent — the SE has not done anything wrong.

Resolve `<skill-dir>` from the absolute path of this `SKILL.md` file. Typical locations: `~/.cursor/skills/afd360-poc-docs-skill/`, `~/.claude/skills/afd360-poc-docs-skill/`, or `~/.agents/skills/afd360-poc-docs-skill/`.

### Phase 1 — Intake

**Required fields (4):**

1. **Customer name** — e.g. "Acme Corp"
2. **POC name** — e.g. "Service Cloud Agent POC"
3. **Product area** — one of: `Agentforce`, `Data 360`, `Agentforce + Data 360`, `Salesforce Platform`
4. **Target directory** — absolute path. Default: `~/Documents/cursor_orgs/<customer-slug>-docs`

**Optional fields (fill in after scaffold by editing `site.config.ts`):**

5. Primary personas — defaults to `Admin, End User`
6. Key integrations — defaults to `None`
7. Deploy target — defaults to `Not decided`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dylandersen/afd360-poc-docs-skill](https://github.com/dylandersen/afd360-poc-docs-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
