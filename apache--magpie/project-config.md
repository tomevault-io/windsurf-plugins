---
trigger: always_on
description: <!-- SPDX-License-Identifier: Apache-2.0
---

<!-- SPDX-License-Identifier: Apache-2.0
     https://www.apache.org/licenses/LICENSE-2.0 -->

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Gemini CLI harness](#gemini-cli-harness)
  - [Harness contract](#harness-contract)
  - [Invoke a Magpie skill](#invoke-a-magpie-skill)
  - [Install](#install)
    - [Authentication with the clean-environment wrapper](#authentication-with-the-clean-environment-wrapper)
  - [Security model](#security-model)
    - [Tool-sandboxing boundaries](#tool-sandboxing-boundaries)
    - [What asks and what denies](#what-asks-and-what-denies)
    - [Policy loading and precedence](#policy-loading-and-precedence)
    - [Deterministic guard rules](#deterministic-guard-rules)
  - [Reuse framework MCP servers](#reuse-framework-mcp-servers)
  - [Spec-loop runner](#spec-loop-runner)
  - [Verify](#verify)
  - [Update](#update)
  - [Doctor](#doctor)
  - [setup-isolated lifecycle](#setup-isolated-lifecycle)
  - [Known limitations](#known-limitations)
  - [Developer checks](#developer-checks)
  - [Upstream references](#upstream-references)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

<!-- SPDX-License-Identifier: Apache-2.0
     https://www.apache.org/licenses/LICENSE-2.0 -->

# Gemini CLI harness

**Capability:** capability:platform

**Harness:** Gemini CLI

Gemini CLI is an agent harness that runs Magpie's shared skills with repository instructions, an action guard, tool sandboxing, and per-action approval policies.
The adapter is **experimental**: its Linux sandbox does not provide the Claude Code reference setup's home-directory read isolation or domain allowlist.
The integration follows the harness contract in [add-a-harness](add-a-harness.md)
and [RFC-AI-0004](../rfcs/RFC-AI-0004.md).

## Harness contract

| Magpie requirement | Gemini CLI implementation |
|---|---|
| Skill discovery | Gemini reads the canonical `.agents/skills/magpie-*/SKILL.md` links. The existing `universal` row in `skills/setup/agents.md` covers this path. |
| Repository instructions | The framework's `GEMINI.md` imports its `AGENTS.md`. |
| Tool bridges | Skills invoke the existing `tools/*` CLI adapters through shell calls, subject to the approval policy and sandbox grants below. Each adapter declares its own prerequisites. |
| Deterministic guard | Project `.gemini/settings.json` registers Magpie's `agent_guard/__init__.py --gemini` adapter for Gemini's `BeforeTool` shell events. The adapter calls Magpie's `dispatch()` function to check the command. Adopters register the hook through the install lifecycle below. |
| Spec-loop | The `gemini` profile forwards the prompt, model, and output format. |
| Credential isolation | `agent-iso gemini` launches the CLI through the generic clean-environment wrapper, which filters inherited environment variables. |
| Filesystem and network | `security.toolSandboxing: true` enables Gemini's tool sandboxing. The shipped profile adds no extra writable directories or network grant. See [Tool-sandboxing boundaries](#tool-sandboxing-boundaries) for the difference between native file tools and shell access. |
| Tool approval | Explicit `policyPaths` loads `policies/magpie.toml`: scoped reads are allowed, other shell calls and native edits require confirmation, and selected commands and credential paths are denied. MCP calls require confirmation. |
| MCP servers | Register the same server commands in Gemini's user settings; see [Reuse framework MCP servers](#reuse-framework-mcp-servers). No servers or credentials are installed by this profile. |

## Invoke a Magpie skill

Install the [Gemini extension](../setup/marketplace-install.md#google-gemini-cli) through [setup](../../skills/setup/SKILL.md), or use the canonical `.agents/skills/` links with a pinned snapshot.
Both methods expose the same skill sources.
The framework's [`GEMINI.md`](../../GEMINI.md) imports `AGENTS.md`; adopters retain their own project instructions alongside that context.

From the adopter repository root:

1. Complete [installation](#install) and launch Gemini through the clean-environment wrapper.
2. Use `/memory show` to inspect repository instructions and `/skills list` to check discovery.
3. Ask `Use the magpie-list-skills skill.` and review the activation request.
4. Approve the skill's catalogue script when prompted; interpreter commands require tool approval even for a read-only workflow.

Skill activation consent and tool approval are separate decisions.
Skills use the existing `tools/*` adapters, whose READMEs declare their prerequisites.

## Install

Use Gemini CLI **0.59.0 or later**.
The validation baseline is 0.59.0 on Linux; later versions require [verification](#verify) before use.
The tested Linux backend requires `bwrap` (bubblewrap), usable user namespaces, and ordinary shell utilities.
Use the framework's [sandbox primitive versions](../../tools/agent-isolation/pinned-versions.toml) when installing bubblewrap.
Python 3.11+ is required for profile linting and the existing action guard.

For guided setup, ask Gemini: `Use the magpie-setup-isolated-setup-install skill.`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/magpie](https://github.com/apache/magpie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
