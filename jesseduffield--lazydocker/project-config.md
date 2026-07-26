---
trigger: always_on
description: - All Go commands need `GOFLAGS=-mod=vendor` (deps are vendored, including the `jesseduffield/gocui` fork and the Docker SDK).
---

# CLAUDE.md

## Build & test
- All Go commands need `GOFLAGS=-mod=vendor` (deps are vendored, including the `jesseduffield/gocui` fork and the Docker SDK).
- Unit tests: `GOFLAGS=-mod=vendor go test ./...`

---
> Source: [jesseduffield/lazydocker](https://github.com/jesseduffield/lazydocker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
