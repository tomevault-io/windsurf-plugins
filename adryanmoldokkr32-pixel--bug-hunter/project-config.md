---
trigger: always_on
description: Adversarial bug hunting with a sequential-first pipeline (Recon, Hunter, Skeptic, Referee) that can optionally use safe read-only parallel triage. Finds, verifies, and auto-fixes real bugs by default (with --scan-only opt-out) using checkpointed verification and resume state for large codebases. Use this skill whenever the user wants bug finding, security audits, regression checks, or code review focused on runtime behavior.
---


# Bug Hunt - Adversarial Bug Finding

Run a sequential-first adversarial bug hunt on your codebase. Use parallelism only for read-only triage and independent verification tasks.

## Table of Contents
- [Usage](#usage)
- [Target](#target)
- [Context Budget](#context-budget)
- [Execution Steps](#execution-steps)
- [Step 7: Present the Final Report](#step-7-present-the-final-report)
- [Self-Test Mode](#self-test-mode)
- [Error handling](#error-handling)

**Phase 1 — Find & Verify:**
```
Recon (map) --> Hunter (deep scan) --> Skeptic (challenge) --> Referee (final verdict)
                    ^                 (optional read-only dual-lens triage can run here)
                    |
             state + chunk checkpoints
```

**Phase 2 — Fix & Verify (default when bugs are confirmed):**
```
Baseline --> Git branch --> sequential Fixer (single writer) --> targeted verify --> full verify --> report
                    ^                                                              |
                    +------------------------ checkpoint commits + auto-revert -----+
```

For small scans (1-10 source files): runs single Hunter + single Skeptic (no parallelism overhead).
For large scans: process chunks sequentially with persistent state to avoid compaction drift.

## Usage

```
/bug-hunter                              # Scan entire project
/bug-hunter src/                         # Scan specific directory
/bug-hunter lib/auth.ts                  # Scan specific file
/bug-hunter -b feature-xyz              # Scan files changed in feature-xyz vs main
/bug-hunter -b feature-xyz --base dev   # Scan files changed in feature-xyz vs dev
/bug-hunter --pr                        # Easy alias for --pr current
/bug-hunter --pr current                # Review the current PR end to end
/bug-hunter --pr recent --scan-only     # Review the most recent PR without editing code
/bug-hunter --pr 123                    # Review a specific PR number
/bug-hunter --pr-security               # PR security review: PR scope + threat model + dependency scan
/bug-hunter --last-pr --review          # Easy mnemonic for “review the last PR”
/bug-hunter --review-pr                 # Alias for --pr current
/bug-hunter --staged                    # Scan staged files (pre-commit check)
/bug-hunter --scan-only src/            # Scan only, no code changes
/bug-hunter --review src/               # Easy alias for --scan-only
/bug-hunter --fix src/                   # Find bugs AND auto-fix them
/bug-hunter --plan-only src/             # Build fix strategy + plan, but do not edit files
/bug-hunter --plan src/                  # Easy alias for --plan-only
/bug-hunter --safe src/                  # Easy alias for --fix --approve
/bug-hunter --preview src/               # Easy alias for --fix --dry-run
/bug-hunter --autonomous src/            # Alias for no-intervention auto-fix run
/bug-hunter --fix -b feature-xyz        # Find + fix on branch diff
/bug-hunter --fix --approve src/        # Find + fix, but ask before each fix
/bug-hunter src/                         # Loops by default: audit + fix until all queued source files are covered
/bug-hunter --no-loop src/               # Single-pass only, no iterating
/bug-hunter --no-loop --scan-only src/   # Single-pass scan, no fixes, no loop
/bug-hunter --deps src/                 # Include dependency CVE scan
/bug-hunter --threat-model src/         # Generate/use STRIDE threat model
/bug-hunter --security-review src/      # Enterprise security workflow: threat model + CVEs + validation
/bug-hunter --validate-security src/    # Force vulnerability-validation for security findings
/bug-hunter --deps --threat-model src/  # Full security audit
/bug-hunter --fix --dry-run src/        # Preview fixes without editing files
```

## Target

The raw arguments are: $ARGUMENTS

**Parse the arguments as follows:**

0. Default `LOOP_MODE=true`. If arguments contain `--no-loop`: strip it from the arguments and set `LOOP_MODE=false`. The `--loop` flag is accepted for backwards compatibility but is a no-op (loop is already the default).

0b. Default `FIX_MODE=true`.
0c. If arguments contain `--scan-only`: strip it from the arguments and set `FIX_MODE=false`.
0d. If arguments contain `--fix`: strip it from the arguments and set `FIX_MODE=true`. The remaining arguments are parsed normally below.
0e. If arguments contain `--autonomous`: strip it from the arguments, set `AUTONOMOUS_MODE=true`, and force `FIX_MODE=true` (canary-first + confidence-gated).
0f. If arguments contain `--approve`: strip it from the arguments and set `APPROVE_MODE=true`. When this flag is set, Fixer agents run in `mode: "default"` (user reviews and approves each edit). When not set, `APPROVE_MODE=false` and Fixers run autonomously.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adryanmoldokkr32-pixel/bug-hunter](https://github.com/adryanmoldokkr32-pixel/bug-hunter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
