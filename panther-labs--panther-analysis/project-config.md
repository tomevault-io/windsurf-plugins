---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.) working in `panther-analysis`. Human contributors should also find this useful — it consolidates the rules, conventions, and gotchas that keep the repo healthy.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Cursor, Copilot, etc.) working in `panther-analysis`. Human contributors should also find this useful — it consolidates the rules, conventions, and gotchas that keep the repo healthy.

This is the canonical source. Tool-specific entrypoints (`CLAUDE.md`, `.cursorrules`, etc.) should reference this file rather than duplicate its contents.

---

## 1. Critical rules — read before doing anything

These are non-negotiable. Violating them creates security, legal, or release-process problems.

### 1.1 This is a PUBLIC repository
- **Never commit customer data, customer names, internal hostnames, real user emails, real IPs, real account IDs, API keys, tokens, or any other sensitive information.**
- Sample logs in unit tests **must be redacted/synthesized**. Use placeholder values like `123456789012` for AWS account IDs, `user@example.com` for emails, `192.0.2.x` (TEST-NET-1) for IPs.
- If you're adapting a real-world detection from an incident, scrub identifiers and rephrase any specifics that could fingerprint the source.
- Internal Panther context (Jira tickets, Slack threads, customer specifics) belongs in PR descriptions or commits **only if** it does not leak protected information — when in doubt, leave it out.

### 1.2 All PRs target `develop`, not `main`
- The default working branch is `develop`. `main` is the released branch and is updated by Panther's release process.
- When opening a PR with `gh pr create`, pass `--base develop` explicitly. Do **not** rely on the GitHub default.
- Do not push directly to `develop` or `main`. Always go through a PR.

### 1.3 CLA required
- All external contributors must sign the [Contributor License Agreement](https://gist.githubusercontent.com/jacknagz/5d097acd8c6ea462361e2d375b87e519/raw/21d2f529bf52c07e7e5c7be8acfe2f7d66688eae/Panther-Labs-CLA.txt) before a PR can merge. CI will block unsigned contributions.

### 1.4 Don't disable, skip, or weaken tests/linters to "make CI green"
- If `make lint` or `make test` fails, fix the underlying issue. Do not add `# pylint: disable=...`, delete failing tests, or use `--no-verify` on commits to bypass hooks.

---

## 2. Repository at a glance

| Path | Purpose |
| --- | --- |
| `rules/` | Streaming detection rules (analyze logs in real time) |
| `policies/` | Cloud resource configuration / compliance checks |
| `queries/` | Scheduled queries and signals for threat hunting |
| `correlation_rules/` | Multi-step / multi-signal attack patterns |
| `data_models/` | Field normalization across log sources (UDM-style) |
| `global_helpers/` | Reusable Python utilities (per-platform: `panther_aws_helpers`, `panther_okta_helpers`, etc.) |
| `lookup_tables/` | Reference data (CIDR ranges, account allowlists, etc.) |
| `packs/` | YAML manifests grouping detections for deployment |
| `templates/` | Starter templates for new detections — copy from here |
| `style_guides/` | Detailed style guides (read these) |
| `indexes/` | Auto-generated indexes — do not hand-edit |
| `deprecated.txt` | Tracks deleted detection IDs for customer cleanup |

### Dual-file architecture
Every detection is **two files** with the same basename:
- `foo_bar.py` — Python detection logic
- `foo_bar.yml` — metadata, configuration, and unit tests

The `.yml` `Filename:` field must match the `.py` filename exactly. Both files must be committed together.

---

## 3. Local development workflow

### Setup
```bash
make install                    # pipenv sync --dev
make install-pre-commit-hooks   # one-time
pipenv shell                    # activate venv
```

### The "before pushing" trio
Always run all three locally before opening a PR. CI runs the same checks.
```bash
make fmt    # isort + black (line length 100)
make lint   # pylint + bandit + isort check + black check
make test   # global helper unit tests + data model unit tests + pat test
```

### `panther_analysis_tool` (pat) — the workhorse
`pat` is the CLI used to test, validate, and upload detections. Always invoke through pipenv.

| Task | Command |
| --- | --- |
| Run all detection tests | `pipenv run panther_analysis_tool test` |
| Test one directory | `pipenv run panther_analysis_tool test --path rules/aws_cloudtrail_rules/` |
| Test one rule by ID | `pipenv run panther_analysis_tool test --filter RuleID=AWS.CloudTrail.Example` |
| Run a single named test case | `pipenv run pat test --filter RuleID=<id> --test-names "Specific test name"` |
| Filter by severity | `pipenv run pat test --filter Severity=CRITICAL` (comma-separate for multiple: `High,Critical`) |
| Filter by log type | `pipenv run pat test --filter LogTypes=AWS.GuardDuty` |
| Filter by analysis type | `pipenv run pat test --filter AnalysisType=rule` (or `policy`, or `rule,policy`) |
| Enforce minimum coverage | `pipenv run pat test --minimum-tests 2` (requires both a true and false case) |
| Debug a single test (print/breakpoints work) | `pipenv run pat debug <RuleID> "<unit test name>"` |
| Validate against a live instance (required for correlation rules) | `pipenv run pat validate --api-token ... --api-host ...` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panther-labs/panther-analysis](https://github.com/panther-labs/panther-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
