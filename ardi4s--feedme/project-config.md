---
trigger: always_on
description: Working notes for an agent (or a person) changing this repository. Everything
---

# AGENTS.md

Working notes for an agent (or a person) changing this repository. Everything
here describes the checkout as it actually is, including the parts that are
deliberately local to this machine. Read it before editing anything.

---

## 1. What this is

`feedme` turns a web page that lists things into a feed a reader can subscribe
to. It serves RSS 2.0, Atom 1.0, or JSON Feed 1.1 from one URL per feed, and it
is a single static Go binary with no build step at deploy time.

The idea is that the source site needs no cooperation: the server fetches the
listing page, works out which elements are the items, and renders them as a
feed. Where a listing is built by JavaScript it can optionally ask a headless
browser for the rendered HTML.

State lives in one SQLite database: an HTTP cache, a rendered-feed cache, the
per-item bodies used for full text, and the build history that the management
page lists from. Nothing else is persisted, so the binary itself is disposable.

Module path is `feedme`. `go.mod` says `go 1.25.0` with `toolchain go1.25.11`.

---

## 2. Commands

The Makefile exports `GOTOOLCHAIN ?= go1.25.11`, so `make` targets pick the
right toolchain on their own. Running `go` directly outside `make` needs
`export GOTOOLCHAIN=go1.25.11` unless the system Go already matches.

| Command | What it does |
| --- | --- |
| `make all` | `vet` + `test` + `build`. The gate to run before committing. |
| `make build` | Builds `bin/feedme` with `-ldflags "-s -w -X main.version=$(VERSION)"`. |
| `make test` | `go test ./...` |
| `make vet` | `go vet ./...` |
| `make fmt` | `gofmt -l -w .` |
| `make tidy` | `go mod tidy` |
| `make install` | `go install` with the version stamped in |
| `make run` | Builds, then runs `bin/feedme` |
| `make clean` | Removes `bin/` |

`VERSION` is read from the git tag with `git describe`, so a local `make build`
reports the tag it came from: `0.2.0` at the tag, `0.2.0-3-g9e82db4` three
commits later, `-dirty` with uncommitted changes, and `docker` where there is no
git at all. Releases ignore it; `release.yml` stamps the tag itself.

Other gates that matter, and that a change should not break:

```sh
gofmt -l .                 # must print nothing
go vet ./...
go test -count=1 ./...     # -count=1 defeats the test cache
docker compose config      # the published compose file must stay valid
```

Run the server locally:

```sh
go run ./cmd/feedme serve -addr :8080 -db /tmp/feedme.db -site-dir configs/sites
```

Diagnose why a URL produces nothing:

```sh
go run ./cmd/feedme probe https://example.com/news
go run ./cmd/feedme https://example.com/news     # shorthand for the above
```

Two subcommands exist: `serve` and `probe`. `feedme <url>` is `feedme probe
<url>`. `feedme version` and `feedme help` are also there.

---

## 3. Repository layout

```
cmd/feedme/            the CLI: dispatch, wiring, and the adapters
internal/              everything that does not need a main function
configs/sites/         per-host site overrides, plus two bundled ones
docs/brand/            the images the README links to
tools/                 Python helpers for brand art and screenshots
.github/workflows/     ci.yml and release.yml
```

### `cmd/feedme`

| File | Role |
| --- | --- |
| `main.go` | Subcommand table, help and version output, the CLI banner. |
| `serve.go` | Flags, config loading, wiring every component, running the HTTP server. This is the only place that knows about all the pieces at once. |
| `probe.go` | Fetches one URL and reports what extraction found. The way to answer "why is this feed empty". |
| `admin.go` | `storeAdmin`: adapts the SQLite store to `web.FeedAdmin`. |
| `feedcache.go` | `storeFeedCache`: adapts the store to the interface the web layer wants for cached feeds. |

### `internal`

| Package | Role |
| --- | --- |
| `web` | HTTP only: routing, status codes, cache headers, the Go-assembled pages. |
| `feedurl` | Parses the stateless feed URL into a plan for building one. |
| `pipeline` | Runs one feed request end to end: fetch, detect, extract, filter, render. |
| `listpage` | Decides which elements of a listing page are the items. |
| `extract` | Pulls an article body out for `fulltext=1`. |
| `domx` | Low-level HTML helpers shared by the HTML-reading packages. |
| `fetch` | The HTTP client layer: rate limiting, robots, size caps, SSRF checks. |
| `render` | Asks a headless browser for rendered HTML, for `render_js=1`. |
| `feed` | Renders items as RSS 2.0, Atom 1.0, or JSON Feed 1.1. |
| `feedread` | Parses an existing RSS/Atom/JSON feed, so feeds can be merged. |
| `filter` | `filter`, `filterout`, `strip`, and deduplication. |
| `dates` | Parses the many date formats that appear in news markup. |
| `urlx` | URL helpers, including the registrable-domain logic the page groups by. |
| `store` | SQLite: the caches and the build history. |
| `config` | Global settings and per-host site overrides. |

The split is deliberate: **HTTP belongs in `web`, feed content belongs in
`pipeline` and its helpers, and request parsing belongs in `feedurl`.** A change
that puts extraction logic in `web`, or status codes in `pipeline`, is going
against the grain of the codebase.

---

## 4. How a request becomes a feed

For `GET /extract?url=…`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ardi4s/feedme](https://github.com/ardi4s/feedme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
