---
trigger: always_on
description: This file is for agents working *on* this repo (the SDKs in `go/`, `python/`, `js/`, `java/`, `dotnet/` and the launchers in `plugins/`).
---

# Working on agento11y

This file is for agents working *on* this repo (the SDKs in `go/`, `python/`, `js/`, `java/`, `dotnet/` and the launchers in `plugins/`).

For agents working in a *consumer* project (instrumenting their app, or installing one of our plugins), point them at [`llms.txt`](llms.txt) instead. That is the file we ship to users.

Read the README and `mise tasks` for the obvious stuff: layout, package names, where languages live. This file only documents what you can't discover by looking.

## Proto is the source of truth

`proto/agento11y/v1/*.proto` is the source of truth. Generated stubs live under each language tree:

- Go: `go/proto/agento11y/`
- Python: `python/agento11y/internal/gen/agento11y/`
- JS: `js/proto/agento11y/` (the runtime loads `.proto` files directly, no codegen)
- Java, .NET: compiled on build via the gradle protobuf plugin and `Grpc.Tools`; no committed stubs.

Never edit generated files. Edit the `.proto`, then:

```sh
mise run generate:proto
```

CI runs `mise run check:proto` and fails the build if the committed stubs drift from the proto. Tool versions are pinned in `mise.toml` so output is byte-identical across machines. See `docs/development.md` for the full table.

## Secret patterns are the second source of truth

`redaction/patterns.json` is the only editable secret-pattern table. Each engine gets its list generated from it, next to hand-written code: `go/agento11y/redaction_patterns_gen.go`, `python/agento11y/_redaction_patterns.py`, `js/src/redaction-patterns.generated.ts`, `dotnet/src/Grafana.Agento11y/RedactionPatterns.g.cs`, `plugins/agento11y/internal/redact/patterns_gen.go`.

Never edit those five files. Edit `patterns.json`, then:

```sh
mise run generate:redaction
```

CI runs `mise run check:redaction` in the same job as the proto drift check. `redaction/README.md` covers the pattern fields and the shared fixtures.

## Releases run off `.github/sdk-releases.json`

That table holds each release line's id, tag prefix, changelog path, and the commit paths its changelog section is generated from. Seven lines are listed: the five SDKs plus `plugins/pi` and `plugins/opencode`.

The commit paths live only there, and every release workflow reads them with `jq`. Tag prefixes and changelog paths are duplicated: the five SDK workflows still spell out their own `sdk-python/v*`-style prefix and `python/CHANGELOG.md`-style path inline, and `sdk-github-releases.yml` hand-copies all seven prefixes into `on.push.tags`, which GitHub cannot template from a file. So a new release line needs a row *and* a pass over those workflows; a row on its own gets the line tagged with no release page.

Each SDK row carries `:(exclude)` pathspecs for tests and READMEs, because a conformance test under `go/` would otherwise put a JS-only commit in the Go changelog. The two plugin rows own their whole directory and need no excludes.

Three steps run per release, and none of them creates a tag on the release PR:

1. The release workflow generates a section with `changelog-for-release.sh` and opens a PR that changes `CHANGELOG.md`.
2. `tag-releases-on-merge.yml` fires on the merge, reads the top version with `changelog-top-version.sh`, and tags the merge commit, so every tag stays reachable from `main`.
3. `sdk-github-releases.yml` fires on the tag and publishes the section from `changelog-latest-section.sh` as the release body.

`plugins/agento11y` sits outside the table. It keeps its own pair of workflows (`release-agento11y.yml`, `tag-agento11y-on-merge.yml`) because it also drives GoReleaser and Homebrew.

## Workspace gotchas

- The Go workspace (`go.work`) covers `go/`, `go-providers/*`, `go-frameworks/google-adk`, and `plugins/agento11y`. Adding a new Go module means updating `go.work` *and* `go.work.sum`. Lint tasks use `GOWORK=off` and iterate per-module via `find . -name go.mod`, so each module must also lint and build on its own.
- The pnpm workspace covers `js/` and `plugins/*`. Use `pnpm --filter <name>` from the root; `mise.toml` does this via tasks like `lint:ts:sdk-js`.
- When adding a JS workspace package, plugin, or private example, update `js/scripts/check-js-dependency-pinning.mjs` so dependency pinning enforcement covers the new manifest. Published packages should keep runtime `dependencies` and `peerDependencies` as compatible ranges, but pin `devDependencies`; private examples should pin external dependencies exactly.
- Java uses a single gradle multi-project rooted at `java/`; modules are listed in `java/settings.gradle.kts`.
- .NET uses a single solution at `dotnet/Agento11y.DotNet.slnx`; projects are listed there.

## Plugins layout

`plugins/` ships two flavors of launcher. They are not uniform; don't assume they are.

| Plugin dir | What it actually is |
|------------|---------------------|
| `plugins/agento11y/` | The shared Go binary, installed as `agento11y` (`brew install grafana/grafana/agento11y`; the old `sigil` name still works but will be removed). Has subcommands `claude`, `codex`, `copilot`, `cursor`, `opencode`, `pi`, `vibe`, `login`, `doctor`, `local`. This is also what consumers use. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grafana/agento11y](https://github.com/grafana/agento11y) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
