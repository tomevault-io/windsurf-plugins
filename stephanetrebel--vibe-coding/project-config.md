---
trigger: always_on
description: "Mon Budget" is a personal budgeting PWA for teenagers. Frontend-only application (local-first):
---

# AGENTS.md - AI Agent Guidelines

## Project Overview

"Mon Budget" is a personal budgeting PWA for teenagers. Frontend-only application (local-first):
- **Frontend**: SvelteKit 5 (JavaScript, not TypeScript) with static adapter, IndexedDB storage
- **Deployment**: GitHub Pages (branch `trunk`) with `BASE_PATH` support
- **Locale**: French (fr-FR)

## Build & Run Commands

### Frontend (from `/frontend`)
```bash
npm run dev          # Start dev server (http://localhost:5173)
npm run build        # Production build (runs prebuild/postbuild scripts)
npm run preview      # Preview production build
```

### Environment Variables
```bash
BASE_PATH=           # Empty for dev; set to /repo-name for GitHub Pages (e.g. /vibe-coding)
```

### Build Scripts (auto-run)
- `scripts/generate-manifest.js` — injects `BASE_PATH` into `static/manifest.json` from template
- `scripts/copy-404.js` — copies `index.html` → `404.html` for GitHub Pages SPA routing

## Testing

### Framework: Playwright (E2E only) + axe-core (accessibility)

Test files are in `/frontend/e2e/`.

```bash
npm run test:e2e                              # Run all tests
npx playwright test e2e/transactions.spec.ts  # Single file
npx playwright test -g "create-expense"       # By name pattern
npm run test:e2e:headed                       # With visible browser
npm run test:e2e:ui                           # Interactive UI mode
```

### Test Files

| File | Coverage |
|------|----------|
| `transactions.spec.ts` | CRUD transactions, type/category switching, edit prefill, delete |
| `dashboard.spec.ts` | Stats, recent transactions (limit 5), balance styling, monthly isolation |
| `budget.spec.ts` | Set/update budget, progress bar states, month navigation, category exclusion |
| `goals.spec.ts` | CRUD goals, +1/+10/-1/-10 increments, floor at 0, achieve badge, validation |
| `navigation.spec.ts` | Desktop navbar, mobile bottom nav (375×667), active link, responsive behavior |
| `accessibility.spec.ts` | axe-core scans on all pages (empty + with-data states), zero violations expected |
| `charts.spec.ts` | BarChart/PieChart/LineChart visibility (hidden when no data), period toggle |
| `integration.spec.ts` | Cross-page: transaction affects budget/dashboard, edit/delete propagation |

### Writing Tests
- Tests use French UI labels (e.g., "Ajouter", "Supprimer", "Aucune transaction")
- Use `data-testid` attributes when adding new testable elements
- Chart wrappers use `data-testid="bar-chart"`, `"pie-chart"`, `"line-chart"`
- Follow existing patterns in `e2e/*.spec.ts`

## Application Features

### Pages & Routes

| Route | Description |
|-------|-------------|
| `/` | Dashboard — solde, stats mensuelles, transactions récentes (5), BarChart |
| `/transactions` | CRUD transactions, switch type income/expense, categories par type |
| `/budget` | Budget mensuel, navigation ← →, PieChart dépenses, LineChart historique |
| `/goals` | Objectifs d'épargne, +1/+10/-1/-10, badge "Atteint", suppression |

### Transaction Categories

**Dépenses** : Alimentation, Transport, Loisirs, Shopping, Abonnements, Education, Autre

**Revenus** : Argent de poche, Job etudiant, Cadeaux, Autre

### Chart Components (SVG — no external library)

| Component | Usage | Props |
|-----------|-------|-------|
| `BarChart.svelte` | Dashboard — revenus vs dépenses (6 derniers mois) | `data: [{ month: 'YYYY-MM', income, expenses }]` |
| `PieChart.svelte` | Budget — dépenses par catégorie (mois courant) | `data: [{ category, amount }]` |
| `LineChart.svelte` | Budget — budget vs réel (toggle 6/12 mois) | `data: [{ month, budget, spent }]`, `period: 6 \| 12` |

Charts are hidden when there is no data to display.

### Navigation (dual system)
- **Desktop** (≥ 768px): top navbar (`.navbar`) with text links
- **Mobile** (< 768px): fixed bottom bar (`.bottom-nav`, `data-testid="bottom-nav"`) with emoji icons + labels + `jiggle` animation on active item

## Code Style Guidelines

### Frontend (JavaScript/Svelte)

**Imports**
```javascript
import { onMount } from 'svelte';    // Svelte imports first
import { db } from '$lib/db.js';     // Then lib imports with $lib alias
```

**Svelte Component Structure**
```svelte
<script>
  // 1. Imports  2. Props/state  3. Reactive ($:)  4. Functions
</script>
<svelte:head><title>Page - Mon Budget</title></svelte:head>
<!-- Template -->
<style>/* Scoped styles */</style>
```

**Formatting**
- Tabs for indentation
- Single quotes, no semicolons
- camelCase for variables/functions, kebab-case for CSS classes

**Event syntax — use Svelte 5**
```svelte
<!-- Correct (Svelte 5) -->
<button onclick={handler}>...</button>

<!-- Avoid (Svelte 4, deprecated) -->
<button on:click={handler}>...</button>
```

**Error Handling**
```javascript
try {
  result = await db.someOperation();
} catch (e) {
  error = e.message;
} finally {
  loading = false;
}
```

**Locale Formatting** (always fr-FR)
```javascript
new Intl.NumberFormat('fr-FR', { style: 'currency', currency: 'EUR' }).format(amount)
new Date(dateStr).toLocaleDateString('fr-FR')
```

## CSS Guidelines

**Variables** (in `app.css`):

| Variable | Value | Usage |
|----------|-------|-------|
| `--primary` | `#a5b4fc` | Primary color |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StephaneTrebel/vibe-coding](https://github.com/StephaneTrebel/vibe-coding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
