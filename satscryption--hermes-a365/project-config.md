---
trigger: always_on
description: Use when integrating a Hermes agent into the Microsoft 365 ecosystem — agent-as-tenant-directory-identity (AI Teammate / agentic user) or Copilot Chat surfacing (Custom Engine Agent + Azure Bot Service). Distinct from Hermes' sibling Teams adapter (`plugins/platforms/teams/`) which covers classic Bot-Framework Teams chat. Wraps the GA `Microsoft.Agents.A365.DevTools.Cli` verbs, ships the BF activity bridge that backs the `agent365` gateway platform, and emits both AI Teammate and Custom Engine A
---


# Hermes A365

## Overview

Hermes-A365 is the **M365 Copilot ecosystem path** for Hermes agents.
It covers the surfaces a classic Bot Framework Teams bot structurally
cannot reach:

- **Path A — AI Teammate (M365 agentic user):** the Hermes agent
  appears as a first-class agentic identity in your M365 tenant
  directory, in the "Built for your org" picker, in M365 People
  search, and in agentic-user audit trails. Teams 1:1 chat with
  M365-native identity. No Azure subscription required.
- **Path B — Custom Engine Agent (Azure Bot Service + 1.21 manifest):**
  the Hermes agent appears in M365 Copilot Chat's agents picker, in
  Copilot side-panels inside Word / Excel / PowerPoint / Outlook,
  and reaches the Copilot fabric's invoke surfaces (Microsoft Search,
  Outlook compose-action). Requires an Azure subscription so the
  blueprint Entra app can be registered as an Azure Bot Service
  resource.

Both paths share the same blueprint Entra app + service principal +
bot endpoint, and operators with both prerequisites can run both
surfaces from one Hermes-A365 install.

**Distinct from Hermes' sibling Teams adapter** at
`plugins/platforms/teams/adapter.py` (shipped v2026.4.30 + in-flight
work in `hermes-agent#10037` and `#13767`). That adapter is the right
tool for generic Teams chat bots — DM, channels, group chats,
threading, file attachments — using classic Bot Framework with an
Azure App Registration + client secret / certificate / Managed
Identity. It gives no M365 directory identity and no Copilot Chat
surfacing.

Microsoft Agent 365 (A365, GA 2026-05-01) is the governance / identity
/ observability control plane Hermes-A365 plugs into: Entra-backed
agentic identity, tenant licensing, agent blueprints, MCP-mediated
Microsoft 365 data access ("Work IQ"), OpenTelemetry, and the
channel adapters for Teams / Outlook / M365 Copilot.

The wrapper is built directly on the GA `Microsoft.Agents.A365.DevTools.Cli`
verbs documented in [`references/a365-cli-reference.md`](references/a365-cli-reference.md).
The skill composes them into idempotent plan/apply flows, fills the
gaps the CLI doesn't cover (license decision, admin consent grant,
runtime `.env` generation, Custom Engine Agent manifest transform),
and ships a Bot Framework activity bridge + Hermes gateway platform
plugin (`agent365`) that round-trips messages between A365 and the
Hermes agent loop.

## When to Use

Use when the user wants any of:

- Hermes registered as a first-class M365 **agentic identity** —
  appears in the tenant directory, agentic-user audit, "Built for
  your org" picker. Path A.
- Hermes available in **M365 Copilot Chat** (agents picker /
  `@`-mention / side-panels in Word / Excel / PowerPoint / Outlook).
  Path B. Requires Azure subscription.
- A Bot Framework activity bridge backed by **A365 governance**
  (Entra-backed identity, MCP-mediated Microsoft 365 data, OTLP
  audit trails).
- Migrating an OpenClaw-on-A365 deployment to Hermes (the
  blueprint stays; only the runtime endpoint changes).

Don't use when:

- The goal is a **generic Teams chat bot** with no M365 directory
  identity or Copilot surfacing — use Hermes' sibling Teams adapter
  (`plugins/platforms/teams/`, shipped v2026.4.30). It handles
  DM / channels / group / threading / file attachments via classic
  Bot Framework without A365 prerequisites.
- Generic Microsoft Graph access is the goal — use a Graph-only skill.
- Deploying a Bot Framework bot **outside** any M365 / A365
  governance surface — pick a classic BF skill.
- Setting up OpenAI Agents SDK or another framework end-to-end — A365
  governs the runtime; pick the appropriate framework skill for the
  agent itself.

## Prerequisites

- A Microsoft 365 tenant where the operator has **Global Administrator**
  or **Agent Administrator** role and is enrolled in Microsoft's
  Frontier Preview Program.
- The A365 CLI on PATH: `Microsoft.Agents.A365.DevTools.Cli` (.NET tool,
  ships as `a365`). Only the .NET tool ships at
  GA — the npm `atk` variant referenced in pre-GA documentation never
  landed. No CLI build is currently live-verified clean for
  Microsoft#408: the secret-persistence regression still reproduced on
  1.1.181 during the 2026-05-15 R9 walk. Doctor therefore warns for all
  versions and live setup should keep `--auto-recover-secret` enabled
  until a fixed build is walked clean.
- `az` CLI ≥ 2.55.0, signed into the target tenant. Many `a365`
  subcommands shell out to `az` for Entra reads.
- **PowerShell 7+ (`pwsh`) on PATH.** The CLI invokes `pwsh` for some
  setup steps; missing `pwsh` causes `a365 setup requirements` to fail.
- A custom Entra client app (Microsoft's convention: display name
  `Agent 365 CLI`) registered in the tenant. The CLI uses it as the
  device-code/auth-code client. Doctor verifies discoverability via the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [satscryption/Hermes-A365](https://github.com/satscryption/Hermes-A365) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
