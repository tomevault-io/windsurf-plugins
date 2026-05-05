---
trigger: always_on
description: Sandboxed AI coding agent runner. Runs AI coding CLI agents (Claude Code, Gemini, Codex) inside disposable Docker containers with copy/diff/apply workflow. Additional agents (Aider, Goose, etc.) in future versions.
---

# yoloAI

Sandboxed AI coding agent runner. Runs AI coding CLI agents (Claude Code, Gemini, Codex) inside disposable Docker containers with copy/diff/apply workflow. Additional agents (Aider, Goose, etc.) in future versions.

## Project Status

Public beta. Breaking changes are allowed but must be tracked in `docs/BREAKING-CHANGES.md`.

## Key Files

User-facing docs live in `docs/`:

- `docs/GUIDE.md` — Full usage reference: commands, flags, workdir modes, agents/models, configuration, sandbox state, security, development.
- `docs/BREAKING-CHANGES.md` — Tracks breaking changes made during beta. Each entry documents previous behavior, new behavior, rationale, and migration steps. Include in release notes.
- `docs/ROADMAP.md` — Future plans: agents, network isolation, profiles, overlayfs, etc.

Design specs live in `docs/design/`:

- `docs/design/README.md` — Goal, value prop, architecture, directory layout, prerequisites, resolved decisions.
- `docs/design/commands.md` — Command table, agent definitions, all command specs.
- `docs/design/config.md` — Docker images, config.yaml format, recipes, profiles.
- `docs/design/setup.md` — First-run experience, tmux configuration.
- `docs/design/security.md` — Credential management, security considerations.

Development docs live in `docs/dev/`:

- `docs/dev/ARCHITECTURE.md` — Code navigation guide: package map, file index, key types, command→code map, data flows, "where to change" recipes, testing. Keep in sync when architecture changes.
- `docs/dev/CODING-STANDARD.md` — Code style: Go 1.22+, gofmt, golangci-lint, Cobra, project structure, naming, error handling, dependency policy.
- `docs/dev/CLI-STANDARD.md` — CLI design conventions: argument ordering (options first), flag naming, exit codes, error messages, help text format.
- `docs/dev/RESEARCH.md` — Index of research documents. Detailed research split into topic files in `docs/dev/research/`: competitors, agents, security, sandboxing, implementation.
- `docs/dev/CRITIQUE.md` — Rolling critique document. After a critique pass, findings are applied to design docs and research files, then CRITIQUE.md is emptied for the next round.
- `docs/dev/OPEN_QUESTIONS.md` — Questions encountered during design/implementation that need resolution.
- `docs/dev/plans/TODO.md` — Consolidated list of designed-but-unimplemented features with design references.
- `docs/dev/old/PLAN.md` — Historical implementation plan (phases, architecture decisions). Reference for how yoloAI was built.
- `docs/dev/backend-idiosyncrasies.md` — **Read this before diagnosing any backend problem.** Catalogs observed behaviors that contradict official documentation, required non-obvious workarounds, or have caused bugs before. Includes a symptom index for fast lookup.

## Architecture (from design docs)

- Go binary, no runtime deps — just the binary and Docker (or Tart for macOS VMs, or Seatbelt for lightweight macOS sandboxing).
- Pluggable runtime backend via `runtime.Runtime` interface in `internal/runtime/`. Three backends: Docker (`internal/runtime/docker/`), Tart (`internal/runtime/tart/`), and Seatbelt (`internal/runtime/seatbelt/`). CLI dispatches via `newRuntime()` in `internal/cli/helpers.go`. No backend-specific types leak outside their packages.
- Docker containers or Tart VMs with persistent state in `~/.yoloai/sandboxes/<name>/`.
- Containers are ephemeral; state (work dirs, agent-state, logs, meta.json) lives on host. Credentials injected via file-based bind mount (not env vars).
- Agent abstraction: per-agent definitions specify install, launch command, API key env vars, state directory, network allowlist, and prompt delivery mode. Ships Aider, Claude, Codex, Gemini, and OpenCode agents.
- CLI separates workdir (primary project dir, positional) from aux dirs (`-d` flag). Directories mounted at mirrored host paths by default. Custom paths via `=<path>` override.
- `:copy` directories use full directory copies with git for diff/apply.
- `:overlay` directories use Linux overlayfs inside the container for instant setup with diff/apply workflow. Changes are captured in an upper layer; no file copying. Docker-only, requires CAP_SYS_ADMIN. Container must be running for diff/apply (git commands exec inside container).
- `:rw` directories are live bind-mounts. Default (no suffix) is read-only.
- Profile system: each profile is a directory in `~/.yoloai/profiles/<name>/` containing a `Dockerfile` and `config.yaml`. The base profile at `~/.yoloai/profiles/base/` is auto-created if missing and serves as the default. "base" is a reserved profile name.
- Two config files: global config (`~/.yoloai/config.yaml`) for user preferences (tmux_conf, model_aliases) and profile config (`~/.yoloai/profiles/base/config.yaml`) for profile-overridable defaults (agent, model, backend, env, etc.). `IsGlobalKey()` routes config commands to the correct file. Operational state (`setup_complete`) lives in `~/.yoloai/state.yaml`.

## Code Quality Gate


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kstenerud/yoloai](https://github.com/kstenerud/yoloai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
