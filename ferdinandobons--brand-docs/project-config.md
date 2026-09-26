---
trigger: always_on
description: <!-- SPDX-License-Identifier: MIT -->
---

<!-- SPDX-License-Identifier: MIT -->
# brand-docs - Frozen Conventions

This is the **single source of truth** for the vocabulary of `brand-docs`. It is
owned by `scripts/brandkit/profile/schema.py`; this document is its human-readable
mirror. Every `SKILL.md`, command, and reference doc quotes these names **verbatim**.
If a name here disagrees with code, the code (the schema module) wins and this file
is the bug.

> **The one hard rule.** Brand-specific identifiers - style names, theme color
> tokens, font names, cover aliases, layout ids, chart-template ids - live in the
> Brand Profile and **nowhere else**. No IntermediateDocument block, and no writer,
> ever contains a literal style name, hex color, or font. The resolver is the only
> code that reads them. Off-brand output is impossible by construction.

---

## 1. The five verbs

Every skill (`brand-docx`, `brand-pptx`, `brand-xlsx`) implements the same contract:

| Verb | Input | Output |
|---|---|---|
| **extract** | a company `.docx`/`.pptx`/`.xlsx` template | a reusable **Brand Profile** |
| **comprehend** *(optional, model-driven)* | a saved profile + a model-authored `comprehension.json` | the profile with a validated, cached `comprehension` block |
| **verify** | a saved Brand Profile | QA findings + a verdict (the role map lives in `PROFILE.md`) |
| **generate** | content (free text or an IntermediateDocument) + a profile | a new on-brand document |
| **learn** *(optional, deterministic)* | a saved profile + its cross-run `generation_report.json` history | the profile with a distilled, ADVISORY `rules.overrides` lesson (live only with `--accept`; see §14) |

`verify` reports deterministic QA findings and a verdict; it does **not** render a
proof image, and the role-mapping table is written to `PROFILE.md` at extract time
(not re-emitted by verify). Pass `verify --accept` to mark a passing profile as
accepted (`verification.accepted = true`).

**`comprehend` is optional and model-driven** (schema 1.2.0). `generate` works on
the deterministic profile alone (CI/no-model); when a current comprehension is
present it additionally reconciles preserved cover/index structures with the new
content. The verb is realized as two CLI steps:

```bash
python scripts/brandkit/cli.py comprehend-input --name <brand>    # prints {facts, excerpt} for the model
python scripts/brandkit/cli.py comprehend --name <brand> --input comprehension.json  # the ONLY writer
```

`comprehend-input` surfaces the bounded, format-uniform bundle the model reasons
over (deterministic facts + a length-capped text excerpt; never raw OOXML).
`comprehend` is the **single writer** of the `comprehension` block: it
merge-validates the model's JSON **fail-closed** (schema shape + verbatim
membership of every load-bearing ref against the surfaced inventories), and on a
clean pass freezes it into `profile.json` with `status='present'`, stamping
`source_shell_sha256` from the live shell hash. On any finding it writes
`status='rejected'` with the findings and exits non-zero; the model retries. The
merge is **idempotent**: comprehend-twice yields a byte-identical `profile.json`.

The canonical engine entrypoint is `scripts/brandkit/cli.py`, run from the plugin
root (the directory containing `.claude-plugin/`):

```bash
python scripts/brandkit/cli.py extract --name <brand> --template <t>
```

Set `BRAND_DOCS_ROOT` to the plugin root to invoke the CLI from any working
directory (the skill `cli.py` shims honor it; if unset they walk up to the nearest
`.claude-plugin/`). `${CLAUDE_PLUGIN_ROOT}` never appears in author-facing SKILL.md.

Beyond the five skill verbs, the engine CLI exposes: `comprehend-input`
(read-only model bundle, above), `refine` (model-authored qualitative delta
overlaid on the present comprehension, ADVISORY until `--accept`; §14 documents
its `learn`/`propose-overrides` siblings), `compare-profiles` (read-only brand
drift report between two saved profiles; exit 1 on drift), the `extract
--blend` flag (multi-template value-fact blending, §16), `list`, and `doctor`.
The full verb table lives in `documentation/PLUGIN_WORKFLOW.md` (drift-guarded
by `tests/test_doc_drift.py`).

---

## 2. The Brand Profile directory

A profile is a **self-contained, copyable directory**. All internal paths are
relative to the profile root, so it can move between stores unchanged.

```
brand-kit/<name>/
├─ profile.json          # the index: schema-versioned envelope          [always]
├─ PROFILE.md            # human-readable palette + role table            [always]
├─ template/
│  └─ shell.<ext>        # the byte-for-byte shell the generator opens FROM [always]
├─ provenance.sha256     # the shell hash (drift detection)               [always]
├─ assets/               # logos + manifest.json                          [optional]
├─ components/           # reusable fragments + index.json                [optional]
├─ sections/             # multi-page/slide units + index.json            [optional]
├─ specimens/            # opaque chart/SmartArt captures                 [optional]
├─ samples/              # source.png (template) + smoke.png (proof)      [optional]
└─ .cache/               # resolved-hex / render caches (safe to delete)  [optional]
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ferdinandobons/brand-docs](https://github.com/ferdinandobons/brand-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
