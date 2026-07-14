---
trigger: always_on
description: - RightMemory is a tree + graph hybrid memory system designed primarily for AI agents. Human readability matters, but it is not the main design center.
---

# RightMemory Agent Notes

## Project Shape
- RightMemory is a tree + graph hybrid memory system designed primarily for AI agents. Human readability matters, but it is not the main design center.
- Core runtime code lives in `rightmemory/`: config loading, command orchestration, standalone tools, CLI-agent delegation, transcript review, async update batching, isolated semantic writes, and provider transcript adapters.
- Canonical role prompts live in `rightmemory/prompts/`. Edit role behavior there first; installed skills do not contain generated role prompts.
- `skills/rightmemory-schema.md` is the schema source for memory files. `MEMORY.example.md` is the installer seed and the source for the managed example block that can be refreshed on reinstall.
- `install.sh` and `install.ps1` are platform bootstraps for the shared stdlib-only `rightmemory.install_core` transaction. Both modes preserve existing user memory files and refresh the managed example block when present.
- `retrieve` model config is independent. Other roles may reuse the configured writer executor when their own `[<role>.model]` or `[<role>.agent_cli]` table is absent, so upgrade-added roles can run without rewriting user config.

## Development Commands
- Run the test suite with `python -m unittest discover -s tests`.
- For syntax-only checks, use `python -m compileall -q rightmemory tests`.
- Use `./install.sh [--mode cli-agent|standalone] <memory-root> <skills-target>` on macOS/Linux/WSL or `.\install.ps1 [--mode cli-agent|standalone] <memory-root> <skills-target>` on Windows PowerShell when verifying install behavior.
- `uv` is available on PATH. Use `uv --version` if you need to check it before running the installer.
- Useful review commands are `rightmemory review scan --once`, `rightmemory review watch`, and `rightmemory review normalize --source <codex|claude> --path <file>`.
- Use `rightmemory prune` to run generation-based active memory pruning, and `rightmemory history --session <id> <query>` for explicit retrieval from pruned memory.
- Use `rightmemory shared-view list|build-file|build-question|approve|pull|status|ask|credential|accept-invite|note|notes|inbox|inbox-http` when debugging `MF#`/`MQ#` shared-view connections, provider view source files, HTTP hubs, credentials, or interaction records.
- Use `rightmemory hub init|status|token|serve` when debugging self-hosted HTTP shared-view hubs.
- Use `rightmemory watch start|status|stop|restart` to manage background review, dreamer, insight, pruner, and sync watchers. Use `rightmemory dreamer watch`, `rightmemory insight watch`, or `rightmemory prune watch` directly when debugging lower-level loops.
- Use `rightmemory doctor agent-cli` after configuring cli-agent mode to check provider commands, role config, and basic read/write probes.
- Semantic upgrade notes are Markdown files under `rightmemory/semantic_upgrades/`; validate them with `python -m unittest discover -s tests -p 'test_semantic_upgrades.py'`.

## Maintaining This File (IMPORTANT!)
- Treat this file (./AGENTS.md) as operational instructions for coding agents, not as a design document. Keep durable design explanation in `README.md` or `DESIGN_NOTES.md`.
- Update this file when setup commands, test commands, install behavior, role boundaries, or git/memory safety rules change.
- Remove stale commands or environment assumptions as soon as they stop matching the repo; bad instructions are worse than missing instructions.
- Keep it concise enough to stay useful in Codex project instructions. Prefer scoped nested `AGENTS.md` files if a subdirectory needs special rules.

## Memory Runtime Rules
- A memory root contains `MEMORY.md`, optional sibling `MEMORY_*.md` detail files, `shared_views.toml`, `shares.toml`, optional provider-owned `shared_views/<view-id>/` source files, `insight_logs/`, `rightmemory.toml`, and `.runtime/`.
- Named profiles are registered in `<default-memory-root>/profiles.toml`. `rightmemory profile create <name>` defaults new roots to a sibling profile area such as `~/.rightmemory-profiles/<name>` for the normal default root.
- Runtime commands can select a profile with `--profile <name>`, or by a user-managed `.rightmemory-profile` file in the project tree. Tracking that file is a user/project choice.
- Profile roots are ordinary memory roots with separate `MEMORY.md`, `rightmemory.toml`, `.runtime/`, Git history, watcher state, async queues, sessions, and insight logs.
- The installer creates a memory-root `.gitignore` allowlist so git status normally shows `MEMORY.md`, `MEMORY_*.md`, `shared_views.toml`, `shares.toml`, provider view source files under `shared_views/<view-id>/`, and `insight_logs/*.md`.
- Runtime/session/review state belongs under `.runtime/` and should not be committed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RightL/RightMemory](https://github.com/RightL/RightMemory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
