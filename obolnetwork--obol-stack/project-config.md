---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Obol Stack: framework for AI agents to run decentralised infrastructure locally. k3d cluster with a Hermes default AI agent, optional OpenClaw instances, blockchain networks, payment-gated inference (x402), and Cloudflare tunnels. CLI: `github.com/urfave/cli/v3`.

## Conventions

- **Commits**: Conventional commits — `feat:`, `fix:`, `docs:`, `test:`, `chore:`, `security:` with optional scope
- **Branches**: `feat/`, `fix/`, `research/`, `docs/`, `chore/` prefixes
- **GitHub branch policy**: never push `codex/`-prefixed branches to GitHub from this repository; rename to `feat/`, `fix/`, `research/`, `docs/`, `chore/`, or another non-codex branch name before pushing
- **Detailed architecture reference**: `@.claude/skills/obol-stack-dev/SKILL.md` (invoke with `/obol-stack-dev`)
- **Review scope**: Avoid broad, vague review/delegation boundaries. State the exact files, invariants, and expected evidence before reviewing or spawning agents. Prefer concrete checks such as "controller cannot access signer/Secrets", "agent write RBAC is namespace-scoped", and "flow uses real obol CLI path" over generic "review architecture".
- **Planning docs vs user docs**: Implementation plans, design notes, and feature retrospectives belong in `plans/` — they're useful to revisit when picking work back up later. Keep `docs/` for durable, user-facing documentation. Don't mix the two.
- **Release descriptions**: Use `.github/release-template.md` for future GitHub releases. The release workflow creates a draft with generated notes; rewrite the narrative body from the template, keep generated `What's Changed`, `New Contributors`, and `Full Changelog` sections at the bottom, and never include private keys, seed phrases, passwords, hostnames, personal paths, or raw bearer tokens.

## Build, Test, Run

```bash
just build                                    # Build with version info
go build -o .workspace/bin/obol ./cmd/obol    # Build to specific location
go build ./...                                # Check compilation
go test ./...                                 # All unit tests
go test -v -run 'TestName' ./internal/pkg/    # Single test

# Integration tests under internal/openclaw/ (legacy local matrix — requires running cluster + host Ollama)
export OBOL_DEVELOPMENT=true OBOL_CONFIG_DIR=$(pwd)/.workspace/config OBOL_BIN_DIR=$(pwd)/.workspace/bin OBOL_DATA_DIR=$(pwd)/.workspace/data
go build -o .workspace/bin/obol ./cmd/obol    # MUST rebuild after code changes
go test -tags integration -v -timeout 15m ./internal/openclaw/

# Validated paid-inference commerce loop (legacy local — requires host Ollama qwen3.5:9b)
# Does NOT replace release-gate flows 11/13/14, which require OBOL_LLM_ENDPOINT (vLLM / llama.cpp).
# If reusing a cluster from another worktree, point OBOL_CONFIG_DIR at that cluster's .workspace/config
go test -tags integration -v -run TestIntegration_Tunnel_SellDiscoverBuySidecar_QuotaAndBalance -timeout 30m ./internal/openclaw/

# Release-gate seller/buyer smoke (requires OBOL_LLM_ENDPOINT pointing at OpenAI-compatible vLLM/llama.cpp)
RELEASE_SMOKE_INCLUDE_OBOL=true RELEASE_SMOKE_INCLUDE_OBOL_FORK=true \
  OBOL_LLM_ENDPOINT=http://127.0.0.1:8000/v1 OBOL_LLM_MODEL=qwen36-deep \
  bash flows/release-smoke.sh

just up    # obol stack init + up
just down  # obol stack down + purge
just clean # Remove build artifacts

OBOL_DEVELOPMENT=true ./obolup.sh  # One-time dev setup, uses .workspace/, go run wrapper
```

Integration tests use `//go:build integration` and skip gracefully when prerequisites are missing.

## Architecture

**Two parts**: `obolup.sh` (bootstrap installer, pinned deps) + `obol` CLI (Go binary, all management).

**Design**: Deployment-centric (unique namespaces via petnames), local-first (k3d), XDG-compliant, two-stage templating (CLI flags → Go templates → Helmfile → K8s).

**Routing**: Traefik + Kubernetes Gateway API. GatewayClass `traefik`, Gateway `traefik-gateway` in `traefik` ns. Local-only routes (restricted to `hostnames: ["obol.stack"]`): `/` → frontend, `/rpc` → eRPC. Public routes (accessible via tunnel, no hostname restriction): `/services/<name>/*` → x402 ForwardAuth → upstream, `/.well-known/agent-registration.json` → ERC-8004 httpd, `/skill.md` → service catalog. Tunnel hostname gets a storefront landing page at `/`. NEVER remove hostname restrictions from frontend or eRPC HTTPRoutes — exposing the frontend/RPC to the public internet is a critical security flaw.

**Config**: `Config{ConfigDir, DataDir, BinDir}`. Precedence: `OBOL_CONFIG_DIR` > `XDG_CONFIG_HOME/obol` > `~/.config/obol`. `OBOL_DEVELOPMENT=true` → `.workspace/` dirs. All K8s tools auto-set `KUBECONFIG=$OBOL_CONFIG_DIR/kubeconfig.yaml`.

## CLI Commands

```
obol
├── stack           init, up, down, purge
├── agent           init (deploys obol-agent singleton),
│                   auth --runtime <runtime> obol-agent  (Bearer token — canonical)
├── network         list, install, add, remove, status, sync, delete
├── sell            inference, http, list, status, stop, delete, pricing, register

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ObolNetwork/obol-stack](https://github.com/ObolNetwork/obol-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
