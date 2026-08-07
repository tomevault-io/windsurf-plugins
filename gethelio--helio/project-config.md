---
trigger: always_on
description: Helio is an open-source MCP governance proxy. It sits between AI agents and external tools, applying policies, enforcing limits, requiring evidence, routing approvals, and recording an audit trail without modifying the agent or the tools.
---

# Helio

Helio is an open-source MCP governance proxy. It sits between AI agents and external tools, applying policies, enforcing limits, requiring evidence, routing approvals, and recording an audit trail without modifying the agent or the tools.

**GitHub org:** `gethelio` · **npm scope:** `@gethelio` · **PyPI:** `helio` · **Config file:** `helio.yaml`

## Architecture

Hybrid model: MCP proxy (core) + thin client SDKs (Python first, TypeScript planned).

```
Agent → Helio Proxy (policies, audit, limits, approvals) → Upstream MCP Server → External Tools
              ↑ optional sideband
         Python SDK (evidence context, dependency annotations)
```

- The proxy intercepts `tools/call` JSON-RPC methods. All other MCP methods pass through untouched.
- The SDK communicates with the proxy via sideband HTTP. It NEVER makes governance decisions.
- SDK constraint: must stay under 500 lines per language. Governance logic belongs in the proxy.

## Monorepo Structure

pnpm workspaces. Three packages:

```
packages/proxy/         → @gethelio/proxy (Node.js, Hono, TypeScript)
packages/dashboard/     → @gethelio/dashboard (React 19 + Vite + Tailwind CSS, internal workspace package bundled into proxy dist; not published)
packages/python-sdk/    → helio on PyPI (Python, <500 lines)
```

Supporting directories: `examples/`, `docs/`, `docker/`, `scripts/`, `.github/`

## Tech Stack

- **Runtime:** Node.js 22+, TypeScript 5.x (strict mode), pnpm 11 workspaces
- **HTTP:** Hono (`@hono/node-server`)
- **Validation:** Zod v4 (single source of truth for helio.yaml schema)
- **YAML:** js-yaml
- **Input matchers:** custom dot-path (`$.field`) conditions with `eq`/`neq`/`gt`/`gte`/`lt`/`lte`/`contains`/`regex` operators — no JSONPath library
- **Globbing:** picomatch (tool-name patterns); safe-regex2 (validates operator-supplied `regex` conditions to reject catastrophic backtracking)
- **Database:** better-sqlite3 (SQLite)
- **File watching:** chokidar
- **CLI:** Commander.js
- **Dashboard:** React 19, React Router 7, Vite, Tailwind CSS 4, Recharts 3
- **Slack:** @slack/web-api
- **Testing:** Vitest (JS), pytest (Python SDK)
- **Linting:** ESLint + Prettier
- **Build:** tsup (proxy), Vite (dashboard)
- **CI:** GitHub Actions; Husky pre-commit hooks (secret scan + lint/format/typecheck + docs-drift check)

## Commands

```bash
pnpm install          # Install all workspace dependencies
pnpm dev              # Start proxy (tsup --watch) + dashboard (vite) in parallel
pnpm build            # Build proxy release artifact (includes dashboard asset bundling)
pnpm test             # Run all tests — Vitest (JS, all packages) + pytest (Python SDK)
pnpm test:js          # Vitest only (proxy + dashboard)
pnpm lint             # ESLint across all packages
pnpm format:check     # Prettier format check (runs in CI)
pnpm typecheck        # tsc --noEmit across all packages
pnpm secrets:scan     # gitleaks-based full repository secret scan
```

Package-specific (run from package directory or use `--filter`):

```bash
pnpm --filter @gethelio/proxy test
pnpm --filter @gethelio/proxy build
pnpm --filter @gethelio/dashboard dev
```

## Security Standards

Helio sits in the critical path between AI agents and external systems. A vulnerability in the proxy is a vulnerability in every downstream tool it governs. Security is not optional — it is the product.

### Input Validation

- **Validate all external data with Zod.** Config files, HTTP request bodies, SDK payloads, query parameters, JSON-RPC messages — every external boundary uses Zod schemas. Never trust unvalidated input.
- **Never pass raw user input to shell commands, SQL queries, or file paths.** All audit queries use prepared statements. Config file paths are resolved and validated before access.
- **Sanitize log output.** Never log raw request bodies that may contain secrets, credentials, or PII. Log tool names, decisions, and metadata — not tool arguments or downstream responses unless explicitly configured via `audit.include_responses`.

### Dependency Security

- **Pin all npm dependencies to exact versions** (no `^` or `~`). The `.npmrc` enforces `save-exact=true`. This prevents supply chain attacks via malicious patch releases.
- **Every dependency must be justified** in `DEPENDENCIES.md` with a clear rationale. If you add a dependency, document why it's necessary and why alternatives were rejected.
- **Minimize the dependency surface.** Prefer standard library functionality over third-party packages. Every new dependency is an attack vector. The bar for adding a production dependency is high.
- **Run `pnpm audit --audit-level=high`** in CI. PRs with known high-severity vulnerabilities in dependencies must not merge.
- **Dependabot is configured** for weekly version bump PRs. Review these for breaking changes before merging.

### Transport & Network Security


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gethelio/helio](https://github.com/gethelio/helio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
