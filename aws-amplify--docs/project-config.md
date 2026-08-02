---
trigger: always_on
description: corepack enable && yarn set version berry
---

# Project Info

## Setup

```bash
# Prerequisites: Node.js 20+ (below 22.0.0)
corepack enable && yarn set version berry
yarn && yarn dev
# Site available at http://localhost:3000/
```

## Build

```bash
yarn build                                    # Full build (prebuild + next build + postbuild)
yarn prebuild                                 # Generate directory.json, flatDirectory.json, llms.txt
node src/directory/generateDirectory.mjs      # Regenerate directory.json only
node src/directory/generateFlatDirectory.mjs  # Regenerate flatDirectory.json only
```

## Testing

```bash
yarn test          # Run all tests (Jest)
npx jest           # Run all tests directly
npx jest <path>    # Run specific test file
```

All tests must pass before merging. Currently 259 tests across 60 suites.

## Linting

```bash
yarn lint          # ESLint + Next.js lint
```

## Project Coding Conventions & Patterns

### Project Overview
- **Language**: TypeScript, MDX
- **Framework**: Next.js 16 (Pages Router, static export with `output: 'export'`)
- **UI Library**: @aws-amplify/ui-react
- **Testing**: Jest + @testing-library/react
- **Styling**: SCSS modules (imported via `src/styles/styles.scss`)
- **Package Manager**: Yarn Berry

---

### 1. File & Directory Naming
- **PascalCase** for component directories and files: `GlobalNav/`, `MenuItem.tsx`, `CrossLink.tsx`
- **camelCase** for utility files: `findDirectoryNode.ts`, `getPageSection.ts`, `useCurrentPlatform.ts`
- **kebab-case** for page directories: `build-a-backend/`, `set-up-auth/`, `connect-to-API/`
- **SCSS** files use kebab-case: `global-nav.scss`, `feedback.scss`
- Component directories have `index.ts` barrel exports: `export { CrossLink } from './CrossLink';`
- Tests live alongside source in `__tests__/` directories

### 2. Page Structure
- All Gen2 pages live under `src/pages/[platform]/`
- **Backend pages**: `src/pages/[platform]/build-a-backend/<feature>/`
- **Frontend pages**: `src/pages/[platform]/frontend/<feature>/`
- **Gen1 pages**: `src/pages/gen1/[platform]/` — MUST NOT be modified
- Every page is an MDX file (`index.mdx`) that exports `meta`, `getStaticPaths`, and `getStaticProps`

### 3. Page Meta Pattern (Required on ALL pages)
```tsx
import { getCustomStaticPath } from '@/utils/getCustomStaticPath';

export const meta = {
  title: 'Page Title',
  description: 'Page description.',
  platforms: [
    'android',     // Keep alphabetically sorted
    'angular',
    'flutter',
    'javascript',
    'nextjs',
    'react',
    'react-native',
    'swift',
    'vue'
  ]
};

export const getStaticPaths = async () => {
  return getCustomStaticPath(meta.platforms);
};

export function getStaticProps(context) {
  return {
    props: {
      platform: context.params.platform,
      meta
    }
  };
}
```

- `platforms` array MUST only include platforms where the content is relevant
- `platforms` array MUST be alphabetically sorted
- Overview pages that show child cards use `getChildPageNodes(meta.route)` in `getStaticProps`

### 4. Section-Based Navigation Architecture

The docs use a section-based navigation system that filters content by category:

- **Sections**: `quickstart`, `backend`, `frontend`, `ui`, `hosting`, `reference`
- **Section config**: `src/data/sections.ts` — labels, subtitles, URL helpers
- **Directory tree**: `src/directory/directory.mjs` — source of truth for navigation hierarchy
- **Section tags**: Every node in `directory.mjs` has a `section` property (`'backend'`, `'frontend'`, `'quickstart'`, `'hosting'`, `'ui'`, `'reference'`)
- **Filtering**: `isNodeVisibleInSection()` in `sections.ts` — shared utility used by Menu, MenuItem, and Overview

#### Key files:
| File | Purpose |
|------|---------|
| `src/data/sections.ts` | Section definitions, `getSectionFromPath()`, `getDefaultPathForSection()`, `isNodeVisibleInSection()` |
| `src/utils/getPageSection.ts` | Directory tree walk for section detection + CrossLink feature route targeting |
| `src/directory/directory.mjs` | Hand-maintained navigation tree (source of truth) |
| `src/directory/generateDirectory.mjs` | Enriches directory.mjs with page metadata → writes `directory.json` |
| `src/components/GlobalNav/GlobalNav.tsx` | Top navigation bar with section tabs |
| `src/components/Menu/Menu.tsx` | Sidebar menu with section filtering |
| `src/components/Menu/MenuItem.tsx` | Recursive menu item with section-aware child filtering |
| `src/components/CrossLink/CrossLink.tsx` | Banner linking between backend ↔ frontend sections |
| `src/components/Overview/Overview.tsx` | Card grid for overview pages with section filtering |

#### Section tagging rules:
- Pages showing `defineAuth`, `defineData`, `defineStorage`, CDK config → `section: 'backend'`
- Pages showing client API calls (`signIn`, `query`, `uploadData`) → `section: 'frontend'`
- Overview pages visible in multiple sections → `section: 'both'` (only used if page appears in both backend and frontend sidebars)
- `'quickstart'` tagged on `how-amplify-works` and `start` sections (hidden from nav, shown on homepage)

### 5. Directory Tree (`directory.mjs`)

This is the source of truth for all navigation. When adding or moving pages:

1. Add/move the entry in `directory.mjs` with the correct `path` and `section` tag

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aws-amplify/docs](https://github.com/aws-amplify/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
