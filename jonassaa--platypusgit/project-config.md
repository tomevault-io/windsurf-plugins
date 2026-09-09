---
trigger: always_on
description: Context for future Claude sessions working on this repo.
---

# CLAUDE.md

Context for future Claude sessions working on this repo.

**Keep this file SHORT.** It is loaded into every session's context, so it holds
only operational rules and pointers. Deep dives — postmortems, design rationale,
annotated source trees — live in `docs/dev/` and in the specs. When a change
earns a written lesson, add it to the matching `docs/dev/*.md` file (or the
feature's spec), never here; this file grew to 2,456 lines once and was cut back
to what you are reading. A new section here needs a reason a pointer cannot serve.

## What this is

`platypusgit` — cross-platform, developer-focused git desktop app. Tauri 2 (Rust) backend + React/TS frontend. Dev-first TortoiseGit alternative with "extreme usability" as north star. Standalone GUI only — shell integration (Finder/Explorer overlays) out of scope.

## Detailed docs — read the matching one BEFORE working in its area

- `docs/dev/architecture.md` — the annotated backend + frontend source trees:
  every module, command, feature directory, and the traps between them. The map
  of the codebase; start here for any non-trivial change.
- `docs/dev/testing.md` — the four test layers, headless e2e in Docker, CI
  workflows and gates, e2e sharding, the `test/` doc invariants.
- `docs/dev/frontend.md` — diff rendering, the paged log, navigation model,
  state management (multi-repo tabs), styling/design system, PGSelect,
  resizable panes, dialogs, file lists, drag and drop.
- `docs/dev/backend.md` — errors, forge tokens, the rebase engine, network ops
  and credentials, signing, stash, spawning processes, bisect, async/threading.
- `docs/dev/distribution.md` — `pgit` CLI packaging per channel, the launch
  detach, Tauri permissions.
- `docs/dev/releasing.md` — what a version number means and when to bump which
  part, the cut-a-release runbook (changelog lands on `main` FIRST), and the
  prerelease-promotion traps. Read it before tagging anything.
- `docs/superpowers/specs/` + `docs/superpowers/plans/` — approved design docs
  and implementation plans per feature (`ls` for the current set). New feature
  beyond MVP slice → write new spec + plan there first.

`test/docs.test.ts` fails the build when the doc set falls behind the tree
(commands, backend modules, feature directories) — see `docs/dev/testing.md`.

## Toolchain

- **Node 22** + **pnpm** (at `~/Library/pnpm`). Not npm, not yarn.
- **Rust stable** via rustup (`~/.cargo/bin`).
- Assistant's Bash tool does not inherit interactive shell rc → prepend `export PATH="$HOME/Library/pnpm:$HOME/.cargo/bin:$PATH"` when running `pnpm` or `cargo`.

## Common commands

```bash
pnpm install                                # frontend + tauri-cli deps
pnpm tauri dev                              # run app (first build ~2 min, reruns ~10s)
pnpm tsc --noEmit                           # type-check
pnpm vite build                             # bundle frontend only
cargo check --manifest-path src-tauri/Cargo.toml
cargo test --manifest-path src-tauri/Cargo.toml
pnpm tauri build                            # production bundle (.msi/.dmg/.deb/.AppImage)
pnpm tauri build --no-sign                  # ...without updater signing (see note below)
pnpm test                                   # vitest (unit logic + component tests + doc invariants)
pnpm test:e2e:docker                        # e2e — THE way to run e2e (headless, same stack as CI)
pnpm test:e2e:docker run --spec e2e/specs/X.e2e.ts   # ...one spec against this worktree's snapshot
pnpm exec tsc -p e2e/tsconfig.json --noEmit # e2e typecheck gate (root tsc excludes e2e/)
```

**Local production builds need the updater signing key.** `tauri.conf.json`
sets an updater pubkey + `createUpdaterArtifacts`, and a pubkey with no private
key is a **hard error** on any target producing an updater artifact. Export
`TAURI_SIGNING_PRIVATE_KEY` (+ `_PASSWORD`) or build with `--no-sign`. CI passes
both from repo secrets.

## Testing — the operational rules

Four layers (details + CI shape in `docs/dev/testing.md`):

- **Rust backend integration** — `cargo test` against real temp repos.
- **Frontend pure logic + component tests** — `pnpm test` (vitest project
  `unit`, jsdom + mocks from `src/test/setup.ts`).
- **Doc/tree invariants** — `pnpm test` (vitest project `docs`, node env,
  reads `CLAUDE.md`, `docs/dev/`, `src-tauri/`, `e2e/`, `.github/`).
- **E2E** — WebdriverIO specs in `e2e/specs/` driving the real binary.

**E2E always runs in Docker (`pnpm test:e2e:docker …`) — never natively, never
in a UI window.** A native run pops a real WKWebView window, steals focus, is
flaky and slow, and does not predict the CI gate. The one exception is a
genuinely WKWebView-specific question the user explicitly asks for. Run e2e
only when DONE developing a change, and only the spec file(s) relevant to what
you touched — CI runs the full suite. After a `src/` or `src-tauri/` change,
rebuild the snapshot first: `pnpm test:e2e:docker build`, then
`pnpm test:e2e:docker run --spec e2e/specs/<file>.e2e.ts`. Never rely on a
stale snapshot. One cold container build at a time across ALL worktrees
(memory), different worktrees may otherwise run concurrently.

**Before writing or debugging any e2e spec, read the `e2e-testing` project
skill** (`.claude/skills/e2e-testing/SKILL.md`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonassaa/platypusgit](https://github.com/jonassaa/platypusgit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
