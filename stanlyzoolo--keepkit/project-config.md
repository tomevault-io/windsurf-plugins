---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
go build .          # build binary
go install .        # install to ~/go/bin/keepkit
go run .            # run without installing
go test -race ./... # run all tests (the version package has real mutex-guarded state — keep -race)
go vet ./...        # static analysis
golangci-lint run   # lint (config in .golangci.yml; requires golangci-lint v2)
```

CI (`.github/workflows/ci.yml`) runs build / vet / `test -race` / golangci-lint on every push and PR to `main`, plus a **cross-compile step** (`GOOS=windows go build`+`vet`, `GOOS=darwin go build`): the runner is linux-only, so without it the build-tagged files for the other platforms (`restart_windows.go`, `restart_unix.go`, `internal/proc/detach_windows.go`) would be compiled by no automated check at all. Release is triggered by pushing a `v*` tag; GitHub Actions builds for darwin/linux/windows via `.github/workflows/release.yml`, which runs GoReleaser off `.goreleaser.yaml`, injecting the tag with `-ldflags "-X main.version=<tag>"`. **The release body assembles itself from commit subjects**, so a `feat:`/`fix:` subject is user-facing copy, not a note to yourself — it is what appears under *New features* / *Bug fixes* on the release page and in keepkit's own changelog block in panel `[2]`. `.goreleaser.yaml`'s changelog filters drop `docs:`/`test:`/`chore:`/`ci:`/`build:`, merge commits and `go mod tidy`; **anything else stays visible** under *Other changes* — `refactor:`, `perf:`, or an off-nomenclature subject like `rebrand:`. **Both the group regexps and the excludes match the bare commit subject, so every pattern anchors at `^`** — grouping runs before an entry is formatted and therefore never sees the SHA prefix the rendered line carries. A leading `^.*?` on a group is actively wrong rather than merely redundant: it degrades the pattern into a substring match, and since `git revert` writes `Revert "feat: x"` by default, a reverted feature then gets announced as a new one — at the very top, because `sort: asc` compares bytes and `R` precedes `f`. `fixup!`/`squash!` subjects leak the same way. Note the excludes are bare prefixes, not `prefix:` matches, so a subject starting `testutil…` or `circular…` would also be dropped; tightening them to `^ci(\(.+\))?!?:` is available if that ever bites. `main.version` (default `"dev"`) exists so that ldflag actually takes effect — it seeds the session-log header, the `--version` output and — via `WithAppVersion` — the self-check gate, whose `dev` default is what keeps that feature off in a working copy. `buildVersion()` falls back to the module version from `debug.ReadBuildInfo()` (set by `go install …@<tag>` and VCS-stamped `go build`) when the ldflag wasn't injected; only a build with no usable buildinfo shows `dev`. Note what that fallback means since Go 1.24: a plain `go build .` from a checkout reports a **pseudo-version** (`v0.0.0-<ts>-<hash>`, `+dirty` with uncommitted changes), not `dev` — which is why the self-check gate rejects that shape too (see **Self-update**).

## Architecture

**`keepkit`** is a terminal TUI tracker for CLI tools built with Bubble Tea. It is a pure TUI app — running `keepkit` launches the interface directly; the only CLI surface is `--version`/`--help`.

**Deep design docs.** Three features carry more rationale than belongs in a file loaded on every session, so their full text lives under `docs/design/` and this file keeps the invariants plus a link. **Read the linked file before changing anything in its area** — each one records what already broke once and why the current shape is the way it is:

| File | Covers |
|---|---|
| [`docs/design/updating.md`](docs/design/updating.md) | `enter` in `[2]`: `updater.Detect`'s manager chain, the confirm dialog, the streaming update log |
| [`docs/design/self-update.md`](docs/design/self-update.md) | `U`/`X`: the version gate, `selfState`, the restart and its path resolution |
| [`docs/design/readme-pipeline.md`](docs/design/readme-pipeline.md) | panel `[3]`: the three sources, `cleanReadmeMarkdown`, the glamour theme |

Everything else stays here. **Never re-inline these three sections.** When a change makes one of them wrong, fix it in its own file, and touch this one only when the invariant summary itself went wrong — the split is the layout, not drift.

### Entry point


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stanlyzoolo/keepkit](https://github.com/stanlyzoolo/keepkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
