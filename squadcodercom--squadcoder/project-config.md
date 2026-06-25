---
trigger: always_on
description: <!-- SQUADCODER:start (merge=ours) -->
---

<!-- SQUADCODER:start (merge=ours) -->
# SquadCoder

This repo is **SquadCoder** — an open-source, **RTL/Hebrew-first** AI coding assistant, a rebranded
fork of **MiMoCode** (`XiaomiMiMo/MiMo-Code`, itself a fork of `sst/opencode`).

- **Goal:** keep pulling bug fixes/updates from MiMo/opencode (`upstream`/`opencode` remotes)
  while keeping our brand and developing our way. Isolate our diff — see `docs/FORK_STRATEGY.md`.
- **What's ours (don't expect upstream to maintain):** `.squadcoder/` (config, skills, plugins),
  `packages/squadcoder-plugin-*`, and rebrand edits tagged `SQUADCODER:`. These are `merge=ours`.
- **Working method:** reuse before build (core → plugin/config → safe third-party → build);
  only i18n/RTL/branding touch core, kept small and marked.
- **⛔ BEFORE BUILDING ANY FEATURE, READ `docs/REUSE_AUDIT.md`** — the canonical per-feature verdict
  (already-in-core? safe extension to adopt? or build?). It exists so we **never double-work**.
  `docs/PROJECT_STATUS.md` is the living status board (requirements, done, pending, user's notes).
- **Durable context lives outside the repo:** the memory store at
  `~/.claude/projects/C--Users-raviv-OneDrive-Desktop-SquadCoder/memory/` (`MEMORY.md` index) and
  the plan at `~/.claude/plans/i-decided-to-create-compiled-corbato.md`. Read them first.

## Build & release (auto-rebuild rule)
- When a set of fixes is COMPLETE and requires a rebuild for the user to install, REBUILD THE PRODUCTION INSTALLER automatically (without waiting to be asked) to the canonical path:
  `C:\Users\raviv\OneDrive\Desktop\MuminAI\mumin\release\SquadCoder-desktop-win-x64-installer.exe`
- Recipe (run from `packages/desktop`, prod channel). If `seed/` or `.squadcoder/plugin-src/*` changed, first run `bun script/make-seed.ts` from repo root:
  1. Clear `out/` + `dist/` first (OneDrive locks files).
  2. `$env:OPENCODE_CHANNEL='prod'; bun run build`
  3. `$env:OPENCODE_CHANNEL='prod'; bun run package:win`
  4. `Copy-Item -Force packages/desktop/dist/SquadCoder-desktop-win-x64-installer.exe release/` (also copy the `.blockmap`).
- Renderer-only edits (packages/app components / i18n) ship via `bun run build` alone — SKIP `prebuild`. A `.squadcoder/plugin-src/*` or engine (`packages/opencode`) change is engine-side → run `bun run prebuild` (re-bundles the plugin + rebuilds the engine node bundle) BEFORE `package:win`.
- `bun run build` (electron-vite/babel+solid) is a SEPARATE pass from `bun typecheck` — a file can pass typecheck yet fail babel parse (e.g. a duplicate import). Always run the real build before declaring a fix shipped.
- **SQUADCODER: desktop release version is single-sourced from `packages/desktop/package.json`** (what Settings displays via `app.getVersion()`). Never hand-type a release tag — derive it. To cut a release: (1) `bun script/set-version.ts <v>` (writes the same version to desktop + app, anti-drift), (2) build the installer per the recipe above, (3) `bun script/release-desktop.ts` (derives the `v<v>` tag from package.json and creates/refreshes the GitHub release; `--dry-run` previews). This is separate from the opencode CLI/npm flow (`script/version.ts`/`script/release.ts`, sourced from `packages/opencode/package.json`).

## GitHub: use the configured MCP (DEFAULT for all GitHub work)
- A GitHub MCP is configured and authenticated (account `snipecoder`, target `squadcodercom/squadcoder`). For ANY GitHub/repo operation — PRs, issues, releases, code search, reading/creating/updating files on the remote, comments, reviews — ALWAYS use the GitHub MCP tools FIRST. Do NOT shell out to `git`/`gh` for these.
- What the MCP CAN do: it makes REAL commits on the remote via the REST API — `create_or_update_file` (one file) and `push_files` (multiple files in one commit), plus branches, PRs, issues, releases, code search, comments, reviews. For any NEW change authored against the remote, the MCP is the correct tool.
- What the MCP CANNOT do: replicate your EXISTING LOCAL git commit history. The REST API authors fresh commits server-side file-by-file; it cannot upload pre-existing local commit objects/graph. So when your local branch already has unpushed commits, use `git push` to sync them — otherwise the MCP would create divergent fresh commits on the remote that don't match local HEAD.
- Rule of thumb: local branch has unpushed commits → `git push` to mirror them. Editing/adding files directly on the remote with no local-history concern → GitHub MCP.
- `git push` to this repo requires a token with BOTH `repo` AND `workflow` scopes — the repo contains `.github/workflows/*`, and GitHub rejects any push that touches workflow files when the token lacks `workflow` scope. The MCP uses the same token, so it hits the same wall on workflow files.
- The repo has a husky PRE-PUSH hook that runs `turbo typecheck`; a typecheck error in ANY package blocks the push — fix it (do NOT use `--no-verify` unless the user explicitly asks).
- Remote `origin` = https://github.com/squadcodercom/squadcoder.git ; default branch `main`.

The opencode/MiMoCode developer style guide follows and still applies.
<!-- SQUADCODER:end -->

- Always use superpowers skill instead of builtin plan mode.
- To regenerate the JavaScript SDK, run `./packages/sdk/js/script/build.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [squadcodercom/squadcoder](https://github.com/squadcodercom/squadcoder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
