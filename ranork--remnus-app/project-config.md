---
trigger: always_on
description: `AI.md` is the canonical shared guide for Claude Code and Codex. Read and follow it before coding, debugging, reviewing, or making architectural changes.
---

# Codex project instructions

`AI.md` is the canonical shared guide for Claude Code and Codex. Read and follow it before coding, debugging, reviewing, or making architectural changes.

Use `.ai/CURRENT_TASK.md`, the current Git state, and relevant Serena memories to recover active context. Keep this file as the detailed product/source map; structural changes must update it and the matching Serena memories.

---

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.

<!-- END:nextjs-agent-rules -->

# Keeping This File Up to Date

**Every agent that makes structural changes to the project MUST update this file before finishing.**
Structural changes include: adding/removing tables, adding routes, adding/removing components, adding server actions, or changing architectural patterns.
If you skip this step, future agents will work from a stale map and make mistakes.

## Serena Memory Sync

Serena is an MCP code-intelligence assistant with its own persistent memory. Its memories must be kept in sync with this file.

**After updating AGENTS.md, also sync Serena:**

1. Activate the project: call `mcp__plugin_serena_serena__activate_project` with `"remnus-app"`.
2. List existing memories: call `mcp__plugin_serena_serena__list_memories` to see what needs updating.
3. Edit stale entries: call `mcp__plugin_serena_serena__edit_memory` for any memory whose content no longer matches AGENTS.md.

**Key memories to check:**

- `conventions` — i18n namespace list and count, coding rules
- `core` — source map, DB tables, component inventory

Do NOT create new Serena memories for every change — prefer editing existing ones. Only create a new memory if it covers a clearly standalone domain not already tracked.

---

# Project Details: Remnus

**Remnus** is a Notion-like application built around a **workspace** model. Users can create standalone pages (title + markdown) and customizable databases (dynamic columns, table/kanban/calendar views) — both living side by side in a unified sidebar. Each database row is also a page with markdown content.

## i18n & Localization

Remnus is fully internationalized using **next-intl v4** (App Router native). All user-facing text is loaded from translation files — **no hardcoded strings in components**.

### Supported Languages

| Code | Language          |
| ---- | ----------------- |
| `en` | English (default) |
| `tr` | Türkçe            |
| `hi` | हिन्दी            |
| `es` | Español           |
| `fr` | Français          |
| `de` | Deutsch           |
| `zh` | 中文 (Simplified)  |
| `ru` | Русский           |

### Locale Resolution (priority order)

1. `NEXT_LOCALE` cookie (user picks via `LanguageSwitcher`, 1-year expiry)
2. `Accept-Language` header (auto OS language detection via `negotiator` + `@formatjs/intl-localematcher`)
3. `en` fallback

**Clean URLs:** `localePrefix: 'never'` — URLs stay as `/db/123`, never `/en/db/123`. All pages live under `src/app/[locale]/`.

**Translation files:** `messages/{locale}.json` — `en.json` is the source of truth, **8 locales** (`en`, `tr`, `hi`, `es`, `fr`, `de`, `zh`, `ru`). **English fallback:** `src/i18n/request.ts` deep-merges a non-default locale's messages onto `en.json`, so any untranslated key gracefully falls back to English instead of showing a raw key name. **31 namespaces:** `Layout`, `Home`, `Auth`, `Workspace`, `WorkspaceSettings`, `Templates`, `Database`, `Editor`, `Page`, `IconPicker`, `Admin`, `Errors`, `LanguageSwitcher`, `MobileNav`, `Landing`, `Billing`, `Pricing`, `Contact`, `Download`, `Privacy`, `Updater`, `Sharing`, `UserSettings`, `OAuthAuthorize`, `Security`, `Consent`, `Onboarding`, `DemoFeedback`, `Brand`, `Docs`, `Mailing`. (`Docs` = UI chrome for the public **Wiki + Docs** sections; the long-form content bodies stay English-only — see **Public Docs: Wiki + Blog**.)

### Rules for All Future Development

**Every new component or server action that surfaces user-facing text MUST follow these rules:**

1. **Client components** — `import { useTranslations } from 'next-intl'` and call `useTranslations('Namespace')` inside the component body.
2. **Server components / layouts** — `import { getTranslations } from 'next-intl/server'` and `await getTranslations('Namespace')`.
3. **Server actions** — same as above; use `getTranslations('Errors')` for error messages returned to the client.
4. **Add all new keys to ALL 8 files** before committing. Missing keys fall back to English (`en.json`) via the deep-merge in `request.ts`, then to the key name.
5. **No hardcoded display strings** — not even English fallbacks like `|| 'Untitled'`. Always use `t('key')`.
6. **Date formatting** — use `useLocale()` (client) or locale from `getRequestConfig` (server) instead of `'en-US'`.
7. **Namespace selection** — pick the closest existing namespace. Create a new one only for a clearly standalone domain (add to all 8 files and document here).

### Agent Task Management & Work Plan


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ranork/remnus-app](https://github.com/Ranork/remnus-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
