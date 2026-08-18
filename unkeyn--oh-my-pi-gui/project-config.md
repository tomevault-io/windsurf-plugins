---
trigger: always_on
description: This repo contains multiple packages, but **`packages/coding-agent/`** is the primary focus. Unless otherwise specified, assume work refers to this package.
---

# Development Rules

## Default Context

This repo contains multiple packages, but **`packages/coding-agent/`** is the primary focus. Unless otherwise specified, assume work refers to this package.

**Terminology**: When the user says "agent" or asks "why is agent doing X", they mean the **coding-agent package implementation**, not you (the assistant). The coding-agent is a CLI tool — questions about its behavior refer to code in `packages/coding-agent/`, not your current session.

### Package Structure

| Package                 | Description                                          |
| ----------------------- | ---------------------------------------------------- |
| `packages/ai`           | Multi-provider LLM client with streaming support     |
| `packages/catalog`      | Model catalog: bundled models.json, provider descriptors, model identity/classification |
| `packages/agent`        | Agent runtime with tool calling and state management |
| `packages/coding-agent` | Main CLI application (primary focus)                 |
| `packages/tui`          | Terminal UI library with differential rendering      |
| `packages/natives`      | Bindings for native text/image/grep operations       |
| `packages/stats`        | Local observability dashboard (`omp stats`)          |
| `packages/utils`        | Shared utilities (logger, streams, temp files)       |
| `crates/pi-natives`     | Rust crate for performance-critical text/grep ops    |

**Catalog import convention**: code in this repo imports catalog *values* (bundled models, model-thinking helpers, identity, descriptors, model manager/cache) from `@oh-my-pi/pi-catalog/<module>` — never via `@oh-my-pi/pi-ai`. The pi-ai barrel re-exports only the model/effort *types* its own signatures use (`Model`, `Api`, `ThinkingConfig`, `Effort`, …); type-only imports of those from `@oh-my-pi/pi-ai` are fine.

## GitHub

Unless user tells you exactly what to write:
- **Never comment on GitHub** (issues, PRs, discussions).
- **Never create issues on GitHub**.

## OMP and Desktop Ownership

The upstream runtime is in `packages/ai`, `packages/catalog`, `packages/agent`,
and `packages/coding-agent`. The custom Desktop layer is `packages/desktop`,
`crates/pi-desktop`, and `scripts/start-desktop.ps1`. Custom provider pools,
endpoint discovery, and Desktop RPC/session files cross that boundary and must
be merged deliberately during an OMP update.

For an OMP update, use a separate migration branch/worktree, record the
upstream tag, merge source changes first, then regenerate generated catalogs and
lockfiles. Never overwrite `.omp/skills`, `.omp/extensions`, plugins, or user
data. Never edit `packages/catalog/src/models.json` directly; change its
descriptors/resolvers and regenerate it.

The Desktop model picker must consume OMP's registry/RPC model projection and
the shared catalog classification. Do not add a second provider/model allowlist
to Desktop. Provider-specific models belong in catalog discovery; the picker
should only apply capability and availability rules exposed by OMP.

Validate an update with `bun install`, `bun check`, focused package type checks,
`cargo check -p pi-desktop`, and the Desktop frontend/Tauri build. If the native
pipeline changes, update `scripts/start-desktop.ps1` in the same migration and
verify a clean first-run build on a machine with no cached native artifact.

## Plugin Settings and Secrets

- Declare plugin configuration in `package.json#omp.settings`; use `secret: true` for tokens, cookies, passwords, and authenticated sessions, plus `multiline: true` for long values.
- Extensions read only their own effective settings through `ctx.pluginSettings`. Never read plugin lockfiles or Desktop storage directly.
- Desktop plugin settings are manifest-driven. Add a reusable schema/input capability instead of hard-coding one plugin's credential fields.
- Secret settings are write-only at UI/RPC boundaries: expose schema and configured state, never current/default/layer values, logs, transcript text, or reflected errors.
- Treat `secret: true` as disclosure control, not at-rest encryption. Persisted plugin settings are user-scoped and outside the project; use an environment fallback or a dedicated credential broker/keychain when encrypted storage is required.
- Model source privacy explicitly. Public requests must never receive private cookies or local-browser state; private requests must be opt-in, exact-host scoped, local-only, and fail closed when credentials are missing.
- Use `.omp/skills/author-omp-plugin` when creating or changing an OMP plugin that needs configuration or authenticated network access.

## Code Quality

- No `any` unless absolutely necessary.
- **NEVER use `ReturnType<>`** — use the actual type name.
- **NEVER use inline imports** — no `await import()`, no `import("pkg").Type` in type positions, no dynamic type imports. Always top-level.
- Check `node_modules` for external API types instead of guessing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unkeyn/oh-my-pi-gui](https://github.com/unkeyn/oh-my-pi-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
