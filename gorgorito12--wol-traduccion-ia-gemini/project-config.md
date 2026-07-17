---
trigger: always_on
description: enables/disables the Traducir/Solo caché/Stop trio together. Rarely-used options (Retry empty cache,
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A localization tool for the **Age of Empires III: Wars of Liberty** mod. It translates the game's
XML string tables (e.g. `stringtabley.xml`) from English into other languages via Google Gemini
(`gemini-2.5-flash`), with aggressive caching so that re-translating across mod versions costs
almost no API calls.

## Commands

```bat
pip install google-genai tqdm          REM core deps; tkinterdnd2 is optional (drag & drop)

REM CLI translation (see docs/USAGE.md for the full flag list)
python translate_gemini.py "INPUT.xml" "OUTPUT.xml" --api-key "KEY" --cache-file "wol_es.cache.json"

REM Cache-only pass: applies cache, never hits the API (preview / rebuild cache)
python translate_gemini.py "INPUT.xml" "OUTPUT.xml" --cache-file "wol_es.cache.json" --cache-only

REM Run the self-tests (the only tests in the repo) and exit
python translate_gemini.py --self-test-quality-gate
python translate_gemini.py --self-test-merge

REM Launch the GUI. Translator.bat is self-sufficient: it resolves a real Python
REM (rejecting the Microsoft Store alias stub), auto-installs Python 3.13 via winget
REM if missing, auto-installs google-genai/tqdm/tkinterdnd2, and launches with the
REM sibling pythonw.exe. Every failure path prints a message and pauses.
python translate_gui.py
```

There is no build, lint, or external test framework. "Tests" = the six `self_test_*` functions in
`translate_gemini.py`: `--self-test-quality-gate` runs four (quality gate, source casing, glossary,
user glossary) and `--self-test-merge` runs two (merge by `_locID`, cache-key parity). Both flags
work without the `input output` positionals. The API key falls back to the
`GEMINI_API_KEY` / `GOOGLE_API_KEY` env vars.

## Architecture

Two files, one engine. **`translate_gemini.py` is both the CLI and the library; `translate_gui.py`
imports it as `tg` and calls its functions directly** (`tg.translate_strings`,
`tg.parse_strings_xml`, `tg.iter_translatable_elements`, `tg.build_skip_rules`). The GUI does *not*
shell out to the CLI. **Consequence: changing the signature or behavior of `translate_strings`,
`parse_strings_xml`, or `iter_translatable_elements` will break the GUI** — check `translate_gui.py`
when touching them.

### Translation pipeline (the core flow)

`main()` → `parse_strings_xml` → `iter_translatable_elements` → `translate_strings` → write XML.

1. **Parse & preserve format.** `decode_auto` detects BOM/encoding, `DocumentFormat` captures
   encoding + newline + BOM + xml-declaration so `serialize_tree` can write the file back
   byte-faithfully. A custom `CommentedTreeBuilder` keeps XML comments. This fidelity matters — the
   game is picky about its string-table format.
2. **Select elements.** `iter_translatable_elements` yields `<string>` and `<plurals><item>` nodes.
   `should_skip_element` (driven by `SkipRules`) marks folder/path-like and user-excluded nodes as
   `skip=True`; skipped text is passed through untouched and re-verified by `assemble_full_texts`.
3. **Protect, then tokenize.** Before sending text to the model, two layers of masking are applied
   *in order*: `protect_phrases` replaces protected phrases/acronyms with `__PROTECT_#__`, then
   `protect_tokens` replaces placeholders (`%s`, `%1$s`, `\n`, …) with `__TOK#__`. After
   translation, `restore_all_tokens` reverses both. If a protected token goes missing in the output,
   the code falls back to the original source string rather than emit a corrupted string.
4. **Cache lookup.** Keyed by the *protected* source text → translated text. Misses are batched.
5. **Batch & parallelize.** `yield_batches` packs strings up to `MAX_BUDGET_BYTES`; a
   `ThreadPoolExecutor` (`--max-workers`, default 8) runs `translate_batch_with_retry` →
   `translate_batch_gemini`. The model is asked for a strict JSON array; `reconcile_batch_length`
   repairs length mismatches (pad with source / truncate) so a bad response never desyncs the list.
6. **Quality gates (Spanish-target only, applied per result).** `apply_postprocess_overrides`
   (Home City→Metrópoli, team→equipo, game ages like *National Age*→*Edad Nacional*),
   `enforce_acronym_integrity`, `apply_source_casing`, and
   `has_english_residue`. These are gated by `target_is_spanish()` so other target languages are
   unaffected. The forced-terminology rules (Home City, team, game ages) live in one place,
   `SPANISH_GLOSSARY` (a list of `GlossaryEntry`): each entry carries the `prompt_hint` fed to Gemini
   *and* the deterministic `output_fixes` applied here, so both layers share one source of truth and
   adding a term is one entry. `output_fixes` run only when the entry's `source_trigger` matches the
   English original, and are anchored to whole phrases (e.g. `(Era|Edad) Nacional`→`Edad Nacional`) —
   never a bare `Era`→`Edad`, since `Era` is also the verb *was*. Guarded by `self_test_glossary`.

   **User glossary (pair-agnostic, NOT Spanish-gated).** `glossary.txt` next to the script (or CLI
   `--glossary-file`): `source term = target term` lines, `#` comments. Loaded by

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gorgorito12/WoL-traduccion-IA-Gemini](https://github.com/Gorgorito12/WoL-traduccion-IA-Gemini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
