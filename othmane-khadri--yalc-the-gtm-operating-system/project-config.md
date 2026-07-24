---
trigger: always_on
description: The modern GTM stack is fragmented, expensive, and hostile to small teams: a dozen point tools, opaque pricing, data scattered across vendors. YALC is one open-source CLI plus IDE chat layer that owns the full outbound loop — prospecting, enrichment, qualification, campaigns, and email — under a single configuration. It's built for operators and founders who run their own outbound and want to keep their data on their own machine. Architecture splits cleanly: read context from your knowledge base
---

# YALC GTM-OS — Claude Code Rules

## What YALC is

The modern GTM stack is fragmented, expensive, and hostile to small teams: a dozen point tools, opaque pricing, data scattered across vendors. YALC is one open-source CLI plus IDE chat layer that owns the full outbound loop — prospecting, enrichment, qualification, campaigns, and email — under a single configuration. It's built for operators and founders who run their own outbound and want to keep their data on their own machine. Architecture splits cleanly: read context from your knowledge base, execute from this repo. Local-first by default, your data stays on your machine.

## When the user opens this repo

If the user pastes the repo URL (`https://github.com/Othmane-Khadri/YALC-the-GTM-operating-system`), drops a "what is this" / "let's start" / "let's go" / "how do I use this" / "i just cloned this" / "where do I begin" message, or otherwise signals first-touch intent, invoke the `yalc-orientation` skill (`.claude/skills/yalc-orientation/SKILL.md`).

If the user has a specific operational request — "import these leads", "launch a campaign for X", "scrape this LinkedIn post" — route directly to the matching capability or skill. Do NOT force them through orientation; they already know what they want.

## Common stuck states

### Localhost won't open
**Symptom:** browser doesn't reach `http://localhost:3847/setup/review`.
**Fix:** confirm `yalc-gtm start` exited cleanly, copy the URL from the terminal banner, and check port 3847 isn't taken (`lsof -i :3847`).

### `yalc-gtm: command not found`
**Symptom:** shell can't resolve the binary.
**Fix:** run `npm i -g yalc-gtm-os` (or `pnpm link --global` from the repo root if hacking on YALC itself).

### `~/.gtm-os/` missing or `.env` empty
**Symptom:** first-run paths absent or env file blank.
**Fix:** run `yalc-gtm start` to scaffold; the CLI writes the template and opens it for editing.

## Project Identity
Open-source AI-native GTM operating system. Stack: Next.js 14, Tailwind, Drizzle + SQLite, Jotai, Anthropic SDK.

## Directory Structure
- `src/cli/index.ts` — CLI entry point, all commands registered here
- `src/lib/services/` — external integrations (Crustdata, Unipile, FullEnrich, Instantly, Notion, Firecrawl, Slack)
- `src/lib/qualification/` — 7-gate lead qualification pipeline
- `src/lib/campaign/` — campaign creation, tracking, scheduling, intelligence
- `src/lib/context/` — context adapters (e.g., markdown-folder for reading external knowledge bases)
- `src/lib/memory/` — tenant memory store, embeddings, retrieval, dream cycle
- `src/lib/framework/` — GTM framework derivation from company context
- `src/lib/agents/` — background agents, launchd integrations
- `src/app/` — Next.js web UI (chat, onboarding)
- `~/.gtm-os/` — per-tenant config, framework YAML, adapters
- `docs/` — architecture, commands, troubleshooting

## CLI
```
npx tsx src/cli/index.ts <command> [options]
```
Env loaded from `.env.local`. Use `--tenant <slug>` or `GTM_OS_TENANT` env var for multi-tenant (default: `default`).

## Lemlist integration

YALC ships with 24 curated [lemlist Claude Code skills](https://www.lemlist.com/claude-skills) under `.claude/skills/lemlist/`, plus a flagship orchestration skill at `.claude/skills/lemlist-campaign-from-icp/` and a lemlist MCP server pre-declared in `.mcp.json`. Together they cover the full outbound loop inside Claude Code: ICP → personas → sourcing → agentic enrichment → seniority-routed sequencing → quality gate → paused campaign in lemlist.

**Flagship skill:** `lemlist-campaign-from-icp` — turn one natural-language ICP prompt into a paused live lemlist campaign in ~5 minutes. Chains 24 atomic lemlist skills + the lemlist MCP. Hard approval gate before push. Landing page: <https://yalc.ai/skills/lemlist-campaign-from-icp/>.

| Layer | Lemlist atomic skills |
|---|---|
| Strategic foundation | `icp-definer`, `persona-definer`, `pain-identifier`, `value-prop-lister`, `offer-definer`, `competitor-finder`, `trigger-finder` |
| Sourcing | `company-finder`, `list-builder`, `people-finder` (+ MCP `lemleads_search`) |
| Per-lead angle | `linkedin-outbound-angle` |
| Campaign design | `campaign-angle-finder`, `outbound-campaign-architect` |
| Writing (seniority-routed) | `copywriting-vp-sequence`, `copywriting-manager-sequence`, `copywriting-ic-sequence`, `copywriting-first-touch`, `copywriting-follow-up`, `cta-designer` |
| Quality gate | `copywriting-refiner`, `copywriting-analyzer`, `gtm-action-thinker` |
| Loop (companion) | `reply-handler`, `outbound-analyst` |

**MCP setup (one time, per machine):**

Option A — API key (recommended for CI / scripted use):
```
export LEMLIST_API_KEY=...   # generate in lemlist → Settings → Integrations
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Othmane-Khadri/YALC-the-GTM-operating-system](https://github.com/Othmane-Khadri/YALC-the-GTM-operating-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
