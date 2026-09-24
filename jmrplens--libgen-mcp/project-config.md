---
trigger: always_on
description: Guidance for working on this repository. Read it before adding a tool, a
---

# libgen-mcp — Development Context

Guidance for working on this repository. Read it before adding a tool, a
download source, or a docs page; it records the conventions the quality gates
enforce and the architecture decisions behind them.

## Project Overview

libgen-mcp is a Model Context Protocol server, written in Go, that exposes
Library Genesis (and a set of open-access fallbacks) to an LLM client. It is
**keyless by default**: every core capability works with no account, no API key,
and no configuration. Keys are strictly opt-in and, when supplied per call, are
used once and never persisted.

The server presents a deliberately small surface — four tools plus a handful of
prompts:

- `search` — search the LibGen catalog, escalating to Anna's Archive and the
  open-access providers (arXiv, Crossref, OpenLibrary) when configured or when
  the catalog comes up empty.
- `get_details` — full metadata for a record by md5, edition/file id, or DOI,
  with optional keyless enrichment (Crossref/OpenLibrary).
- `download` — resolve and download a book (by md5) or article (by DOI) through
  an ordered source chain with transparent failover.
- `read` — extract text, search within, and outline a downloaded PDF/EPUB/TXT.

The module path is `github.com/jmrplens/libgen-mcp/v2`, and the suffix is not
decoration: Go requires it from major 2 onward, and a repository tagged `v2.0.0`
while its `go.mod` still says the unsuffixed path is not broken loudly. It is
broken **silently** — `go install …@v2.0.0` refuses outright ("module contains a
go.mod file, so major version must be compatible"), and `…@latest`, which is the
form the README and the installation page tell people to run, quietly keeps
resolving the newest v1 tag forever. Bumping the major therefore means the
`go.mod` module line, every import of this module, and the `go install` command
wherever it is documented, in one change.

The single source of truth for the version is the `VERSION` file.

## Project Structure

`ls cmd/ internal/` gives the layout; every package carries a doc comment saying
what it is, which `make godoc-check` enforces. What that does not tell you:

### Package roles worth knowing

- `internal/tools` is where the four tools are wired onto the server
  (`tools.Register`). Input/output types, their `jsonschema` field descriptions,
  the handlers, and the Markdown renderers all live here.
- `internal/libgen` owns the download pipeline. Sources are pluggable via the
  `DownloadSource` interface; the ordered chain is assembled in
  `Client.buildSourceChain`.
- `internal/discovery` owns keyless search beyond the catalog via the `Provider`
  interface, fanned out concurrently by `Federate`.
- `internal/config` defines every `LIBGEN_MCP_*` environment variable and the
  canonical `KnownSources` list.

## Build & Test Commands

Everything is driven through the `Makefile`; run `make help` for the full list.
Two things `make help` does not tell you:

`validate-http-stateless` is a hand-run smoke check, not a CI gate: it builds the binary,
serves it, and asserts the wire-level promises an HTTP deployment makes — no
`Mcp-Session-Id`; `GET` on the MCP endpoint → 405 with `Allow: POST`; an unknown path → 404
with a JSON body naming the endpoint, never the 405 a catch-all used to give; the five
security headers, checked on a response an inner layer writes itself (the 404) as well as on
a plain route; both server-card locations answering the same bytes under their own media
types, with the card's `Cache-Control` override; and the `--json-response` content type. Run
it after touching `internal/transport` or the HTTP wiring in `cmd/server`.

Coverage is scoped to `./internal/...` and `./cmd/...` — everything this module
builds — against a **90%** floor (`COVERAGE_PKGS`, mirrored into both
`sonar-project.properties` and the CI profile; **all three have to agree**, or a
package ends up counted and uninstrumented, which reports as 0% and is not).

It was narrower twice, and each narrowing was wrong for the same reason.
`cmd/server` was out until 2026-08-27 on the premise that it was thin wiring;
the rest of `cmd/` was out until 2026-09-20 on the premise that build tooling is
gated by its own `check-*` targets rather than by a number. **Excluding a
package from the metric hides more than a number** — `cmd/gen_tool_schema`
shipped with no test file at all and nothing reported it, because the rule was
prose and the exclusion was configuration.

**`cmd/eval` is the one exclusion left, and it is measurement rather than
policy**: its files are behind the `eval` build tag, CI does not set it, so no
profile CI produces can carry a line of it. Counting it would report a package
as 0% for being untestable here rather than untested.

**The number is measured on one platform.** The unit suite runs on three, but
the profile CI keeps is Linux's; a per-platform floor would measure the same
tests three times and gate on whichever runner was slowest to warm its cache.

### What the server costs to run

`make bench-resources` measures it, from the real binary, on both transports,
against an in-process stand-in for the catalog on loopback. The record is
`docs/benchmarks/resource-benchmark.json` and the page beside it is generated

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmrplens/libgen-mcp](https://github.com/jmrplens/libgen-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
