---
trigger: always_on
description: validates against it, `presetLayout.js` converts coordinates with it, and the stylesheet draws it.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

SnmpLens is a cross-platform SNMP MIB browser / network-management desktop app built with **Wails v2**: a Go backend and a Svelte 5 + Vite frontend compiled into a single native binary. The Go module is `SnmpLens` (see `go.mod`); the frontend lives entirely under `frontend/`.

## Commands

```bash
wails dev                 # hot-reload dev server (regenerates Go→JS bindings, then runs vite + Go)
wails build               # production binary → build/bin/
wails build -platform linux/amd64 -tags webkit2_41   # Linux REQUIRES the webkit2_41 build tag
wails build -platform windows/amd64
wails build -platform darwin/universal
```

Frontend-only scripts (run from `frontend/`): `npm run dev`, `npm run build`. You normally don't invoke these directly — `wails dev`/`wails build` drive them via `wails.json`.

`go run ./tools/cataloguecheck <directory>` validates a catalogue of contributed simulator models and
dashboard presets — the shape [SnmpLens/snmplens-community](https://github.com/SnmpLens/snmplens-community)
has — by BUILDING each one through `pkg/simulator` and `pkg/preset` rather than reimplementing their rules: a
checker that drifts accepts what the import then refuses. That repository's workflow checks out both and runs
it on every pull request.

The Go toolchain is pinned by the `toolchain` directive in `go.mod`, and every workflow reads it with
`go-version-file` rather than naming a version — one source of truth. It matters: Go supports only the two most
recent majors, so a project sitting on an older one stops receiving security fixes without anything failing.

Security workflows: `codeql.yml` (SAST for Go and the frontend, weekly as well as per-push, because a query added
after a commit lands would otherwise never see it), `dependency-review` on pull requests (govulncheck reports on
what is already merged; this blocks it before), `govulncheck`, and `npm audit --audit-level=high` — which used to
end in `|| true` and therefore never failed. Actions are pinned by commit SHA, with Dependabot configured for
gomod, npm and github-actions, because pinning without something to move the pins just freezes them.

CI verification gate (`.github/workflows/ci.yml`, all run from repo root):

```bash
go vet -tags webkit2_41 ./...
go test -race -count=1 -tags webkit2_41 ./...       # on ALL THREE platforms, not just Linux
staticcheck -tags webkit2_41 ./...                  # PINNED: honnef.co/go/tools/cmd/staticcheck@v0.8.1
go mod tidy && git diff --exit-code go.mod go.sum   # go.mod must stay tidy
go build -tags webkit2_41 ./...                     # the tripwire, see below
govulncheck -tags webkit2_41 ./...
```

Three of those lines are load-bearing in a way the obvious version is not.

**The tests run on every platform we ship.** They used to run on Linux alone "to avoid redundant runs", and they
are not redundant: `pkg/autostart` is an HKCU Run key, a LaunchAgent and an XDG entry; `pkg/secrets` is DPAPI,
the Keychain and a file backend; `pkg/network` runs `traceroute6` on macOS and `tracert` on Windows. Three
implementations behind one interface, one of them ever exercised. The trap-listener race reproduced on Linux and
never on Windows; the converse is as likely and would have been as invisible.

**`go build` before `govulncheck` is a tripwire, not ceremony.** govulncheck's analysis is only as complete as
the packages it manages to LOAD, and it does not fail when one does not build — it reports on the rest and
prints a clean bill of health. The security job had neither the `frontend/dist` placeholder nor the GTK/WebKit
headers, so on Linux the main package dropped out of every scan it ever ran. Reproduced by deleting
`frontend/dist`: `go build ./...` fails on the embed while `govulncheck ./...` answers "No vulnerabilities
found" and exits 0.

**staticcheck is pinned and govulncheck is not**, deliberately and in opposite directions. A linter that floats
fails a commit that changed nothing when a new check lands; a vulnerability scanner that is pinned reports an
old world. Dependabot does not see versions inside a `run:` step, so staticcheck is bumped by hand.

Go unit tests live beside the code they cover (`go test ./...`, run in CI). They are deliberately few and target
the logic that is subtle and easy to break silently — storage pragmas/migrations, aggregation, threshold
semantics, the poll clock, counter-wrap maths — not coverage for its own sake.

Two of them guard what gosnmp does, in the two ways that are available. This repository states gosnmp's
behaviour rather than assuming it, and most of those statements can be ASSERTED against gosnmp itself:
`pkg/snmp/gosnmpcontract_test.go` pins the debug log's wording, the version strings the Traps tab filters by, and
a user table that accepts a user which can never authenticate, the way `informack_test.go` already pinned a
handler's right to decline an acknowledgement. A test beats a citation wherever one is possible, because it is
re-checked on every commit and fails on the upgrade rather than on whoever next reads the paragraph — and because

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SnmpLens/SnmpLens](https://github.com/SnmpLens/SnmpLens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
