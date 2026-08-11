---
trigger: always_on
description: Security-focused linter for Docker Compose files. Python >=3.10, PyYAML only runtime dep, MIT license.
---

# compose-lint

Security-focused linter for Docker Compose files. Python >=3.10, PyYAML only runtime dep, MIT license.

## Architecture

- **Parser**: `LineLoader(yaml.SafeLoader)` captures line numbers. `load_compose(path) -> (data, lines)`. Handles v2/v3, anchors, merge keys (`<<:`), extension fields (`x-`). Leaves `${VAR}` unresolved. Validates `services:` exists and is a mapping. Does NOT do full Compose schema validation.
- **Rules**: Classes inheriting `BaseRule`, registered via `@register_rule`. IDs are `CL-XXXX`, zero-padded. From 1.0 they are permanent and never reused; pre-1.0 a rule that should not have shipped may be removed and its id reclaimed. Rules receive plain Python types (`dict`, `list`, `str`, `int`, `bool`), never parser-specific types.
- **Findings**: Dataclasses in `models.py`, not dicts.
- **Formatters**: Modules in `formatters/` exposing `format(findings) -> str`.
- **Engine**: `engine.py` runs rules, collects results, applies config overrides.

## Severity

CRITICAL > HIGH > MEDIUM > LOW. A rule's severity is **derived**, not chosen: it is what the two-axis matrix in `docs/severity.md` produces for the rule's cell, under a stated attacker baseline and the grounded Docker posture (ADR-020). Shipping a different number is legal only as a declared override from the closed reason list, with a link — `tests/test_severity_matrix.py` fails otherwise. Evaluating cells until one yields the number you expected is the documented failure mode the model exists to prevent, so if the result disagrees with instinct, fix an axis definition or file an override rather than trying a different cell.

## Exit codes

- 0: No findings at/above threshold
- 1: Findings at/above threshold
- 2: Usage error (bad args, file not found, invalid Compose)
- Default threshold: HIGH. Configurable via `--fail-on`.

## CLI output

Stdout carries data (findings in `check`; future artifacts in operations like `fix` or `completion`). Stderr carries human status and errors. Today's text-mode banner, per-file summary, aggregate summary, and verdict are the exception — gated on `output_format == "text"` in `cli.py` so JSON/SARIF redirects stay clean. When a second stdout-emitting mode lands, decide in that feature's ADR whether to extend the gate or move human-status lines to stderr permanently.

## Config file (`.compose-lint.yml`)

Disables still produce suppressed findings. `reason` flows to `suppression_reason` (JSON), `justification` (SARIF), or after `SUPPRESSED` (text). No inline suppression comments.

## Quality checks

`ruff check src/ tests/`, `ruff format --check src/ tests/`, `mypy src/` (strict), `pytest`. All four must pass, scoped exactly as written — CI lints only `src/` and `tests/`, and a bare `ruff check` also sweeps `scripts/`, which has known, accepted violations. CI test matrix: Python 3.10–3.14 on ubuntu-24.04.

## Adding a rule

See CONTRIBUTING.md for the full checklist. Every rule must cite OWASP, CIS, or Docker docs **that demonstrate the need in a container context** — generic host/Linux hardening a container's defaults already neutralize is not enough (see CL-0022/CL-0023). If a container-context source is thin, the rule's premise must be **validated at runtime**: a rule that describes container runtime state gets a check in `scripts/validate_rule_premises.py`, which proves the insecure state is Docker's *default* (for absence rules) or that the flagged config produces the insecure behavior (for presence rules). That suite runs in CI (`rule-premises` job), and asserts the daemon is at Docker's defaults before measuring — a premise measured on a departed posture cannot ground a rule. Every finding must be actionable with specific fix guidance.

A new rule also needs its derivation block (with an **Evidence** line naming the premise check or captured observation), a row in `docs/severity.md`'s assignment table, a listing on the README/`docs/index.md`/mkdocs-nav surfaces, and an ATT&CK mapping in `src/compose_lint/attack.py` or a recorded reason none fits. Each is enforced by a test.

## Contributor workflow

CONTRIBUTING.md is the source of truth for commits, signing, and PRs. Key points:
- One logical change per commit, imperative subject under 72 chars, no Conventional Commits prefixes
- All commits signed (SSH). Verify with `git log --format='%h %G? %s'` — every commit shows `G`
- All changes go through a PR, squash-merge to main
- Releases: follow `docs/RELEASING.md` checklist — version lives in both `pyproject.toml` and `src/compose_lint/__init__.py`

## Rule docs (docs/rules/)

- H1 format: `# CL-XXXX: <directive> — <symptom phrasing>` (query-phrased, id first).
  The docs-site `<title>` comes from the matching nav label in `mkdocs.yml` — keep
  both in sync. `tests/test_cli.py` pins CL-0003's H1; update it if that changes.
- "Reading the failure" symptom tables quote **verbatim, live-captured** error
  strings. Busybox wordings must be re-proven by a mapping check in
  `scripts/validate_rule_premises.py` (see the ADR-016 amendment); other wordings
  (coreutils/bash/application) are captured-but-not-asserted and labeled as such.
  Mechanism claims are observed against a live container, never reasoned from docs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmatens/compose-lint](https://github.com/tmatens/compose-lint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
