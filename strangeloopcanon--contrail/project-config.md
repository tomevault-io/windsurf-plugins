---
trigger: always_on
description: Operating manual for autonomous coding agents across this monolith workspace. Default to AGENT_MODE=baseline; escalate per Appendix E triggers.
---


# Drop‑In Rules for Autonomous Coding Agents
> Constitution + lean appendices; ship minimal, correct, succinct code.

## Quick start (read this first)
- Set `AGENT_MODE` (baseline by default; see Appendix E for production triggers).
- Prefer `uv` over `pip`.
- Run Interface Contract commands (stop after first failure):
  - `setup` / `bootstrap`: env + deps; install/refresh hooks (pre-commit, commitlint).
  - `check`: format check, lint, types (mode-appropriate), quick Bandit, detect-secrets on staged changes.
  - `test`: unit/integration; enforce coverage by mode; no live network unless explicitly marked.
  - `llm-live` (if LLM code exists): goldens with cost + p95 latency.
  - `deps-audit`: advisory in baseline; blocking in production; SBOM in production.
  - `all`: `check` → `test` (+ `llm-live` if applicable) → (`deps-audit` in production).
  - `release` (production): semantic release + changelog + tag.
- Local vs CI: local runs fast gates; CI enforces full suite (strict types in production, LLM-live, SAST, dep audit).
Agents MUST call these targets, not raw tools.

## Stack & paths (project knowledge)
- Monolith, single process; all architecture code lives here. Modules only call via `some_module/api.py` with statically typed interfaces (Pydantic preferred; no opaque dicts). CI/pre-commit enforce this.
- Read from module code; write only through declared interfaces and allowed dirs for the task.
- `.env` is the source of secrets; never commit secrets.
- Do not touch vendor/prod configs, generated/binary assets, or `node_modules/`/`vendor/` unless asked.

## Boundaries
- **Always:** follow Interface Contract commands; prefer `uv`; update README/docstrings when behaviour changes (start with “so what”; make optional sections collapsible; avoid exhaustive technical lists); structured logs (JSONL, no f-strings in log calls; pass fields); stop and ask for help if blocked; find root cause (no band-aids).
- **Ask first:** schema changes; new dependencies; CI/hook policy changes; module renames; prod/dev deploys; changing coverage/type thresholds.
- **Never:** commit secrets/PII; edit prod configs or generated/vendor folders; add “improved/better/new” comments; push directly to `main`; skip tests because they fail; hard-reset user work.

## Quality gates by mode
| Topic | Baseline | Production |
| --- | --- | --- |
| Format | Prettier (JS/TS) | + Black (Py) |
| Lint | Ruff defaults; ESLint v9 flat | Expanded rules; exceptions documented |
| Types | Basic | `mypy --strict`; TS `strict`; no lingering ignores |
| Coverage | Global ≥80%; no regression | Global ≥90%; changed lines ≥90%; mutation tests on critical modules (scheduled) |
| LLM live | Mandatory if LLM code exists | + faithfulness/relevance, OWASP LLM Top-10 probes, SLO gates |
| SAST | Bandit advisory; Semgrep minimal advisory | Bandit + Semgrep blocking on high severity |
| Deps | `pip-audit` advisory | `pip-audit` blocking; SBOM |
| Releases | Conventional Commits | Semantic release + changelog |

## Development cycle
1) Plan and log assumptions when the task is large enough to need them.
2) Implement smallest viable slice with typed interfaces.  
3) Run Interface Contract (halt on first failure; triage via Appendix B).  
4) Open PR with tests (LLM live if relevant), risks, rollback.  
5) If blocked, split scope and log deferrals as issues.  
6) If asked to update GitHub: create PR, merge via CLI, sync, return to main, confirm.

## Tooling and style
- Format/Lint: Black, Ruff (Py); ESLint v9 + Prettier (JS/TS); flake8/pylint if present.
- Types: mypy; `tsc`.
- Tests: pytest + pytest-cov; property-based where stateful/numeric/parser-like; no live network in unit tests; integration uses local fakes unless marked.
- Secrets: detect-secrets (pre-commit) blocks new secrets.
- Security: Bandit; Semgrep (advisory → blocking in production).
- Deps audit: `pip-audit`.
- Repo ops: `gh`/`glab`; commitlint.
- Python env: use existing venv or create/log if needed; prefer `uv` installs.
- OS: macOS (M4).

## LLM-specific rules
- Live tests are non-negotiable; staging credentials only.  
- Output shape: choose JSON vs free-text; validate (JSON Schema/Pydantic) or enforce invariants.  
- Providers: lock provider+model+version for goldens; default GPT-5 unless specified; see https://github.com/strangeloopcanon/llm-api-hub for latest docs.  
- Determinism: `temperature=0`, `top-p=1` for goldens.  
- Retries: transient (5xx/429/timeout) up to 3 attempts with jitter; deterministic errors fail fast. Max 9 total calls/job; if exhausted on transients, exit code 3.  
- Cost ceilings: Baseline ≤ $3, Production ≤ $10; exit code 2 if exceeded (advisory in baseline).  
- Logging: redact PII/secrets; no raw model I/O when user data may appear.

## Git & PR workflow
- Branches: `feat/*`, `fix/*`, `chore/*` off `main`.  
- Commits: Conventional Commits; use `BREAKING CHANGE:` footer when needed.  
- PRs: summary, rationale, reproduction commands; link issues; focused diffs.  
- Checklist: format/lint; types by mode; tests + coverage; LLM live (or N/A reason); secrets/SAST clean; dep audit reviewed; docs updated; if rollback claimed, add `@pytest.mark.rollback` test.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [strangeloopcanon/contrail](https://github.com/strangeloopcanon/contrail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
