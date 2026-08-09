---
trigger: always_on
description: This file is the project's working memory — how the system *thinks*, not what files exist. Update it **in the same change/commit** whenever you: alter dispatch or sourcing flow; add/rename/remove a module or helper; change the config/knob contract; change an fzf-binding pattern (`--bind` / `--preview` / `reload` / `change:`); or discover a new gotcha. Keep it conceptual — prune file-listings, line numbers, counts, and rotted claims on sight. If it drifts toward a `tree` dump, it has already fail
---

# AGENTS.md — git-fuzzy mental model

## Self-Maintenance Directive

This file is the project's working memory — how the system *thinks*, not what files exist. Update it **in the same change/commit** whenever you: alter dispatch or sourcing flow; add/rename/remove a module or helper; change the config/knob contract; change an fzf-binding pattern (`--bind` / `--preview` / `reload` / `change:`); or discover a new gotcha. Keep it conceptual — prune file-listings, line numbers, counts, and rotted claims on sight. If it drifts toward a `tree` dump, it has already failed. State current truth only; this is not a changelog.

## What This Is

git-fuzzy is a **thin bash orchestration layer over `fzf`**: every command builds an fzf menu whose **list, preview, and actions are all `git` commands**. It owns no real state of its own — it is a TUI skin over git. Read it as a dispatcher and a pile of menu-builders, nothing more.

## Execution / Dispatch Model

This is the spine. Get this and the rest follows.

**Two-process model.** Two kinds of process run, and the distinction governs every perf decision:
- **PARENT** = the interactive `gf_<cmd>` function. It builds and owns the fzf TUI, runs **once**, and does all the heavy setup: validation, invariants, color, geometry.
- **HELPER** = `git fuzzy helper <name>` subprocesses that fzf spawns on **every keystroke / preview / action** via `--bind` and `--preview`. This is the **HOT PATH**: a fresh `bin/git-fuzzy` process fires per binding event.

**All perf design exists to keep helper spawns cheap.** When optimizing, that is the lens.

**Entry & flow.** `bin/git-fuzzy` is the ONLY executable; everything else is a sourced lib. Boot order:
1. Resolve the real script dir (symlink-aware — it follows `readlink`).
2. Source CORE.
3. Selectively source the needed module(s) (helpers source exactly one file; top-level commands may also pull transitive deps).
4. Run the invariant block — **parent only**.
5. Call `git_fuzzy "$@"`.

CORE source order is **load-bearing**: `load-configs → utils → debug → core`. Do not reorder it.

`git_fuzzy`: no args → `gf_menu`; args → `gf_run`. **`gf_run` lives only in `bin/git-fuzzy`**, not in any lib.

**Routing (inside `gf_run`).**
- `helper <sub>` → `gf_helper_<sub>`.
- `interactive <cmd>` → forces `gf_<cmd>` (skips the direct variant).
- plain `<cmd>` → **prefers `gf_<cmd>_direct`, else `gf_<cmd>`**.

It resolves by `type`-checking that the function exists, then forwards params via `quote_params` + `eval`.

**Helper fast path — why it skips validation.** The invariant block is guarded so helpers skip it (`[ "$1" != helper ]`). WHY: a helper always runs inside an already-validated, in-repo parent with `GH_AVAILABLE` exported — re-running git / fzf / repo / gh checks would be pure forking waste on every keystroke. This is **deliberate, not an oversight. Do not "fix" it by re-adding checks to the helper path.**

**Implicit helper allowlist.** There is NO allowlist array. The "allowlist" is simply the set of `gf_helper_*` functions that got sourced. An unknown helper name sources nothing → the function does not exist → `gf_run` errors. (This is the *implicit* allowlist; the *explicit* debounce allowlists in the security section are a different mechanism that happens to share the word.)

## Conceptual Directory Roles

Roles and relationships, not a tree. What each part is *for*:

- **`bin/git-fuzzy`** — the only executable. Home of entry, dispatch (`gf_run`), selective sourcing (`gf_source_for` / `gf_source_helper`), and the invariant block.
- **`lib/modules/<cmd>.sh`** — the interactive UI shell, ONE per user command. Owns the fzf invocation (PARENT side).
- **`lib/modules/helpers/<cmd>.sh`** — pure stdout-producing helper functions, no fzf (HELPER side / hot path).
- **CORE libs** `lib/core.sh`, `lib/utils.sh`, `lib/debug.sh` — sourced first; order is load-bearing (see Dispatch Model).
- **`lib/load-configs.sh`** — knob defaults plus per-repo override layering (see Config Contract).
- **`lib/modules/main.sh`** — builds the top-level menu (the `gf_menu` surface).
- **`lib/modules/helpers/generic.sh`** — cross-module helper infra (debounce / reload machinery).
- **GitHub-PR integration** mirrors the module/helper split — a role, not an enumeration.

**Selective sourcing ties roles to perf.** `gf_source_for` sources a single module (helpers source exactly one file; top-level commands may also pull transitive deps), **mirroring `gf_run` routing**. Helpers route through `gf_source_helper`, a `case` on the **name prefix** that sources exactly ONE helper file. **Prefix order is load-bearing — specific before general** (e.g. `diff_checkout_*` before `diff_*`). WHY: bulk-sourcing every module would dominate hot-path cost; selective sourcing avoids it.

## Core Patterns / Idioms


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bigH/git-fuzzy](https://github.com/bigH/git-fuzzy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
