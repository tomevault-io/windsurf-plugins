---
trigger: always_on
description: generates the hook commands. The scripts in `src/shell/scripts/` are embedded and templated at
---

# Agent Instructions

General coding guidelines, commit conventions, and agent workflows for this repository.

## Project Overview

Oh My Posh is a cross-shell prompt theme engine written in Go. It renders prompt segments by
querying an `Environment` abstraction that wraps all OS/shell interactions.

## Tech Stack

| Layer                     | Technology                    |
| ------------------------- | ----------------------------- |
| Core engine               | Go (module root: `src/`)      |
| Documentation site        | Docusaurus (MDX) - `website/` |
| Themes                    | JSON - `themes/`              |
| Config format             | TOML / JSON / YAML            |
| Package/installer scripts | `packages/`                   |
| Build scripts             | `build/`                      |

## Key Commands

```bash
# Go - run from src/
go test ./...
go test ./segments/... -run TestFoo  # single test
golangci-lint run

# Docs - run from website/
npm run start    # local dev server
npm run build    # validate before opening a docs PR
```

## Codebase Exploration

**Always explore the actual codebase before planning or writing code.** Do not rely on memory
or assumptions. Use the file system tools to read relevant files first - the codebase evolves
and the feature you're asked to add may already exist.

## Repository Layout

```text
src/
  segments/   # One Go file + one _test.go per segment
  prompt/     # Core rendering engine
  runtime/    # OS/shell abstraction layer
themes/       # Bundled JSON theme files
website/      # Docusaurus docs site (MDX pages, sidebar config, JSON schema)
packages/     # Installer/package manifests
build/        # CI build helpers
```

Key paths inside `src/`:

| Path                          | Purpose                                             |
| ----------------------------- | --------------------------------------------------- |
| `src/segments/`               | One `.go` + one `_test.go` per segment              |
| `src/config/segment_types.go` | Segment type registry (gob + string constants)      |
| `src/cli/`                    | CLI commands (Cobra); `root.go` is the entry point  |
| `src/prompt/engine.go`        | Segment rendering loop                              |
| `src/cache/`                  | Existing TTL/file/command-path cache infrastructure |
| `src/runtime/`                | `Environment` abstraction + mock                    |

## Segment Development

Every segment lives in `src/segments/` and implements the `SegmentWriter` interface. Use the
`Environment` abstraction (`env`) for **all** OS/shell calls - never call OS APIs directly.

Adding a segment requires **five** artifacts - use the `segment-create` skill to scaffold all
of them automatically:

1. `src/segments/<name>.go` - segment source
2. `src/segments/<name>_test.go` - unit tests
3. `website/docs/segments/<name>.mdx` - user-facing docs
4. Update `website/sidebars.js` and `website/static/schema.json`
5. Register the type in `src/config/segment_types.go` via `gob.Register(&segments.MySegment{})`

Missing step 5 causes the segment to fail silently at runtime.

See the `segment-docs` skill for the canonical mapping between Go source constructs and MDX
documentation fields (template properties, type representations, option tables).

## Shell Integration

`oh-my-posh init <shell>` is how users wire oh-my-posh into their shell. It:

1. Writes a shell-specific init script to the cache (source: `src/shell/scripts/omp.<ext>`)
2. Returns a one-liner for the shell to `eval` - this sources the cached script, which hooks
   into prompt rendering

The `src/shell/` package contains per-shell logic (`pwsh.go`, `bash.go`, `zsh.go`, etc.) that
generates the hook commands. The scripts in `src/shell/scripts/` are embedded and templated at
init time. When modifying shell behaviour, changes typically span both the `.go` file and the
corresponding script.

Supported shells: `bash`, `zsh`, `fish`, `powershell`/`pwsh`, `cmd`, `nu`, `elvish`, `xonsh`.

## CLI Commands

CLI commands use [Cobra](https://github.com/spf13/cobra) and live in `src/cli/`. To add a new
command:

1. Create `src/cli/<name>.go` with a `var <name>Cmd = &cobra.Command{...}`
2. Register it in `src/cli/root.go` via `RootCmd.AddCommand(<name>Cmd)`

## Caching

`src/cache/` provides the existing caching infrastructure - use it instead of building new
cache logic. It supports TTL-based key/value storage, file-based persistence, and command-path
caching. Do not introduce new cache packages unless `src/cache/` genuinely cannot meet the
requirement.

## Go Conventions

Follow the `golang` skill for project-specific Go standards.

## Documentation (website/)

- Follow the `markdown` skill for `.md`/`.mdx` formatting rules.
- Segment doc pages live in `website/docs/segments/` and use MDX frontmatter with `title`, `sidebar_label`, and `id`.

## PowerShell

PowerShell helper scripts live in `packages/` and `build/`. Follow the `powershell` skill for cmdlet conventions.

## Themes

Themes are plain JSON files in `themes/`. New themes must validate against
`website/static/schema.json`. Do not introduce breaking schema changes without updating the
schema file.

## Skills


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JanDeDobbeleer/oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
