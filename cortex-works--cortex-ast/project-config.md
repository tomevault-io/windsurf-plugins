---
trigger: always_on
description: **The Golden Rule (Non‑Negotiable):**
---


## MCP Tool Usage Rules (CortexAST + cortex-scout)

### CortexAST Priority Rules

**The Golden Rule (Non‑Negotiable):**
- NEVER use standard IDE/shell tools (`grep`, `rg`, `cat`, `head`, `tree`, `ls`, `git diff`) for codebase exploration, symbol lookup, or refactor verification.
- ALWAYS use CortexAST Megatools. They are AST-accurate, token-efficient, and designed to keep agents on rails.
- If a tool returns an error telling you which parameter you forgot, treat it as an instruction and retry the tool call (do not guess).

**Megatool Quick‑Reference**

| Task | Megatool | Action Enum | Required Params | Key Optional Params |
|---|---|---|---|---|
| Repo overview (files + public symbols) | `cortex_code_explorer` | `map_overview` | `target_dir` (use `.` for whole repo) | `exclude` (dir-name array), `search_filter`, `max_chars`, `ignore_gitignore` |
| Token-budgeted context slice (XML) | `cortex_code_explorer` | `deep_slice` | `target` | `exclude` (dir-name array), `budget_tokens`, `skeleton_only`, `query`, `query_limit`, `single_file` (bool — exact target only, no vector spill), `only_dir` (string — restrict query to this subdir) |
| Extract exact symbol source | `cortex_symbol_analyzer` | `read_source` | `path` + `symbol_name` *(or `path` + `symbol_names` for batch)* | `instance_index` (0-based), `skeleton_only` |
| Find all usages before signature change | `cortex_symbol_analyzer` | `find_usages` | `symbol_name` + `target_dir` | |
| Find trait/interface implementors | `cortex_symbol_analyzer` | `find_implementations` | `symbol_name` + `target_dir` | |
| Blast radius before rename/move/delete | `cortex_symbol_analyzer` | `blast_radius` | `symbol_name` + `target_dir` | |
| Cross-boundary update checklist | `cortex_symbol_analyzer` | `propagation_checklist` | `symbol_name` *(or legacy `changed_path`)* | `aliases`, `only_dir` (restrict scan to one microservice dir) |
| Save pre-change snapshot | `cortex_chronos` | `save_checkpoint` | `path` + `symbol_name` + `semantic_tag` | `namespace` |
| List snapshots | `cortex_chronos` | `list_checkpoints` | *(none)* | `namespace` |
| Compare snapshots (AST diff) | `cortex_chronos` | `compare_checkpoint` | `symbol_name` + `tag_a` + `tag_b` *(use `tag_b="__live__"` + `path` to diff against current state)* | `namespace`, `path` |
| Delete old snapshots (housekeeping) | `cortex_chronos` | `delete_checkpoint` | `symbol_name` and/or `semantic_tag` *(optional: `path`, `namespace`)* — Automatically searches legacy flat `checkpoints/` if no matches in namespace. | `namespace` |
| Compile/lint diagnostics | `run_diagnostics` | *(none)* | `repoPath` | |

## The Ultimate CortexAST Refactoring SOP

Whenever you are asked to perform a non-trivial refactor or update a core feature, you MUST generate and print this Markdown checklist into the chat **before writing any code**, and check the boxes as you proceed:

### 🧠 Refactoring Orchestration Plan
- [ ] **Phase 1: Recon & Blast Radius**
  - [ ] Use `map_overview` to understand the domain.
  - [ ] Use `blast_radius` (or `find_usages`) on the target symbol.
- [ ] **Phase 2: Snapshot**
  - [ ] Use `save_checkpoint` on the target files/symbols.
- [ ] **Phase 3: Execution**
  - [ ] Read minimal context using `read_source` (`skeleton_only: true` if large).
  - [ ] Write the code edits.
- [ ] **Phase 4: Verification & Sync**
  - [ ] Use `run_diagnostics` to catch compiler errors.
  - [ ] Use `compare_checkpoint` to verify structural intent.
  - [ ] Use `propagation_checklist` to ensure TS/Python/Proto boundaries are updated.

**The Autonomous Refactoring Flow (Rails)**

Follow this sequence for any non-trivial refactor (especially renames, signature changes, or cross-module work):

1. **Explore** → `cortex_code_explorer(action: map_overview)`
2. **Isolate** → `cortex_symbol_analyzer(action: read_source)` (get the exact symbol source before editing)
3. **Measure Impact** →
  - Use `cortex_symbol_analyzer(action: find_usages)` BEFORE changing any signature
  - Use `cortex_symbol_analyzer(action: blast_radius)` BEFORE any rename/move/delete
4. **Checkpoint** → `cortex_chronos(action: save_checkpoint, semantic_tag: pre-refactor)`
5. **Edit Code** → make the minimal change
6. **Verify** →
  - `run_diagnostics` immediately after editing
  - `cortex_chronos(action: compare_checkpoint)` to verify semantics (never use `git diff`); prefer `tag_b="__live__"` for "before vs now"
7. **Cross‑Sync** → `cortex_symbol_analyzer(action: propagation_checklist)` when touching shared types/contracts

**Output safety (spill prevention):**
- Output is truncated server-side at `max_chars` (default **8000**). VS Code Copilot writes responses larger than ~8 KB to workspace storage — the 8000 default is calibrated to stay below that threshold. Set `max_chars` explicitly (e.g. `3000`) for large-scope queries; increase only if your client handles larger inline output.

**`exclude` best practice (map_overview + deep_slice):**
- If `map_overview` returns “Massive Directory” or counts look inflated (e.g. `node_modules/`, build outputs, generated code), pass `exclude: ["node_modules", "vendor", "__pycache__", "build", "dist"]`.
- `exclude` matches directory **base names** (not full paths) and prunes at every depth.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cortex-works/cortex-ast](https://github.com/cortex-works/cortex-ast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
