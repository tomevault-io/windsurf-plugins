---
trigger: always_on
description: This repository is a documentation-only runbook for an authorized, disposable AI red-team lab. It contains no application implementation, model weights, cloud state, or secrets. Codex, Claude, and other coding agents must follow this file when asked to build, operate, recover, or destroy the lab.
---

# AGENTS.md — end-to-end deployment contract

This repository is a documentation-only runbook for an authorized, disposable AI red-team lab. It contains no application implementation, model weights, cloud state, or secrets. Codex, Claude, and other coding agents must follow this file when asked to build, operate, recover, or destroy the lab.

## Read first

Read these documents completely before changing cloud state:

1. `README.md`
2. `docs/architecture.md`
3. `docs/security.md`
4. `docs/gpu-rental.md`
5. `docs/model-choice.md`
6. `docs/capability-evaluation.md`
7. `docs/azure-goad.md`
8. `docs/runpod-vllm.md`
9. `docs/tailscale.md`
10. `docs/agent-stack.md`
11. `docs/parallel-agents.md`
12. `docs/tracing-replay.md`
13. `docs/validation.md`
14. `docs/operations.md`
15. `docs/lessons-learned.md`

Use current official provider/project documentation to verify commands, versions, SKU availability, pricing, and API behavior. The dates and prices in this repository are historical observations.

## Mission

When the user asks for an end-to-end build, produce a working system with:

- private GOAD in an explicitly selected Azure subscription/resource group;
- a B300-class RunPod or equivalent GPU serving the selected model with authenticated vLLM;
- an Azure Tailscale subnet router and a RunPod userspace Tailscale client;
- a constrained web agent, model gateway, egress proxy, headless jumpbox runner, and authenticated web UI;
- when enabled, a bounded Pi subagent coordinator that uses the same model gateway and preserves per-run isolation;
- live/archived JSONL trace capture and replay;
- completed positive and negative validation;
- a handoff that includes cost, URLs, secret locations (not secret values), backup state, and teardown instructions.

The deployment application is not present here. Either integrate an implementation the user supplies or create it in a separate private implementation repository/workspace following `docs/agent-stack.md` and `docs/tracing-replay.md`. Do not claim the documentation repository contains runnable app source.

## Non-negotiable safety rules

1. Work only in cloud accounts and targets explicitly authorized by the user.
2. GOAD Windows VMs must not receive public IPs.
3. Restrict jumpbox public SSH to the user's current `/32`; never use `0.0.0.0/0`.
4. The agent's only target scope is the configured jumpbox and GOAD subnet.
5. Never scan or interact with public internet targets, cloud metadata, unrelated private ranges, or the user's production systems.
6. Never commit or print API tokens, passwords, private keys, Tailscale state/auth keys, Terraform state, recovered credentials, tickets, hashes, model weights, JIT binaries, or traces containing secrets.
7. Use separate admin and runner SSH keys. The runner must be unprivileged and have no sudo.
8. Do not give the public web agent a shell unless an enforceable network/process sandbox exists and the user explicitly approves the expanded design.
9. Do not expose internal ports 1055, 1056, 13128, 18080, Docker, or an admin shell.
10. Never delete a resource based only on a remembered name. Inventory the active account, subscription, resource group, pod, and volume first.
11. Obtain explicit user approval immediately before creating billable resources and before permanent teardown unless the user's current request already clearly authorizes that exact action.
12. Preserve unrelated resources and pre-existing local changes.
13. Parallel workers must have explicit, non-overlapping assignments. Use one writer per working tree and never let concurrency expand network scope or secret access.

## Required user inputs

Discover safe values where possible. Ask only for choices that materially change the deployment:

- Azure subscription and acceptable region/budget;
- intended GOAD CIDR (default reference `10.42.42.0/24`) and confirmation it does not overlap existing routes;
- operator public SSH CIDR;
- RunPod account/budget and whether direct port 8888 must be public;
- preferred GPU provider if not RunPod;
- Tailscale tailnet authorization method and who can approve routes;
- single-user/team count and whether isolation must be per-user;
- location of an existing agent/web implementation, or authorization to create one privately;
- data-retention requirements for traces and reports;
- whether cloud resources should be stopped or destroyed after validation.

Never request a secret in chat when an interactive login, local secret file, environment variable, keychain, or provider CLI can supply it.

## Execution phases and gates

Maintain an explicit plan. Do not skip a gate.

### Phase 0 — local and account preflight

- Check Git status and preserve user changes.
- Verify `az account show`, RunPod identity/balance/spend limit, Tailscale access, and GitHub/source access.
- Inventory existing Azure groups/resources and RunPod pods/volumes.
- Check Azure quota and live B300 availability.
- Present estimated hourly cost, storage durability, region, and exact resources to be created.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antojoseph/adv26](https://github.com/antojoseph/adv26) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
