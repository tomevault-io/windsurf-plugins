---
trigger: always_on
description: > **This file is the always-loaded context.** Keep it lean — it loads into *every* session. Detailed reference lives in `docs/` and is read on demand:
---

# DE-LIMP Project Context for Claude

> **This file is the always-loaded context.** Keep it lean — it loads into *every* session. Detailed reference lives in `docs/` and is read on demand:
> - **Gotchas / known-issue fixes** → `docs/GOTCHAS.md` (index below)
> - **Subsystem patterns** (Shiny, bslib, SSH, DIA-NN flags, Comparator, Proteogenomics, UI) → `docs/PATTERNS.md`
> - **Proteogenomics architecture** → `docs/PROTEOGENOMICS.md`
> - **HPC paths/containers** → `docs/HPC_PATHS.md` · **Queue switching** → `docs/QUEUE_SWITCHING.md` · **TODO** → `docs/TODO.md`

## Working Preferences
- **Update the right file**: project state/patterns → this file or `docs/`; change history → `CHANGELOG.md`; new gotcha → `docs/GOTCHAS.md`.
- **Bump the patch version after every user-visible fix**: update (a) the `VERSION` file, (b) the `# Version:` line in the `app.R` header comment, and (c) add a `CHANGELOG.md` entry under the new version. (a) drives the runtime console banner; (b) is what the user sees opening `app.R` in the editor. Keep them in sync.
- **NEVER run heavy computation on HPC login nodes** — submit via `sbatch` or request an interactive node with `srun`. Login nodes are shared; CPU/memory-heavy tasks can get the user flagged.
- **Check primary sources before guessing — NEVER guess anything verifiable.** Applies to EVERYTHING: algorithms, formulas, file paths, container locations, module names, binary paths, HPC config, API formats, parameters. If it can be checked, check it FIRST — SSH and run `find`/`which`/`ls`, fetch source from GitHub, read config files. Do NOT answer from memory. Past failures: tof-to-mz formula guessed from first principles was off by 155 Da (correct one was in timsrust source); claimed `module load diann` when DIA-NN is an Apptainer container; used depthcharge's default peak filtering thinking it was Cascadia's.

## Architectural rules (NEVER violate — discovered the hard way in v3.9.x)

These four rules exist because each was violated in early DE-LIMP and produced wrong-but-plausible exports that misled real analyses. Before writing code that touches user-facing exports, methods text, AI prompts, info modals, or reproducibility logs — stop and check this list.

1. **Pipeline objects must self-describe — never hardcode a description of "what we did".** Every quantification path returns a self-describing object; downstream consumers read `$pipeline_id`, `$methods_paragraph`, `$rollup_method` from it. Never write `if (isTRUE(values$pipeline_mode_used == "maxlfq")) ...` in a new file — put that branch in a single helper. Adding a third pipeline must require zero edits to downstream files. (Real users once got reports describing the wrong pipeline because methods/AI/Comparator/repro text all hardcoded `"DPC-Quant... dpcDE()"`.)

2. **`%||%` defaults that flow into user-facing text must be tagged.** For any value that ends up in an export / AI prompt / methods string, either render `(DEFAULT — not user-confirmed)` next to it, or replace the `%||%` with `NA_character_` and have the prompt-builder skip the line. Never silently substitute a fabricated value (e.g. `coalesce_setting(..., "0.05")`) for missing user input — a reviewer reading "FDR threshold: 0.05" wrongly assumes the user set it.

3. **Concepts have one definition.** Every shared concept (detection status, pipeline label, covariate display name, DIA-NN defaults, classifier rules) lives in exactly **one** file as a function/constant, imported elsewhere. If you find yourself copying logic to a second site, refactor instead. ("Detected vs Inferred" once had 4 independent classifiers; the covariate display name is read 22 times.)

4. **Silent catch is banned in export paths.** In every multi-file export bundler use `safe_section(manifest, name, expr)` from `R/helpers.R` — records `[OK]` on success, `[SKIPPED] <name> -- <reason>` to a `MANIFEST.txt` in the ZIP root on failure. Never write `tryCatch(error = function(e) NULL)` around CSV writes / ZIP entries — a downstream user gets a ZIP silently missing whole sections.

When reviewing your own changes: ask whether the export still describes the analysis correctly under *every* pipeline + *every* input shape. If you can't answer "yes" with certainty, branch the code or add a test before shipping.

## Review Agents (spawn before major releases)
After significant changes, spawn these 5 in parallel: (1) **Biological researcher** — workflow intuitiveness, jargon, missing biology features; (2) **Proteomics expert** — DIA-NN integration, QC, core-facility readiness, instrument support; (3) **Statistician** — statistical validity, multiple testing, no-replicates caveats; (4) **Error handling & UX audit** — silent failures, blank `req()` screens, missing validation; (5) **Documentation audit** — stale references, version mismatches, missing features.

## Project Overview
DE-LIMP is a Shiny proteomics data analysis pipeline using the LIMPA R package for differential expression analysis of DIA-NN data.
- **GitHub**: https://github.com/bsphinney/DE-LIMP · **Hugging Face**: https://huggingface.co/spaces/brettsp/de-limp-proteomics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bsphinney/DE-LIMP](https://github.com/bsphinney/DE-LIMP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
