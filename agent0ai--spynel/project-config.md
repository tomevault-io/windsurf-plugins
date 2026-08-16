---
trigger: always_on
description: - DOX is the AGENTS.md hierarchy used by this project.
---

# DOX framework

- DOX is the AGENTS.md hierarchy used by this project.
- Agents must follow the applicable AGENTS.md chain before editing files.

## Core Contract

- AGENTS.md files are binding work contracts for their subtrees.
- Durable docs, source files, prompts, tests, and packaging metadata must stay understandable from the nearest applicable AGENTS.md plus every parent AGENTS.md above it.
- Spynel is the Go application rooted in this repository. It owns chat transports, harness sessions, deterministic markdown orchestration, persisted runtime state, executable project extensions, documentation, and distribution.
- Describe Spynel consistently as a classic, non-AI program that coordinates and oversees external AI/coding harnesses. It is not itself an agent and does not compete with or duplicate harness intelligence. The product relationship is one human through one assistant-facing relationship to many agents; “One human → one agent → infinite agents” is a conceptual expression of scalable leverage, never a literal resource guarantee. Use “Simplicity at scale.” as the primary slogan and “Simplicity. Leverage. Quality.” as the defining words. Do not invent capabilities, integrations, guarantees, or adoption claims when adapting this positioning to public copy.
- The retired Python prototype and the former nested Go layout are not part of the repository. Product behavior, tests, documentation, packaging, and automation use root-relative paths.
- Produce one cross-platform application that hosts the styled TUI, long-running communication channels, coding-harness sessions, Markdown orchestration, and project extensions. Release archives include the sherpa-onnx native runtime beside the executable.
- `cmd/spynel` is composition only. Channels translate external messages into the provider-neutral application contract and never call a coding harness directly.
- Every channel conversation and orchestrated Markdown file has an independent harness session. Persisted leases prevent duplicate orchestration and recover stale work across restarts.
- Every running process in one workspace participates in a single primary-server election. The owner alone runs Telegram, WhatsApp, and continuous Markdown orchestration; every TUI reaches the owner's application service over an authenticated loopback API. Primary leases carry a non-secret digest of a private per-user/per-environment installation token so a client rejects a known-foreign host/container loopback before dialing; missing or invalid identifiers remain fresh-owner fenced and receive only bounded compatibility readiness. Interactive secondary TUI startup reports connection progress before entering the alternate screen, while automation remains quiet. The first TUI in an ownerless workspace resumes the latest TUI communication session, while additional live TUIs receive independent sessions. Five-second heartbeats, a 30-second stale threshold, lock-free discovery of atomically published leases, serialized compare-and-replace takeover, and target-fenced `/primary` handoffs prevent duplicate owners while allowing secondary startup, explicit promotion, or recovery from a dead or stalled server.
- The main communication agent is a responsive dispatcher: it answers questions and reports durable state, routes finite work into tasks and long-term objectives into goals, and leaves substantive execution to independent orchestrator sessions.
- The communication agent performs bounded inspection without narrating routine tool steps, then gives one brief, natural, outcome-first result. Ordinary question, status, and delegation turns have no preamble or progress messages. Routine task and goal confirmations hide internal files, paths, IDs, metadata, lifecycle details, and orchestration mechanics; explicit detail requests and dedicated diagnostic commands such as `/status`, `/jobs`, `/tasks`, and `/goals` remain technically precise. Telegram and WhatsApp routine confirmations never contain local-path Markdown links. Agents must obtain current UTC from the environment instead of estimating timestamps written into durable Markdown state.
- The communication agent carries a framework-owned evidence-grounded honesty contract even when a workspace preserves a customized chat template. It must distinguish observation and durable or source-backed evidence from inference, uncertainty, and unknowns; never lie, fabricate evidence, invent causality, imply inspection it did not perform, or present stale or unverified claims as facts; and verify materially uncertain current, workspace-specific, causal, high-impact, completion, dispatch, delivery, release, external-state, or code-behavior claims when bounded authoritative inspection is practical. Otherwise it says that it does not know yet and offers or dispatches investigation. Corrections identify the unsupported claim and changed evidence rather than reflexively agreeing. Stable low-risk knowledge remains responsive without ritual lookup, and this behavioral contract is not represented as a mathematical guarantee of model perfection.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent0ai/spynel](https://github.com/agent0ai/spynel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
