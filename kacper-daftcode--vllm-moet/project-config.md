---
trigger: always_on
description: Multiple agents and humans work on this project concurrently, often in the
---

# AGENTS.md — the commit contract

Multiple agents and humans work on this project concurrently, often in the
**same checkout**. Work has been lost here three separate times by treating
the generated patch as a source file. Read this before your first commit;
every rule below traces back to a real incident.

## The iron rule

`patch/vllm-moet-v0.24.0.patch` is a **generated artifact**: byte-for-byte
`git diff v0.24.0 moet-v0.24.0` from the vllm fork clone. It is **never
edited by hand**, never patched incrementally, never regenerated from
anything but the fork branch. The only sanctioned way to change it:

```bash
python3 tools/check_patch_files.py --update
```

If your change touches vLLM code, it goes to the **fork branch first**; the
patch is derived afterwards. A change that exists only inside the patch file
WILL be erased by somebody's next regeneration.

## The two repos on this box

| repo | path | branch | role |
|---|---|---|---|
| **vLLM-Moet** (this one, public) | `/workspace/vllm-moet` | `main` | publication: generated patch, kernels + cubins, bench system, docs |
| **vllm fork clone** | `/workspace/vllm-v0.24.0` | `moet-v0.24.0` | **source of truth for ALL vLLM code**; remotes: `fork` = `kacper-daftcode/vllm`, `origin` = `vllm-project/vllm` |

`/root/workspace` used to be a symlink to `/workspace`; it was **removed
2026-08-07** (IDE workspace scans followed it into 4.3 TB and choked the
sessions). Always use `/workspace` paths directly. Upstream-PR branches
and experiments live in worktrees off the same clone
(`git -C /workspace/vllm-v0.24.0 worktree list`).

`moet-v0.24.0` is the ship lineage: everything committed there is meant to
ship in the next patch regen. Park half-done work on a side branch or
worktree, not on `moet-v0.24.0`.

## Where a change goes

| change | commit where | must also update |
|---|---|---|
| vLLM runtime code (`moe_w2_*`, loaders, runner, attention, …) | fork branch `moet-v0.24.0` | regen the patch here — procedure below |
| SASS kernels / cubins | `kernels/` | a `kernels/MANIFEST.md` row (generator + validation status) |
| serve configs | `bench/recipes/` | `bench/models.yaml`, `bench/matrix.yaml`; run `bench/runner/lint.py` |
| bench results | `bench/results/<release>/` | `bench/runner/render.py` — the README table and per-release report are **generated**; never hand-edit the marked README block |
| docs | `docs/`, README outside the generated block | — |
| session notes / handoffs / experiment scraps | `internal/` (gitignored, stays local) | never into `docs/` |

Never commit: wheels (`patch/*.whl` is ignored), checkpoints, expert packs,
smoke results (`bench/results/smoke/`).

## Shipping a vLLM code change — the procedure

1. **Commit on the fork branch** (`/workspace/vllm-v0.24.0`,
   `moet-v0.24.0`). If the remote may have moved, fetch and merge first —
   the regen tool refuses to run when the local branch is missing pushed
   commits.
2. **Regenerate** from this repo:

   ```bash
   python3 tools/check_patch_files.py --update
   ```

   This rewrites the patch from the branch tip and updates the two
   committed fingerprints: `patch/FILES.txt` (file list) and
   `patch/SOURCE.txt` (the fork SHA the patch was generated from). It
   refuses to move `SOURCE.txt` backwards along the branch, so a
   regeneration can never roll back work that already shipped.
3. **Review `git diff patch/`.** An entry *vanishing* from `FILES.txt`
   means the patch carried work that never reached the fork branch —
   someone skipped step 1. **Stop and merge that work into the branch**;
   never ship the loss, never `--update` a second time to silence it.
4. **Validate what the change class requires.** Byte-exact generation from
   the branch already guarantees `git apply --check` on the tag. For
   GPU-relevant changes run the relevant suites
   (`tools/test_moe_w2_forward.py`, `tools/test_store_backends.py`, the
   three-tier tests) and put the results in the commit message, as the
   existing history does.
5. **Commit both repos, cross-referenced.** The vLLM-Moet commit that ships
   a regen names the fork SHA, following the established style:

   > `Three-tier starvation fix ships: step-scoped seen windows (vllm 9736e4d34)`

6. **Push both together** (`fork moet-v0.24.0` + `origin main`) once the
   pre-push checklist passes — or leave both unpushed. Avoid a lasting
   state where only one side is pushed.

## Concurrency — several agents, one checkout

- `git status` before you start. Dirty paths you did not create belong to
  another live session: **leave them alone**. No `git add -A`, no
  `git commit -a`, no `git stash`, no `git checkout --` / `git reset` over
  someone else's files, ever.
- Stage **explicit paths only**: `git add <file> <file> …`.
- `main` and `moet-v0.24.0` are shared trunks: no amending commits you did
  not just create, no rebase, no force-push, no history rewrite.
- The `patch/` trio (patch, `FILES.txt`, `SOURCE.txt`) changes **only** via
  `--update`. If your commit would touch any of them for another reason,
  you are doing something wrong.
- **Hardware is shared too, and a written claim is not a lock.** Two sessions
  landed on the same GPU 48 s apart on 2026-07-25 because both had surveyed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kacper-daftcode/vLLM-Moet](https://github.com/kacper-daftcode/vLLM-Moet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
