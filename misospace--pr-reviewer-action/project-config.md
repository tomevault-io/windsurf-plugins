---
trigger: always_on
description: This is a GitHub Action that analyzes pull requests using OpenAI-compatible or Anthropic-compatible models (cloud or self-hosted) and publishes the review as a sticky PR comment or a native GitHub review.
---

# Agents Guide: pr-reviewer-action

This is a GitHub Action that analyzes pull requests using OpenAI-compatible or Anthropic-compatible models (cloud or self-hosted) and publishes the review as a sticky PR comment or a native GitHub review.

## What it does

The action collects rich PR context (diff, files, linked issues, version hints, image digests, repo impact/history, standards files), runs a deterministic rule-based classification (PR kind, risk flags, required checks), assembles a review corpus, routes the review to a fast or smart model (optional), sends it to an LLM via OpenAI `POST /chat/completions` or Anthropic `POST /messages`, parses the JSON verdict + markdown body + optional structured findings, validates/enforces the result (required checks, findings severity gating, carried-forward findings, evidence/tool enforcement), and publishes via one of three modes (`comment`, `review_comment`, `review_verdict`).

## Key files

### Action definition and orchestration

- **`action.yml`** — Action definition with all inputs/outputs and composite run steps (precheck → CI wait → review → publish). The publish steps live inline in this file using helpers from `scripts/publish_helpers.sh`.
- **`scripts/platform_api.sh`** — Platform seam (#221): every host-forge API call goes through `platform_*` functions (github backend = the exact pre-seam `gh` invocations; forgejo backend = `pr_reviewer/forgejo_backend.py`, rolling out across 1.4.x). `github_enrich_*` functions are for linked-source enrichment and always target github.com. `pr_reviewer/platform.py` is the Python mirror for script consumers.
- **`scripts/check_review_needed.sh`** — Precheck: computes `git patch-id --stable` fingerprint, decides full vs. incremental scope, and skips if unchanged since last managed comment (unless `force_review=true`)
- **Re-review trigger** — adding the `rereview_label` (default `ai-review`) to a PR forces a fresh review (`check_review_needed.sh` reads the `labeled` event from `GITHUB_EVENT_PATH`, sets `force_review`, and skips unrelated labels; the label is removed post-publish in `action.yml`). Labels are maintainer-only, so no command-auth gate is needed.
- **`scripts/wait_for_ci.sh`** — Optional CI gating: polls the Checks API until checks reach a terminal state (`ci_status_check=true`), then renders the per-check outcomes to `CI_CHECKS_FILE` for the review corpus
- **`scripts/run_review.sh`** — Main review orchestration script (collects context, builds corpus, classifies, routes, calls model, validates and enforces verdicts)
- **`scripts/model_call.sh`** — Shared model-call layer: request building, streaming/SSE handling, retries, error-body preservation for both API formats
- **`scripts/default_system_prompt.txt`** — Bundled system prompt used when no override is provided

### Python package (`pr_reviewer/`)

- **`classifier.py`** — Deterministic PR classification: `pr_kind`, `risk_flags`, `must_check` checklist (no model calls)
- **`completeness.py`** — Required-check completeness validation: keyword-matches `review_markdown` against `must_check` items
- **`enforcement.py`** — Verdict policy (`model` / `findings_severity_gated`), findings normalization, evidence/tool enforcement; records `verdict_source`
- **`escalation.py`** — Post-hoc escalation triggers for fast reviews (request_changes, low confidence, incomplete checks, blockers, dirty baseline)
- **`carry_forward.py`** — Carried-forward open findings for incremental reviews; surviving blockers force `request_changes` (`verdict_source: carry_forward`)
- **`metadata.py`** — Managed metadata marker (fingerprint, scope, open findings) embedded in published comments
- **`github_context.py`** — PR metadata/linked-issue context helpers
- **`response_parser.py`** — Tolerant model-output parsing (JSON in fences/prose, verdict + findings extraction)
- **`sse_reassembler.py`** — Reassembles streamed SSE responses into complete bodies (including streamed tool-call deltas; `function.arguments` is the accumulated JSON string, OpenAI non-streaming shape, per #233)
- **`conversation.py`** — Multi-turn conversation/request builder for native tool calling (#202, 2/7 of #197 Option B): append-only neutral state, OpenAI/Anthropic wire rendering, per-API tool-schema catalogue, `truncate_oldest_tool_results` budget helper, `verdict_turn` mode that drops `tools` and switches to the strict JSON `response_format`

### Publishing and output hygiene

- **`scripts/publish_helpers.sh`** — Shared publish functions: sanitize, metadata marker build, native review cleanup, finding-thread resolution
- **`scripts/sanitize_review_markdown.py`** — Neutralizes upstream GitHub auto-links (PR/issue/commit URLs, `owner/repo#123`, bare `#123`) in review output
- **`scripts/strip_metadata_markers.py`** — Strips reserved `<!-- ai-pr-review-*:... -->` markers from model output before publishing
- **`scripts/redact.py`** — Shared secret-redaction pipeline applied to tool and evidence-provider output
- **`scripts/build_review_comments.py`** — Builds line-anchored inline review comments from structured findings, validated against the PR diff

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [misospace/pr-reviewer-action](https://github.com/misospace/pr-reviewer-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
