---
trigger: always_on
description: Observability SDK for web applications built on OpenTelemetry. Captures Spans (traces) and Logs to help debug and monitor user experiences.
---

# Embrace Web SDK

Observability SDK for web applications built on OpenTelemetry. Captures Spans (traces) and Logs to help debug and monitor user experiences.

## Behavior Guidelines

**NEVER**:

- Reference change-history in code, comments, or commits (`updated`, `legacy`, `old`, `previously`, "renamed from"). Describing a freshly-created runtime entity as `new` (e.g. "the new span") is fine
- Name competing observability vendors in source or docs

**ALWAYS**:

- Review existing patterns and prefer editing an existing file over creating a new one
- After committing, update PR body if one exists

## Quick Reference

```bash
# Build all packages (turbo)
npm run build

# Lint with Biome (add :fix to auto-fix)
npm run lint
npm run lint:fix

# Typecheck + ESLint baseline (tsc + eslint --max-warnings 0)
npm run check

# Validate built artifacts (es-check, bundle size, ESM/CJS split); run after build
npm run validate

# Run demo dev server (http://localhost:4847)
npm run dev
```

Run `lint`/`check` from the repo root so turbo and Biome cover every workspace. Scoping them to one package leaves the others unchecked.

## Architecture

Turbo + npm-workspaces monorepo (`packages/*`, `demo/*`, `server`, `tests/integration`). The published SDK is `packages/web-sdk`, and the source layout below is rooted there.

### Source Layout (`packages/web-sdk/src/`)

```
api-*/          Public APIs with no-op defaults (traces, logs, sessions, users, page)
managers/       Concrete implementations (EmbraceTraceManager, EmbraceLogManager, etc.)
processors/     Span/Log processing chain (scrubbing, batching, session correlation)
exporters/      OTLP serialization for Embrace backend
instrumentations/  Auto-capture plugins (web-vitals, clicks, rage-click, navigation, exceptions, etc.); fetch/XHR use upstream OTel instrumentations
sdk/            Entry point (initSDK) and configuration
transport/      HTTP transport with retry logic
```

### Key Patterns

**Proxy/No-Op Pattern**: Public APIs (`trace`, `log`, `session`, `user`, `page`) default to no-ops until SDK initializes. Safe to call before initialization.

**Processor Chain**: Spans and logs flow through processors that add attributes, scrub sensitive data, and batch for export.

**Layered Architecture**: `api-*` (interfaces/proxies) → `managers/` (implementations) → `processors/exporters/` (infrastructure)

### Distribution

| Format | Target | Use Case                                |
| ------ | ------ | --------------------------------------- |
| ESM    | ES2022 | npm package (import)                    |
| CJS    | ES2022 | npm package (require)                   |
| IIFE   | ES6    | CDN script tag (`window.EmbraceWebSdk`) |

## Code Conventions

### TypeScript

- **Import extensions required**: Always use `.ts` extension in imports
- **Type imports**: Use `import type` for type-only imports
- **No re-exports**: `export * from` forbidden (use explicit exports)

### Naming

- **Classes**: PascalCase with `Embrace` prefix for implementations
- **Static-only classes**: Used for API singletons (OTel convention)
- **Attributes**: `emb.` prefix for Embrace-specific
- **Session terminology**: never write bare "session". Use **user session** or **session part**, in prose and in identifiers (`UserSession*`, `SessionPart*`). They are distinct concepts
- **No abbreviations**: spell identifiers out: `terminationInfo` not `termInfo`, `timestamp` not `ts`
- **`Id` casing**: never mix `ID` and `Id` suffixes. Pick one and stay consistent

### File Organization

- **Co-located tests**: `*.test.ts` next to implementation
- **Index files**: Each module has `index.ts` for exports
- **Types**: Separate `types.ts` files for interfaces

### Comments

- **TSDoc** on public APIs. **WHY-only** on internals (the code shows the what)
- No performance trivia, and no bare "§X.Y" / "per section N" spec reference without a link (drop the marker if there is none)

## Testing

### Unit Tests

Framework: @web/test-runner + Playwright + Mocha + Chai

```bash
npm run test                  # Headless (from packages/web-sdk/)
npm run test:manual           # Browser with DevTools
npm run test:watch            # Watch mode
```

**Run a single test file** (paths are workspace-relative, i.e. relative to `packages/web-sdk/`):

```bash
npx turbo run test --filter=@embrace-io/web-sdk -- --files "src/utils/throttle.test.ts"
```

### Integration Tests

Test SDK against bundlers (Webpack 5, Vite 6/7, Next.js 15/16):

```bash
npm run build                             # Build first
npm run test:integration                  # Run tests
npm run test:integration:update-golden    # Update golden files
```

Golden files are nondeterministic: instance IDs, trace/span IDs, and timestamps regenerate on every run. Never hand-edit them. Regenerate with the update-golden script and review the semantic diff.

### Conventions

- Hardcode contract / wire-format values in tests (attribute keys, payload shapes). Importing them defeats the test. Importing purely-internal constants is fine
- No test-only escape hatches (`_setX`/`_resetX`). If defensive code is unreachable, delete it rather than expose a hook to cover it

## Constraints


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [embrace-io/embrace-web-sdk](https://github.com/embrace-io/embrace-web-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
