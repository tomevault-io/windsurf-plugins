---
trigger: always_on
description: > **⚠️ Project Status: Early Development**
---

# alan - AI Agent Guide

> **⚠️ Project Status: Early Development**
>
> This project is actively being developed. APIs may change without notice.

---

## Component Names

Use these names consistently in docs, specs, code comments, UI copy, and review
comments:

| Name | Meaning |
| ---- | ------- |
| **Alan** | The product: a programmable personal computing environment. Do not use "programmable environment" as a separate product name. |
| **Alan OS** | The operating-system boundary for Alan: Alan Kernel, file-server system services, Service Manager, Root Agent Process, Agent Runtime Service, hosts, and app integration conventions. It is not the CLI, HTTP/WS compatibility transport, TUI, macOS app, or agent engine. |
| **Alan Kernel** / `alan-kernel` | The file-tree substrate inside Alan: namespace and mounts, paths, files, descriptors, access rights, credentials, process table, and a single `Process` category. Agent-ness is a file-layout convention, not a Kernel type. Streams are file kinds; process output, requests, actions, and events are files. |
| **Standard Namespace** | The canonical Alan OS root layout: `/proc`, `/agent`, `/srv`, `/bin`, `/lib`, `/man`, and `/mnt`. Alan-specific packages and mounted service trees live under `/lib` or `/mnt`, not as new top-level roots by default. |
| **Service Manager** | The Alan OS system Process that starts, stops, restarts, and supervises system services and boot units. It replaces the former daemon as the canonical lifecycle concept. |
| **File-Server Service** | A long-running Process that exports a file tree which other processes mount or bind into their namespace. Alan OS services are file servers, not HTTP APIs. |
| **Service Handle Registry** / `/srv` | The Plan 9-style rendezvous tree where running file servers post mountable handles. `/srv` is not the service state tree. |
| **Agent Runtime Service** | The file-server service that executes Agent Processes and serves AgentFS at `/agent`. It is an internal system service, not an app-facing API. |
| **Process** | A bounded execution with PID, parent, descriptors, credentials, lifecycle, input/output streams, status, and exit state. |
| **Agent Process** | An ordinary Process that runs an agent, recognized by conforming to the agent file-layout — not a separate Kernel type. It lives in `/proc/<pid>` (the source of truth) and is surfaced through the `/agent/<pid>` view. |
| **Root Agent Process** | The always-available Agent Process at the root of the agent process tree, exposed through `/agent/root`. It coordinates child Agent Processes; it is not root permission, the Service Manager, a root chat session, or the Alan Agent UI. |
| **Agent Executable** | An executable that creates an Agent Process when spawned. Agent executables are command files bound into `/bin`, not RPC/API methods. |
| **Tool** | A reusable executable installed into the Alan OS command namespace. Tools provide actions; permissions come from descriptors, access rights, and policy. |
| **Skill** | A manual-like knowledge package installed into the Alan OS namespace and passed to Agent Processes by descriptor. Skills provide understanding; they do not execute. |
| **Memory Stores** | Personal, system-continuity, app, and workspace file trees that own memory authority. Agent memory kinds such as working, episodic, semantic, and procedural describe how memory is used, not who owns it. |
| **Alan Agent** | A built-in but optional Agent Workspace app for inspecting, steering, and organizing Agent Processes. It is not required to run agents and is not the Root Agent Process or Agent Runtime Service. |
| **Agent Execution Engine** / `alan-agent-engine` | Current implementation of the agent Turing-machine loop: tape, model calls, tool compatibility, skills, policy, memory, and persistence. It backs Agent Runtime Service work; it is not Alan Kernel. (Renamed from `alan-runtime`; lives in `crates/agent-engine`.) |
| **Alan for macOS** | Native Apple host for Alan: renderer, input shell, windowing, and OS integration surface. |
| **Alan Shell** / future `alan-shell` | The primary shell for Alan OS: a Plan 9 `rc`-like and Acme-like interaction surface for files, processes, Agent Processes, Tools, Skills, Memory Stores, and services. The current implementation path is Ratatui in `crates/tui`. |
| **Alan Agent App Module** / future `alan-agent` | Optional workspace module that reads agent files (status, io, requests, actions, machine) as a client and renders from those files, not from core-owned view snapshots. |
| **Alan Apps** | Apps such as Alan Agent and Groove Master that run on Alan OS with app-owned domain cores and Alan adapters. |

---

## Architecture Progression Principle

When implementing OpenSpec changes, move the codebase toward the accepted Alan
OS target shape, not just the smallest local patch.

- Prefer directory structure, crate boundaries, module names, and public APIs
  that match the target architecture already recorded in OpenSpec and this
  guide.
- When touching old compatibility code, add or extract the next durable layer
  if it clarifies ownership and reduces future migration risk. For example,
  keep Alan Kernel semantics in `alan-kernel`, Agent Runtime Service and
  AgentFS-oriented execution behind `alan-agent-engine`, Alan

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realmorrisliu/alan](https://github.com/realmorrisliu/alan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
