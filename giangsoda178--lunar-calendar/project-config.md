---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development
bin/dev                      # Start Rails + Vite with HMR (port 8888)

# Testing
bin/rspec                    # All tests
bin/rspec spec/requests      # Request specs only
bin/ci                       # Run all CI checks locally

# Code Quality
pnpm lint:js && pnpm fix:js  # ESLint
pnpm format:fix              # Prettier
pnpm check                   # TypeScript/Svelte type checking
bin/rubocop -a               # Ruby linting with auto-fix

# Security
bin/brakeman                 # Static analysis
bin/bundler-audit            # Gem vulnerabilities

# Utilities
rake js:routes               # Generate JS routes after modifying config/routes.rb
```

## Architecture Overview

### Rails + Inertia.js + Svelte 5 Integration

Monorepo combining Rails 8.1 backend with Svelte 5 frontend via Inertia.js. No separate API layer - Inertia bridges server and client.

**Request Flow:**
1. Request hits Rails controller inheriting from `InertiaController`
2. Controller renders Inertia response: `render inertia: { users: users.as_json(...) }`
3. `app/frontend/entrypoints/inertia.ts` resolves Svelte component from `app/frontend/pages/`
4. Component receives props and renders with `PersistentLayout` wrapper

**Key Files:**
- `app/controllers/inertia_controller.rb` - Base controller with shared data (flash, auth)
- `app/frontend/entrypoints/inertia.ts` - Inertia app setup, page resolution, theme init
- `app/frontend/layouts/PersistentLayout.svelte` - Default layout with Toaster

### Frontend Structure (`app/frontend/`)

```
components/ui/    # Reusable UI components
pages/           # Inertia page components (maps to controller actions)
layouts/         # Page layouts
runes/           # Svelte 5 runes (composables) - use .svelte.ts extension
routes/          # Auto-generated JS routes from Rails
types/           # TypeScript definitions
```

**Import Alias:** `@/*` maps to `app/frontend/*`

### Svelte 5 Patterns

- Use `$props()` for component props with TypeScript interfaces
- Use `.svelte.ts` extension for runes files
- Flash messages: `useFlashSvelte()` rune + Svelte Sonner
- Theme: `useAppearance` rune with mode-watcher
- Available runes: `use-appearance.svelte.ts`, `use-flash.svelte.ts`, `is-mobile.svelte.ts`, `use-initials.ts`

### Styling

TailwindCSS 4 with dark mode support. CSS-first configuration (no tailwind.config.js). Use `dark:` variants for dark mode styles.

## Skills Activation

Activate skills based on task type:

| Task Type | Skill to Activate |
|-----------|-------------------|
| **Backend** | |
| Rails controllers, API design, authentication | `backend-development` |
| Database schema, migrations, query optimization | `databases` |
| **Frontend (Svelte 5)** | |
| Svelte components, TailwindCSS, responsive design | `ui-styling` |
| Premium UI/UX design, design systems | `ui-ux-pro-max` |
| Distinctive interfaces, bold aesthetics | `frontend-design` |
| Production-grade UI with real photos/prompts | `frontend-design-pro` |
| Beautiful interfaces, design principles | `aesthetic` |
| **Quality & Process** | |
| Feature planning, architecture decisions | `planning` |
| Bug investigation, root cause analysis | `debugging` |
| Code quality review, before merge | `code-review` |
| Complex multi-step reasoning | `sequential-thinking` |
| Problem decomposition, stuck resolution | `problem-solving` |
| **Media & Assets** | |
| Image/video/audio analysis, generation | `ai-multimodal` |
| Image editing, format conversion, backgrounds | `media-processing` |
| Browser automation, screenshots, performance | `chrome-devtools` |
| **Research & Docs** | |
| Package/library documentation lookup | `docs-seeker` |
| Codebase packaging for AI analysis | `repomix` |
| Technical research, solution exploration | `research` |
| **Infrastructure** | |
| Docker, deployment, cloud infrastructure | `devops` |
| Payment integrations (Stripe, Polar, SePay) | `payment-integration` |

**Frontend UI workflow:**
1. Design: `aesthetic` or `frontend-design` → extract design guidelines
2. Generate assets: `ai-multimodal` → generate images, `media-processing` → edit
3. Implement: `ui-styling` or `ui-ux-pro-max` → build Svelte components
4. Verify: `chrome-devtools` → screenshot → `ai-multimodal` → compare to design

**Feature development workflow:**
- `planning` → `backend-development` + `databases` + `ui-styling` → `code-review`

**Bug fix workflow:**
- `debugging` (root cause) → fix → `code-review`

## Documentation Management

Keep docs in `./docs` folder and update them:

```
./docs
├── project-overview-pdr.md   # Project goals, features, requirements
├── code-standards.md         # Coding conventions and best practices
├── codebase-summary.md       # High-level overview and directory structure
├── system-architecture.md    # Technical architecture and design decisions
├── design-guidelines.md      # Design patterns (create when needed)
├── deployment-guide.md       # Deployment procedures (create when needed)
└── project-roadmap.md        # Development phases (create when needed)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Giangsoda178/lunar-calendar](https://github.com/Giangsoda178/lunar-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
