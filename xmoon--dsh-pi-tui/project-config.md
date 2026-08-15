---
trigger: always_on
description: dsh-pi-tui — a third-party TUI mode for DeepSeek Harness (`dsh`), built on a vendored fork of [pi-tui](https://github.com/MoonshotAI/kimi-code/tree/main/packages/pi-tui). Read this file before editing.
---

# AGENTS.md

dsh-pi-tui — a third-party TUI mode for DeepSeek Harness (`dsh`), built on a vendored fork of [pi-tui](https://github.com/MoonshotAI/kimi-code/tree/main/packages/pi-tui). Read this file before editing.

## Naming (hard rules)

Collision-avoidance is a deliberate choice: the official dsh project will plausibly ship its own `dsh-tui` / `@deepseek-ai/dsh-tui`, so nothing here may use that family.

| Thing | Name | Notes |
|---|---|---|
| Repository | `dsh-pi-tui` | repo root (this directory) |
| Profile (`dsh --profile`) | `pi-tui` | **Never `tui`** — that is reserved territory |
| Vendored fork package | `@xmoon76/pi-tui` | rescopped from `@moonshot-ai/pi-tui`; `private: true`, **never published** — bundled into the release package at build time |
| Bundle package | `@xmoon76/dsh-pi-tui` | the `dsh.bundle` patch layer; the **only** published package |
| Plugin row ids | `tui-startup`, `tui-app` | internal Loader ids, fine as-is |
| Startup service | `tuiStartup` (`TUI_STARTUP_SERVICE`) | |

## Repository layout

```
packages/pi-tui/    Vendored @moonshot-ai/pi-tui fork. The vendored version
                    and upstream commit live in ONE place —
                    packages/pi-tui/package.json `repository.note` (see
                    that field, never a copy in this file or README). Its
                    own AGENTS.md (kept from the fork) is the source of
                    record for the local divergence fixes and their
                    guarding tests; re-verify every entry on each
                    re-vendor. native/ prebuilds are NOT vendored; loading
                    degrades gracefully without them.
packages/dsh-pi-tui/   The dsh bundle (the only published package). cordis.patch.yml
                    inserts the startup row (parses `dsh --profile pi-tui` flags)
                    and the runner row (starts the TUI). src/tui-app.ts is the
                    testable surface core (terminal injected); src/theme.ts the
                    palette; demo.ts a standalone interactive demo. Builds with
                    tsdown into dist/, bundling @xmoon76/pi-tui (deps.onlyBundle)
                    so the tarball is self-contained; dist/ is gitignored —
                    build before install. The tarball is verified by
                    scripts/tarball-smoke.mjs (prepack builds+verifies,
                    postpack smokes the exact bytes; root `pnpm pack:release`).
```

## Key decisions (do not silently reverse)

1. **In-process bundle, not BFF client.** Like `dsh-headless`, the TUI runs inside the Cordis context and consumes `ctx.*` services directly. The web surface's remote RPC exists only because a browser cannot be in-process; a TUI has no such constraint. Remote attach via the apiproxy is **explicitly not planned** (removed from the roadmap).
2. **Vendored fork, not npm dependency.** `@moonshot-ai/pi-tui` is not published (npm 404). Vendored from the kimi-code fork (not upstream pi-mono) to keep its five local fixes: CJK wrap recursion guard, container width clamp, overwide-line truncation instead of throw, negative-width guards, per-frame processed-line reuse. Re-verify those on every re-vendor — the fork's AGENTS.md lists them with guarding tests.
3. **`TuiMainScreen`, not `TUI`.** In this fork the constructible entry is `TuiMainScreen` (main screen + scrollback, `mode: "regular"`); the README's `new TUI(...)` is stale upstream docs. `TuiAltScreen` is the alternative.
4. **Source exports, built artifacts.** Both packages build with tsdown (`dist/`); dsh-pi-tui bundles the vendored pi-tui fork (`deps.onlyBundle: ['@xmoon76/pi-tui']`, the kimi-code pattern) so the published tarball is self-contained. `exports` point at built files; neither `dist/` is committed — build before installing into a profile. Node 26 refuses type-stripping inside `node_modules` (`ERR_UNSUPPORTED_NODE_MODULES_TYPE_STRIPPING`), so a `.ts`-exporting package cannot load from a profile's node_modules.
5. **No native prebuilds.** darwin/win32 modifier-key addons are optional; the loader returns `undefined` on other platforms without attempting a load. Revisit only if modifier detection matters on macOS/Windows.
6. **`chalk` is a runtime dependency** of `dsh-pi-tui` (theme.ts lives in `src`, unlike pi-tui's tests-only chalk).
7. **Single-package release model.** `@xmoon76/pi-tui` is `private: true` and never published (same as `@moonshot-ai/pi-tui` in kimi-code); `@xmoon76/dsh-pi-tui` is the only registry package and carries the fork inside its dist. Its `dependencies` therefore list pi-tui's runtime deps (`marked`, `get-east-asian-width`) directly, and `@xmoon76/pi-tui` lives in `devDependencies` (build-time only).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XMoon/dsh-pi-tui](https://github.com/XMoon/dsh-pi-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
