---
trigger: always_on
description: Bento packages AI agent workspace state into portable, layered OCI artifacts. It captures everything git doesn't: agent memory, installed dependencies, build caches, conversation history, and session state. Checkpoints are stored as standard OCI images in any container registry.
---

# AGENTS.md - Bento Development Guide

## Project Overview

Bento packages AI agent workspace state into portable, layered OCI artifacts. It captures everything git doesn't: agent memory, installed dependencies, build caches, conversation history, and session state. Checkpoints are stored as standard OCI images in any container registry.

**Repository:** `github.com/kajogo777/bento`
**Language:** Go (1.25+)
**License:** Apache 2.0

## Architecture

```
cmd/bento/main.go           → entrypoint, version injection
internal/
  cli/                       → cobra commands (init, save, open, list, diff, etc.)
  config/                    → bento.yaml parsing, validation, platform defaults
  extension/                 → composable extensions (agent, deps, tool detection)
  workspace/                 → file scanning, layer packing (tar+gzip), .bentoignore
  registry/                  → OCI store (local image layout + remote push)
  manifest/                  → OCI manifest/config construction, annotations, DAG
  secrets/                   → secret scanning (gitleaks), env hydration, providers, scrubbing, backends
  hooks/                     → lifecycle hook execution (pre_save, post_restore, etc.)
  policy/                    → garbage collection (retention tiers, blob pruning)
  watcher/                   → file-system watcher for auto-checkpointing
```

### Key Design Decisions

1. **Standard OCI media types** - All layers use `application/vnd.oci.image.layer.v1.tar+gzip` so `docker pull`, `COPY --from`, and containerd work natively. Layer semantics are carried by `org.opencontainers.image.title` annotations.

2. **Shared blob store** - Local OCI store at `~/.bento/store/` uses a shared content-addressed blob pool. Identical layers across workspaces are stored once.

3. **Composable extensions** - Each agent framework, language, and tool gets a small extension that contributes patterns to the right layer. Extensions auto-detect and merge. No monolithic harnesses.

4. **Three core layers** - deps (rarely changes, large), agent (changes often, small), project (catch-all). Extensions can add more layers (e.g., `build-cache`).

5. **Secret safety** - Pre-save scanning via [gitleaks](https://github.com/zricethezav/gitleaks) (~200+ rules), automatic scrubbing of detected secrets (replaced with unique placeholders in OCI layers), secrets stored locally and encrypted with a one-time key per checkpoint, `.gitleaksignore` for false positives, SHA256 scan cache, credential file exclusion. Key is shown at push/export time, not save time. See `specs/secret-scrubbing.md` for the full design.

## Extension System

Extensions are the core abstraction. Each extension has one concern and implements three methods:

```go
type Extension interface {
    Name() string                          // e.g., "claude-code", "node", "python"
    Detect(workDir string) bool            // check if relevant to this workspace
    Contribute(workDir string) Contribution // return patterns, ignore, hooks
}

type Contribution struct {
    Layers      map[string][]string // layer name → patterns to add
    ExtraLayers []LayerDef          // new layers (e.g., "build-cache")
    Ignore      []string            // patterns to exclude
    Hooks       map[string]string   // default lifecycle hooks
}
```

On every `save`/`diff`/`watch`, all extensions are auto-detected, their contributions are merged into a unified set of layer definitions, and the workspace is scanned against those patterns.

### Built-in Extensions

**Agent extensions** (contribute to `agent` layer):

| Extension | Detects | Patterns |
|-----------|---------|----------|
| `claude-code` | `.claude/` or `CLAUDE.md` | `CLAUDE.md`, `CLAUDE.local.md`, `.claude/**`, `.mcp.json`, `.worktreeinclude`, `~/.claude/projects/<path-with-dashes>/`, `~/.claude/{CLAUDE.md, settings.json, keybindings.json, rules/, skills/, commands/, agents/, agent-memory/, output-styles/}`, `~/.claude.json` |
| `claude-cowork` | Cowork sessions referencing workspace | Session metadata JSON, session workspace dirs (`.claude/`, `audit.jsonl`, `outputs/`, `uploads/`), `cowork_settings.json`, `spaces.json`, `cowork_plugins/` under `~/Library/Application Support/Claude/local-agent-mode-sessions/<user>/<org>/` |
| `codex` | `.codex/` | `.codex/**`, `~/.codex/sessions/` (workspace-scoped), `~/.codex/state_N.sqlite` (global), `~/.codex/memories/`, `~/.codex/{AGENTS.md, config.yaml, config.json}` |
| `opencode` | `.opencode/` or `opencode.json` | `.opencode/**`, `opencode.json`, `~/.local/share/opencode/opencode.db` (global), `~/.local/share/opencode/storage/` (legacy), `~/.config/opencode/commands/`, `~/.opencode/commands/` |
| `openclaw` | `SOUL.md` or `IDENTITY.md` | `SOUL.md`, `MEMORY.md`, `memory/**`, `skills/**`, `canvas/**`, `~/.openclaw/openclaw.json`, `~/.openclaw/agents/<id>/sessions/`, `~/.openclaw/workspace/skills/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kajogo777/bento](https://github.com/kajogo777/bento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
