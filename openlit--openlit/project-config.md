---
trigger: always_on
description: These instructions supplement the repository-root `AGENTS.md`.
---

# OTLP receiver instructions

These instructions supplement the repository-root `AGENTS.md`.

- This is an independent Go module. Validate with `go test ./...`.
- Do not log API keys, Authorization headers, or ClickHouse passwords.
- Tenant writes must stay isolated by OpenLIT API key → organisation → project → environment → DatabaseConfig.

---
> Source: [openlit/openlit](https://github.com/openlit/openlit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
