---
trigger: always_on
description: Drive ggen CLI. Generate zero-copy, zero-reflection JSON encode/decode for annotated Go structs. Use when user want faster JSON codec than `encoding/json`, validation baked into decode, compile-time-checked custom validators/transforms. Cover when invoke ggen, flag→intent map, annotation surface, regen-after-edit workflow.
---


# ggen — JSON codegen for Go

ggen parse annotated Go structs. Emit `DecodeFrom`, `DecodeFromStream`, `JSONSize`, `AppendJSON`. Generated code = hand-rolled byte scan. No reflection, no token layer. Bytes path (`DecodeFrom` over caller `[]byte`) strings alias input via `unsafe.String` — zero-copy. Stream path (`DecodeFromStream` over `*scan.Stream`) copy strings out of intermediate buffer so buffer compact safely. See _Stream is not zero-copy_ below.

Module: `github.com/sirkostya009/ggen`. Binary: `ggen`. Go ≥ 1.26.

## When to reach for ggen

Use ggen when ANY hold:

- Hot-path JSON decode/encode where `encoding/json` (v1 or v2) show in CPU/alloc profiles.
- Validation belong at decode time (length, range, regex-light patterns, custom `func(T) error`) — ggen fold into parser. Invalid payloads short-circuit before allocating full value.
- Long/slow streams + validation required AND invalid payloads frequent enough that fail-fast mid-body (vs finish read first) save real bandwidth/CPU.

Skip ggen when:

- Wire shape need `encoding/json` v1 quirks ggen diverge from (URL struct-dump, `sql.NullX` `{Valid:…}` wrapper).

## Install

```sh
go install github.com/sirkostya009/ggen@latest # CLI binary
go get github.com/sirkostya009/ggen            # runtime subpackages
```

## Invocation

```sh
ggen .                 # current package
ggen ./...             # every package matched by the pattern — module-scoped, same as `go build ./...`
ggen ./pkg/...         # subtree pattern (relative paths must start with `./`)
ggen <dir>             # one package
ggen <file.go>         # one file
ggen <file.go> Foo Bar # one file, only structs named Foo or Bar, will fully overwrite existing <file_ggen.go> file
```

Test-only packages (no non-`_test.go` files) skipped in pattern mode. Invoke `ggen <dir>` directly when target only has `_test.go` sources.

**Run ggen under same `GOEXPERIMENT` as user build** — `packages.Load` honor build tags. Files behind `goexperiment.jsonv2` invisible without it. Repos using jsonv2:

```sh
GOEXPERIMENT=jsonv2 ggen ./...
```

## Agent-mode output (do not truncate)

ggen auto-detect when driven by coding agent. Switch logger from pretty multi-line/ANSI-colored (humans) to concise one-line-per-record. Concise mode also fires under CI or non-TTY stderr.

Any non-empty value of these env vars enable:

- `AI_AGENT` (generic cross-vendor)
- `CLAUDECODE` (Anthropic's Claude Code)
- `CURSOR_TRACE_ID` (Cursor IDE)
- `AIDER_AUTO_COMMITS` (Aider)
- `CI` / `GITHUB_ACTIONS` / `GITLAB_CI` / `CIRCLECI` / `JENKINS_HOME` /
  `BUILDKITE` / `TRAVIS` / `APPVEYOR` / `TF_BUILD` /
  `TEAMCITY_VERSION` / `CONTINUOUS_INTEGRATION`
- non-TTY stderr (piped/redirected)

Each line self-contained, pattern: `<level>: [file:line:col:] <msg> [(hint)]`. Levels: `inf:` / `dbg:` / `trc:` / `err:`. Every line signal — **do not truncate** (`head` / `tail` / `grep -v`).

## Output file naming

- Package mode: `<dir-basename>_ggen.go` (and `_ggen_test.go` if annotated struct in `_test.go`).
- Single-file mode: `<basename>_ggen.go`.
- Source with `//go:build foo`: land in `<dir>_foo_ggen.go`, constraint preserved. Multi-term constraints get slugified filename (`//go:build foo && bar` → `<dir>_foo_bar_ggen.go`).
- `-o <path>` override path in single-file or single-package mode.

## Flags (global) and per-struct annotations (local)

Most flags have matching annotation token (no leading dash). Annotations space-separated after `//ggen:generate`.

| CLI flag         | annotation      | effect                                                                                                 |
| ---------------- | --------------- | ------------------------------------------------------------------------------------------------------ |
| `-o <path>`      | —               | override output path (single-file / single-package only)                                               |
| `-pkg <name>`    | —               | override the package name in the generated file                                                        |
| `-marshal`       | `marshal`       | also emit `MarshalJSON` so the type satisfies `encoding/json.Marshaler`                                |
| `-unmarshal`     | `unmarshal`     | also emit `UnmarshalJSON` for `encoding/json.Unmarshaler`                                              |
| `-multierr`      | `multierr`      | accumulate every validation failure into `validation.Errors` (slice) instead of returning on the first |
| `-allowdups`     | `allowdups`     | accept duplicate JSON keys, first-wins (default: error on second occurrence)                           |
| `-novalidate`    | `novalidate`    | drop validation, required-field checks, and mods                                                       |
| `-ignoreunknown` | `ignoreunknown` | silently drop unknown JSON keys (default: error). Overridden when an inline catch-all map is present   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sirkostya009/ggen](https://github.com/sirkostya009/ggen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
