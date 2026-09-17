---
trigger: always_on
description: Orientation for AI agents and human contributors working inside the
---

# AGENTS.md

Orientation for AI agents and human contributors working inside the
**Nerya** repository.

Read this file *first*. It answers the "where does X live, and how do I
run things" questions so you don't grep through the whole tree on
every session.

---

## 1. What Nerya is

Nerya is a **skill-first, trading-native, self-evolving autonomous
agent runtime**. It runs as:

- `nerya.api.local_server` — FastAPI-like HTTP server on `:18317`
(launched by `nerya service start`).
- `nerya.agent.kernel.AgentKernel` — the main decision loop.
- `dashboard/` — Next.js 14 control panel on `:3001`, talking to the
API server through `/api/proxy/`*.

The agent runtime never calls exchanges or LLMs directly. **Every
model-driven external capability call is mediated by a Skill and its
approved scripts**. Operator-only control-plane maintenance (for example,
an `admin:ops` workspace backup/restore) may call its dedicated adapter
directly when it is not exposed as an agent tool and validates remote
content before applying it. A Skill is the model/operator-facing playbook
declared by `SKILL.md`. Executable logic belongs under `scripts/`, not in
`actions.py` or YAML manifests. Do not create new `skill.yml`, `skill.yaml`,
`manifest.yml`, `manifest.yaml`, or `actions.py` files to define skills.
Legacy YAML/action files in existing skill directories are migration
artifacts only and must be removed or converted when touching that
capability.

## 2. Absolute rules (read before editing code)

1. **Never log or return plaintext secrets.** Use
  `nerya.security.secrets.SecretVault`; values resolved via
   `vault://<ref>` stay in memory for a single call.
2. **Live trading** is off unless `runtime.live_trading_enabled: true`
  in `nerya.yml` *and* the Approval Gate signs off. Do not bypass
   `nerya.trading.risk_gate` or `approval_gate`.
3. **All agent-authored changes are proposals.** Writing a strategy,
  a trigger, a script, or a skill goes through
   `nerya.evolution.PatchProposal`. Never mutate `workspace/` state
   directly from an action. Operator-initiated workspace restore is not an
   agent-authored action: it may apply a curated snapshot directly only
   behind `admin:ops`, with secret/runtime paths excluded, manifest hashes
   verified, and conflicts rejected unless the operator explicitly forces
   the restore.
4. **Don't introduce native CEX connectors.** All CEX venues are
  handled by `nerya.connectors.ccxt_adapter.CcxtConnector`. If you
   want a new venue, add an `ExchangeProviderSpec` (see
   `nerya/connectors/provider_spec.py`).
5. **Dashboard state is not authoritative.** The browser's
  `localStorage` caches UI prefs only; every action hits the API
   proxy, which talks to the Python runtime.
6. **Research knowledge is lazy-loaded skill content.** Professional
  research frameworks, data-source decision trees, report templates,
   factor/quant methodology, and market-analysis checklists belong in
   `SKILL.md` or files under `references/`. Keep always-on prompts,
   default subagent prompts, route manifests, and team templates small:
   they should name the role, preferred skills, and output contract only.
   Load the relevant skill (`Skill`, `skill_view`, or script docs) only
   when a task actually needs that research capability.

## 3. Repo layout (top-level)

```
Nerya/
├── nerya/                 # Python runtime
│   ├── agent/             # AgentKernel, Planner, ContextBuilder, Memory, Reflection
│   ├── api/               # local_server + HTTP routes (/chat, /strategies, /integrations, /wallet, /exchanges)
│   ├── cli/               # `nerya ...` CLI (cli/app.py)
│   ├── connectors/        # ccxt_adapter, polymarket, evm/solana/bsc native, registry, provider_spec
│   ├── core/              # config, paths, errors, yaml_io, devmode
│   ├── data/              # news / social / tvl / funding / whale-event feeds (real APIs)
│   ├── evolution/         # reflection, strategy mutation, patch proposals, promotion, rollback
│   ├── install/           # service / host-process helpers
│   ├── llm/               # ModelRouter, adapters/, credential_pool, compression, session, budget
│   ├── mcp/               # FastMCP server exposing skills as MCP tools
│   ├── messaging/         # Telegram / Discord / webhook transports + outbound pipeline
│   ├── sdk/               # trading + trigger SDKs (user-authored scripts import these)
│   ├── security/          # SecretVault, prompt firewall, redaction, structured output validation
│   ├── skills/            # skill kernel + builtin/<name>_skill/*
│   ├── subagents/         # spawn/run sub-agents per strategy
│   ├── trading/           # risk_gate, approval_gate, accounts, ledger, reconciliation
│   ├── triggers/          # cron + trigger router, schedules.yml parser
│   └── wallet/            # WalletProvider abstraction + self_custody / okx_os / bitget / binance_agentic / coinbase
├── dashboard/             # Next.js 14 control panel (app router + Tailwind)
├── scripts/               # CLI helpers (install / doctor / e2e)
├── tests/                 # pytest suite — 500+ tests
├── workspace_template/    # reference layout (the actual layout is built at runtime by `workspace/layout.py::required_dirs`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NeryaAI/Nerya](https://github.com/NeryaAI/Nerya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
