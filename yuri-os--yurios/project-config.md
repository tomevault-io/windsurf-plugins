---
trigger: always_on
description: - `SPEC.md` is the normative specification: RFC-2119 language, cited from code as `SPEC §n` comments (hundreds of them). When changing specified behavior, update `SPEC.md` in the same change; when code and spec disagree, the spec is authoritative by convention — treat the mismatch as a bug.
---

# YuriOS Agent Guide

## Specification

- `SPEC.md` is the normative specification: RFC-2119 language, cited from code as `SPEC §n` comments (hundreds of them). When changing specified behavior, update `SPEC.md` in the same change; when code and spec disagree, the spec is authoritative by convention — treat the mismatch as a bug.
- Section numbers are stable and never renumbered — code, tests, and scripts cite them, so only ever append or revise within a section.
- A bare `SPEC §n` means *this* spec and must resolve — `tests/test_spec_citations.py` fails on a citation to a section that doesn't exist, or to one SPEC.md marks superseded. A predecessor build's spec carries its prefix instead (`B1`/`B2`/`B4`, per `PROVENANCE.md`'s map of which package came from which build).
- `docs/spec-map.md` is the inverse index — section → the code that implements it — generated from those citations by `python scripts/spec_map.py`. Use it to go from a section to its modules; the file's own docstring goes the other way. Regenerate it with `python scripts/spec_map.py` after adding or moving a citation — `tests/test_spec_citations.py` fails while it is stale, so this is the suite's rule rather than a habit, and `--check` is the same answer outside pytest.
- The citation test catches a `SPEC §n` that does not exist and one that lands on a superseded stub. It cannot catch the third case — a citation that resolves perfectly, to the wrong section, which is what a predecessor build's number looks like once it has lost its `B1`/`B2`/`B4` prefix. `python scripts/spec_map.py --audit` narrows where to look by flagging the odd package out of each section's citers; it is a review list, not a verdict. Run it after moving code between packages.
- `docs/test-map.md` is the other generated index: module → the test files that actually *ran* its lines, measured with coverage contexts by `python scripts/test_map.py` (`--check` verifies it, `--reuse` rebuilds without rerunning). Use it before running the whole suite — for the 100-odd modules with no `test_<name>.py`, it is the only way to derive the cheap subset. It is not a gate stage: the instrumented run is serial and takes minutes.
- `docs/` is the plain-language companion; `SPEC.md` wins on any disagreement. Don't treat `docs/` as a source of truth for behavior.

## Toolchain and verification

- Support Python `>=3.11,<3.14`; Python 3.12 is the installer target. Use the project interpreter when present: `.venv/bin/python`.
- Run the offline suite with `.venv/bin/python -m pytest -q -n 8` — 1,744 tests in ~80s across eight workers, minutes serial, so pass `-n 8`. Focus a change with `.venv/bin/python -m pytest -q tests/test_file.py::test_name`, and drop `-n` when you need a readable traceback (xdist interleaves eight workers' output). Don't raise the worker count much past 8: most workers pay a one-off torch import (~5s warm, ~20s cold), and `-n auto` measured slower than `-n 8` on a 20-core box.
- Tests deliberately replace dotenv loading and use fake voice, tools, image, model, and clock seams. Do not require a configured model, API key, GPU, or live service for test coverage.
- `./scripts/check.sh` is the gate: `ruff check`, `mypy`, pytest, and the web suite, each stage running even when an earlier one failed. Use `--fast` to skip pytest while iterating, `--release` to add the install smoke test. It needs `pip install -e ".[dev]"`. There is no CI; this is the whole gate.
- Lint and typecheck are configured in `pyproject.toml` and are green — keep them that way. Ruff is `check` only, never `format` (the formatting is hand-set). Mypy skips the 30 modules listed in its overrides, which predate it: that list may only shrink, and new modules are checked from the start.
- For `install.sh`, run `bash -n install.sh` (and `shellcheck install.sh` when available).
- The browser app is a separate Vite build. After changing `web/`, run `(cd web && npm ci && npm run build)`; FastAPI serves the ignored `web/dist/` output at `/`. Use `(cd web && npm run dev)` only for Vite development.
- `web/tests/` holds the frontend suite (`npm test`, vitest). Scope it to modules that decide something, not to the three.js room: a test that mocks WebGL asserts only that the mock was called.
- After changing a dependency in `pyproject.toml`, regenerate the pins with `./scripts/pin_deps.sh` and run `./scripts/smoke_install.sh` — the resolve check is the only thing that sees a version ceiling breaking, and pytest never will.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuri-os/YuriOS](https://github.com/yuri-os/YuriOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
