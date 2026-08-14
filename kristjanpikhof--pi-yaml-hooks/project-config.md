---
trigger: always_on
description: Agent contract for `pi-yaml-hooks`. Facts only; tutorials in `docs/`.
---

# AGENTS.md

Agent contract for `pi-yaml-hooks`. Facts only; tutorials in `docs/`.

## Facts

- Runtime: one `pi-yaml-hooks` package with native Pi and OMP extension entries. Type-only `pi-yaml-hooks/types` exports `HookConfig`, `HookEvent`, `BashHookContext`, `SessionDeletedReason`; smoke: `src/public-types-smoke.test.ts`.
- Package engine: Node `>=22.19.0`; macOS/Linux only (`src/pi/register-adapter.ts` guards win32). OMP runtime smoke is verified with Bun `1.3.14`.
- Pi host peers: `@earendil-works/pi-coding-agent` + `@earendil-works/pi-tui` as optional `*`; dev-tested at `0.84.1`; compatibility matrix covers exact `0.74.0` + `0.79.3` + `0.80.10` + `0.84.1`. Current live Pi evidence is exact `0.84.1`, including `--no-builtin-tools`; this is not a broader range claim.
- OMP host peers: `@oh-my-pi/pi-coding-agent` + `@oh-my-pi/pi-tui` as optional `*`; compile/runtime-tested at exact `17.0.1` + `17.2.12`.
- No direct `@mariozechner/*` host deps. Transitive `@mariozechner/clipboard` may appear under Pi SDK packages in `package-lock.json`.
- `package-lock.json` is canonical; no bun.lock.

## Structure

- `src/index.ts`: shared registration plus the default Pi factory; registers profile, policy, adapter, commands, autocomplete, diagnostics, and prompt support.
- `src/omp/index.ts`: OMP factory; gets and validates the active OMP agent directory before shared registration.
- `src/core/`: host-agnostic. `runtime.ts` owns state; `load-hooks.ts` is a barrel; hook loading is in `core/hooks/*`; dispatch/actions/path/async are in `core/runtime/*`. Never import `src/pi/*` or host SDK types from core.
- `src/pi/`: shared Pi-compatible adapter/register/lifecycle/registry, host event mappers, commands, autocomplete, diagnostics, prompt, `user_bash`, session lineage, unsupported policy; `adapter.ts` is a barrel.
- `extensions/`: native TS entrypoints. `extensions/pi-yaml-hooks/index.ts` selects Pi; `extensions/omp-yaml-hooks/index.ts` selects OMP; generic `extensions/index.ts` remains Pi-compatible.
- `examples/`: shipped: `pre-tool-developer-guards`, `post-tool-developer-feedback`, `README.md`. Repo-only: `atomic-commit-snapshot-worker`, snapshot helpers; not built-ins.
- `docs/`: `README.md` is the router; `setup.md` owns paths, trust, imports, and environment variables; `hooks-reference.md` owns the runtime contract; `examples.md` owns copyable YAML; `debugging-hooks.md` and `maintaining.md` cover operations.
- `scripts/`: test runner, SDK/host matrices, tail-log, Pi/OMP smoke helpers.
- `dist/`: generated; do not edit. `build`/`build:publish` regenerate extension stubs.

## Runtime contracts

- Built-ins: events `tool.before.*`, `tool.after.*`, `file.changed`, `session.{created,idle,deleted}`; actions `bash`, `tool`, `notify`, `confirm`, `setStatus`; commands `/hooks-{status,validate,trust,reload,tail-log}`.
- Diagnostics use host custom messages when available; prompt awareness runs at agent start.
- `command:` actions rejected. `tool:` gives the current matching Pi or OMP session a follow-up prompt; it does not execute tools or target other sessions.
- `runIn: main` rejected for non-`bash`; for bash it does not change process/session context. Prefer `scope` for main-vs-child routing.
- `action: stop` only affects `tool.before.*`; `async: true` + `action: stop` rejected at parse time and runtime warns once per source/runtime.
- `session.deleted` is best-effort; optional host `reason` is an opaque string forwarded to the internal envelope/debug telemetry, not a closed enum or matching field.
- UI actions gate on `ctx.hasUI` + method. RPC may expose UI; no-UI/headless degrades and `confirm` fails closed.
- `/hooks` autocomplete is TUI-only: register only when `ctx.mode === "tui"` or older SDKs omit `mode`, and `ctx.ui.addAutocompleteProvider` exists.
- `user_bash` opt-in: `PI_YAML_HOOKS_ENABLE_USER_BASH=1`.
- `tool_args` redacted by `sanitizeToolArgsForSerialization` before bash stdin (`src/core/runtime/actions.ts`).

## Paths, trust, imports

- Conditions: `matchesCodeFiles` legacy single-file; `matchesAnyPath` / `matchesAllPaths` only on `file.changed`, `session.idle`, `tool.after.*` when paths exist; pathless/non-mutating tools never match.
- Mutation paths come from `src/core/tool-paths.ts`: `write|edit|multiedit|patch|apply_patch|bash`; unknown tools are pathless.
- One global root config + one project root config; project discovery is repo/worktree-aware, not exact-cwd-only.
- Hook trust is separate from host package/project trust. Persistent trust uses the active Pi or OMP agent directory; OMP legacy `.pi` project fallback requires OMP trust and never inherits Pi trust. Entries are absolute canonical repo/worktree anchors.
- Shortcuts: `/hooks-trust`, or the opt-in per-process `PI_YAML_HOOKS_TRUST_PROJECT=1`.
- Imports: global-root needs `PI_YAML_HOOKS_ALLOW_GLOBAL_IMPORTS=1`; package needs `PI_YAML_HOOKS_ALLOW_PACKAGE_IMPORTS=1`; project imports must stay inside the trusted repo/worktree anchor unless `PI_YAML_HOOKS_ALLOW_PROJECT_IMPORTS_OUTSIDE_TRUST_ANCHOR=1`.

## Commands

- Setup/deps: `npm install`.
- TS: `npm run typecheck` after TS changes.
- Build: `npm run build` before direct `dist/**/*.test.js`; emits `dist/extensions/*`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KristjanPikhof/Pi-YAML-Hooks](https://github.com/KristjanPikhof/Pi-YAML-Hooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
