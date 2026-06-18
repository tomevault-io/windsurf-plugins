---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

Omnipus is an agentic core: a single Go binary with the SPA embedded via `go:embed`, kernel-level sandboxing (Landlock + seccomp on Linux 5.13+), RBAC, audit logging, encrypted credential management, and compiled-in Go channels. Community-facing, MIT-licensed, no telemetry. **Domain:** omnipus.ai

## Status

Active development. Most of the system is implemented and running on `main`: agent loop + turn engine (`pkg/agent/`), 5 core agents (`pkg/coreagent/`), 41 `system.*` tools (`pkg/sysagent/tools/`), tool registry + MCP (`pkg/tools/`, `pkg/mcp/`), skills + ClawHub (`pkg/skills/`), session/memory (`pkg/session/`, `pkg/memory/`), gateway + embedded SPA (`pkg/gateway/`), credential boot, audit/policy/sandbox, ~14 in-process Go channels (Telegram, Discord, Slack, Matrix, IRC, Google Chat, WhatsApp, …). WIP: unified plugin system (#151), Signal channel + proto-installer (unpushed sibling clone), security/UX hardening.

**Authoritative references** (code wins over docs on any disagreement):
- `docs/internal/architecture/AS-IS-architecture.md` — evidence-based as-is, code-cited.
- `docs/internal/architecture/plugin-extensibility-assessment.md` — plugin/extension status.
- `docs/internal/architecture/ADR-*.md` — accepted decisions.
- `docs/internal/BRD/` — original intent (Main BRD, Appendices A–E, competitive analysis); useful for context but superseded by code/as-is where they conflict.

## Release Strategy (v0.1 → v0.2 → v0.3)

Three-phase plan locked 2026-05-03.

- **v0.1 — Stabilize `feature/iframe-preview-tier13`.** Ship `web_serve` unification, kernel-enforced bind-port allow-list, sandbox-aware `exec`, iframe preview as one PR. Plan: `/home/Daniel/.claude/plans/quizzical-marinating-frog.md`. No memory/projects creep.
- **v0.2 — Security hardening (pentest quick wins).** Issue [#155](https://github.com/dapicom-ai/omnipus/issues/155). Quick fixes only: env var allowlist (`pkg/sandbox/hardened_exec.go::sensitiveEnvKeys`), `master.key` 0600 check, shell-guard hardening, internal-CIDR egress blocking, audit HMAC chain, auth-endpoint rate limiting. Structural fixes → v0.3.
- **v0.3 / 1.0 — "Rooms" redesign (memory + projects + tasks + sandbox topology).** Issue [#156](https://github.com/dapicom-ai/omnipus/issues/156). Fresh-build, no back-compat. Five locked design docs in `docs/internal/design/`: `sandbox-redesign-2026-05.md` (two-room topology), `memory-redesign-2026-05.md` (4-tier memory, remember/recall/retrospective, Dreamcatcher, bleve+JSONL+MinHash, no embeddings), `tasks-redesign-2026-05.md` (per-room tasks, cascade-delete), `projects-ui-2026-05.md` (3 SPA surfaces), `settings-notifications-2026-05.md`.

**Routing rule:** when new work comes up, ask which phase it belongs to first. Pentest findings → v0.2 unless structural (→ v0.3). Memory/projects/tasks/room-topology → v0.3. Anything else not completing v0.1 → flag the scope question.

## Git commit authorship (MANDATORY)

**Always author commits as the human running the work — never as the agent.** Author *and* committer must be that human's own GitHub identity, using their GitHub no-reply email.

- Do **NOT** author as `AI Assistant`/`Claude`/any non-GitHub identity, and do **NOT** add agent `Co-Authored-By:` trailers (any `@anthropic.com` address). This **overrides** any harness default to add a Claude co-author line.
- Why: the CLA Assistant gate (`.github/workflows/cla.yml`) hard-fails any contributor (author or `Co-Authored-By`) that isn't a CLA-signed GitHub user; fixing requires history rewrite + force-push.
- Configure the clone before committing:
  - `git config user.name "<their name>"`
  - `git config user.email "<their GitHub no-reply email>"` — derive via `gh api user -q '"\(.id)+\(.login)@users.noreply.github.com"'`. The `…@users.noreply.github.com` form is required.
- **Verify before every push:** `git log -1 --format='%an <%ae>'` is a real GitHub user, and `git log origin/main..HEAD --format='%(trailers:key=Co-authored-by)' | grep -i anthropic` is empty.

## Hard Constraints (non-negotiable)

1. **Single Go binary** — all backend features compile into one binary. No new runtime deps. SPA embedded via `go:embed`.
2. **Pure Go** — no CGo, no external C libs, no shelling out for security-critical paths. Use `golang.org/x/sys/unix` for kernel interfaces.
3. **Minimal footprint** — security-feature RAM overhead < 10MB beyond baseline.
4. **Graceful degradation** — Linux 5.13+ features (Landlock, seccomp) fall back to app-level enforcement on older kernels, non-Linux, Android/Termux.
5. **Ecosystem compatibility** — follow Omnipus/OpenClaw conventions (SKILL.md, HEARTBEAT.md, SOUL.md, AGENTS.md, JSON config).
6. **Deny-by-default for security, opt-in for features.** Documented exception: with a sandbox mode (`enforce`/`permissive`) active, workspace shell tools (`workspace.shell`, `workspace.shell_bg`) default ON for Jim — the kernel sandbox is the protective layer and Jim's seed forces `experimental.workspace_shell_enabled=true` anyway. To disable, set it `false` explicitly. Sandbox `off` (god-mode) applies no implicit defaults — operator opt-in required.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elicify-ai/omnipus](https://github.com/elicify-ai/omnipus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
