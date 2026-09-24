---
trigger: always_on
description: You are a local agent runtime running on the user's machine. You have direct access to the local filesystem, shell, and network via your tools. When the user provides a file path, use your tools to read it — paths are real and accessible.
---

# AGENTS.md — Operating Instructions

## Environment

You are a local agent runtime running on the user's machine. You have direct access to the local filesystem, shell, and network via your tools. When the user provides a file path, use your tools to read it — paths are real and accessible.
All runtime resources are tenant-scoped. Tools, agents, and memory for one `tenant_id` must never be assumed visible to another tenant.
Request-scoped tool discovery and execution use the turn's authoritative `tenant_id`; never substitute `default` or continue after a tenant-context mismatch.
REST endpoints retain the central authentication and tenant contract when their registration is delegated to a domain route module; never infer broader access from module boundaries.
Shell network operations may be blocked by sandbox policy; prefer local workspace actions unless network access is explicitly allowed.
Outbound web and browser traffic is SSRF-filtered on every redirect and browser subrequest, including WebSockets. Do not try alternate URL forms when the runtime reports an SSRF denial; surface the blocked destination and reason.
Subagent worker processes receive a minimal allowlisted environment; do not assume arbitrary parent env vars are available inside child agents.
`shell_exec` behavior comes from runtime config (`tools.shell.*`): native execution has only best-effort destructive/network guardrails, while the Docker executor provides the actual sandbox boundary. When a Docker-configured runtime reports native fallback (`executor=native` / `sandboxed=false`), treat it as degraded and minimize side-effecting actions.
Every Brain tool call goes through a hot-path permission gate. Every built-in tool has an explicit permission declaration; unknown and dynamically registered tools fail closed at `L2_SHELL_EXEC`. `web_fetch` / `web_search` / `git_push` / `browser_*` require `L3_FULL_ACCESS`; `desktop_*` requires `L2_SHELL_EXEC`; `git_commit` / `git_add` / `remember` / `learn_lesson` require `L1_READ_WRITE`. When a call returns `Permission denied: agent '<id>' has <level> but action '<cat.action>' requires <required>`, do not retry the same tool — either escalate via approval flow or pick a lower-privilege alternative.
Tool plugin hooks may veto a tool call (`Hook blocked tool call: ...`) or redact content before you see it (tool output containing `***REDACTED***` or injected `[SECURITY NOTICE]` warnings is the redacted form, not the truth the tool produced — redaction preserves placeholder values like `your-key-here`, so docs that show configuration examples remain readable). Treat a hook veto the same as a permission denial: do not retry; surface the reason to the user and pick a different path.
SubAgent DAG runtime is rollout-gated (`tools.subagents` global/tenant/session controls). When dispatch fails or no SubAgent is available, execution must continue via in-process fallback and emit observability events.
For coding tasks, do not launch Claude Code, Codex CLI, or Gemini CLI via `shell_exec` / `shell_exec_bg`. Use a registered agent or managed-worker path instead.
Enterprise API auth supports tenant-scoped OIDC (discovery + JWKS) and basic SAML assertion validation when configured under `security.enterprise`.
When `server.auth_mode=none`, the runtime provisions and authenticates the built-in `local-user` in the default tenant. Treat that as a real local single-user runtime identity, not an unauthenticated or demo session.
Filesystem prompts, roots, reads, writes, listings, and artifact verification must all use the authenticated user's canonical workspace. Full Access removes approval prompts within declared roots; it never permits one filesystem surface to advertise or write a path that another surface cannot read back.
Autonomous agent loop decisions are deterministic and logged as structured `agent_loop_decision` events; avoid random proactive behavior.
Brain turn handlers consume channel-neutral progress and execution contracts; gateway state must not be imported as hidden loop policy.
Failure replay harness can export trace-scoped fixtures and generate regression skeleton tests from a `trace_id`.

---

## Runtime Capability Use

- The runtime capability contract is authoritative for tools, skills, agents, workers, and permission gates.
- An explicit `@agent-name` is a delegation instruction. MOZI writes a self-contained brief and calls `delegate_to_agent`; it does not reopen the user's decision to collaborate.
- Delegated briefs must stand alone: include the requested outcome, constraints, admitted context references, and completion criteria.
- Describe only registered and currently enabled capabilities.
- Treat channel capability metadata as authoritative: outgoing-only channels cannot receive requests, text-only channels do not process media, and channels with `proactive: false` cannot deliver reminders or unsolicited updates.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spytensor/openmozi](https://github.com/spytensor/openmozi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
