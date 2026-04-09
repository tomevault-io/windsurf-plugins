---
trigger: always_on
description: Project-specific configuration and quick reference. For detailed rules, see `.claude/skills/*/SKILL.md`.
---

# AYA Development Guidelines

Project-specific configuration and quick reference. For detailed rules, see `.claude/skills/*/SKILL.md`.

## Quick Reference

| Topic                | Skill                     |
| -------------------- | ------------------------- |
| Project structure    | `architecture-guidelines` |
| Code quality         | `coding-practices`        |
| JS/TS patterns       | `javascript-practices`    |
| Go patterns          | `go-practices`            |
| Security             | `security-practices`      |
| Development workflow | `workflow-practices`      |
| CI/CD                | `ci-cd-practices`         |
| Tooling              | `tooling-standards`       |

## gstack

Use the `/browse` skill from gstack for all web browsing. **Never use `mcp__claude-in-chrome__*` tools.**

Available skills:

- `/browse` — Web browsing and page interaction (Playwright-based)
- `/plan-ceo-review` — CEO-level plan review
- `/plan-eng-review` — Engineering plan review
- `/review` — Code review
- `/ship` — Ship code (commit, PR, deploy workflow)
- `/qa` — Quality assurance testing
- `/setup-browser-cookies` — Import browser cookies for authenticated browsing
- `/retro` — Retrospective analysis

## Project Structure

```
aya.is/
├── apps/
│   ├── webclient/                # TanStack Start frontend (Deno)
│   │   └── src/
│   │       ├── routes/           # File-based routing
│   │       ├── components/       # UI components (shadcn in ui/)
│   │       ├── modules/          # Feature modules (backend, auth, i18n)
│   │       └── lib/              # Utilities
│   └── services/                 # Go backend (Hexagonal Architecture)
│       └── pkg/
│           ├── api/business/     # Pure business logic
│           ├── api/adapters/     # External implementations
│           └── ajan/             # Framework utilities
└── .claude/skills/               # Development rules
```

## Development Commands

```bash
# Frontend (in /apps/webclient)
deno task dev         # Start dev server
deno task build       # Build for production
deno lint && deno fmt # Lint and format
deno task test        # Run snapshot tests (writes snapshots)
deno task test:update # Regenerate snapshots after intentional changes
deno task test:ci     # Run tests read-only (CI mode, used by make ok)

# Backend (in /apps/services)
make restart          # Restart after changes
make lint             # Run golangci-lint
make test             # Run tests

# Root
make ok               # All quality checks (includes frontend tests)
```

## Key Conventions (CRITICAL)

### Explicit Checks

```typescript
// Correct
if (value === null) {}
if (array.length === 0) {}

// Wrong - never use truthy/falsy except for booleans
if (!value) {}
if (!array.length) {}
```

### Backend Object Pattern

```typescript
// Correct - use centralized backend object
import { backend } from "@/modules/backend/backend.ts";
const profile = await backend.getProfile("en", id);

// Wrong - direct imports
import { getProfile } from "@/modules/backend/profiles/get-profile.ts";
```

### CSS Modules with @apply

```css
/* product-card.module.css */
.card {
  @apply border rounded-lg p-4;
}
```

```tsx
import styles from "./product-card.module.css";
<div className={styles.card}>...</div>;
```

### Single Props Object

```tsx
// Correct
function Component(props: Props) {
  return <div>{props.title}</div>;
}

// Wrong - destructured
function Component({ title }: Props) {}
```

### Pure Function Extraction for Testability

```typescript
// Correct - pure logic in separate file, testable with Deno
// locale-utils.ts (pure, no import.meta.env)
export function isValidLocale(locale: string) { ... }

// config.ts (re-exports + env-dependent config)
export { isValidLocale } from "@/lib/locale-utils.ts";
export const siteConfig = { host: import.meta.env.VITE_SITE_URL };

// Wrong - pure logic mixed with env dependency
import { siteConfig } from "@/config.ts";
export function buildUrl(locale: string) { return `${siteConfig.host}/${locale}`; }
```

## Project-Specific Notes

### Internationalization & Locale Fallback (CRITICAL)

- 13 locales: ar, de, en, es, fr, it, ja, ko, nl, pt-PT, ru, tr, zh-CN
- Translation keys use English text: `t("Section", "English text")`
- Messages in `/apps/webclient/src/messages/[locale].json`
- **NEVER put English text in non-English locale files** — every value in a locale file MUST be translated to that
  locale's language. When adding new i18n keys, always provide proper translations for ALL 13 locales.
- All `_tx` tables use `CHAR(12)` for `locale_code` — **always `strings.TrimRight(value, " ")` when mapping to Go
  business types**

#### 3-Tier Locale Fallback Pattern

All `_tx` table joins (story_tx, profile_tx, profile_page_tx, profile_link_tx) MUST use the 3-tier fallback subquery —
never restrict with `AND (locale = X OR locale = Y)`:

```sql
-- CORRECT: 3-tier fallback — always finds a translation if one exists
AND st.locale_code = (
  SELECT stx.locale_code FROM "story_tx" stx
  WHERE stx.story_id = s.id
  ORDER BY CASE
    WHEN stx.locale_code = sqlc.arg(locale_code) THEN 0          -- requested locale
    WHEN stx.locale_code = <default_locale_reference> THEN 1     -- entity's default

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/eser) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
