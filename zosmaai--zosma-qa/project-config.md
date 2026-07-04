---
trigger: always_on
description: This file is automatically loaded by OpenCode, Pi, Claude Code, and any agent that follows the `AGENTS.md` standard. It gives you the full context you need to work on this project without guessing.
---

# zosma-qa — Agent Instructions

This file is automatically loaded by OpenCode, Pi, Claude Code, and any agent that follows the `AGENTS.md` standard. It gives you the full context you need to work on this project without guessing.

---

## What This Project Is

**zosma-qa** is an open-source, zero-config QA platform. Users run `npx zosma-qa init`, choose TypeScript or Python, and get a fully working Playwright test setup with AI agent support out of the box.

The project ships three publishable npm packages plus a working example test suite that runs against the live [https://www.zosma.ai](https://www.zosma.ai) site.

---

## Monorepo Structure

```
zosma-qa/                          ← pnpm workspace root
├── packages/
│   ├── core/                      ← @zosmaai/zosma-qa-core  (types, config, discovery)
│   ├── playwright/                ← @zosmaai/zosma-qa-playwright  (runner + base config)
│   └── cli/                       ← @zosmaai/zosma-qa-cli  (interactive CLI)
├── templates/
│   ├── playwright/                ← reference scaffold for TypeScript projects
│   └── playwright-python/        ← reference scaffold for Python projects
├── examples/zosma-ai/             ← working example: tests against zosma.ai
│   ├── playwright.config.ts
│   ├── tests/
│   │   ├── seed.spec.ts
│   │   ├── home.spec.ts
│   │   ├── about.spec.ts
│   │   ├── openzosma.spec.ts
│   │   └── contact.spec.ts
│   └── specs/zosma-site.md        ← AI planner spec
├── tests/                         ← root test directory (users put tests here)
├── specs/                         ← AI planner markdown output
├── .github/
│   ├── agents/                    ← Playwright agent definitions
│   └── workflows/                 ← CI/CD (ci.yml, release.yml)
└── docs/
    ├── ARCHITECTURE.md
    ├── GETTING_STARTED.md
    └── VISION.md
```

---

## Package Dependency Graph

```
@zosmaai/zosma-qa-cli
  ├── @zosmaai/zosma-qa-core
  └── @zosmaai/zosma-qa-playwright
        └── @zosmaai/zosma-qa-core
```

`@playwright/test` is a **peer dependency** of `@zosmaai/zosma-qa-playwright` — do not add it as a direct dependency.

---

## Tech Stack

| Layer | Tool |
|---|---|
| Language | TypeScript 5, strict mode, compiled to **CommonJS** |
| Package manager | pnpm 9 (workspaces) |
| Test runner (TS) | Playwright 1.52+ |
| Test runner (Python) | pytest-playwright (dispatched via CLI) |
| Python env manager | uv (preferred) or venv + pip |
| CLI prompts | `@inquirer/prompts` (NOT the old `inquirer` package) |
| Build | `tsc` per package, `tsconfig.base.json` extended by each package |
| Lint + Format | **Biome** (`pnpm lint` / `pnpm lint:fix` / `pnpm format`) |
| Node | >=18 |

---

## Essential Commands

```bash
# Install all dependencies
pnpm install

# Build all three packages (run from repo root)
pnpm build

# Run the example test suite (zosma.ai live site, 32 tests)
pnpm test:examples

# Run only a specific spec file
cd examples/zosma-ai && pnpm exec playwright test tests/home.spec.ts

# Type-check all packages
pnpm typecheck

# Open the Playwright HTML report
pnpm report

# Lint the entire repo (Biome)
pnpm lint

# Lint and auto-fix
pnpm lint:fix

# Format all files
pnpm format

# Clean build artifacts and test results
pnpm clean
```

Each package also has its own `pnpm build` / `pnpm typecheck` scripts.

---

## Code Conventions

- **TypeScript strict mode** everywhere — no `any`, no implicit `any`
- **CommonJS output** (`"module": "commonjs"` in all `tsconfig.json` files)
- All source files live under `src/`, compiled output goes to `dist/`
- `tsconfig.base.json` at the root is extended by every package
- Biome handles all linting and formatting — run `pnpm lint:fix` before committing
- Imports use `@zosmaai/zosma-qa-core` workspace aliases, not relative paths between packages
- All Node.js built-in imports use the `node:` prefix (`node:fs`, `node:path`, etc.)

### `@inquirer/prompts` — important note

The CLI uses `@inquirer/prompts` (the new modular API). The old `inquirer` + `@types/inquirer` pattern is **not used**. The `checkbox` validate callback receives `readonly NormalizedChoice<T>[]`, not `T[]`.

---

## Config System (two files, both optional)

### TypeScript projects (`plugins: ['playwright']`)

```typescript
// zosma.config.ts
import { defineConfig } from '@zosmaai/zosma-qa-core';
export default defineConfig({ baseURL: 'https://example.com', browsers: ['chromium'] });

// playwright.config.ts — extends @zosmaai/zosma-qa-playwright base config
import { defineConfig } from '@zosmaai/zosma-qa-playwright';
export default defineConfig({ use: { baseURL: 'https://example.com' } });
```

### Python projects (`plugins: ['pytest']`)

```typescript
// zosma.config.ts — same shape; only the plugin value differs
import { defineConfig } from '@zosmaai/zosma-qa-core';
export default defineConfig({ plugins: ['pytest'], baseURL: 'https://example.com', browsers: ['chromium'] });
```

```toml
# pyproject.toml — pytest-playwright native config (no playwright.config.ts)
[tool.pytest.ini_options]
testpaths = ["tests"]
base_url = "https://example.com"
addopts = "--browser chromium"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zosmaai/zosma-qa](https://github.com/zosmaai/zosma-qa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
