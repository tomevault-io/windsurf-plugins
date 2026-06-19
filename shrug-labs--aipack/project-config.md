---
trigger: always_on
description: aipack is a package manager for AI agent knowledge. Packs contain rules, skills, workflows, agents, prompts, MCP configs, and settings. A sync engine renders pack content to any supported harness (Claude Code, OpenCode, Codex, Cline) via profiles. Go 1.25+ module.
---

# aipack

aipack is a package manager for AI agent knowledge. Packs contain rules, skills, workflows, agents, prompts, MCP configs, and settings. A sync engine renders pack content to any supported harness (Claude Code, OpenCode, Codex, Cline) via profiles. Go 1.25+ module.

## Commands

```bash
make build          # Build for current platform → dist/
make test           # Run all Go tests
make lint           # go vet + staticcheck + go fix (applies fixes in-place)
make fmt            # go fmt ./...
make fmt-check      # Fail if source is unformatted
make install        # Build + copy to ~/.local/bin
make dist           # Cross-compile all platforms (darwin, linux, windows)

# Single test
go test ./internal/app/ -run TestSyncAndApply -v

# Single package
go test ./internal/engine/...
```

`VERSION` is the source of truth for the release line. Injected via ldflags at build (`-X main.version`, `-X main.commit`). Full release process in `RELEASING.md`.

## Architecture

Three layers enforced by `cmd/aipack/architecture_test.go`:

| Layer | Path | Role |
|-------|------|------|
| CLI | `cmd/aipack/` | Kong adapters — parse flags, delegate to app |
| Service | `internal/app/` | Request → Run → Result. No CLI deps, no I/O globals |
| Domain | `internal/` (config, domain, engine, harness, render) | Business logic, no upward imports |

Import rules (test-enforced, violations fail `go test`):
- `internal/` NEVER imports `cmd/`
- `harness/` and `render/` NEVER import `config/` (depend on domain + engine only)
- No imports of deleted v1 packages (full blocklist in `architecture_test.go`)

## Key abstractions

Services in `internal/app/` follow a Request → Run → Result pattern with injected I/O — no global state. See the existing services for the shape.

Sync produces a **Plan** by accumulating **Fragments** from each harness adapter. Settings and MCP are separate plan vectors with different gating behavior — details in `internal/harness/AGENTS.md`.

Four harness adapters (`claudecode`, `opencode`, `codex`, `cline`) handle both forward sync (Plan) and reverse save (Capture). Scope branching is per-harness. Interface contract and patterns: `internal/harness/AGENTS.md`.

## Content model

A pack contains these content vectors, all auto-discovered from standard directories when not explicitly listed in the manifest:

| Vector | Directory | Discovery | Extension |
|--------|-----------|-----------|-----------|
| Rules | `rules/` | `*.md` | `.md` |
| Agents | `agents/` | `*.md` | `.md` |
| Workflows | `workflows/` | `*.md` | `.md` |
| Skills | `skills/` | `*/SKILL.md` | dir-based |
| Prompts | `prompts/` | `*.md` | `.md` |
| Profiles | `profiles/` | `*.yaml` | `.yaml` |
| Registries | `registries/` | `*.yaml` | `.yaml` |
| MCP | `mcp/` | `*.json` (via manifest) | `.json` |
| Hooks | `hooks/` | `*/HOOK.yaml` | dir-based |
| Configs | `configs/` | per-harness settings and drop-ins (via manifest) | varies |
| Extras | — | via manifest `extras` field | arbitrary |

Manifest fields for all vectors are **ID-based** (e.g., `"rules": ["anti-slop"]`, `"profiles": ["dev"]`). Extras are the exception — they use relative paths because they can reference files outside standard directories.

### Bundled content and `--with`

Core content (rules, skills, workflows, agents, prompts, hooks, MCP, configs) is always installed. Profiles, registries, and extras are **bundled content** — gated by `--with` (`-w`). Remote installs without `--with` preview bundled content then strip it from the installed pack. `WithSet` in code tracks which categories are approved; `applyWithFilter` in `pack_extract.go` removes unapproved files and updates the manifest.

### Content extraction

Remote installs (clone and HTTP tarball) produce clean content-only packs. `extractPackContent` in `pack_extract.go` copies only standard content directories and declared extras from the clone into a staging directory, then atomically moves it into the packs directory. The clone is discarded.

### Multi-pack settings

Any pack with harness config files (`configs/` directory) contributes base settings automatically. Multiple packs' settings are deep-merged in profile order (first pack wins at leaf conflicts). Set `settings.enabled: false` on a pack entry to opt out.

### Pack root references

MCP server definitions can use `{pack:root}` to reference extras — scripts, data files, binaries — bundled with the pack. Resolved at sync time to the installed pack's absolute path. Expansion order: `{pack:root}` → `{params.*}` → `{env:*}`.

## Conventions

- Wrap errors with `%w` — always preserve context
- Exit codes: `cmdutil.ExitOK` (0), `ExitFail` (1), `ExitUsage` (2)
- Tests: `t.Parallel()` where safe, `t.TempDir()` for isolation, NEVER `t.Parallel()` with `t.Setenv()` or `t.Chdir()`
- Use `make fmt` (not raw `gofmt -w`) for formatting
- `--skip-settings` skips settings only; MCP configs and plugins always sync
- Version injected via ldflags at build time (`-X main.version`, `-X main.commit`)
- All commits require `Signed-off-by` — use `git commit --signoff`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shrug-labs/aipack](https://github.com/shrug-labs/aipack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
