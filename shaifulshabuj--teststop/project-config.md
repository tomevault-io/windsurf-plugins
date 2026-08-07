---
trigger: always_on
description: > teststop is an agent-native CLI tool that triggers AI to test any system the way a real adversarial user would break it. Zero configuration. Any language. Self-reducing.
---

# GitHub Copilot Instructions — teststop

> teststop is an agent-native CLI tool that triggers AI to test any system the way a real adversarial user would break it. Zero configuration. Any language. Self-reducing.

## Core Philosophy (6 Non-Negotiables)

1. **ZERO CONFIGURATION** — `teststop run` works with no setup on any project
2. **UNIVERSAL** — any language, any age, any system type
3. **SELF-REDUCING** — tests reduce over time. Not grow.
4. **AGENT-NATIVE** — JSON output for AI agents by default
5. **NO NEW LOOP** — never becomes a maintenance burden
6. **EXIT CONDITION** — success = the user needs it less, not more

## Technology Stack

- **Language:** Go (single binary, cross-platform, `CGO_ENABLED=0`)
- **CLI:** Cobra (`github.com/spf13/cobra`)
- **AI:** `os/exec` — shells out to `claude` or `copilot` CLI. **No SDK. No API keys. Ever.**
- **Sandbox:** Apple Container (`container` CLI) — AI runs inside isolated linux/arm64 VM
- **Embed:** `//go:embed` for mandate file (no external deps at runtime)
- **Memory:** JSON files in `.teststop/` (human-readable, version-controllable)
- **Build:** `CGO_ENABLED=0` for cross-platform binaries

## Package Structure

```
cmd/teststop/     → CLI entry point only (calls Execute())
internal/cli/     → Cobra command handlers (run, status, memory, report, mandate)
internal/reader/  → Static codebase scanner (scanner, detector, analyzer, types)
internal/mandate/ → Mandate composer (injects context + memory into base.md)
internal/sandbox/ → Container isolation layer (detector, runner, types)
internal/ai/      → AI adapter (adapter interface + Detect(), claudecli, copilotcli)
internal/memory/  → Confidence persistence (store, confidence, retire)
internal/reporter/→ Output (json, text, markdown, types)
pkg/scenario/     → Scenario types — STABLE CONTRACT
mandate/          → base.md (THE KEY FILE) + embed.go
Dockerfile.agent  → Minimal runtime image (AI CLIs only, no dev tools)
Dockerfile.dev    → Full dev environment (Go + CLIs + gh, for coding agent)
```

## Build & Test Commands

```bash
go build ./...        # Must pass on every commit
go test ./...         # Must pass on every commit
go run ./cmd/teststop # Local CLI run
go mod tidy           # Keep go.sum in sync
go vet ./...          # Static analysis
```

## Critical File: mandate/base.md

This is **the most important file**. It is the adversarial user instruction sent to AI. Everything else serves this file. When in doubt, improve the mandate before improving the code.

## Key Constants

```go
// internal/memory/confidence.go
RetirementThreshold = 0.95   // retire when confidence >= this
PassWeight          = 0.19   // 15 passes → retirement (math: 15 * 0.19 = 0.9576 > 0.95)
FailPenalty         = 0.30   // significant drop on failure
```

## Exit Codes

```
0 = confidence threshold met (safe to deploy)
1 = below threshold (review required)
2 = critical failures (do NOT deploy)
3 = teststop internal error
```

## Environment Variables

```bash
TESTSTOP_CLI=auto      # auto | claude | copilot | ollama  (default: auto-detect)
TESTSTOP_MODEL=        # optional — passed as --model to claude CLI
TESTSTOP_SANDBOX=auto  # auto | required | none
                       #   auto     = use container if available, else run AI direct
                       #   required = error if Apple Container not running
                       #   none     = always run AI CLI directly (CI, non-macOS)
```

**No API keys. No SDK.** teststop calls `claude -p "..."` or `copilot -p "..." -s --no-ask-user`.

## AI CLI — Non-Interactive Usage

teststop uses the **non-interactive (programmatic)** mode of each CLI:

**Claude Code CLI** (`claude`):
```bash
claude -p "<mandate>"                        # outputs response to stdout, exits
claude -p "<mandate>" --model claude-opus-4-5  # with specific model
```
- `-p` = print mode (non-interactive, no TUI, stdout only)
- Exit code 0 on success, non-zero on error
- Response is the raw AI output (teststop parses JSON from it)

**GitHub Copilot CLI** (`copilot`):
```bash
copilot -p "<mandate>" -s --no-ask-user
```
- `-p` = prompt (non-interactive)
- `-s` = silent/clean output (no spinner, no formatting)
- `--no-ask-user` = no clarifying questions (fully automated)

## Container Isolation (internal/sandbox/)

teststop runs the AI CLI inside an Apple Container VM when available. The AI is sandboxed — it cannot access the user's host filesystem.

**Runtime pipeline:**
```
teststop run
  → sandbox.Detect()               # checks: container system status = running?
  → ai.GenerateScenarios(mandate)
       ├─ [sandbox=auto, available] → container run --rm teststop-agent:latest claude -p "..."
       └─ [sandbox=none / fallback] → exec.Command("claude", "-p", mandate)
```

**Permission map:**

| Layer | Context | What AI can access |
|---|---|---|
| Dev container | Coding agent building teststop | R/W `/workspace` (repo), read-only `~/.claude`, `~/.config/gh` |
| Runtime container | teststop running AI for scenarios | v0.1: none (mandate passed as arg); v0.2: project read-only |
| Direct fallback | No container available | Full host user permissions |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shaifulshabuj/teststop](https://github.com/shaifulshabuj/teststop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
