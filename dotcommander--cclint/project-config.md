---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Build & Run

```bash
#-Build and symlink to ~/go/bin
go build -o cclint . && ln -sf $(pwd)/cclint ~/go/bin/cclint

#-Build with version override
go build -ldflags "-X github.com/dotcommander/cclint/cmd.Version=1.0.0" -o cclint .

#-Version
cclint --version              # show version (defaults to "dev")
cclint -V                     # short form

#-Run linter (defaults to ~/.claude)
cclint                        # lint all component types
cclint agents                 # lint only agents
cclint commands               # lint only commands
cclint agents commands        # lint multiple types
cclint skills                 # lint only skills
cclint settings               # lint only settings.json files
cclint rules                  # lint only rule files
cclint context                # lint only CLAUDE.md files
cclint plugins                # lint only plugin manifests
cclint output-styles          # lint only output style files
cclint summary                # show quality summary
cclint fmt                    # format component files
cclint fmt --write            # format and write in place

#-File and directory mode
cclint ./agents/my-agent.md            # lint one file (auto-detect type)
cclint path/to/file.md                 # lint by path
cclint a.md b.md c.md                  # lint multiple files
cclint ./commands/                     # lint all files in a directory
cclint ./command/                      # singular dir names auto-detected
cclint --type agent ./custom/file.md   # override type detection

#-Git integration (pre-commit hooks)
cclint --staged                        # lint only staged files
cclint --diff                          # lint all uncommitted changes

#-Baseline mode (gradual adoption)
cclint --baseline-create                  # create baseline from current issues
cclint --baseline                         # lint with baseline filtering (only new issues fail)
cclint --baseline-path custom.json        # use custom baseline file path

#-Common flags
cclint --root /path/to/project agents    # specify project root
cclint --scores agents                    # show quality scores (0-100)
cclint --improvements agents              # show improvement recommendations
cclint --format json --output report.json # JSON output for CI
cclint --quiet                            # errors only, no suggestions
cclint --verbose                          # detailed processing info
```

## Testing

```bash
go test ./...                             # run all tests
go test ./internal/scoring/...            # test specific package
```

## Architecture

```
cmd/                    # Cobra commands (root, agents, commands, skills, etc.)
internal/
├── discovery/          # File discovery using doublestar glob patterns
├── types/              # Shared types: ValidationError, constants
├── textutil/           # Shared text utilities: KnownTools, frontmatter parser
├── cue/
│   ├── validator.go    # CUE-based schema validation
│   └── schemas/        # Embedded CUE schemas (agent, command, settings, claude_md)
├── baseline/           # Baseline support for gradual adoption
│   ├── baseline.go     # Baseline creation, loading, filtering
│   └── baseline_test.go
├── scoring/            # Quality scoring (0-100) with tier grading (A-F)
│   ├── types.go        # QualityScore, ScoringMetric interfaces
│   ├── agent_scorer.go
│   ├── command_scorer.go
│   ├── skill_scorer.go
│   └── plugin_scorer.go
├── lint/               # Lint orchestration per component type
│   ├── crossfile/         # Cross-file validation
│   │   ├── crossfile.go   # Reference validation, orphan detection
│   │   ├── triggers.go    # Ghost trigger detection in reference files
│   │   ├── refs.go        # Reference extraction helpers
│   │   └── graph.go       # Cycle detection
│   └── baseline_filter.go  # Baseline filtering logic
├── output/             # Formatters (console, json, markdown)
├── outputters/         # Output coordination
├── config/             # Viper-based config (.cclintrc.json/.yaml)
└── project/            # Project root detection
schemas/                # Root-level CUE schemas (duplicated in internal/cue/schemas)
```

## Key Patterns

**Validation Pipeline**: Discovery → Frontmatter Parse → CUE Schema Validation → Go-based Best Practice Checks → Scoring → Output

**CUE Schemas**: Embedded via `//go:embed` in `internal/cue/validator.go`. Validate frontmatter structure for agents, commands, settings.

**Quality Scoring**: Four categories (structural 0-40, practices 0-40, composition 0-10, documentation 0-10) summed to 0-100. Tier grades: A≥85, B≥70, C≥50, D≥30, F<30.

**Discovery Patterns**: Searches `.claude/{agents,commands,skills}/**/*.md`, `{agents,commands,skills}/**/*.md`, and `**/.claude-plugin/plugin.json` paths. Lowercase `skill.md` files are also discovered and flagged as naming errors.

## Lint Rules Enforced

| Component | Size Limit | Key Checks |
|-----------|------------|------------|
| Agent | 200 lines | name format, model, triggers, Foundation/Workflow sections, Skill() loading, body-tool mismatch (suppressed for 8+ tools) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dotcommander/cclint](https://github.com/dotcommander/cclint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
