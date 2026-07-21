---
trigger: always_on
description: This guide helps AI coding agents work effectively in the `blint` repository.
---

# AGENTS.md

This guide helps AI coding agents work effectively in the `blint` repository.

## Mission

`OWASP blint` is a binary linter, disassembler, and SBOM generator for ELF, PE, Mach-O, WASM, and Android artifacts.

Primary workflows:

1. Security posture checks (PIE, NX, RELRO, canary, signing, PE hardening features).
2. Capability reviews from symbols/imports/functions using YAML rules.
3. Optional function disassembly (`--disassemble`) with nyxstone for advanced behavior detection.
4. CycloneDX SBOM generation (`blint sbom`) for binaries and Android apps.

## Repository map

- `blint/cli.py`: CLI parser, subcommands, and app entrypoint.
- `blint/config.py`: defaults, env vars, rule constants, disassembly indicator lists.
- `blint/lib/runners.py`: default analysis flow and SBOM flow orchestration.
- `blint/lib/analysis.py`: rule loading, checks execution, review engine, report writing.
- `blint/lib/review_runner.py`: `ReviewRunner` coordination for imports/symbols/entries/functions.
- `blint/lib/review_utils.py`: generic pattern-review matching and rule-option coercion.
- `blint/lib/function_reviews.py`: `FUNCTION_REVIEWS` heuristics and metric evaluation.
- `blint/lib/binary.py`: format parsing and metadata extraction (ELF/PE/Mach-O/WASM).
- `blint/lib/disassembler.py`: nyxstone-backed function disassembly and metrics.
- `blint/lib/sbom.py`: CycloneDX object construction and dependency modeling.
- `blint/lib/android.py`: APK/AAB metadata extraction and component mapping.
- `blint/db.py`: blintdb v2 SQLite-assisted component identification from symbols, binary-name hints, and disassembly hashes.
- `blint/data/rules.yml`: built-in hardening/security checks.
- `blint/data/annotations/*.yml`: built-in capability and behavior reviews.
- `docs/METADATA.md`, `docs/DISASSEMBLE.md`, `docs/RULES.md`: deep reference docs.
- `tests/`: unit tests and fixture metadata.

## Operational flow (default mode)

1. CLI builds `BlintOptions` (`blint/cli.py`).
2. `run_default_mode` gathers candidate binaries (`gen_file_list`).
3. Each binary is parsed (`blint/lib/binary.py::parse`).
4. Raw metadata is exported as `*-metadata.json`.
5. Security checks run from `rules.yml` (`run_checks`).
6. Capability reviews run from annotation rules (`blint/lib/review_runner.py::ReviewRunner`).
7. Optional fuzzable targets are generated (`run_prefuzz`).
8. Findings/reviews/fuzzables are exported as JSON plus HTML console output.

## Operational flow (SBOM mode)

1. `run_sbom_mode` discovers binary + Android inputs.
2. `generate` builds CycloneDX 1.6 model.
3. `process_exe_file` parses binaries and maps dependencies/components.
4. Optional `--use-blintdb` enriches component identification.
5. `--use-blintdb --deep` automatically enables disassembly and uses function-hash lookup before symbol fallback.
6. Output is written to the configured file (or stdout).

## Agent coding guidelines for this repo

- Preserve existing CLI compatibility and JSON schema shape.
- Prefer additive metadata fields over breaking renames/removals.
- Keep cross-format behavior consistent (ELF/PE/Mach-O/WASM field normalization).
- Rule engine behavior must remain deterministic and case-insensitive where expected.
- Keep heavy operations behind explicit flags (`--disassemble`, `--deep`, `--use-blintdb`).
- Avoid weakening error handling around malformed binaries.
- Always check for Windows path separators/characters (`\` vs `/`, drive letters, basename handling) when writing or updating filesystem logic, fixtures, and especially unit tests; avoid POSIX-only raw-string assertions when the behavior is path-based.
- Nyxstone currently provides disassembly text, but not structured operand/register metadata; register usage and call-target heuristics in `blint/lib/disassembler.py` must therefore remain text-based.
- For blintdb-backed SBOM matching, prefer exact project evidence over permissive fuzzy expansion. False positives in SBOM output are harder to review than missed low-confidence hints.
- Keep `blint/lib/indicators.py` high-signal and avoid turning it into a generic phrase dump. Prefer compact technical tokens over broad prose/CVE strings, and prefer high-quality disassembly-based detection or `FUNCTION_REVIEWS` heuristics when those can express the behavior reliably.

## Common task playbooks

### Add a new security/capability rule

1. Add or edit YAML in `blint/data/rules.yml` or `blint/data/annotations/*.yml`.
2. Ensure `id` is unique and includes required fields.
3. If using `FUNCTION_REVIEWS`, confirm `check_type` and fields align with
   `blint/lib/function_reviews.py` evaluation logic.
4. Add/adjust tests in `tests/test_analysis.py` with fixture metadata.

### Add metadata extraction for a format

1. Extend relevant parser section in `blint/lib/binary.py`.
2. Keep cleanup compatibility (`cleanup_dict_lief_errors`).
3. Update `docs/METADATA.md` for any new top-level or nested keys.
4. Add focused tests and fixtures under `tests/data/`.

### Extend SBOM mapping

1. Implement or refine component conversion in `blint/lib/sbom.py`.
2. Keep `bom_ref` stable and dependency refs consistent.
3. Preserve deep-mode behavior and avoid huge default output growth.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [owasp-dep-scan/blint](https://github.com/owasp-dep-scan/blint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
