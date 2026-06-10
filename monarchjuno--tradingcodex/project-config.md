---
trigger: always_on
description: This file is the durable working guide for the root source repository at `/Users/junhoyoon/codex_pjt/tradingcodex`. It is not the generated workspace guide; generated `AGENTS.md` content comes from `workspace_templates/modules/codex-base/files/AGENTS.md`.
---

# TradingCodex Source Repository Guide

This file is the durable working guide for the root source repository at `/Users/junhoyoon/codex_pjt/tradingcodex`. It is not the generated workspace guide; generated `AGENTS.md` content comes from `workspace_templates/modules/codex-base/files/AGENTS.md`.

## Current Direction

- TradingCodex is now a Python/Django-native local-first trading harness, not a Node package workspace.
- Harness is the top-level product model. Guardrails and Improvement sit under it: guidance guardrails, enforcement guardrails, and information barriers reduce or block risk; Improvement covers workflow quality, research memory, skill proposals, postmortems, and validation feedback.
- The service target follows the product docs: latest LTS-oriented Python/Django stack, currently Django 5.2.x and the current supported Python line defined in `pyproject.toml` and `docs/tradingcodex-prd.md`.
- Django is the durable service plane. The product web app at `/` is the visual harness dashboard. Django Admin is the advanced harness operations console. Django Ninja is the typed local/staff control API. Django-hosted MCP is the agent/tool execution boundary.
- Public equity remains the deepest first investing sleeve, but the product must not be limited to public equities. Preserve extensibility for ETF/index, public crypto, macro/rates/FX/commodities, options, credit-signal, and cross-asset workflows.
- Runtime state and research memory are central-DB-first. Codex projects are clients/provenance; markdown and JSON files are Codex-readable exports, caches, or artifacts unless the docs explicitly say otherwise.
- OpenAI SDK embedding, semantic-search, AI-review, and SDK-backed agent orchestration surfaces are intentionally not part of the core harness right now. Do not add them back unless the user explicitly reopens that decision.

## Documentation Source Of Truth

- Treat `docs/` as the source of truth for durable product direction, core concepts, safety rules, role boundaries, and execution policy.
- Read [docs/README.md](./docs/README.md) before changing product rules, generated workspace behavior, guardrails, subagent roles, artifact contracts, module capabilities, MCP tools, or Admin operations.
- Use [docs/tradingcodex-prd.md](./docs/tradingcodex-prd.md) as the source of truth for product definition, goals, non-goals, architecture, initial scope, and test expectations.
- Use [docs/core-concepts-and-rules.md](./docs/core-concepts-and-rules.md) as the source of truth for role responsibilities, guardrails, information barriers, execution lifecycle, artifact paths, and MCP role boundaries.
- Use [docs/harness.md](./docs/harness.md), [docs/guardrails.md](./docs/guardrails.md), and [docs/improvement-loop.md](./docs/improvement-loop.md) when changing the top-level harness taxonomy, safety taxonomy, or quality/improvement loops.
- Update the relevant `docs/` files in the same change whenever product direction, rules, permissions, workflows, templates, policy behavior, or MCP/Admin behavior changes.
- If implementation and docs disagree, resolve the mismatch in the same change. Do not let hidden product rules live only in code, tests, templates, or prompts.

## Source Layout

- Keep Python CLI code under `tradingcodex_cli/`.
- Keep CLI command implementations under `tradingcodex_cli/commands/`; keep `tradingcodex_cli/workspace.py` as the public compatibility facade for generated wrappers and imports.
- Keep Django project code under `tradingcodex_service/`.
- Keep shared Django service implementation under `tradingcodex_service/application/`; keep `tradingcodex_service/domain.py` as the public compatibility facade.
- Keep modular Django apps under `apps/`.
- Keep generated workspace templates under `workspace_templates/modules/*`.
- Keep TradingCodex product documentation under `docs/`.
- Keep tests under `tests/`.
- Do not reintroduce Node runtime surfaces such as `package.json`, `packages/*`, old `templates/*`, or Node MCP scripts unless the user explicitly reverses the migration direction.

## Service Layer Rules

- Admin, Django Ninja, MCP, generated hooks, and CLI must call shared application service functions for durable behavior.
- Do not duplicate policy, order, approval, execution, portfolio, research, audit, or harness logic per interface.
- Executable actions must flow through `principal -> capability -> policy -> schema -> approval/idempotency -> adapter -> audit`.
- Revalidate policy and approval immediately before adapter submission.
- Live broker adapters remain disabled and unimplemented in the initial core. Paper/stub adapters are the only executable adapters unless the docs and user request explicitly change that.
- Do not store broker API keys, tokens, or secrets in this repository.
- Do not add direct live broker paths outside the TradingCodex MCP/service-layer boundary.

## DB-First Runtime

- The default runtime DB is the central local SQLite ledger at `~/.tradingcodex/state/tradingcodex.sqlite3`, unless `TRADINGCODEX_HOME` or `TRADINGCODEX_DB_NAME` overrides it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monarchjuno/tradingcodex](https://github.com/monarchjuno/tradingcodex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
