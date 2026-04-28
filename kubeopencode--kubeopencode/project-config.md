---
trigger: always_on
description: This document provides guidelines for AI assistants working on the KubeOpenCode project.
---

# Development Guidelines for KubeOpenCode

This document provides guidelines for AI assistants working on the KubeOpenCode project.

## Project Overview

> **Disclaimer**: This project uses [OpenCode](https://opencode.ai) as its primary AI coding tool. KubeOpenCode is not built by or affiliated with the OpenCode team.

KubeOpenCode brings Agentic AI capabilities into the Kubernetes ecosystem. By leveraging Kubernetes, it enables AI agents to be deployed as services, run in isolated virtual environments, and integrate with enterprise management and governance frameworks.

> **IMPORTANT FOR AI ASSISTANTS**: The OpenCode project source code is located at `../opencode/` (sibling directory to kubeopencode). Always prioritize searching the local OpenCode codebase first before using web search.

**Key Technologies:** Go 1.25, Kubernetes CRDs, Controller Runtime (kubebuilder), Helm

**Architecture Philosophy:**
- No external dependencies (no PostgreSQL, Redis) — Kubernetes-native (etcd for state, Pods for execution)
- Simple API: Task (WHAT to do) + Agent (HOW to execute)
- Use Helm/Kustomize for batch operations (multiple Tasks)
- Event-driven triggers delegated to [Argo Events](https://argoproj.github.io/argo-events/)

**Unified Binary:** Single container image (`ghcr.io/kubeopencode/kubeopencode`) with subcommands: `controller`, `git-init`, `context-init`, `url-fetch`. Image constant: `internal/controller/pod_builder.go` → `DefaultKubeOpenCodeImage`.

## Core Concepts

### Resource Hierarchy

1. **Task** - Single task execution (the primary API)
2. **CronTask** - Scheduled/recurring task execution (creates Tasks on cron schedule)
3. **Agent** - AI agent configuration (HOW to execute)
4. **AgentTemplate** - Reusable base configuration for Agents (optional)
5. **KubeOpenCodeConfig** - Cluster-scoped system-level configuration (optional, singleton named `cluster`)

### Important Design Decisions

- **Agent** (not KubeOpenCodeConfig) - Stable, project-independent naming
- **Agent = running instance** - Always creates Deployment + Service (no Pod mode vs Server mode)
- **agentRef or templateRef** - Task must reference exactly one: an Agent (persistent) or AgentTemplate (ephemeral)
- **No Batch/BatchRun** - Use Helm/Kustomize (Kubernetes-native approach)
- **CronTask** - Scheduled task execution (CronTask:Task = CronJob:Job). Creates Tasks on cron schedule with concurrency policy, maxRetainedTasks (blocks creation, does NOT delete), and manual trigger support
- **Two-container pattern**: Init container (`agentImage`) copies OpenCode binary to `/tools`, Worker container (`executorImage`) runs the server

### Context System

Tasks and Agents use inline **ContextItem** for additional context. Types: Text, ConfigMap, Git, Runtime, URL.

Key behaviors:
- Empty `mountPath` → content written to `${WORKSPACE_DIR}/.kubeopencode/context.md` with XML tags
- Relative paths prefixed with `workspaceDir`; absolute paths used as-is

> See `website/docs/features.md` for detailed context examples and field reference.

### Agent Configuration (Summary)

Key Agent spec fields: `templateRef`, `profile`, `agentImage`, `executorImage`, `attachImage`, `command` (optional, has default), `workspaceDir` (required), `port` (default: 4096), `extraPorts` (additional Service/Deployment ports for DinD, VS Code, etc.), `persistence`, `suspend`, `standby` (automatic suspend/resume lifecycle), `contexts`, `skills` (external SKILL.md sources from Git repos, auto-injects `skills.paths` into OpenCode config), `plugins` (OpenCode plugins installed via npm; controller creates a plugin-init container that runs `npm install` into a shared `/plugins` volume, loads via `file:///plugins/node_modules/<package>` path into OpenCode config's `plugin` array — executor container does not need npm), `config` (inline YAML/JSON object → `/tools/opencode.json`), `credentials`, `caBundle`, `proxy`, `imagePullSecrets`, `podSpec`, `serviceAccountName`, `maxConcurrentTasks`, `quota`, `share` (shareable terminal link).

> See `website/docs/features.md` for detailed YAML examples of Agent configuration, proxy, credentials, concurrency, quota, persistence, and Git auto-sync.

**Docker in Docker (DinD)**: The devbox executor image includes Docker CLI. Full DinD (build + run containers) is supported via `podSpec.runtimeClassName` (Sysbox recommended) or `podSpec.securityContext` (privileged/rootless). No code changes needed — existing `AgentPodSpec` fields are sufficient. See `website/docs/use-cases/docker-in-docker.md`.

**Git Auto-Sync**: Git contexts support `sync` field with `enabled`, `interval`, and `policy` (HotReload or Rollout). HotReload uses a `kubeopencode git-sync` sidecar for in-place updates. Rollout triggers Deployment rolling update with Task protection (waits for active Tasks, 1h safety timeout). See ADR 0027.

### AgentTemplate

Reusable blueprint serving two roles: (1) base configuration for Agents via `spec.templateRef.name`, (2) blueprint for ephemeral Tasks via `Task.spec.templateRef`. Merge strategy: Agent wins for scalars; Agent **replaces** template for lists. Agent-only fields: `profile`, `port`, `persistence`, `suspend`, `standby`, `templateRef`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kubeopencode/kubeopencode](https://github.com/kubeopencode/kubeopencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
