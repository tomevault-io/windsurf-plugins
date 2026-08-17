---
trigger: always_on
description: Project instructions for AI coding agents (Claude Code, Cursor, Copilot, Windsurf, etc.) working in this repository.
---

# AGENTS.md

Project instructions for AI coding agents (Claude Code, Cursor, Copilot, Windsurf, etc.) working in this repository.

> **Single source of truth — edit `AGENTS.md`, not `CLAUDE.md`.** `CLAUDE.md` is a symlink to this file (`AGENTS.md`). This is deliberate: `AGENTS.md` is the cross-agent standard filename (Cursor / Copilot / Windsurf read it), while Claude Code reads `CLAUDE.md` — the symlink serves both from one source. Most editors and tools (including the Edit tool) **reject writing through a symlink** ("Refusing to write through symlink"), so always open and edit `AGENTS.md` directly. Do **not** replace the symlink with a real file, delete it, or duplicate the content into `CLAUDE.md`.

## Project Overview

Headroom is a browser extension built with [WXT](https://wxt.dev/) (next-gen web extension framework). **Manifest V3 only** — Chrome, Edge, and Firefox. MV2 is not supported (`manifestVersion: 3` is set in `wxt.config.ts`, overriding WXT's Firefox-default MV2).

## Commands

```bash
npm run dev              # Dev + HMR → .output/chrome-mv3-dev/ (DIFFERENT dir from build!)
npm run dev:firefox      # Dev mode for Firefox
npm run build            # Production build → .output/chrome-mv3/
npm run build:chrome     # Same as build (alias)
npm run build:firefox    # Production build → .output/firefox-mv3/
npm run build:edge       # Production build → .output/edge-mv3/
npm run zip              # Package .zip for distribution (Chrome)
npm run zip:firefox      # Package .zip for Firefox
npm run zip:edge         # Package .zip for Edge
npm run lint             # ESLint check
npm run lint:fix         # ESLint auto-fix
npm run typecheck        # TypeScript type check (tsc --noEmit)
npm test                 # Run tests in watch mode
npm run test:run         # Run tests once (CI + pre-commit)
npx wxt prepare          # Regenerate types in .wxt/ (auto-runs on postinstall)
```

## Architecture

**WXT uses file-based routing** — entrypoints are auto-discovered from `entrypoints/` directory.

- **HTML entrypoints** must use directory structure: `entrypoints/<name>/index.html` + `entrypoints/<name>/main.ts`. Do NOT place `.html` and `.ts` sibling files with the same name — WXT treats them as duplicate entrypoints.
- **Script entrypoints**: `background.ts` uses `export default defineBackground(() => {...})`. Content scripts use `export default defineContentScript({ matches: [...], main() {...} })`.
- **Auto-imports**: `defineBackground`, `defineContentScript`, `defineConfig`, `browser` etc. are auto-imported by WXT. Do not add explicit import statements for these.
- **Cross-browser API**: Use `browser.*` (WXT wrapper) instead of `chrome.*` directly.

### Upstash (Redis) data model — the cloud storage layer

Upstash Redis (user-owned, BYOK) is the **cross-device merge point + cloud persistence**; `browser.storage.local` is the acceleration cache the live gauge reads from. The **token truth is the platform's conversation-history text** — tokens are always _estimated_ from that text by the 001 engine, never trusted from the platform; Upstash only persists the resulting counts. The transport layer is spec [002](specs/002-upstash-data-layer.md); the reconciliation that reads/writes these records (open = full recompute, union-merge by round-n, delete sync, zombie cleanup) is spec [003](specs/003-cross-device-sync.md). The extension reaches Upstash only over the HTTPS **REST API** — the browser can't speak native Redis.

**REST contract** (`utils/upstash.ts`): one HTTPS POST per command.
`POST {UPSTASH_REDIS_REST_URL}/` · header `Authorization: Bearer {UPSTASH_REDIS_REST_TOKEN}` · body = JSON command array (`["GET",key]` / `["SET",key,val]` / `["DEL",key]`) → `{ "result": <string|null> }`. 8s `AbortController` timeout — a wedged Upstash must not hang the SW. Empty creds ⇒ every op silently no-ops (Upstash is optional; the gauge works off local state).

**Free-tier budget** ([pricing](https://upstash.com/pricing/redis)): 256 MB storage, **500K commands/month** (account-level, not per-key). Each round costs 2 commands (GET + SET in the read-modify-write), a delete costs 1 (DEL), a settings save costs 1, a side-panel open costs 1 (settings-pull GET, spec 003). 500K/month ≈ 250K rounds/month — well beyond a single user. Storage is a non-issue: a `DialogueRecord` stores **only token counts** per round (no prompt/answer text — see `utils/dialogue-record.ts`), so a 50-round conversation is ~4 KB; 256 MB ≈ 65K conversations. **Architectural implication**: 003's zombie-cleanup / open-reconcile can burst commands after a long offline period (there is **no outbox** — missed rounds are simply re-reconciled on next open), but the total stays within budget because they are real user activity that would have been counted anyway. If a user ever exceeds the free tier, Upstash bills ~$0.20/100K extra commands — that's the user's account, not Headroom's concern.

**Key scheme — only two value types live on Redis:**

- `headroom:conv:{platform}:{dialogueId}` → `DialogueRecord` JSON (shape in `utils/dialogue-record.ts`; carries `updatedAt`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZM-BAD/headroom](https://github.com/ZM-BAD/headroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
