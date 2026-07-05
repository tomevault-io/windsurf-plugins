---
trigger: always_on
description: provides O(1) lookup for cursor restoral.  Cursor IDs survive
---

# AGENTS.md — helixel-mode

> User docs: `docs/USER-GUIDE.org` | Extension API: `docs/API.org`
> Architecture: `docs/ARCHITECTURE.org` | Debug: `docs/DEBUG-SKILL.org`

## File Map

| File | Role |
|------|------|
| `helixel-core.el` | **Pure data layer**: `helixel-sel`, `helixel-action` structs, `helixel-last-action`, kind registry, op registry, delimiter protocol, transaction helpers, swap-source type, keyrec utilities. Zero helixel deps (cl-lib only). |
| `helixel-ring.el` | **Event storage + history navigation**: `helixel--action-ring` (commit/dedup/cap), `helixel--global-jump-log`, `helixel--tracking-open`, `helixel--cancel-action`, `helixel--live-action-set`, live-event management, `;' action-cycle, C-o/C-i jump commands. |
| `helixel-macros.el` | **Command definition macros**: `helixel-define-command`, `helixel-define-operator`, `helixel-with-action-tracking`. |
| `helixel-repeat.el` | Dot-repeat (`.`) and selection-repeat (`M-.`): record (`helixel-record-action`), replay, unified `helixel--repeat-advance` (delegates to kind-registry advance fns), all-buffer/all-dir dispatch, kind-specific `:all-buffer-fn`/`:all-dir-fn` from kind registry, line-pass helper, interactive entry points.  Also includes insert-mode key + text recording (segment-based capture via after-change-functions) — each insert-mode command becomes either `(:keys VEC)` (no buffer change) or `(:text STR :delete-before N :offset O)` (any buffer change).  Replay helper `helixel--execute-keys' accepts both segment lists and raw key vectors. |
| `helixel-chain.el` | Chain lifecycle: start/end/cancel.  Chain accumulates a list of `helixel-action' values committed during the chain (via `helixel-action-commit-hook') and stores it as `:action-list' payload.  Replay iterates the list and `helixel-action-replay`s each entry.  No more kmacro / keystroke capture. |
| `helixel-state.el` | Modal state machine, pending-op system, keymap shells, insert entry/exit, visual state, minor modes, shared kill core. |
| `helixel-move.el` | Movement/selection commands (line/rect/word), rect change/replay. |
| `helixel-editing.el` | Editing commands (kill, change, copy, replace, yank) + selection recreate fns + op runners + `helixel--replace-region` + `helixel--delete-selection`. |
| `helixel-keymap.el` | All keymaps. Populates `helixel-state-map-alist`. 7 `declare-function` for flymake/eglot (third-party only). |
| `helixel-search.el` | Search/find-char + `n`/`N` repeat + `helixel--active-search` state. |
| `helixel-textobj-engine.el` | Forward primitives (forward-word/WORD/symbol/sentence/paragraph/function), generic select-inner/a-object + restricted variants, range struct, type-properties, motion-loop / with-restriction macros, activate-textobj-range, recreate-textobj + advance-textobj. Pure primitives, no per-textobj-type code. |
| `helixel-textobj-pair.el` | Paren / quote / xml-tag selection (the matched-pair families): get-block-range, select-block, up-paren, select-paren, forward-quote, select-quote, select-xml-tag, tag-* helpers, make-pair-delimiter, make-tag-delimiter. |
| `helixel-textobj-block.el` | Regex / fenced block text objects: up-regex-block, select-regex-block, up-block-at-point, select-block-at-point, block-textobj-alist (customs), block-spec-at-point, block-adjust-for-jump, regex-adjust-for-jump, make-block-delimiter, make-regex-delimiter. |
| `helixel-textobj-marks.el` | User-facing surface: define-mark-pair/-quote/-object/-regex-textobj macros, mark-inner-*/mark-a-* commands (including tag and block), tree-sitter helper, all default registrations, `textobj' kind registration. |
| `helixel-textobj.el` | Facade: requires engine, pair, block, marks. |
| `helixel-surround.el` | Surround add/delete/replace. |
| `helixel-swap.el` | Swap commands. Depends on `helixel-editing` for `helixel--replace-region` (one-way, no circular dep). |
| `helixel-mc-core.el` | **Multi-cursor core + target computation**: fake-cursor overlays, per-cursor state vars, dispatch loop via `post-command-hook` / `pre-command-hook`, cursor-ID hash table, undo-step management (begin/finish + `buffer-undo-list` `apply` entry injection for cursor-position persistence across undo/redo), whitelist policy, `helixel-multi-cursor-mode`.  Target computation: `helixel-mc--realize-targets`, advance-walk fallback, `helixel-mc--spawn-from-sel/-line/-rect/-find-char`, kind registry hooks. |
| `helixel-mc-spawn.el` | **High-level user commands**: toggle, add-cursor-here, edit-lines, mark-next-like-this, primary/content rotation, keep/remove-matching, merge/trim/align, split-on-regex, restore-cursors. |
| `helixel-mc-integrate.el` | Glue: dot-repeat / chain / insert per-cursor execution + atomic undo. |
| `helixel-shims.el` | `with-eval-after-load` shims for third-party integration (info, help-mode, shortdoc, man, woman, eww) + multi-cursor completion-preview shim. 29 `declare-function` (all third-party). |
| `helixel.el` | Package entry point. Requires all domain files. |

### Test Files

| File | Covers |
|------|--------|
| `test/helixel-test-common.el` | `helixel-test-with-buffer` macro |
| `test/helixel-test-editing.el` | Edit transactions, sel struct, editing commands |
| `test/helixel-test-action.el` | Action tracking and command execution |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jixiuf/helixel-mode](https://github.com/jixiuf/helixel-mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
