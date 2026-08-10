---
trigger: always_on
description: Instructions for AI coding assistants (Claude Code, Codex, Cursor, Cline, Aider, etc.) working in this repository. This file follows the [AGENTS.md convention](https://agents.md/). Vendor-specific files (e.g., `CLAUDE.md`) point here.
---

# AGENTS.md

Instructions for AI coding assistants (Claude Code, Codex, Cursor, Cline, Aider, etc.) working in this repository. This file follows the [AGENTS.md convention](https://agents.md/). Vendor-specific files (e.g., `CLAUDE.md`) point here.

Human contributors: [`CONTRIBUTING.md`](./CONTRIBUTING.md) is the right doc.

## What this project is

`agent-vault-proxy` is a credential broker - a loopback HTTPS proxy that fetches API credentials from Bitwarden Secrets Manager just-in-time and substitutes them into outbound requests, so the calling agent's address space never contains the real secret bytes.

Read [`docs/architecture.md`](./docs/architecture.md) before making non-trivial changes. The whole design hangs on nine binary invariants (G1–G9); changes that affect those need explicit human discussion, not just a passing test.

## Hard constraints

These are non-negotiable. Violating any of them turns a PR into a security incident.

1. **Never log real secret values.** Not in `print()`, not in exception messages, not in audit events. The audit log records *decisions* (which secret name, which destination), never *contents*.
2. **Never silent-swallow exceptions.** `except Exception: pass` is forbidden. If you genuinely need to ignore an error, narrow the exception type, add a comment explaining what's being swallowed and why, and emit something observable.
3. **Never change the audit event schema** without bumping the JSON contract version and updating `docs/architecture.md` §4.4. Operators parse this log.
4. **Never weaken the G1–G9 invariants.** If a refactor moves substitution earlier, eliminates an fsync, or relaxes the SNI/Host consistency check, it's a security-affecting change that needs explicit human sign-off.
5. **Never replace `pip install --require-hashes --only-binary :all:`** with looser variants. Both flags exist for a reason. **`pip --require-hashes` at install time is the actual supply-chain enforcement mechanism** (CI runs it in the `test` workflow; pip itself refuses any install where a requirement lacks a matching hash). The structural pre-commit/CI check `scripts/check-lockfile-hashes.py` is a *fast guard* that the lockfile is shaped correctly - it complements but does not replace the install-time check. Every package in `requirements.lock` and `requirements-dev.lock` must carry at least one `--hash=sha256:` line.
6. **Never modify `.github/workflows/*.yml` without re-running `zizmor` and `pinact`.** Specifically: every third-party action reference must be pinned to a 40-character commit SHA (not `@v1` or other mutable tag); every checkout must set `persist-credentials: false`; every job needs an explicit `permissions:` block scoped to least privilege; downloaded artifacts must extract to `/tmp/`, never the workspace. Use `pull_request`, never `pull_request_target`. The existing workflows in this repo are the reference shape, match them.
7. **Never use `pull_request_target`** in any workflow. Forks would get secret access.
8. **Never commit or push.** Open a PR and let the human merge.
9. **Keep AVP single-license Apache-2.0.** Don't vendor code from sources that aren't Apache-2.0-compatible (MIT/BSD/Apache are fine; GPL/LGPL/AGPL/SSPL are not). Designs from prior art may be referenced — see [`CREDITS.md`](./CREDITS.md) — but code is original Python written from the underlying public specs (RFC / vendor docs / standard).

## Setup

```bash
python3 -m venv .venv
.venv/bin/pip install --only-binary :all: -e '.[dev]'
.venv/bin/pre-commit install --hook-type pre-commit --hook-type pre-push   # mandatory; CI runs the same checks
```

## The loop

```bash
.venv/bin/pytest -q                                  # tests
.venv/bin/ruff check src tests                       # lint
.venv/bin/ruff format src tests                      # apply formatting
.venv/bin/python -m bandit -c pyproject.toml -r src  # Python SAST
.venv/bin/pre-commit run --all-files                 # full pre-commit pass
```

Pre-commit runs ruff, bandit, TruffleHog (secret scan), Semgrep (pattern SAST), OSV-Scanner (CVE), zizmor, pinact, pytest, and basic hygiene hooks. The three Docker-based hooks gracefully skip if Docker isn't running locally, CI is the authoritative gate. Passing pre-commit locally means CI will pass on the same checks.

## Dependency changes

If your change adds, removes, or version-bumps a dependency in `pyproject.toml`, regenerate **both** lockfiles with the 7-day supply-chain cooldown applied:

```bash
CUTOFF=$(python3 -c 'from datetime import datetime, timedelta, timezone; print((datetime.now(timezone.utc) - timedelta(days=7)).strftime("%Y-%m-%dT00:00:00Z"))')
uv pip compile --generate-hashes --exclude-newer "$CUTOFF" \
  pyproject.toml -o requirements.lock
uv pip compile --generate-hashes --exclude-newer "$CUTOFF" --extra dev \
  pyproject.toml -o requirements-dev.lock
```

Two layers enforce this:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inflightsec/keys-on-the-wire](https://github.com/inflightsec/keys-on-the-wire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
