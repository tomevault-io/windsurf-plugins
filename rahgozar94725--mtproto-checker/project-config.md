---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Single source of truth for this repo. `AGENTS.md` is a pointer to this file.

## Commands

```bash
go run .                          # dev server on 127.0.0.1:3000 (PORT/HOST env override)
go build -o mtproto-checker .     # single self-contained binary
go test ./... -short              # unit tests only — skips network/proxy-file tests
go test ./... -v                  # full run, incl. live Telegram handshake tests
go test -run TestDecodeSecret -v  # single test
go test -bench BenchmarkBatchPipeline -benchtime=1x
go vet ./...                      # clean as of last run

# reproduce a release build locally
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -ldflags="-X main.version=v0.0.0-local" -o /tmp/mtc .
```

A host build (`go1.26.4 windows/amd64`) produces a 21,616,640-byte (~20.6 MiB) binary with `public/` baked in.

**Formatting:** `.gitattributes` pins `*.go` (and all text files) to LF in the repository *and* the working copy, so `gofmt -l .` is meaningful on every platform and is expected to be clean. Caveat: a checkout that predates `.gitattributes` may still hold stale CRLF working copies (`git ls-files --eol` shows `w/crlf`), which makes `gofmt -l` flag every Go file on line endings alone — fix with `rm <files> && git checkout -- <files>`, not by re-formatting.

No linter, no formatter config, no test CI. Only CI is `.github/workflows/release.yml`, triggered by `push` of a `v*` tag: cross-compiles 5 platforms (windows/linux/darwin × amd64/arm64, `CGO_ENABLED=0`), injects `-X main.version=<tag>`, uploads to GitHub Releases with a changelog generated from `git log <prev-tag>..<tag>`.

## Architecture

Single-process Go server (`main.go`, ~600 lines) + vanilla-JS frontend embedded into the binary. No build step for the frontend, no framework, no TypeScript.

**Backend — three endpoints, all wrapped in `recoverMiddleware` (panic → 500 JSON):**
- `POST /check` — one proxy, returns `{ok, ping?}`. The supported scripting endpoint, documented in the READMEs' HTTP API section.
- `POST /check-batch` — **deprecated, removal planned for a future release**: answers with `Deprecation: true` + `Link` headers and logs a warning per hit. JSON array in, array out. Two strict phases with a barrier between them: all TCP pre-checks finish (`tcpWg.Wait()`), *then* MTProto checks run on survivors.
- `POST /check-stream` — SSE, the only endpoint the UI actually calls. Per-proxy goroutine does TCP check then MTProto check inline (no barrier), emitting `event: progress` per result and `event: done` at the end. Writes are serialized by `mu` because `http.ResponseWriter` is not concurrency-safe.

**Request limits:** every handler caps the body at `maxBodySize` (8 MiB) via `http.MaxBytesReader`; the two batch endpoints additionally reject more than `maxBatchSize` (10 000) entries. Either violation answers `413` with `{"error": …}` — shared logic in `readCheckRequests`, which `/check-stream` runs *before* committing to SSE so a rejected request gets plain JSON, not an empty event stream. Note the UI posts the whole pasted list in one request, so a pasted list over 10 000 entries now fails with the generic error toast.

`public/` is baked in with `//go:embed public` + `fs.Sub` and served by `http.FileServer` at `/`. Nothing is read from disk at runtime — editing `public/` requires a rebuild (or `go run .`) to take effect.

**Startup** logs the version (`-ldflags -X main.version=<tag>` in releases, `"dev"` otherwise), binds explicitly with `net.Listen` (bind failure dies before any browser launch), then opens the local browser at the bound address — only when the bound host is loopback (`shouldOpenBrowser`); `NO_BROWSER` set to any non-empty value suppresses it, and a non-loopback `HOST` suppresses it automatically. The launcher (`browserCommand`: `rundll32`/`open`/`xdg-open`) is fire-and-forget; a failed launch logs one line and never affects the server.

**Proxy verification** (`checkProxy`) is a real MTProto handshake, not a TCP ping: `dcs.MTProxy(addr, secret)` resolver → `telegram.NewClient` with public test creds (`testAppID = 6`, `testAppHash = "eb06d4…"`, hardcoded in `main.go`, intentionally public — no login required) → `help.getNearestDC`. Round-trip time of that call is the reported ping. It carries its own `recover()` in addition to the middleware's; the reason is not recorded anywhere in the repo.

`decodeSecret` tries the raw input and then a junk-right-trimmed copy (the trim set overlaps the base64 alphabets, so raw must come first); per candidate it tries hex first (both candidates), then base64 RawURL → URL → RawStd → Std. Known limitation: a base64 secret whose last character is `+`, `/` or `_` *and* is followed by junk still decodes to the wrong bytes — the raw pass fails on the junk, and the trim pass strips that final alphabet character along with it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rahgozar94725/MTProto-Checker](https://github.com/rahgozar94725/MTProto-Checker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
