---
trigger: always_on
description: ui-clone-skills is motion forensics for the animated web — four jobs against a live URL (`decode` / `clone` / `verify` / `extract`) backed by one extraction engine that pulls real CSS + animation params from JS bundles and verifies with AE/SSIM diff.
---

# AGENTS.md — ui-clone-skills development guide

ui-clone-skills is motion forensics for the animated web — four jobs against a live URL (`decode` / `clone` / `verify` / `extract`) backed by one extraction engine that pulls real CSS + animation params from JS bundles and verifies with AE/SSIM diff.

Canonical project guide for Claude Code, Codex, and other agent hosts. Claude Code loads `CLAUDE.md`, which imports this file via `@AGENTS.md`; Codex and other `AGENTS.md`-aware tools read this file directly.

## Project structure

```
ui_clone/          Python package (gates, hooks, pipeline, DAG, metrics)
skills/            3 reusable skills; skill-owned primitives live under skills/<skill>/scripts/
scripts/           Repo automation: ci/, hooks/, extract/, verify/
hooks/             Claude/Codex host hook registration and shim plumbing
tests/             pytest suite for gates, hooks, metrics, and pipeline behavior
.claude-plugin/    Claude Code plugin manifest and marketplace metadata
.codex-plugin/     Codex plugin manifest
CLAUDE.md          Thin Claude Code entrypoint that imports this file
```

## Script location policy

- Host lifecycle hook registration stays in `hooks/` (`hooks.json`, `codex-hooks.json`, `shim.sh`). Do not move `shim.sh` into `scripts/`; hook manifests call it as host integration plumbing.
- Repo-level automation lives in `scripts/`: `scripts/ci/` for CI/review/security, `scripts/hooks/` for local git/tool hooks, `scripts/extract/` for shared extraction helpers, and `scripts/verify/` for shared verification wrappers.
- Skill-owned primitives live under `skills/<skill>/scripts/`. `skills/visual-debug/scripts/` owns AE/SSIM, section, transition, and visual-diff primitives.
- Orchestration skills should reference shared/skill-owned scripts by their canonical location instead of duplicating compatibility shims.

## Verification gate (must pass before commit)

```
[] bash scripts/ci/ci-local.sh — 0 failures (mirrors GitHub Actions test job; runs pytest + mypy + ruff + shell + review)
[] bash scripts/ci/pre-push-security.sh — 0 blockers (security + cross-ref + version sync)
```

`scripts/ci/ci-local.sh` is the single source of truth for what CI runs. `scripts/hooks/pre-push-guard.sh` calls it automatically before `git push` when configured as an agent hook (bypass for emergencies: `UI_RE_SKIP_CI_LOCAL=1 git push`). If you change CI, update `ci-local.sh` to match — and vice versa. Run a single test module with `uv run python -m pytest tests/test_<module>.py`; validate gates with `python -m ui_clone.gate tmp/ref/<c> all`.

### Verification tier (cost control)

`skills/visual-debug/scripts/verification-plan.sh` accepts `--tier=quick|standard|comprehensive` (env: `UI_CLONE_VERIFY_TIER`, default `comprehensive`). Each `add_check` row is tagged with a `min_tier`; the plan emits only checks at or below the active tier.

- `quick` (~10s) — static analysis + JSON-comparison rows only (`hydration-check`, `tailwind-transform-conflict`, `transition-spec-coverage`, `runtime-spec-coverage`). Use during inner iteration loops where running the full browser sweep on every change is wasteful.
- `standard` (~1min) — `quick` + one-shot browser interactions (`scroll-end-completion`, `reveal-trigger`, `transition-compare`, `font-parity`). No 60fps video recording.
- `comprehensive` (~5min+) — `standard` + 60fps frame-by-frame motion compares (`video-motion-compare`, `hover-state-compare`, `click-state-compare`). **Default** — preserves the unconditional dispatch from before the tier system was added.

Default stays `comprehensive` so existing callers and CI keep their current safety guarantees. Drop to `quick`/`standard` only when iterating against a single signal class and you want a faster feedback loop.

## Rules

### Language
- All skill docs (`skills/**/*.md`), README, CHANGELOG, eval fixtures (`skills/*/evals/*.json`): **English only**. Enforced by `scripts/ci/review.sh` language check.
- Code comments: English only
- Commit messages: English only

### Source fidelity
- This repo does not enforce identity/name redaction. Preserve observed site names, service names, brand text, asset URLs, image/video/Lottie references, alt/title/aria labels, and other user-visible identity strings when they are part of the reference, clone output, benchmark, test fixture, or explanatory evidence.
- Generic examples may use placeholders when they are clearer, but placeholders are a style choice, not a compliance rule. Do not rewrite source copy to `Example`, generic brands, sanitized labels, emoji, gradients, or placeholder text for policy reasons.
- Do not treat HTTP 200, page title, section ids, build success, source string presence, or a proxy/cache of the original site runtime as fidelity evidence. Clone quality requires actual rendered text, visible assets, DOM/section structure, and motion runtime behavior to match the reference from generated implementation source.

### Naming
- Python package: `ui_clone`
- npm packages: `ui-clone-cli` (binary `ui-clone`), `agent-browser`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [voidmatcha/ui-clone-skills](https://github.com/voidmatcha/ui-clone-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
