---
trigger: always_on
description: >
---


# git-conflict-resolver

Mechanical conflict resolution is automated. Judgment is explicit. If intent
cannot be inferred, halt and ask.

For rationale and empirical grounding, read
`${CLAUDE_SKILL_DIR}/references/design-rationale.md` only when needed.

## Constitutional Rules

Read `${CLAUDE_SKILL_DIR}/constitution.md`. These rules override this procedure:

1. No conflict markers in tracked source files.
2. No blanket resolution across human-authored files.
3. Halt when intent cannot be inferred for `other` files.

## Modes

Two entry points:

- **Resolve** (default): an operation is paused with conflicts. Follow the
  Procedure below.
- **Split** (`--split`, or a request to decompose an oversized PR/branch with no
  active conflict): propose smaller PRs along functional/structural boundaries.
  See "PR Decomposition" below. The Procedure's large-conflict gates also route
  here when a conflict is too large to resolve in place.

## Procedure

### Step 1 — Detect context

```bash
${CLAUDE_SKILL_DIR}/scripts/conflict-status.sh
```

Output: `context<TAB>progress<TAB>branch<TAB>unmerged_count`.

| Context | Unmerged | Argument | Action |
|---|---:|---|---|
| `none` | n/a | `--abort` | Report "no operation in progress" |
| `none` | n/a | none / `--continue` | Report "no conflicts" |
| active | n/a | `--abort` | `git <ctx> --abort` |
| active | 0 | any | `git <ctx> --continue` (merge: `git commit --no-edit`) |
| active | >0 | `--continue` | Continue without resolving |
| active | >0 | none | Proceed to Step 2 |

Abort recommendation criteria:

- rebase step has 20 or more conflicted files
- five or more consecutive rebase steps have conflicts
- same conflict reappears three or more times with rerere enabled

These are configurable heuristics, not empirical laws. See
`${CLAUDE_SKILL_DIR}/references/recurring-conflicts.md`. When the unmerged count
is 20 or more, run
`${CLAUDE_SKILL_DIR}/scripts/suggest-pr-split.sh --conflicts --scope both --json` before
categorizing. If high-confidence groups exist, recommend aborting and re-landing
as smaller PRs (refactor-baseline first); see
`${CLAUDE_SKILL_DIR}/references/pr-decomposition.md`. This is advisory —
low-confidence (cross-cutting) groups are structural-only boundaries, not safe
splits.

### Step 2 — Categorize

```bash
${CLAUDE_SKILL_DIR}/scripts/categorize-conflicts.sh
```

Categories: `lockfile`, `migration`, `submodule`, `binary`, `generated`,
`snapshot`, `notebook`, `mergiraf`, `other`.

Report categorization before editing.

### Step 3 — Resolve by category

#### 3a. Lockfiles

Accept theirs to clear markers; regenerate in Step 5.

```bash
git checkout --theirs <file> && git add <file>
```

#### 3b. Migrations

Do not auto-resolve. Show both sides and ask. State operation-specific
ours/theirs semantics, especially during rebase.

#### 3c. Submodules

Halt. Show both pinned SHAs and ask which commit the submodule should point to.

#### 3d. Binary files

Halt. Show file type/size and ask whether to keep ours or theirs.

#### 3e. Generated files

Resolve the source spec first, then regenerate. If the source spec cannot be
identified or both specs changed incompatibly, halt.

#### 3f. Snapshots

Use the side that matches the intended code state, then regenerate snapshots.
During rebase, explicitly state that "ours" is upstream and "theirs" is the
replayed commit before choosing.

#### 3g. Notebooks

If conflict is output-only (`outputs` or `execution_count`), strip outputs.
If source cells conflict, treat as `other` and run intent inference.

#### 3h. mergiraf files

```bash
timeout 30 mergiraf solve -- <file> --compact --keep-backup=false
```

Exit code 124 means timeout; fall through to `other`. If markers remain after
mergiraf, fall through to `other`. Otherwise stage the file.

#### 3i. Other files

Before per-file resolution, route deterministically (advisory; the prose steps
below are unchanged):

```bash
${CLAUDE_SKILL_DIR}/scripts/meta-route.sh --unmerged-only --json
```

Record each file's `route`, `reason`, and `confidence` into the per-file
Decision Record. The router augments and audits judgment; it does not replace
it. See `${CLAUDE_SKILL_DIR}/references/meta-resolver.md`.

For each file:

1. Measure balance. If one side is more than 3x longer, LLM analysis is more
   appropriate. If balanced, prefer line-combination reasoning and lower
   confidence. If total conflict content is over 300 lines, halt and recommend
   decomposition: run `${CLAUDE_SKILL_DIR}/scripts/suggest-pr-split.sh --conflicts --scope both`
   to propose split groups, and fall back to `git-imerge` (see
   `${CLAUDE_SKILL_DIR}/references/recurring-conflicts.md`) when the change cannot
   be split (all groups low-confidence/cross-cutting).
1a. If balanced (with a diff3 base) and ≤ 400 lines, enumerate line-combination
    candidates:

    ```bash
    ${CLAUDE_SKILL_DIR}/scripts/sbse-recombine.sh --file <file> --top 3 --json
    ```

    Advisory: a `clear-winner` verdict (top ≥ 95, gap ≥ 10) supports the
    `additive` / `trivial` classification at step 9. An `ambiguous` verdict
    (top three within 5) is itself a HALT signal — pick by intent, not by
    score. `deferred` means the block exceeds the SBSE bound (>400 lines or
    >3x imbalance); fall through to the LLM path. See H-02 / I-27.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Raudbjorn/conflict-of-interest](https://github.com/Raudbjorn/conflict-of-interest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
