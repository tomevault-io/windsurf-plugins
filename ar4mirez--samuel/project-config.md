---
trigger: always_on
description: AI-assisted development instructions for the Samuel CLI project.
---

# CLAUDE.md

AI-assisted development instructions for the Samuel CLI project.

> **AGENTS.md Compatible**: A copy exists as `AGENTS.md` for cross-tool compatibility (Cursor, Codex, Copilot, etc.)

---

## Operations

### Setup Commands

```bash
git clone https://github.com/ar4mirez/samuel.git && cd samuel
go mod download
make build            # → ./bin/samuel
./bin/samuel version   # Verify installation
```

### Testing Commands

```bash
make test             # Run all tests (verbose, race detection, coverage)
go test ./...         # Quick run
go test -cover ./...  # With coverage
go test -race ./...   # Race detection only
```

### Build & Lint Commands

```bash
make build            # Build binary to ./bin/samuel
go build ./...        # Build all packages
go vet ./...          # Static analysis
golangci-lint run     # Comprehensive lint
go fmt ./...          # Format code
```

### Documentation

```bash
pip install -r requirements-docs.txt
mkdocs serve          # Local preview at http://127.0.0.1:8000
mkdocs build          # Build static site to ./site/
```

---

## Boundaries (Do Not Touch)

### Protected Files
- `go.sum` (dependency lock)
- `.github/workflows/` (CI/CD configurations)
- `.goreleaser.yaml` (release automation)
- `go.mod` (without testing build)

### Never Commit
- Secrets, API keys, credentials, tokens
- `.env` files
- `bin/`, build artifacts, the root `samuel` binary
- `.claude/settings.local.json` (personal settings)

### Ask Before Modifying
- Template files in `template/` (affects all users)
- Public CLI command interfaces (breaks user scripts)
- Registry data in `internal/core/registry.go` (component definitions)
- Build/deploy processes (Makefile, .goreleaser.yaml)

---

## Project Context

### Tech Stack
- **Language**: Go 1.21+
- **CLI Framework**: Cobra
- **UI**: fatih/color, manifoldco/promptui, schollz/progressbar
- **Config**: gopkg.in/yaml.v3
- **HTTP**: net/http (standard library)
- **Archive**: archive/tar, compress/gzip (standard library)
- **Docs**: MkDocs with Material theme (Python)

### Architecture

```
samuel/
├── cmd/samuel/              # Entry point (main.go) — minimal
├── internal/
│   ├── commands/            # 14 CLI commands (init, update, add, remove, list, doctor, version, search, info, config, diff, skill, auto, sync)
│   ├── core/                # Business logic (config, registry, extractor, skill, auto)
│   ├── github/              # GitHub API client (releases, archives)
│   ├── orchestrator/        # v4 component lifecycle (Component interface, structured Error, flock concurrent init lock) — internal foundation, no CLI surface yet
│   └── ui/                  # User interface helpers (prompts, spinner, colors)
├── template/                # Distributable template files
│   ├── CLAUDE.md            # Template AI instructions
│   ├── AGENTS.md            # Cross-tool copy
│   └── .claude/skills/      # 21 language + 33 framework + 25 workflow/utility skills
├── docs/                    # MkDocs documentation source
├── Makefile                 # Build targets (build, test, lint, install)
└── .goreleaser.yaml         # Release automation
```

**Core Flow**: User command → GitHub download → Tar extraction → Local files
**Caching**: Downloaded versions cached in `~/.cache/samuel/`
**Config**: Project config in `samuel.yaml`

### Key Design Decisions

- **template/ directory**: Separates distributable files from project source. CLI paths use `template/` prefix for extraction.
- **GitHub archive downloads**: Uses GitHub's tar.gz API — no git required on user machines.
- **Standard Go layout**: `cmd/` + `internal/` at root. `internal/commands/` (not `cmd/`) to avoid confusion.
- **Go for CLI**: Single binary, cross-platform, no runtime dependencies. Cobra for CLI framework.
- **Auto (Ralph Wiggum)**: Autonomous AI coding loop. Tasks stored in `.claude/auto/prd.json`, progress in `progress.md`, orchestrated by Go-native loop. Supports PRD-based and zero-setup pilot modes.

---

## Quick Reference

**Task Classification:**
- **ATOMIC** (<5 files, clear scope) → Implement directly
- **FEATURE** (5-10 files) → Break into subtasks
- **COMPLEX** (>10 files, new subsystem) → Use PRD workflow | .claude/skills/create-prd/SKILL.md

**Common Guardrails** (validate first):
- Function ≤50 lines | File ≤300 lines | Input validation | Parameterized queries
- Tests >80% (critical) | Conventional commits | No secrets in code

**Emergency Quick Links:**
- Security issue? → .claude/skills/security-audit/SKILL.md
- Tests failing? → .claude/skills/troubleshooting/SKILL.md
- Stuck >30 min? → .claude/skills/troubleshooting/SKILL.md
- Complex feature? → .claude/skills/create-prd/SKILL.md
- Code review? → .claude/skills/code-review/SKILL.md
- Go-specific? → .claude/skills/go-guide/SKILL.md

**Skills** (capability modules - [Agent Skills](https://agentskills.io) standard):
- Create: `samuel skill create <name>` or `.claude/skills/create-skill/SKILL.md`
- Validate: `samuel skill validate`
- List: `samuel skill list`
- Load: `.claude/skills/<skill-name>/SKILL.md` when task matches description

**Autonomous Mode (Ralph Wiggum methodology):**

- Initialize: `samuel auto init --prd .claude/tasks/NNNN-prd-feature.md`
- Start loop: `samuel auto start`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ar4mirez/samuel](https://github.com/ar4mirez/samuel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
