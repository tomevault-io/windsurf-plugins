---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Mission of this repo (as of 2026-06-04): verify the authenticity of the Ascend kernel-agent corpus and toolchain.**
> 语料/工具使用真实性审计。This charter governs the whole `Kernel-Agent-Ascend/` repo — it auto-loads under
> `KernelWiki/`, `msprof-report-skill/`, and `docs/`. It sits *between* two existing CLAUDE.md files, which it does
> not replace:
> - **Schema / controlled vocabulary / page types** → `KernelWiki/CLAUDE.md` (the three-layer architecture, tags/aliases/schemas, confidence + reproducibility ladders).
> - **Two-stack project layout / NV→Ascend mapping / ralph flow** → top-level `/models/wangakang/kernel_agent/CLAUDE.md`.
> This file adds the layer they lack: **what we are verifying, how, and what a future agent is authorized to do about what it finds.**

## Why this mission exists

`validate.py` going green means frontmatter matches the schema and tags are in the controlled vocabulary. It does **NOT**
mean the corpus is true. The repo has already burned itself on this: 7 fabricated blogs (unreachable `hiascend.com/forum/thread-*`
URLs) were deleted on 2026-06-03, and the 2026-06-04 baseline audit found **17 of 19 `verified` pages resting on hollow
evidence** that `validate.py` passed without complaint. The gap between "validates" and "is true" is this mission's entire reason
to exist.

The core insight, stated once: **"test passes" ≠ "claim verified", and "validates" ≠ "real".** Every rule below is a way of
closing that gap.

## Posture: audit → gate (two phases, one set of checks)

1. **Phase 1 — audit the existing corpus.** Run the four axes below over everything already committed, produce an
   `data/authenticity-audit.md` verdict, and remediate per the policy. (Baseline pass done 2026-06-04 — see that file.)
2. **Phase 2 — freeze the same checks as a standing gate.** Anything *new* (page, bundle, PR, doc, blog) must pass the same
   axis checks before it is considered real. The gate is not a separate findings ledger — **the gate IS the verifier scripts
   exiting clean** (`validate.py`, `verify_core_prs.py --strict`, `verify_verbatim.py --strict`, `audit_authenticity.py`,
   `verify_bundle_numerics.py` on-NPU). The `.md` record proves they ran.

## The four authenticity axes

The mission names two halves — **语料 (corpus)** and **工具使用 (tool-usage)**. Corpus splits into three axes; tool-usage is the fourth.

### Axis 1 — PR source authenticity (`sources/prs/**`)

**Tiered, forge-routed, checks merged-state — not just existence.**

- **Offline gate (runs anywhere):** flag seed-pattern pages (`PR-000X` with non-kernel `changed_paths` like `azureml/`),
  any `merge_sha: unknown`, and any `status: merged` not yet substantiated against upstream.
- **Online audit + new-PR gate:** resolve every PR against **its own forge** — `gitee.com` → Gitee v5 REST
  (`scripts/forge_resolve.py`), `github.com` → `gh`. Confirm the PR exists, title/author match the page, and **the recorded
  `status` matches upstream** (`merged_at` / `state`). Backfill the real anchor SHA.
- **Forge SHA semantics differ — this matters:** GitHub exposes `merge_commit_sha`; **Gitee never populates it** (null even
  for merged PRs), so the only anchor is `head.sha`. A page reading `merge_sha: unknown` is not "missing data" — for a Gitee PR
  the right field is `head.sha`, recorded with `merge_sha_kind: head.sha`.
- **The Ascend corpus deliberately includes closed-not-merged PRs** as stock-operator analogues. That is legitimate — so the
  check is **"does the page tell the truth about upstream state"**, NOT "must be merged". The defect is a page claiming `merged`
  when upstream closed it, which the old GitHub-only `--strict` could never catch (it 404'd on every Gitee PR).

### Axis 2 — on-hardware bundle authenticity (`artifacts/kernels/<slug>/variants/`)

**Tiered: static forensics always; on-hardware re-run for audit + required for new bundles.** Bundles are the only evidence that
lifts a page to `verified`, so they are the highest-stakes artifacts.

- **Static forensics (always-on gate):** every `files[*].sha256` in `PROVENANCE.yaml` matches the file on disk; `bench.md`
  headline numbers trace to the `op_summary*.csv` cells; column names are real CANN-8.3 msprof columns; no physically-impossible
  values. Static forensics catches tampering and internal inconsistency but **cannot** catch a self-consistent hand-authored fake.
- **On-hardware spot re-run (audit + new bundles):** only re-running on real 910C silicon distinguishes a genuine capture from a
  self-consistent fabrication. Run `scripts/verify_bundle_numerics.py` on the NPU (it asserts each op computes the right answer
  against an independent reference). For a *new* bundle the gate **requires** this at creation time.
- **All 13 current bundles are `asset_mode: derived`** (a harness we wrote + a real capture), not `verbatim`-upstream like the NV
  stack. That is acceptable — see the synthesis stance below — provided the capture is real and disclosed.

### Axis 3 — doc / blog authenticity (`sources/docs/**`, `sources/blogs/**`)

**Both checks, severity-split.** Two genuinely different failure modes:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ryannnice/Kernel-Agent-Ascend](https://github.com/Ryannnice/Kernel-Agent-Ascend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
