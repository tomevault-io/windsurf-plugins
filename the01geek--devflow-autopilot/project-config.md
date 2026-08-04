---
trigger: always_on
description: DevFlow is a single [Claude Code](https://code.claude.com) plugin published at the repo root; the repo is also its own marketplace. It turns a GitHub issue into a reviewed, documented, merged PR, and a weekly retrospective loop improves the automation. Full system reference: [`docs/DEVFLOW_SYSTEM_OVERVIEW.md`](docs/DEVFLOW_SYSTEM_OVERVIEW.md). Contributor guide: [`CONTRIBUTING.md`](CONTRIBUTING.md).
---

# DevFlow — project memory

DevFlow is a single [Claude Code](https://code.claude.com) plugin published at the repo root; the repo is also its own marketplace. It turns a GitHub issue into a reviewed, documented, merged PR, and a weekly retrospective loop improves the automation. Full system reference: [`docs/DEVFLOW_SYSTEM_OVERVIEW.md`](docs/DEVFLOW_SYSTEM_OVERVIEW.md). Contributor guide: [`CONTRIBUTING.md`](CONTRIBUTING.md).

## Commands

```bash
bash lib/preflight.sh      # verify git/gh/jq/python3.11+/PyYAML on PATH
bash lib/test/run.sh       # full suite: jq filters + shell helpers + python; gh-stubbed, no network/auth
git ls-files '*.sh' | grep -v '^lib/test/' | xargs -r shellcheck --severity=warning -e SC1091
shellcheck --severity=warning -e SC1091 --extended-analysis=false lib/test/run.sh
git ls-files '*.py' | xargs -r ruff check
```

**The `lib/test/run.sh` lint needs ShellCheck ≥ 0.10.0 and the `--extended-analysis=false` flag — both, or it does not work (issue #745).** ShellCheck's dataflow ("extended analysis") pass OOMs on a file this size; the flag disables it (the measured figures and what the flag costs live in the `ci.yml` pin step's own comment — a past-time snapshot to re-measure after a ShellCheck upgrade, not an invariant). But `--extended-analysis` (and the equivalent `# shellcheck extended-analysis=false` directive) landed in **0.10.0**: on 0.9.x — which is what `ubuntu-latest`/Ubuntu 24.04 ships — the flag errors and the directive is silently ignored (SC1107), so the lint OOMs exactly as if the flag were absent. CI therefore installs a pinned ≥ 0.10.0 binary rather than using the runner image's; at the desk, check `shellcheck --version` before concluding the lint is broken. The remaining `lib/test/**/*.sh` files are linted by the explicit list in `.github/workflows/ci.yml`, and `lib/test/lint-carveout-guard.py` (driven from `lib/test/run.sh`) turns the suite RED if a tracked `lib/test` script is neither in that CI-linted set nor under `lib/test/fixtures/`.

CI (`.github/workflows/ci.yml`) runs the same suite + lint on every PR. The **required** status check is the job name **`lib + python tests`** (not "CI", which is the workflow name and never resolves). CI checks out full history (`fetch-depth: 0`) so `origin/main` resolves and the `#434` stale-prose self-scan runs live there rather than self-skipping (issue #456).

**The suite reports three tallies — passed, failed, and *skipped* — and `0 failed` does NOT mean "everything ran" (issue #456).** A check can **self-skip** (it neither passed nor failed because the host could not run it or express its condition). With nothing skipped the summary is byte-identical to before — `N passed, M failed`; with `K` skips it reads `N passed, M failed, K skipped` followed by one line per skipped check (name, kind, reason). The exit code is unchanged (`[ "$FAIL" -eq 0 ]` — a skip never fails the suite). Each skip carries a **kind** — `blocking-gate` (a real gate that should have run here but could not — the `#434` self-scan arms) and `host-capability` (the host cannot express the condition — `#423 T6b`). Every self-skip routes through the single `skip <name> <kind> <reason>` helper in `lib/test/run.sh`; a meta-assertion turns the suite RED if a `printf '  NOTE ` emit appears outside that helper (it is the only sanctioned NOTE-emit; the scan is comment-aware and matches only the contiguous `printf` emit shape — an echo-based or variable-assembled emit is a disclosed non-goal). The renderer is `lib/test/summary.sh` (added to the CI shellcheck scope explicitly, since `lib/test/` is otherwise excluded). **A skipped check is never a clean pass:** when reading a suite run as verification (issue #405), a non-empty skip population must be surfaced, not laundered into `result: pass` — `/devflow:review-and-fix` records skips in `verification_evidence.skipped_checks` (distinct from `result: skipped`, which means the whole suite run was skipped) and a run whose `skipped_checks` is non-empty is not reported as a clean pass.

**Running the suite when the `bash <path>` wrapper above is denied — the tier matters.** The commands above are the *local* form. If the classifier denies the `bash <path>` wrapper (see the *LEADING token* gotcha and the classifier-fallback Convention below), do **not** silently fall back to "CI is the gate" — work the tiers in order:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The01Geek/devflow-autopilot](https://github.com/The01Geek/devflow-autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
