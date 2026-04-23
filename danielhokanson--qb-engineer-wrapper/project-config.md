---
trigger: always_on
description: > Loaded into every Claude Code session. These rules override defaults. Follow exactly.
---

# QB Engineer — Project Rules

> Loaded into every Claude Code session. These rules override defaults. Follow exactly.
> Full specs in `docs/`. When in doubt, check `docs/coding-standards.md` first.

## SELF-MAINTENANCE RULE

**After every session that introduces a new pattern, convention, architectural decision, or workflow change — update this file.** This is the single source of truth for project rules across sessions. If a decision is made during implementation (new shared component, naming convention, SCSS pattern, API convention, etc.), add it here before the session ends. Outdated or missing rules cause rework. Keep this file current.

Also update `docs/coding-standards.md` or the relevant doc file if the change is significant enough to be spec-level.

**Implementation tracking:** Check `docs/implementation-status.md` at the start of every session. When completing a feature or sub-feature, update its status (Not Started → Partial → Done) in that file before ending the session. This is the master progress tracker.

## Auto-Restart API

**When any .NET backend change is made that requires a restart (controller changes, entity changes, Program.cs, appsettings, etc.), automatically rebuild and restart the API container:**

```bash
docker compose up -d --build qb-engineer-api
```

Do not ask the user — just do it after verifying the build passes.

## Visual Verification (Non-Negotiable)

**After any UI fix or visual change, take a Playwright screenshot and examine it before considering the task complete.** This catches issues that code review alone misses (wrong spacing, overlapping elements, broken layouts, missing gaps).

**How to verify:**
1. Run `npx playwright test screenshot-verify` from `qb-engineer-ui/`
2. Examine the screenshot at `e2e/screenshots/{page}.png`
3. If the screenshot shows the fix didn't work (hot-reload missed, wrong CSS, etc.), iterate

**Reusable screenshot script:** `qb-engineer-ui/e2e/tests/screenshot-verify.spec.ts` — set `TARGET_PATH` env var or edit the route inline. Default: `/dashboard`.

Do not ask the user — just verify visually after every UI change.

---

## Space Efficiency (Application-Wide Rule)

**Every UI screen must fit and be usable at 1080p (1920×1080) without scrolling off-screen or hiding primary content.** This is a hard constraint, not a nice-to-have.

### Rules

1. **Reactive design first.** Use CSS to make layouts adapt — tighter spacing at shorter viewports via height-based media queries (`@media (max-height: 900px)`), fewer/smaller gaps, denser grids.

2. **Dedicated mobile/narrow UI** only when reactive design isn't practical (e.g. kanban board, shop floor kiosk).

3. **No redundant chrome.** Do not add a border/divider AND a large gap between every section. Pick one visual separator — either a thin border OR whitespace, not both. A single divider after the first content block (hero/title area) is sufficient; section-to-section separation uses gap alone.

4. **Collapsible empty sections.** Sections that have no items (subtasks, linked cards, parts, etc.) must not render their full add-form at all times. Show just the section header + an inline [+] toggle. Expand on demand. Sections with content are always visible.

5. **Spacing scale for dense panels** (detail panels, dialogs, sidebars):
   - Panel body padding: `$sp-lg` (16px) max, `$sp-md` (8px) preferred
   - Section gap: `$sp-md` (8px) between sections
   - Section internal gap: `$sp-sm` (4px)
   - Sidebar section padding: `$sp-md $sp-lg`
   - Hero/title area: `$sp-xs` gap between title and subtitle

6. **Section titles.** Use `$font-size-xxs` uppercase labels (`color: var(--text-muted)`) — not the larger `$font-size-xs` variant. Section titles are navigational aids, not headings.

7. **Test at 1080p.** After any layout change, screenshot at the default kanban job detail panel size (or relevant view) and verify the primary content is visible without scrolling.

---

## Project Structure

```
qb-engineer-wrapper/
├── qb-engineer-ui/          # Angular 21 + Material 21
│   └── src/
│       ├── styles/           # _variables, _mixins, _shared, _reset
│       ├── styles.scss       # Material theme + overrides
│       └── app/
│           ├── shared/       # Reusable components, services, directives, pipes, utils
│           ├── features/     # Feature modules (kanban, backlog, admin, etc.)
│           └── core/         # Singleton services (layout, nav, toolbar, sidebar)
├── qb-engineer-server/       # .NET 9 solution
│   ├── qb-engineer.api/      # Controllers, Features/ (MediatR handlers), Middleware
│   ├── qb-engineer.core/     # Entities, Interfaces, Models, Enums
│   ├── qb-engineer.data/     # DbContext, Repositories, Migrations, Configuration
│   └── qb-engineer.integrations/
├── docs/                     # Specs: coding-standards, architecture, functional-decisions, ui-components, roles-auth, libraries
└── docker-compose.yml        # 5 core + 3 optional profiles (ai, tts, signing)
```

---

## Critical Rules

### ONE OBJECT PER FILE (Non-Negotiable)
- **Angular:** One component, service, pipe, directive, guard, interceptor, or model per file. No barrel files (`index.ts`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielhokanson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
