---
trigger: always_on
description: The repository law, operating discipline, and code stance — the single canonical constitution for every contributor, human or agent.
---

# CLAUDE.md

The repository law, operating discipline, and code stance — the single canonical constitution for every contributor, human or agent.

> **Activating governance on this existing repository?** Read [SETUP.md](SETUP.md). Preserve the Talos package, history, authored docs and Hatch version source; configure external controls explicitly.

## Authority and activation

User and higher-priority session instructions take precedence over this repository contract. Local work remains authorized by the task. A repository rule is not permission to open a PR, request review, post comments, merge, publish or change remote settings.

The laws describe the intended enforced policy. The checked-in law/configuration/ruleset contract is locally testable; the adopted required-check/review ruleset and privileged audit still require activation through [SETUP.md](SETUP.md). Do not claim a local check proves remote enforcement.

## Motivation

We don't want saga ornamentation. We want commits that move the needle. The needle is the thing that humans actually benefit from in the software. Always ask "what are the key usage paths here, and how is this proposed change moving those?". Never be satisfied with work that seems to check the boxes, but doesn't really move the capability where the actual benefit is.

## The laws

Eleven laws. Ten are workflow gates on every PR; the eleventh is intended branch protection on `master`. Once activated, any required failure blocks merge. Existing debt is recorded in measured budgets, never hidden by disabling gates.

1. **Every PR closes exactly one OPEN slice-labelled issue — plus that slice's parent PRD when, and only when, the slice is the PRD's last open sub-issue.** PR title byte-equals the slice issue's title. Diff stays within the issue's `## Surfaces` globs. Diff touches no path in `## Out of Scope`. Issue body preserves every `> **Significance.**` blockquote from the slice template verbatim, and every `Done Means` checkbox is checked or carries `OVERRULED: <reason>` before merge; the `slice_closeout_guard` workflow writes the closeout evidence fields when a merged PR closes the issue and reopens an evidence-less close. A PR opened by an author in `automation.bot_authors` skips this law while `automation.exempt_gates` lists `slice` — dependency bumps carry no slice issue, and the other nine laws still run on them. *(pr_checks_slice)*

2. **PR title, every non-merge commit, and the linked issue title match Conventional Commits v1.0.0, and no commit message or the PR title names an AI/LLM assistant.** Allowed types: `feat, fix, docs, style, refactor, perf, test, build, ci, chore, revert`. *(pr_checks_cc)*

3. **Typing discipline never weakens.** No new `Any`, `cast(..., Any)`, `# type: ignore`, `# pyright: ignore`, or `# noqa`. Pyright error count cannot rise. The `typing` budget in `.github/budgets.json` cannot be raised, and the scan surface in `governance.yml` cannot be narrowed, by the PR it gates. *(pr_checks_typing)*

4. **Silent-fallback patterns never grow.** No new bare `except:`, empty handler (`pass`, `...`, `return`, `return None`, `continue`, `break`), `contextlib.suppress` (or any alias chain thereof), or `errors='ignore'`. The `fail_loud` budget in `.github/budgets.json` cannot be raised, and the scan surface in `governance.yml` cannot be narrowed, by the PR it gates. *(pr_checks_fail_loud)*

5. **Every PR bumps the version and leaves a CHANGELOG trail.** `__version__` in `talos/__init__.py`, read dynamically by Hatch, advances strictly forward by `MAJOR.MINOR.PATCH`. `CHANGELOG.md`'s first `# v<X.Y.Z>` header equals the new version and carries at least one content line, written imperatively ("Add", not "Added") and free of leftover placeholders. Bump level meets the Conventional Commits type minimum: `type!` → major, `feat` → minor, else patch. Dependency-bot PRs also need the version, changelog and citation bump so every merge has a distinct publishable identity. *(pr_checks_version)*

6. **The lint gate passes.** Ruff, at the version `pyproject.toml` pins, across the declared package, `governance/` and test surfaces; measured Ruff and dead-code debt cannot increase; every package module and every `governance/` module within its declared line budget in `.github/budgets.json` (file-size balance applies to the package tree, where a ratio is meaningful; the gate modules are held by their individual budgets instead), and measured docstring debt cannot increase; the docstring conventions, file-size balance and test/code ratio remain within their measured bounds; test fallback debt cannot increase outside the paths `governance.yml` names as infrastructure, and no honesty-violation is swallowed; changed lines arrive covered; declared runtime dependencies carry no known vulnerability (pip-audit, with time-boxed `.github/vuln_exceptions.json` entries); the documentation corpus passes its locked audit, lint, link, build, route, asset, browser, and accessibility checks; and the coverage floor in `.github/budgets.json` holds and ratchets upward — it cannot be lowered by the PR it gates without a `[coverage-lower: <field>: <reason>]` marker. *(pr_checks_lint)*


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [autonomio/talos](https://github.com/autonomio/talos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
