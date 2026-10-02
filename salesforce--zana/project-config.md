---
trigger: always_on
description: Guidance for working in this repo (Zana Command Center — an Electron + React + TS multi-project terminal hub).
---

# AGENTS.md

Guidance for working in this repo (Zana Command Center — an Electron + React + TS multi-project terminal hub).

## Worktrees

Do not create git worktrees by default. Work in the existing checkout.

If a worktree is explicitly requested, put it under `.worktrees/<branch-name>` inside this repository so repo instructions and tooling stay in the ancestor.

## Engineering Rules

Core rules. Rationale: `docs/review-consensus-2026-06.md`.

1. **The renderer is untrusted — main authorizes.** Validate any path / projectId / cwd in main before it grants access; renderer-side checks are advisory.
2. **Confine paths before trusting them.** A renderer- or agent-supplied path is only a trust anchor after `realpath`-matching a registered project (or a HOME/cloneRoot base).
3. **Subscribe long-lived emitters once, at app init** — never inside `createWindow()` (it re-runs). Release every subscription, timer, and per-session resource on its shutdown path.
4. **Shared-file writes are atomic and serialized** — tmp + uniquely-suffixed rename, and one in-process mutex for read-modify-write (or be strictly append-only).
5. **Keep heavy, unbounded work off the main event loop** — bound/`LIMIT`/paginate growing reads; an unbounded accumulating store needs a retention cap.
6. **Core never names a specific extension in logic** — concrete ids appear only in the `MAIN_MODULES` / `APP_MODULES` registration. `MAIN_MODULES` is empty; `APP_MODULES` registers `docs` (compiled library UI). In the RENDERER the `'zana'` module-id literal must appear NOWHERE in `apps/app/src/**` code. The source-text guard (`apps/app/src/__tests__/rule6-zana-literal.guard.test.ts`) scans comment-stripped renderer code and fails on ANY bare `'zana'`/`"zana"` token. The registration site is guarded by `apps/server/src/services/extensions/__tests__/core-extension-separation.guard.test.ts`.
7. **Promotion to a built-in is deliberate and bounded** — only when the broker can't grant the capability even scoped, and the trusted version (`builtinExec`/`builtinFetch`) is no weaker than its broker-gated twin (redirects, body cap, timeout).
8. **New or modified code needs at least 80% test coverage.** Cover meaningful branches and failure paths, not only line count. For Electron main/renderer seams, unit coverage alone is insufficient: add or update the relevant built-Electron E2E test. Before completion, run the focused tests and the production-boundary E2E required by any affected coupling note.
9. **PR monitoring means diagnose and repair, not only report.** After pushing a PR, watch its checks until complete. On failure, fetch job logs with `gh run view <run-id> --job <job-id> --log-failed`; for external checks, query `gh api repos/<owner>/<repo>/commits/<sha>/check-runs` then `gh api repos/<owner>/<repo>/check-runs/<id>/annotations` to get file, line, rule, and remediation. Fix actionable failures, run focused local verification, push, and repeat until every required check passes. Do not stop at an external failure summary when annotations are available.

## Native runtime and test isolation

- Never flip the installed SQLite addon between Node and Electron. `pnpm rebuild:electron` prepares the Electron ABI cache without replacing Node's binary. Product database connections use `createSqliteDatabase` (or `sqliteNativeBinding` for an existing constructor) from `@zana-ai/zcc-db`.
- Use `pnpm build` for production output and `pnpm test:e2e -- <spec>` for a private build. `pnpm test:e2e:only -- <spec>` snapshots an existing build. Direct Playwright uses the same isolation setup. Do not run `electron-vite build` directly or manually copy native binaries into the installed package.
- Dev output is `out-dev/`; production output is `out/`. The shared build lock protects preparation and snapshots, not running tests. Each Playwright invocation owns `e2e/.artifacts/runs/<id>`; never clear another run's files.
- E2E homes are already isolated by bootstrap. Never write or restore the real user's config from a fixture. See `docs/native-runtime-isolation.md` for the regression checks.

## Product Design Rules

- **Choose a layout per feature; do not expose the choice as a user preference.** A new panel is either a centered reading/configuration surface or a full-width workbench. Make that decision from the feature's task and information density, encode it in the panel's layout classes, and do not add a global "Centered / Full width" control to Settings.
- **Keep catalogues distinct by ownership.** ZCC-installed extensions are presented as **Plugins** in the ZCC Plugins hub. Codex's `~/.Codex/plugins` catalogue is an implementation-specific compatibility surface and must not appear as a competing Settings destination unless a user explicitly asks for it.
- **Sidebar folders are Projects, not Workspaces.** User-facing copy (website, docs, Plugin Guide, in-app guides) says **Project**. Keep API identifiers (`placement: "workspace"`, `--source workspace`, `personal-workspaces/` on disk) — do not rename those tokens in product prose.

## Coupling notes (don't regress these)

- **"Run the Job Team E2E tests" → `pnpm run test:e2e:jobteam`.** The Job Team

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salesforce/zana](https://github.com/salesforce/zana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
