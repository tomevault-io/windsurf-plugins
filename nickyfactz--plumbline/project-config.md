---
trigger: always_on
description: This repository contains the Plumbline plugin for Codex and Claude Code. Keep the plugin skills-first and repository-local: its optional continuity hook is bundled with the plugin, while installation must not create global skills, global agent files, project/global hook configuration, MCP servers, or a custom worktree system. The bundled hook must remain inert until the user explicitly invokes the Plumbline front door; it only restores a compact reminder after resume or compaction and never sele
---

# Plumbline repository guidance

This repository contains the Plumbline plugin for Codex and Claude Code. Keep the plugin skills-first and repository-local: its optional continuity hook is bundled with the plugin, while installation must not create global skills, global agent files, project/global hook configuration, MCP servers, or a custom worktree system. The bundled hook must remain inert until the user explicitly invokes the Plumbline front door; it only restores a compact reminder after resume or compaction and never selects phases, runs setup, edits the repository, or dispatches agents. Claude Code loads this shared guidance through the repository's `CLAUDE.md` `@AGENTS.md` import.

## Project-local agent team

Use only project-local agent definitions when this checkout is explicitly initialized for delegated work: `.codex/agents/*.toml` for Codex or `.claude/agents/*.md` for Claude Code. The global Codex config may be inspected for host capability and a model candidate, but personal/global agent definitions are never selected or used as fallback. Do not edit global Claude settings or enable Claude Agent Teams automatically. Keep `.codex/config.toml`, `.codex/agents/`, `.claude/agents/`, and any installed local router untracked unless the user explicitly chooses otherwise.

Treat explicit user instructions as authoritative over Plumbline defaults and
process preferences, but interpret them in the context of the user's stated
outcome, approved artifacts, and repository evidence. Do not follow an
ambiguous instruction literally when doing so would contradict the apparent
goal, approved contract, or safety boundary. Resolve ordinary ambiguity with a
safe reversible default; ask a focused product question only when competing
interpretations would materially change behavior.

For material multi-step work in a Git checkout, use a `required` Git policy:
create main-thread checkpoint or batch commits at coherent boundaries by
default, unless the user explicitly opts out. State the policy once without
blocking for approval. Record the starting `HEAD`, keep unrelated dirty files,
scratch, and secrets out, and do not advance dependent checkpoints with
uncommitted plan-owned changes. Use checkpoint commits, `git show`, and focused
diffs as worker hydration context. Ask before push, force-push, history
rewriting, or external publication. If Git is absent, recommend establishing it
before Execute; an explicit opt-out is reported as Git-unanchored.

Keep the main thread thin: it owns product decisions, specifications, plans, integration, Git, singleton operations, and all delegation. Read only the controlling artifact, repository guidance, Git state, and named paths needed to route and integrate work. Before broad grep, repository archaeology, multi-file fact gathering, external research, or cross-seam review, dispatch the matching project-local role with a bounded brief when it can return a bounded result that materially reduces main-thread context or improves quality; keep small, tightly coupled, and inherently main-owned work direct. During material Execute work, prefer a matching project-local role for useful bounded research, architecture, implementation, review, testing, or another capability with a clear boundary, especially read-heavy or independent work that can safely run in parallel. Ask read-heavy workers for a compact decision packet containing the conclusion, exact pointers, constraints, residual uncertainty, and next action; do not repeat their exploration on the main thread. Report-only roles receive no write set. Worker recommendations are advisory and return to the main thread; workers never invoke, hand off to, or dispatch another worker. Codex `sandbox_mode = "read-only"` and Claude `permissionMode: plan` are intent; a writable parent may affect effective permissions. Emit one compact dispatch line with selected roles, host-native model/reasoning or effort, and short assignments. Mention standard boundaries or effective values only for an exception, mismatch, or user question; omit routine starting, waiting, return, and unchanged-state narration. Inspect Git status/diff after return and report unexpected edits. Preserve project-local role values and never invent personal/global roles. Restore `delegation_roles` and `delegation_status` after compaction. Keep only already-understood tiny work or tightly coupled main-owned actions direct; use `Direct: <reason>` only when useful bounded work looked delegable but no matching local role remains. Worker leaf behavior is a Plumbline orchestration boundary, not a required Codex depth setting. When independent work has stable contracts, disjoint scopes, no result dependency, and a clear join condition, the main thread may dispatch a parallel wave; otherwise keep it serial.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nickyfactz/plumbline](https://github.com/nickyfactz/plumbline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
