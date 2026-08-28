---
trigger: always_on
description: `herdr-mcp` has distinct source, build, installed, active-runtime, and user-entry identities. Treat them as separate objects at all times.
---

# AGENTS.md

## Binary and runtime ownership

`herdr-mcp` has distinct source, build, installed, active-runtime, and user-entry identities. Treat them as separate objects at all times.

| Layer | Canonical location | Meaning | Allowed use |
| --- | --- | --- | --- |
| Source checkout | repository/worktree | Editable source and scripts | Development only |
| Build artifact | `target/debug/herdr-mcp` or `target/release/herdr-mcp` | Ephemeral Cargo output | Tests, local candidate runs, release preparation |
| Installed generation | `~/.config/herdr-mcp/runtime/generations/rust-<content-id>/herdr-mcp` | Immutable installed binary | Service activation, update, rollback |
| Active runtime | `~/.config/herdr-mcp/runtime/current/herdr-mcp` | Managed symlink to the active installed generation | Production launchd target and authoritative runtime CLI |
| User CLI | `~/.local/bin/herdr-mcp` | Stable command users invoke | User-facing control entrypoint; see migration note below |

### Hard rules

1. Never use a repository build artifact as the production service binary. `target/*/herdr-mcp` may be rebuilt, deleted, or changed by branch switches.
2. `dev.herdr-mcp.server` must execute the stable active-runtime path `~/.config/herdr-mcp/runtime/current/herdr-mcp`. Do not point launchd at a checkout, worktree, `target/`, `bin/`, or a fixed generation path.
3. Installed generations are immutable. Create a new content-addressed generation, validate it, then atomically switch `runtime/current`. Never overwrite an existing generation in place.
4. Do not infer the active runtime from Git `HEAD`, the current worktree, Cargo output, or a process-name heuristic. Query the active runtime and the exact launchd label.
5. Do not use legacy checks such as `pgrep -f "dist/server.js"` or `pkill -f "dist/server.js"` for Rust service lifecycle decisions.
6. Build/test operations must not modify `runtime/current`, installed generations, launchd, or the user CLI unless the task explicitly performs an install/update/cutover.
7. Rollback must reactivate a previously installed managed generation using recorded service state. Do not rebuild a binary as part of rollback.
8. Keep credentials out of source, Git history, CLI diagnostics, AGENTS.md, and non-secret state records. Preserve existing service credentials during generation changes.
9. Any change to the installer, updater, CLI, release path, or service manager must preserve these ownership boundaries and include regression coverage for them.

### Validation ownership and test selection

The native `herdr-mcp` service/runtime is Rust (`crates/herdr-mcp`). The repository still intentionally contains JavaScript/TypeScript for the browser extension, Cloudflare Edge, documentation/site generation, compatibility paths, cross-language contract tests, and the still-Node production Link path until G5 cutover. The existence of `package.json`, `src/`, or `dist/` does **not** make Node the owner of the `herdr-mcp` runtime.

Choose validation by the component being changed. Do not substitute one lane for another:

| Change surface | Authoritative validation | Notes |
| --- | --- | --- |
| Rust runtime, MCP, service manager, updater, native host, Rust Link/candidate code | `cargo fmt --check`; `cargo clippy --workspace --all-targets --all-features -- -D warnings`; `cargo test --workspace` | This is the Rust runtime gate used by `.github/workflows/ci.yml`. `npm test` is never evidence that this lane passed. |
| Browser extension (`extension/**`) | Targeted `node --test ...` for the changed modules, `node tests/manual/extension_smoke.mjs`, and `node tests/manual/background_bind_test.mjs` when background/binding behavior changes | Prefer targeted extension tests during iteration. Run the full JS/TS CI lane only when cross-cutting compatibility coverage is needed. |
| Cloudflare Edge / JS public-contract compatibility | `npm run build` followed by the relevant Node tests and `npm run test:edge` | Edge remains a JS/TS validation lane even though the local service runtime is Rust. |
| Legacy/compatibility TypeScript under `src/**`, generated `dist/**`, or cross-language contracts | `npm run build` before Node tests; run both Node and Rust gates when the contract is shared | Generated `dist/**` is a test/build artifact, not production runtime identity. |

For a full local reproduction of the GitHub JavaScript/TypeScript `test` job, preserve the CI ordering:

```bash
npm ci
npm run build
npm run build:site
scripts/ci-herdr-runtime.sh start
npm test
npm run test:edge
node tests/manual/extension_smoke.mjs
scripts/ci-herdr-runtime.sh stop
```

`scripts/ci-herdr-runtime.sh` starts a pinned headless **Herdr** server used by transport/regression tests; it does not start a Node `herdr-mcp` production runtime.

Never run repository-wide `npm test` in a fresh worktree and interpret the result without first building `dist/`. Several retained compatibility tests import `dist/*.js`; `ERR_MODULE_NOT_FOUND` for `dist/...` after a direct `npm test` is normally a missing-build precondition, not a Rust runtime regression. Do not report it as a Rust service failure.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whshang/herdr-mcp](https://github.com/whshang/herdr-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
