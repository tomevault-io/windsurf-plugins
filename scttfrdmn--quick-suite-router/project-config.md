---
trigger: always_on
description: This is **quick-suite-router** — a CDK-deployable reference
---

# CLAUDE.md — Project Context for Claude Code

## Project Overview

This is **quick-suite-router** — a CDK-deployable reference
architecture that extends Amazon Quick Suite with multi-provider LLM
access through Bedrock AgentCore Gateway.

The core idea: universities have existing AI subscriptions (OpenAI site
licenses, Google Gemini via Workspace, Anthropic agreements). Instead of
asking them to abandon those investments for Quick Suite's built-in LLM,
this router lets them bring their existing subscriptions INTO Quick Suite
with full AWS governance (Bedrock Guardrails, CloudTrail, CloudWatch
metering) applied to every call regardless of provider. The existing AI
subscription becomes the on-ramp to AWS, not the competitor.

## Project Tracking

Work is tracked in GitHub — not in local files. Do not add TODO lists or task
tracking to CLAUDE.md or create TODO.md files.

- **Issues:** https://github.com/scttfrdmn/quick-suite-router/issues
- **Milestones:** https://github.com/scttfrdmn/quick-suite-router/milestones
- **Project board:** https://github.com/users/scttfrdmn/projects/44
- **Changelog:** CHANGELOG.md (keepachangelog format, semver 2.0)

To report a bug or propose a feature, open a GitHub Issue with the appropriate
label. All release planning happens via milestones.

## Current State — v0.12.0

### File Inventory (35 files)

**Code:**
- `app.py` — CDK entry point
- `cdk.json` — CDK config with context flags (enable_cache, cache_ttl_minutes, budget_caps_secret_arn, enable_vpc, vpc_id, cors_allowed_origin, rate_limit_per_minute, rate_limit_per_day, enable_content_logging)
- `requirements.txt` — CDK dependencies
- `requirements-dev.txt` — dev/test dependencies
- `stacks/__init__.py` — empty
- `stacks/model_router_stack.py` — main CDK stack (Cognito, Secrets Manager,
  Lambdas, API Gateway, Bedrock Guardrail, DynamoDB cache + spend ledger, CloudWatch dashboard,
  optional VPC with private endpoints, per-user rate limiting usage plan, content audit logging)
- `stacks/multi_region_stack.py` — multi-region deployment support
- `lambdas/router/handler.py` — task classification, provider selection
  (capability + context window matching), cache check, fallback logic, spend
  record write, budget cap enforcement, PHI routing filter, dry-run mode,
  Cognito claims extraction, CORS origin control, content audit logging
  (SHA-256 hashes)
- `lambdas/common/python/provider_interface.py` — shared governance utilities
  (guardrails, CloudWatch metrics, DynamoDB cache helpers, spend ledger write,
  cost_usd computation, spend department query, token estimation)
- `lambdas/authorizer/handler.py` — Lambda authorizer for per-user rate
  limiting; decodes Cognito JWT, extracts `sub`, returns IAM Allow policy
  with `usageIdentifierKey = sub`
- `lambdas/query-spend/handler.py` — AgentCore Lambda target; queries qs-router-spend
  and aggregates cost by department/user/tool/date; enforces Cognito claims-based authorization
- `lambdas/providers/bedrock_provider.py` — Bedrock Converse + converse_stream API
- `lambdas/providers/anthropic_provider.py` — Anthropic Messages API (blocking + streaming)
- `lambdas/providers/openai_provider.py` — OpenAI Chat Completions API (blocking + streaming)
- `lambdas/providers/gemini_provider.py` — Google Generative AI API (blocking + streaming)
- `lambdas/guardrail-version-updater/handler.py` — internal Lambda; updates SSM param for guardrail version without redeploy
- `lambdas/key-rotation-checker/handler.py` — internal Lambda; weekly check that provider API key secrets have been rotated within `KEY_ROTATION_MAX_AGE_DAYS`; emits `KeyRotationOverdue` CloudWatch metric

**Config:**
- `config/routing_config.example.yaml` — provider preference lists per tool
- `config/routing_config.yaml` — active routing config with `model_capabilities` + `model_context_windows` registries
- `quicksuite/openapi_spec.json` — OpenAPI spec for Quick Suite import
- `quicksuite/agent-template.json` — AgentCore agent template

**Docs:**
- `docs/architecture.md` — full architecture overview with ASCII diagrams
- `docs/setup-bedrock.md` — Bedrock setup (IAM, model access)
- `docs/setup-anthropic.md` — Anthropic direct API setup
- `docs/setup-openai.md` — OpenAI setup with site license instructions
- `docs/setup-gemini.md` — Gemini setup with Google Workspace instructions
- `docs/quicksuite-integration.md` — AgentCore Gateway + Quick Suite MCP setup
- `docs/compliance.md` — HIPAA-ready deployment guide (VPC isolation, PHI tagging, CloudTrail, Guardrail hardening)

**GTM (go-to-market for BD/AM peers):**
- `gtm/bd-playbook.md` — discovery questions, demo script, competitive positioning
- `gtm/am-talking-points.md` — stakeholder messaging, value chain
- `gtm/objection-handling.md` — 10 objections with full responses

**Scaffolding:**
- `.gitignore`
- `LICENSE` (Apache 2.0)
- `README.md`

## Architecture

```
Quick Suite
    | MCP Actions Integration
    v
AgentCore Gateway (MCP server)
    | HTTPS
    v
API Gateway + Cognito (OAuth 2.0 client_credentials)
    |
    +-- Lambda Authorizer (per-user rate limiting via JWT sub)
    |
    v
Router Lambda
    |-- DynamoDB cache (optional, TTL-based)
    |-- Task classification (analyze/generate/research/summarize/code/extract)
    |-- Capability + context window matching

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scttfrdmn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
