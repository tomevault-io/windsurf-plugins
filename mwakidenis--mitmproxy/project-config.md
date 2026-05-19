---
trigger: always_on
description: - This project uses uv. Always use `uv run pytest` and don't run pytest directly.
---

- This project uses uv. Always use `uv run pytest` and don't run pytest directly.
- To run all tests: `uv run tox`.
- When adding new source files, additionally run: `uv run tox -e individual_coverage -- FILENAME`.

---
> Source: [mwakidenis/mitmproxy](https://github.com/mwakidenis/mitmproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
