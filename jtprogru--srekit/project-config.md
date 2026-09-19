---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`srekit` is a single-binary Go CLI that generates SRE text artifacts (investigation log, postmortem, runbook, RFC/ADR, on-call report, SLO, error budget policy, changelog) plus one document that is not an SRE artifact at all — `tasker`, a card for a collection of engineering tasks — from templates compiled into the binary via `//go:embed`. It also *maintains* one artifact it did not necessarily generate: `changelog release` and `changelog validate` edit and lint an existing `CHANGELOG.md`. That is a genuine widening of what the tool is — every other write path renders a fresh document and either creates a file or refuses to — and the conservatism of `internal/changelog` follows from it. `tasker` widens the catalog in the other direction, past the rule that got `capacity`, `retro` and `license` removed in 0.30.0: the spec admits a task card explicitly, by name, and nothing else moved with it. Extracted from the [gch](https://github.com/jtprogru/gch) monolith. Pre-1.0 (0.30.x line). `capacity`, `retro` and `license` were removed in 0.30.0; `cmd/retired.go` keeps hidden stubs that explain the removal until 1.0.

## Commands

```bash
make                 # list every target (help is the default goal)
make ci              # lint + race tests — the one-shot pre-push check
make test            # go test --short -coverprofile=cover.out -v ./...
make test-race       # go test -race -coverprofile=cover.out -v ./...  (what CI runs)
make lint            # golangci-lint run at the pinned version, from ./bin
make govulncheck     # vulnerability scan (what CI runs)
make build           # CGO_ENABLED=0 go build -o ./dist/srekit .
make run ARGS="…"    # go run . <args>
make release-dry     # goreleaser snapshot build into ./dist, no publish
make docs-serve      # MkDocs at http://127.0.0.1:8000
make docs-build      # mkdocs build --strict
```

Single test / subset:

```bash
go test ./cmd/ -run TestSLO -v
go test ./internal/sections/ -run TestRenderArtifact -v
go test ./cmd/ -run TestTemplates -v      # run this whole suite when touching cmd/templates.go — the 3-way merge has subtle invariants
```

Toolchain must match CI or gosec taint rules produce false positives. Go **1.26.4** is on you; `golangci-lint` (**v2.12.2**, ~50 linters including `gochecknoglobals`) and `govulncheck` are pinned in the `Makefile` and installed into `./bin` by their targets, so `make lint` resolves to the same binary locally and on a runner — invoking a system-wide `golangci-lint` directly bypasses that pin.

The `Makefile` must stay compatible with **GNU Make 3.81** — that is the system `make` on macOS and Apple will not ship newer (GPLv3). No `.ONESHELL` / `.SHELLFLAGS` (3.82+), no `!=` / `$(file ...)` (4.0+), no `$(intcmp)` / `$(let)` (4.4+); one command per recipe line. A 4.x runner swallows the incompatibility silently, so verify locally. Every CI workflow except `goreleaser.yaml` calls a Make target rather than a raw command — a target's behaviour is CI's behaviour.

## Architecture

Flow for every generator command:

```
cmd/<name>.go                     flags → <name>Meta struct
  └─ loaderFrom(cmd)              *tmpl.Loader off cmd.Context()
  └─ loader.LoadArtifactBytes()   resolves <name>.yaml through Sources
  └─ sections.ParseArtifact()     → Artifact (structural validation)
  └─ sections.Merge()             defaults + --from overrides, template-evaluated
  └─ out.RenderOptions(cmd, defaultPath)
  └─ render.Render()              → sections.RenderArtifact() → markdown → writeBody
```

Key pieces:

- **`internal/tmpl`** — `Source` interface (`EmbedSource` for `//go:embed templates/*.yaml`, `DirSource{Dir}` for a user dir). `Loader{Sources}` walks them in order treating `fs.ErrNotExist` as fall-through, so a missing file in the user dir transparently falls back to embedded. Also the shared `template.FuncMap` (`default`, `shortID`, `slugify`, `upper`, `lower`, `trim`, `now`, `join`).
- **`internal/sections`** — the v1 artifact runtime. `Artifact` = `version` / `frontmatter` (a `yaml.Node`, so author key order survives parse→render) / `title` / `meta_bullets` / `header_body` / `sections`. Section types are `text` / `list` / `table`. A templated frontmatter scalar renders to a Go string, so it can only be emitted quoted; an explicit YAML tag from `retypedTags` (`!!int`, `!!seq`, …) makes the renderer re-read the rendered text as that type, and a mismatch is a render error naming the key. `!!str` and application tags are deliberately excluded — an explicit `!!str` is the untagged path, and somebody else's tag is theirs to interpret. `Merge` overlays per-section overrides; `RenderArtifact` composes the markdown (frontmatter block → H1 → meta bullets → header body → `## section` blocks).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jtprogru/srekit](https://github.com/jtprogru/srekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
