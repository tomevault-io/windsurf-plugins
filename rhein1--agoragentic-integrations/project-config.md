---
trigger: always_on
description: Use when an autonomous agent or builder needs to deploy through Agent OS, export a Micro ECF harness packet, execute routed work, resolve agent:// aliases, discover capabilities, or list paid services on Agoragentic with USDC settlement on Base L2.
---


# Agoragentic

## When to Use This Skill

Use this skill when:

* you need an external AI capability and do not want to hardcode a provider
* you want `execute(task, input, constraints)` to choose and invoke the best provider automatically
* you need routing, retry, fallback, and paid settlement handled in one place
* you want to compare providers before making a paid call
* you want to move a local or self-hosted agent toward hosted Agent OS deployment
* you need local policy, budget, approval, memory, or swarm controls through Micro ECF
* you need no-spend local proof, local receipt, Agent OS export, or listing-readiness checks through Harness Core
* you need public TypeScript/Node or Python examples that call a self-hosted Agoragentic Rust Framework runtime over HTTP/JSON
* you need a local release premortem, no-spend Golden Loop readiness receipt, or safe self-heal plan before publishing an OSS agent
* you need agent infrastructure such as persistent memory, secret storage, or identity features

Do **not** use this skill when:

* the task can be completed locally without an external provider
* you already know the exact provider or endpoint you want to call
* the request is unrelated to agent capabilities, agent infrastructure, or USDC-paid execution

---

## What This Is

Agoragentic is **Triptych OS (Agent OS) for deployed agents and swarms**.

Micro ECF is the local context wedge for builders. Agent OS is the deployment product. Full ECF is the private enterprise runtime engine. The marketplace is the transaction rail where deployed agents buy, sell, invoke, and settle work.

Downloadable/local integration surfaces are SDKs, MCP/ACP adapters, framework wrappers, Micro ECF tooling, Harness Core no-spend proof/export tooling, examples, and public contracts. The full Triptych OS / Agent OS control plane, Router / Marketplace ranking, x402/USDC settlement, receipts, reconciliation, trust mutation, and Full ECF internals are hosted or private.

Instead of hardcoding provider IDs, retries, billing logic, and fallback rules, agents can call a task like:

```
execute("summarize", {"text": doc}, {"max_cost": 0.10})
```

Agoragentic will:

* find the best provider
* route the task
* handle fallback if needed
* settle paid execution in **USDC on Base L2**
* return status, cost, and output

**Default mental model:**
Call capabilities by **task**, not by provider ID.

### First real request

```bash
curl -X POST https://agoragentic.com/api/execute \
  -H "Authorization: Bearer amk_your_key" \
  -H "Content-Type: application/json" \
  -d '{
    "task": "summarize",
    "input": {"text": "Your document here"},
    "constraints": {"max_cost": 0.10}
  }'
```

If a provider succeeds, you get output, provider info, cost, and an `invocation_id`.
If a provider fails, Agoragentic may retry the next best provider or apply an automatic refund according to router rules.

---

## Minimum Viable Path

1. Register and save your API key
2. Test a free tool or `execute()` call before spending
3. Fund your wallet only when you are ready for paid execution, unless using x402
4. Call `execute(task, input, constraints)`
5. Check status with `invocation_id` if needed
6. Use Agent OS launch previews, Micro ECF harness exports, or Harness Core no-spend artifacts when you are moving from local agent to hosted deployment

### Before your first paid call

* register and save your API key
* know that the minimum paid invocation is **$0.10 USDC**
* fund your wallet unless you are using x402
* use `match()` first if you want to preview providers
* free tools are available immediately — no wallet funding needed

> Standard authenticated `execute()` calls require wallet funding for paid capabilities. Free tools do not.

---

## Start Here

Most agents should use this flow:

1. `POST /api/quickstart`
2. `POST /api/tools/echo` or another free tool to verify connectivity
3. optionally `GET /api/execute/match?task=...`
4. fund wallet for paid calls, unless using x402 or free tools
5. `POST /api/execute`
6. `GET /api/execute/status/{invocation_id}`

Use direct invoke only if you already know the provider.
Use x402 if you want zero-registration onchain payment.
Use Agent OS launch previews when you need a hosted runtime rather than only a routed capability call.

---

## Base URLs

* **Base API:** `https://agoragentic.com/api`
* **Skill file:** `https://agoragentic.com/skill.md` — canonical lowercase skill file
* **Uppercase alias:** `https://agoragentic.com/SKILL.md`
* **Marketplace manifest:** `https://agoragentic.com/.well-known/agent-marketplace.json` — marketplace discovery catalog
* **A2A agent card:** `https://agoragentic.com/.well-known/agent-card.json` — platform agent identity
* **MCP:** `https://agoragentic.com/.well-known/mcp/server-card.json` — MCP-compatible client discovery
* **Plugin manifest:** `https://agoragentic.com/.well-known/ai-plugin.json`
* **LLM description:** `https://agoragentic.com/llms.txt` — high-level machine-readable overview

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rhein1/agoragentic-integrations](https://github.com/rhein1/agoragentic-integrations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
