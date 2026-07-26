---
trigger: always_on
description: - **Do NOT commit any change** unless the user says "commit", "push", "create a PR", or similar explicit instruction.
---

# DocKit - AI Agent Guidelines

## ⚠️ Git: Never commit or push unless the user explicitly asks

- **Do NOT commit any change** unless the user says "commit", "push", "create a PR", or similar explicit instruction.
- Stack changes in the working tree. Batch related fixes together. Let the user decide when to commit.
- Multiple small commits on a PR create noise and make review harder.
- If you're unsure whether a change is commit-worthy, discuss it with the user first.

---

## Project Overview

DocKit is a Tauri v2 + Vue 3 + TypeScript desktop client for NoSQL databases (Elasticsearch, OpenSearch, DynamoDB).

**Tech Stack**: Tauri v2 (Rust backend), Vue 3 (Composition API), TypeScript, Monaco Editor, shadcn-vue + UnoCSS (styling).

**Key Directories**:

- `src/` - Vue frontend application
- `src-tauri/` - Rust backend (Tauri)
- `src/composables/` - Vue composables (reusable logic)
- `src/views/` - Page-level components
- `src/components/` - Shared UI components
- `src/components/ui/` - shadcn-vue components

---

## Coding/Architecture Guidelines

### Functional TypeScript

- Define functions as `const xxx = (...) => ...`. Prefer **functional decomposition** over OOP.
- **Avoid classes** unless strictly necessary.

### Declarative/Functional Collection Handling

- Replace `for`/`while` loops with `map`, `filter`, `find`, `some`, `every`, `reduce`, `flatMap` (and `sort` when appropriate).
- Favor pipeline-style transformations over step-by-step imperative logic.

### Immutability

- Avoid in-place mutation (`push`, `splice`, mutating objects/arrays, shared mutable state).
- Instead, return new arrays/objects and model changes as explicit state-transform functions (e.g., reducers).

### Pure Functions

- Keep functions small, composable, and side-effect-free where possible.
- If effects are required (I/O, logging), isolate them at the boundaries and keep core logic pure.

### Types

- Prefer `type`/`enum` over `interface` where possible.
- Use `type` when it can fully replace an `interface`.

### Module Boundaries

- Each module should export **only** via its `index.ts`.
- Avoid deep imports (e.g., import from `src/composables/useKeyboardShortcuts` → use `src/composables`).

### Export Discipline

- Only export functions/types/constants that are used outside the module.

### Provider-Agnostic Design

- Keep provider-agnostic abstractions and follow clean separation of concerns.

### Comments and Documentation

- Use as few inline comments as possible.
- Behavior should be clear from tests and naming.

---

## Styling Conventions

- **UnoCSS** for utility-first atomic CSS (loaded via `virtual:uno.css`)
- **shadcn-vue** for UI components (Radix Vue-based, headless)
- Theme tokens via CSS variables in `src/assets/styles/index.css`
- See `uno.config.ts` for UnoCSS presets and theme configuration

---

## Code Quality Tools

- **ESLint**: `npm run lint:check` (check violations), `npm run lint:fix` (auto-fix)
- **TypeScript**: Strict mode enabled (`noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch`)
- **Prettier**: Single quotes, 100 char width, 2-space indent, semicolons, arrow parens `avoid`

---

## Build Commands

```bash
npm install        # Install dependencies
npm run tauri dev  # Compile and run (development)
npm run lint:check # Check ESLint violations
npm run lint:fix   # Auto-fix ESLint issues
npx tsc --noEmit   # TypeScript type check
```

---

## Shared Utilities — Impact Analysis Required

Modifying any of these requires the systemic impact analysis defined in the global AGENTS.md (Rule 5). Before changing, trace all call sites and verify no silent regressions.

### Rust (src-tauri/)

| Utility | Location | Callers | Failure Mode If Changed Carelessly |
|---|---|---|---|
| `create_http_client()` | `src/common/http_client.rs` | 10 callers across 7 files | A blanket timeout breaks streaming LLM responses, long ES queries, file downloads |
| `get_base_url()` | `src/agent/provider_adapter.rs` | `harness.rs`, `loop_runner.rs`, `config.rs` | Wrong base URL for any provider → all API calls fail |
| `build_headers()` | `src/agent/provider_adapter.rs` | `harness.rs`, `loop_runner.rs`, `config.rs` | Missing/wrong auth headers → all authenticated requests fail |
| `map_to_api_compatibility()` | `src/agent/provider_adapter.rs` | `harness.rs` | Wrong routing → wrong auth, wrong endpoint, wrong model parser |
| `resolve_spec()` | `src/agent/model_registry.rs` | `compact.rs`, `loop_runner.rs`, `conversation.rs` | Wrong context window → token budget miscalculation → premature compaction or overflow |

### TypeScript (src/)

| Utility | Location | Callers | Failure Mode If Changed Carelessly |
|---|---|---|---|
| `PROVIDER_PRESETS` | `src/store/appStore.ts` | `defaultProviderConfigs()`, `createProviderConfig()`, UI dropdown | Missing/wrong `apiCompatibility` → provider silently broken |
| `storeApi` | `src/datasources/storeApi.ts` | All Pinia stores | Save/load failure → settings lost on restart |
| `chatBotApi` | `src/datasources/chatBotApi.ts` | `aigc.vue`, `appStore.ts` | Wrong provider string → Tauri command maps to wrong API path |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geek-fun/dockit](https://github.com/geek-fun/dockit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
