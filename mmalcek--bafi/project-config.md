---
trigger: always_on
description: - Format Go code with `gofmt -w` before committing.
---

# AGENTS Instructions

## Development
- Format Go code with `gofmt -w` before committing.
- Run `go test ./...` and ensure all tests pass.

## Documentation
- If you modify files under `docs/` or `mkdocs.yml`, run `mkdocs build` to check that the documentation builds successfully (if mkdocs is available).

---
> Source: [mmalcek/bafi](https://github.com/mmalcek/bafi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
