---
trigger: always_on
description: LeetCode TUI built with OpenTUI (React bindings) and Bun.
---

# leettui

LeetCode TUI built with OpenTUI (React bindings) and Bun.

## Quick start

```sh
cd leettui
bun install
bun src/index.tsx
```

## Checks (run before every PR)

`bun run check` is the full gate: Biome (lint + format check), `tsc --noEmit`, and `bun test`. CI (`.github/workflows/ci.yml`) runs that full script on every PR and push to `main`. Locally the gate is **split across two Husky hooks** (installed by the `prepare` script on `bun install`) so committing stays fast:

- **pre-commit** (`.husky/pre-commit`) → `bun run lint && bun run typecheck` (the fast half).
- **pre-push** (`.husky/pre-push`) → `bun test` (so broken tests can't reach the remote).

Bypass either in a pinch with `git commit --no-verify` / `git push --no-verify`. Keep `package.json`'s `lint` / `typecheck` / `test` / `check` scripts as the single source of truth so the hooks and CI never drift.

```sh
bun run check      # the full gate (lint + typecheck + test)
bun run lint:fix   # auto-fix Biome lint + formatting
bun run format     # format only
```

Biome config is `biome.json`. The gate runs with `--error-on-warnings`, so it is **zero-tolerance** — any lint finding (warning or error) fails it; keep the tree clean. The only relaxed rule is `noNonNullAssertion` (off — used deliberately with strict index access). A handful of intentional cases carry inline `// biome-ignore <rule>: <reason>` suppressions (e.g. static, never-reordered render lists keyed by index; the `useMemo(..., [themeVersion])` theme cache-bust pattern); add a justification when you suppress. `bun.lock` pins Biome — run installs with `--frozen-lockfile`.

On first run, leettui runs an auth flow that imports your LeetCode session from Firefox (if logged in) or guides you through a one-time cookie paste, validates it, and writes it to `~/.config/leettui/config.toml`. Re-run any time with `bun src/index.tsx auth`.

### Subcommands

These are matched as argv tokens in `src/index.tsx` before the TUI starts:

- `auth` — force the authentication flow.
- `update` — self-update: download the latest release binary for the platform and atomically replace the running executable (`src/core/update.ts`). Pass `--force` to reinstall even when already current. Refuses unless `IS_RELEASE` (only official release-workflow binaries self-update — a from-source build is never overwritten by an older published release). **Stage 19:** prefers the gzip-compressed release asset (`{asset}.gz`, ~40 MB on the wire vs. ~111 MB raw — the runtime binary is unchanged, only the download is compressed), decompressing it into the atomic-swap temp file; falls back to the raw asset on a tag published before Stage 19. `install.sh` does the same on first install. **Transition caveat:** releases keep shipping the raw asset alongside the `.gz` so pre-Stage-19 clients (whose `update.ts` only knows the raw asset name) keep self-updating; the raw asset can be retired in a later stage once `.gz`-aware clients dominate. **Windows** has a PowerShell installer (`install.ps1`, the counterpart to `install.sh` — `irm … | iex`, downloads the raw `.exe` to `%LOCALAPPDATA%\leettui\bin` and adds it to the user PATH) but **no in-place self-update**: `leettui update` refuses on `win32` (you can't rename over a running, file-locked `.exe`), so updating means re-running `install.ps1`. The Windows `.exe` ships raw (no `.gz` sibling) since there's no programmatic download path that would consume it. **Background auto-update:** the TUI itself now checks for a newer release at boot and every 4 hours (`src/ui/updateScheduler.ts`) and — when `[update] auto` is on (default), the build is `IS_RELEASE`, and the platform can self-update — silently downloads + installs it in the background via the same atomic swap (`downloadAndInstall`, extracted from `runUpdate`; pid-suffixed temp file so a concurrent manual `leettui update` can't collide). The banner then flips from "Update available" to a green "▲ Updated to vX — restart leettui to apply" CTA (no respawn — exit and relaunch), and quitting prints an "installed — takes effect on next launch" notice instead of the "run leettui update" nudge. win32/dev builds fall back to the notify-only banner. An exit-hook cleanup plus a boot-time sweep remove temp files orphaned by an exit/crash mid-download.
- `--version` / `version` — print the embedded version and exit.

**Headless cwd-aware verbs** (Stage 8; `new` = Stage 22) infer the problem from the current dir so they work from inside an editor's `:!` (`src/cli/`, no renderer): `test` (run the seeded cases offline), `run`/`submit` (LeetCode API), and `new <language>` (scaffold a solution + harness + tests for a language — the headless analogue of the TUI solution picker). Unlike the cwd-only verbs, `new` resolves at the **problem** level (run it from the problem **workspace** dir) and takes the language as an argument, so the typical flow is: enter a problem → `w` to open the workspace → `:!leettui new rust` to add a language without leaving the editor. Valid languages are exactly what LeetCode offers for the problem; an unsupported signature scaffolds a plain solution with no harness (never an error); it's idempotent (a pre-existing solution just prints its path). See `src/cli/CLAUDE.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [y4nder/leettui](https://github.com/y4nder/leettui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
