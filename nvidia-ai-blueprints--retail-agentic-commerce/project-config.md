---
trigger: always_on
description: Fast-start guide for coding agents working in Retail-Agentic-Commerce.
---


# AGENTS.md

This document is the fast-start guide for GPT Codex and other coding agents.

Deep context and diagnostic examples live in `docs/agent-playbook.md`.

## Session Workflow (Mandatory)

Use this sequence for every task:
1. Read this file once.
2. Identify task type and read required specs/docs before coding.
3. Read mandatory skill files in `.cursor/skills/`.
4. Implement minimal, spec-aligned changes.
5. Run required verification and report evidence.

## Documentation-First Development (CRITICAL)

**ALWAYS read and follow documentation BEFORE implementing features, modifying code, or reviewing behavior.**

This project implements ACP/UCP flows with strict architecture contracts. Do not infer behavior from existing code alone.

### Mandatory Documentation Review

| Task Type | Required Reading |
|-----------|------------------|
| ACP endpoints, checkout, delegation, webhooks | `docs/specs/acp-spec.md` |
| UCP flows, discovery, A2A transport | `docs/specs/ucp-spec.md` |
| Apps SDK MCP tools and widget behavior | `docs/specs/apps-sdk-spec.md` |
| NAT agent config/orchestration | `docs/NEMO_AGENT_TOOLKIT_DOCUMENTATION.md` |
| Agent integration details | `src/agents/README.md` |
| System architecture and data flow | `docs/architecture.md` |
| Feature-specific behavior | `docs/features/index.md` and `docs/features/feature-XX-*.md` |
| Docker deployment, common operations | `deploy/docker-deployment.md` |
| Local development, `install.sh`/`stop.sh` | `deploy/local-development.md` |

### Specification Source of Truth

#### ACP Specification (`docs/specs/acp-spec.md`)
Use when working on checkout sessions, payment delegation, and webhook contracts.

Validate:
- Data types: `CheckoutSession`, `PaymentDelegation`, `LineItem`, `ShippingOption`
- Endpoints: `/checkout_sessions`, `/agentic_commerce/delegate_payment`, webhook routes
- Security: API key auth + signature requirements
- Flow ownership: who calls what, and in what order

#### UCP Specification (`docs/specs/ucp-spec.md`)
Use when working on discovery, capability negotiation, UCP status lifecycle, or A2A JSON-RPC.

Validate:
- Statuses: `incomplete`, `requires_escalation`, `ready_for_complete`, `complete_in_progress`, `completed`, `canceled`
- Headers: `UCP-Agent`, `API-Version`, `Idempotency-Key`
- A2A methods: `a2a.ucp.checkout.create|get|update|complete|cancel`
- Discovery contract: `/.well-known/ucp`

Note: Merchant Activity ACP/UCP tabs switch backend protocol behavior. Apps SDK mode remains ACP-only.

#### Apps SDK Specification (`docs/specs/apps-sdk-spec.md`)
Use when working on MCP tool schemas, widget lifecycle, cart/checkout tools, and recommendation/search behavior.

Validate:
- Tool input/output schema contracts
- Widget state and event handling
- Merchant API integration boundaries
- NAT integration behavior

#### NeMo Agent Toolkit Documentation (`docs/NEMO_AGENT_TOOLKIT_DOCUMENTATION.md`)
Use when working on NAT YAML configs, multi-agent orchestration, tool registration, and `nat run`/`nat serve` workflows.

### Documentation-First Checklist

Before coding:
- [ ] Read the relevant spec sections.
- [ ] Identify who should call the endpoint/function.
- [ ] Trace end-to-end data flow.
- [ ] Compare current code to documented architecture.
- [ ] Ask user if spec and code conflict.

### Architecture Verification Order

When debugging:
1. What does documentation say should happen?
2. Does implementation match docs?
3. If not, is documentation outdated or code wrong?
4. Only then investigate environment/configuration issues.

## Security & Privacy Guardrails (Mandatory)

When creating or editing docs (including `AGENTS.md` files):
- Never include absolute local filesystem paths (for example: `/Users/<name>/...`, `/home/<name>/...`, `C:\Users\<name>\...`).
- Never include usernames, home directories, machine-specific temp paths, or other host-identifying data.
- Prefer repo-relative paths only (for example: `src/merchant/main.py`).
- Treat any detected local path exposure as a blocking issue and fix before finishing.

Before finalizing documentation changes, run this check and ensure it returns no matches:

```bash
rg -n -e '/Users/[A-Za-z0-9._-]+/' -e '/home/[A-Za-z0-9._-]+/' -e '\\\\Users\\\\[A-Za-z0-9._-]+\\\\' AGENTS.md src docs
```

## Cursor Skills (Mandatory)

Before changing code, read:
- `.cursor/skills/features/SKILL.md` for Python backend standards
- `.cursor/skills/ui/SKILL.md` for frontend standards
- `.cursor/skills/pre-commit-analysis/SKILL.md` for pre-commit validation

### Setup Skill

Trigger: **"setup"** or **"install"** — launches the full Docker deployment with public NIM endpoints. See `.cursor/skills/setup/SKILL.md`.

## Project Overview

Agentic Commerce Protocol (ACP) reference implementation:
- Backend: Python 3.12+, FastAPI, SQLModel
- Frontend: Next.js 15+, React 19, Tailwind, Kaizen UI

Core docs:
- `docs/features.md` for feature status and acceptance criteria
- `docs/architecture.md` for system design

## Scoped AGENTS (Subfolders)

When working in these components, read the local scoped guide after this root file:
- `src/merchant/AGENTS.md`
- `src/apps_sdk/AGENTS.md`
- `src/agents/AGENTS.md`

## Quick Map (Where to Look First)

Backend:
- API routes (shared): `src/merchant/api/routes/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-AI-Blueprints/Retail-Agentic-Commerce](https://github.com/NVIDIA-AI-Blueprints/Retail-Agentic-Commerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
