---
trigger: always_on
description: Router for AI agents working inside the spellbook repo. See
---

# AGENTS.md — Spellbook

Router for AI agents working inside the spellbook repo. See
`harnesses/shared/AGENTS.md` for the universal principles file (symlinked
to every harness); this file is the **spellbook-specific** index.

## Stack & boundaries

| Layer | Lives at | Owns |
|---|---|---|
| **Skills** | `skills/<name>/SKILL.md` + `references/` + `scripts/` | Judgment. <500-line SKILL.md; frontmatter-triggered. |
| **Agents** | `agents/<name>.md` | Scoped personas with tool restrictions and model pins. |
| **Harness configs** | `harnesses/{claude,codex,pi,factory,gemini,shared}/` | Per-harness hooks, settings, principles. `harnesses/shared/AGENTS.md` symlinks into every harness. |
| **CI module** | `ci/src/spellbook_ci/main.py` | 12 Dagger gates + heal loop. Python 3.12. |
| **Bootstrap** | `bootstrap.sh` | Installs minimal globals (`GLOBAL_SKILLS=(tailor seed)` + all agents) via symlink OR download mode. Per-repo subsets are `/tailor` / `/seed`'s job. |
| **Scripts** | `scripts/` | Shell + Python utilities: frontmatter check, index regen, embeddings, external sync, harness lint. |
| **Backlog** | `backlog.d/NNN-*.md` (open), `backlog.d/_done/` (closed), `.spellbook/deliver/<ulid>/` (runtime state, gitignored) | Shaped work ready to build. Single source of truth; closure via `Closes-backlog:` trailers on squash-merge commits (handled by `/ship`). |

## Ground-truth pointers

Stale training data lies about these — always read the file:

- **`ci/src/spellbook_ci/main.py`** — exact set of gates, what each
  enforces, which are healable.
- **`harnesses/shared/AGENTS.md`** — the principles file. Red flags,
  doctrine, anti-patterns. Cited verbatim by `/code-review`.
- **`bootstrap.sh:271`** — `GLOBAL_SKILLS=(tailor seed)`. Anything you
  thought was globally symlinked but isn't in this list: it isn't.
- **`.githooks/pre-commit`** — what runs automatically on every commit
  (index regen, harness-agnostic install wording, `.spellbook/deliver/`
  force-add block).
- **`.githooks/pre-merge-commit`** — verdict gate for non-FF merges.
  Escape: `SPELLBOOK_NO_REVIEW=1`.
- **`index.yaml`** — derived; never edit manually (pre-commit hook
  regenerates from `scripts/generate-index.sh`).
- **`registry.yaml`** — external-skill source registry with
  `alias_prefix` doctrine.
- **`.spellbook/repo-brief.md`** — `/tailor`'s shared spine for every
  rewriter in this repo. Regenerated each `/tailor` run.

## Invariants

- **Cross-harness first (Red Line).** Every new mechanism — skill, hook,
  setting, lint — must work on Claude Code, Codex, AND Pi. Anchoring a
  design on one harness's unique feature is a bug. Prior art:
  `harnesses/pi/settings.json:skills[]` globs.
- **Thin harness, strong models.** Don't compensate for weak models with
  scaffold. `skills/flywheel/SKILL.md` (43 lines) is the reference.
- **Skills are self-contained.** No `../..`, no `$REPO_ROOT/…` sourcing.
  Libs resolve via `readlink -f` + `$SCRIPT_DIR/lib/…`. State roots
  anchor to the *invoking* project's `git rev-parse --show-toplevel`,
  not the skill's install dir. Symlink-install + invoke from a foreign
  project is the canonical self-containment test.
- **No `index.yaml` edits.** Pre-commit regenerates it.
- **No `references/<repo-name>.md` sidecar files.** Spellbook-specific
  content belongs in SKILL.md body. Stack-specific references under their
  own topic (e.g. `references/cross-harness.md`) are fine.
- **`.spellbook/deliver/<ulid>/state.json` + `receipt.json` are agent-
  written, never human-edited, never committed.** Gitignored; pre-commit
  hook blocks force-adds.
- **Base branch: `master`.** Topic branches: `feat/*`, `fix/*`, `chore/*`,
  `docs/*`, `refactor/*`, `backlog/*`, `doctrine/*`.
- **No claim-coordination primitives under `skills/`.** `claims.sh`,
  `claim_acquire`, `claim_release` were dropped per `backlog.d/_done/
  032-deliver-inner-composer.md`; regression guarded by
  `check-no-claims`.
- **`/deliver` must compose atomic phase skills via trigger syntax.**
  Raw `dagger call check`, direct bench-agent dispatch, or inlined
  phase internals inside `skills/deliver/SKILL.md` fail
  `check-deliver-composition`.
- **`harnesses/claude/settings.json` is COPIED by bootstrap**, not
  symlinked (Claude mutates it at runtime). Changes require re-bootstrap.

## Gate contract

**The load-bearing gate is `dagger call check --source=.`** — 12 parallel
sub-gates, all must pass to ship:

| Gate | What it enforces |
|---|---|
| `lint-yaml` | YAML parseability |
| `lint-shell` | `shellcheck --severity=error` on non-`ci/` shell scripts |
| `lint-python` | `py_compile` on non-`ci/` Python |
| `check-frontmatter` | Required fields + line limits (`scripts/check-frontmatter.py`) |
| `check-index-drift` | `index.yaml` matches `scripts/generate-index.sh` output |
| `check-vendored-copies` | Vendored copies match canonical sources |
| `test-bun` | `bun test` under `skills/research/` |
| `check-exclusions` | No `@ts-ignore`, `.skip()`, `eslint-disable`, `as any` |
| `check-portable-paths` | No hardcoded `/Users/<name>/` or `C:\Users\` outside `harnesses/claude/` + `.claude/hooks` |
| `check-harness-install-paths` | No Claude-only install wording for `/seed` or `/tailor` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phrazzld/spellbook](https://github.com/phrazzld/spellbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
