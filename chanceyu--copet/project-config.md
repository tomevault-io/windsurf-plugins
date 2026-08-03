---
trigger: always_on
description: Every new feature module follows the same workflow. Do not start coding on `main`.
---

# CoPet Agent Guide

## Feature Module Development

Every new feature module follows the same workflow. Do not start coding on `main`.

### 1. Worktree-first

New feature modules must be developed in an isolated git worktree on a `feature/*` branch.

- Create a worktree with a `feature/<module-name>` branch before writing any code.
- Worktrees live under `.worktrees/` at the repository root (e.g. `.worktrees/virtual-pet-list/`). Do not place them under `.claude/worktrees/` or anywhere else.
- Use `EnterWorktree` to create the worktree and switch into it; do not hand-craft worktrees with raw `git worktree add` unless the harness is unavailable.
- One worktree per feature module. Do not reuse a worktree for an unrelated feature.
- Keep the worktree and its branch after the feature is merged. Do not delete or prune them.
- Bug fixes, doc updates, and small refactors that touch a single file may be made directly on `main` without a worktree.

### 2. Branch naming

- Feature branches: `feature/<kebab-case-module-name>` (e.g. `feature/virtual-pet-list`).
- The module name should describe the user-visible capability, not the implementation detail.
- Do not use `feat/`, `feature_`, `dev/`, or personal-name prefixes.

### 3. Code layout

- React components live **flat** under `src/components/`. Do not create per-feature subdirectories (e.g. `src/settings/`, `src/components/settings/`). The only allowed subdirectory is `src/components/ui/` for shadcn-style primitives.
- Group related components by **filename prefix**, not by folder. Example: `SettingsNav.tsx`, `SettingsPetsSection.tsx`, `SettingsAboutSection.tsx`.
- Window-level entry components (`PetWindow.tsx`, `SettingsWindow.tsx`) stay at `src/` top level. Auxiliary components they compose go flat into `src/components/`.
- Hooks go in `src/hooks/`, shared utilities in `src/lib/`, static assets in `src/assets/`.
- Asset placement follows the consumer, not the file type:
  - `src/assets/` — assets the frontend `import`s at build time and Vite processes (URL/hashing). Example: agent icon SVGs, `logo.png`.
  - `src-tauri/assets/` — content the Rust backend enumerates at runtime, ships through Tauri `bundle.resources`, and exposes to the webview via the `asset://` protocol (`convertFileSrc`).
  - Decision rule: if Rust lists it with `fs::read_dir`, or the frontend receives a filesystem path that must pass through `convertFileSrc`, the asset belongs under `src-tauri/assets/`. If the frontend imports it directly in TS/CSS, it belongs under `src/assets/`.
- Built-in pet packages live in `src-tauri/assets/pets/<pet-id>/` with `pet.json` + `spritesheet.{webp,png}`. They are discovered by `fs::read_dir` in `config_store.rs`, declared as `bundle.resources` and allowlisted under `assetProtocol.scope` (`$RESOURCE/assets/pets/**`) in `tauri.conf.json`, and share the same package format as user-imported pets under `~/.copet/pets/`. Do not move built-in pets to `src/assets/`; doing so breaks runtime discovery, splits the built-in vs user-imported code paths, and forces a hardcoded manifest list.
- Rust source goes in `src-tauri/src/`; keep it production-only (no tests inline).

### 4. Tauri command boundary

- Frontend calls into Rust through Tauri commands; expose new Rust capability with a typed command and consume it from a hook in `src/hooks/`.
- Do not call `invoke` from inside presentational components — wrap it in a hook so tests can mock the Tauri layer with the shared harness.

### 5. Commit messages

Follow Conventional Commits with a scope matching the feature module:

```
<type>(<scope>): <imperative summary>
```

- `type`: `feat`, `fix`, `refactor`, `style`, `test`, `chore`, `docs`, `perf`.
- `scope`: feature module or surface area (e.g. `settings`, `window`, `pet`).
- Summary in the imperative mood, lowercase, no trailing period.
- Keep the subject short — under ~72 chars. If it doesn't fit, the commit is doing too much; split it.
- No body by default. Add one only when the *why* is non-obvious from the diff (hidden constraint, subtle interaction, workaround), and keep it brief — no multi-paragraph explanations.

Examples:

- `feat(settings): add SettingsAboutSection`
- `fix(settings): align section heading IDs with tabpanel aria-labelledby`
- `refactor(settings): compose window from sectioned shell`

### 6. Git hygiene

- Do not force-add ignored files. Never use `git add -f` or `git add --force` unless the user explicitly names the ignored file and asks for it to be tracked.
- Respect `.gitignore` as authoritative.
- Before every commit, check `git diff --cached --name-status` and verify the staged files match the user-requested scope.
- Before every commit, you must explicitly check every staged path against ignore rules with `git check-ignore -v --no-index -- <path>`. This check is mandatory and cannot be skipped, waived, inferred from `git status`, or replaced with any other form of reasoning. If any staged path matches `.gitignore` or another ignore source, stop and do not commit until the ignored path is unstaged or the user explicitly changes the ignore rules.

### 7. Definition of done

A feature module is done only when **all** of the following hold:

- [ ] Branch is `feature/<module-name>` in a dedicated worktree.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChanceYu/CoPet](https://github.com/ChanceYu/CoPet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
