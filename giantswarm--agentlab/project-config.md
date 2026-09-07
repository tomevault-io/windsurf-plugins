---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A local lab for the **Giant Swarm agent platform** in one Go binary
(`agentlab`): muster + the Kubernetes MCP server (and optionally Giant Swarm
Backstage) on a throwaway kind cluster, so the platform can be tested and
demoed end to end. The platform needs an identity provider, so the lab bundles
its own Dex — throwaway users that exist nowhere else, RBAC driven by the
`groups` claim, the apiserver, muster and Backstage all trusting the same
issuer. All configuration lives in `agentlab.yaml` (created by `agentlab
configure`); every manifest renders from templates embedded in the binary into
`state/` (gitignored). There is no YAML to hand-edit and no shell scripts.

## Always use the lab and its MCP

Testing the agent platform is this repo's purpose, and `.mcp.json` registers
the **`musterkind`** MCP server (`https://muster.127.0.0.1.nip.io/mcp`) —
muster running *inside* the lab cluster, reached through the agentgateway
edge. Interacting with the cluster through it is the point: it exercises the
whole Claude Code → agentgateway → muster → mcp-kubernetes → apiserver chain,
with Dex doing the logins.

- The edge serves a lab-CA certificate. Either the CA is in the system trust
  store (one-time `./agentlab trust`; then launch Claude Code with
  `NODE_USE_SYSTEM_CA=1`, Node >= 22.15) or launch with
  `NODE_EXTRA_CA_CERTS=<repo>/certs/ca.crt` — without one of the two the
  connection fails on TLS. (Fallback for a shell without either: the direct,
  edge-bypassing `http://localhost:8090/mcp`.) Never install trust silently:
  `agentlab trust` is the user's explicit, sudo-gated step.
- If `musterkind` is unreachable or unauthenticated, the lab is down — bring
  it up instead of switching tools: `./agentlab configure --defaults` (once;
  the platform and Backstage are enabled by default), then `./agentlab up`,
  then authenticate via `/mcp` (Dex browser login; users and passwords are in
  `agentlab.yaml`, default `admin@lab.local` / `password`).
- The Kubernetes tools come from the umbrella's bundled `mcp-kubernetes`
  MCPServer and use muster's per-server prefixing: `x_mcp-kubernetes_<tool>`
  (e.g. `x_mcp-kubernetes_list`), no `management_cluster` argument.
- muster's OAuth *client* role is on (`oauth.mcpClient`), and the lab ships
  one `Auth Required` downstream to sign in to: the MCPServer
  `lab-oauth-fixture`, which points muster at its own protected `/mcp`. It
  exists for the per-server sign-in path (`core_auth_login`, the portal's
  Sign in button); `platform-test` and `backstage-test` assert the challenge.
  Not a real integration — never "fix" its Auth Required state, and after a
  muster pod roll it reads `Failed` for about a minute by design.
- With `platform.observability: true` (the default), a minimal Prometheus
  (the GS kube-prometheus-stack constituent of the observability bundle, with
  the server re-enabled) and mcp-prometheus install too; the tools surface as
  `x_mcp-prometheus_<tool>` (e.g. `x_mcp-prometheus_execute_query`) — the way
  to answer CPU/memory questions about the lab. Chart pins are Go consts in
  `internal/lab/observability.go`; the bundle itself is deliberately NOT
  installed (MC-shaped: Flux HelmReleases, Alloy -> Mimir, no local PromQL).
  Backstage's Clusters/Deployments metrics work too: the lab serves the
  Mimir-shaped endpoint (`observability.<domain>/prometheus` on the edge →
  the lab Prometheus) and overrides the umbrella's `mimirEnabled: false` in
  its app-config overlay (backstage-catalog.yaml.tmpl).
- The agents runtime (kagent) installs with the platform by default but is
  optional (`platform.agents` in `agentlab.yaml`) — on real clusters agent
  delivery runs through Flux/GitOps, which the lab does not run as a GitOps
  loop. Backstage's agent create flow (`/agents/new`) deploys by kube:applying
  Flux CRs through the scaffolder Template `agent-deployment` (embedded into
  the lab catalog from `templates/static/`), so `agentlab backstage` also
  installs Flux's source+helm controllers as the delivery engine when agents
  are enabled — nothing watches git. Its default
  ModelConfig and Backstage's ai-chat both use `aiModel` from `agentlab.yaml`
  (Anthropic only); the API key comes from `$ANTHROPIC_API_KEY` on the host at
  deploy time and lives only in the Secrets `kagent/kagent-anthropic` and
  `backstage/backstage-anthropic` — never in `agentlab.yaml` or `state/`.
  Never inline a real key in config, templates, or rendered values.
  `platform.extraModels` adds further ModelConfigs (self-hosted
  OpenAI-compatible endpoints, OpenRouter, Gemini, Ollama) with the same
  env-var -> Secret key handling; entries removed from the config are pruned
  on the next run (see README "Extra model configs").
- `agentlab configure` **discovers this machine on every run** (fresh or
  existing `agentlab.yaml`): the tools `up` shells out to, whether this
  configuration's kind node exists and which host ports it publishes (never
  conflicts; while no node exists, occupied ports move to free ones), the
  host model servers — an Ollama on 11434, a Lemonade Server on 13305 —

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giantswarm/agentlab](https://github.com/giantswarm/agentlab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
