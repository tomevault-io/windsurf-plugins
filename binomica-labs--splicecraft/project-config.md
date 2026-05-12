---
trigger: always_on
description: Agent handoff document. Read before touching the codebase.
---

# CLAUDE.md — AI Agent Context for SpliceCraft

Agent handoff document. Read before touching the codebase.

The project is developed by a human bioinformatician with an AI agent (Claude Opus 4.6+).

## What is SpliceCraft?

A **terminal-based circular plasmid map viewer, sequence editor, and cloning/mutagenesis workbench** built with Python 3.10+ / Textual / Biopython. Renders Unicode braille-dot plasmid maps in the terminal, with a per-base sequence panel, restriction-site overlays, a **collection-driven plasmid library**, Golden Braid L0 assembly tooling, Primer3-backed primer design, and SOE-PCR site-directed mutagenesis.

**Repo:** `github.com/Binomica-Labs/SpliceCraft` (Binomica Labs, user ATinyGreenCell). **PyPI:** `splicecraft`. Latest: **v0.4.5**.

- **Single-file architecture:** entire app is `splicecraft.py` (~17,900 lines). Intentional — keeps the codebase greppable. Sibling project ScriptoScope (~8,600 lines) follows the same convention.
- **Test suite:** ~1,200 tests across 19 files in `tests/`. `pytest -n auto` ~2-3 min on 8 cores; sequential ~13 min. Biology subset (`test_dna_sanity.py`) < 2 s. `test_invariants_hypothesis.py` adds property-based fuzzing. New in 0.5.1: `test_blast.py` (BLASTN/BLASTP/HMMscan engines + sanitisation) and `test_new_plasmid.py` (NewPlasmidModal + `_annotate_seq_from_feature_library`).
- **Dependencies:** `textual>=8.2.5`, `biopython>=1.87`, `primer3-py>=2.3.0`, `platformdirs>=4.9`, `pyhmmer>=0.12`. Tests: `pytest>=9.0`, `pytest-asyncio>=1.3`, `pytest-xdist>=3.8`, `hypothesis>=6.152`. No optional runtime deps — pLannotate integration was removed in 0.4.0.
- Releases via `./release.py X.Y.Z` (bumps version, runs tests, builds, tags, pushes; `publish.yml` uploads to PyPI via OIDC). Pure-Python — no bash/sed/grep dependencies.

## How to run

```bash
cd ~/SpliceCraft
python3 splicecraft.py              # empty canvas
python3 splicecraft.py L09137       # fetch pUC19 from NCBI
python3 splicecraft.py myplasmid.gb # open local GenBank file (.gb/.gbk/.dna)
python3 -m pytest -n auto -q        # full test suite (~3 min on 8 cores)
```

End users: `pipx install splicecraft && splicecraft`.

Logs: `~/.local/share/splicecraft/logs/splicecraft.log` (override with `$SPLICECRAFT_LOG`). Every line prefixed with an 8-char session ID for multi-run grepping.

## Architecture (single file: `splicecraft.py`)

### Top-level structure (line numbers ±50)

| Lines | Section |
|-------|---------|
| 1–200 | Docstring, imports (incl. module-level `from datetime import date as _date`), user data dir, dep check, rotating session-tagged logger, feature-colour palette |
| 201–460 | Atomic JSON persistence (`_safe_save_json` / `_safe_load_json` / `_extract_entries`; envelope schema `{"_schema_version":1,"entries":[...]}` + legacy bare-list back-compat) + library cache loaders |
| 461–575 | **Collections persistence** — `_load_collections` / `_save_collections` (deepcopy-on-read), `_get/_set_active_collection_name`, `_find_collection`, `_collection_name_taken`, `_ensure_default_collection` (Main Collection migration), `_sync_active_collection_plasmids`, `_restore_library_from_active_collection` |
| 576–2400 | NEB enzyme catalog (~204), IUPAC tables + cached regex, `_rc`, `_scan_restriction_sites`, **2D feature packer** (`_pack_features_2d`, `_chunk_lane_groups`, `_render_packed_strand`, `_paint_feature_label`, `_paint_feature_bar`, `_paint_cds_aa`), `_feats_in_chunk` (wrap-CDS halves carry `_orig_start`/`_orig_end`), memoized `_build_seq_inputs`/`_build_seq_text` (per-strand cut bg colours), `_cds_aa_list` (per-CDS LRU translation cache), OSC-52 clipboard, `_translate_cds` |
| 1821–1870 | Char-aspect detection + label helpers + `_cursor_row_key` DataTable utility |
| 1871–2110 | GenBank I/O (`fetch_genbank`, `load_genbank` auto-detect `.gb`/`.dna`, `_record_to_gb_text`, `_gb_text_to_record`, `_normalize_for_genbank`, `_export_genbank_to_path`, `_export_fasta_to_path`) |
| 2111–2200 | _(pLannotate slot — removed in 0.4.0; placeholder comment only)_ |
| 2201–2350 | `_Canvas` + `_BrailleCanvas` (sub-cell braille resolution) + `PlasmidMap` start |
| 2200–3160 | `PlasmidMap` widget — circular/linear draw, label placement, `_draw_cache`; `FeatureSidebar` |
| 3160–3730 | **`LibraryPanel`** — two-mode panel (collections list ↔ active-collection plasmids). Click a collection → enter plasmids view; back button (`←` in plasmids-button row) → return to collections. Per-mode `+ − ✎` CRUD; `set_dirty` does single-cell `update_cell_at` instead of full repopulate. |
| 3730–3990 | `SequencePanel` |
| 3990–4400 | Core modals (`EditSeqDialog`, `FetchModal`, `OpenFileModal`, `ExportGenBankModal`, `FastaExportModal`, `DropdownScreen`, `MenuBar`) |
| 4400–4900 | Golden Braid L0 position catalog (Esp3I/BsmBI overhangs); cloning grammar registry (`_BUILTIN_GRAMMARS`, `_load_custom_grammars`, `_get_active_grammar`); settings.json key/value persistence |
| 4900–5800 | Codon-usage registry, Kazusa parser, NCBI taxid search (`_safe_xml_parse`), CAI/GC. SOE-PCR mutagenesis primer design (`_mut_*`) |
| 5800–7100 | Feature-library workbench, `PlasmidFeaturePickerModal`, `AddFeatureModal`, `ColorPickerModal`, `_FeatureSnippetPanel`, `FeatureLibraryScreen` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Binomica-Labs/SpliceCraft](https://github.com/Binomica-Labs/SpliceCraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
