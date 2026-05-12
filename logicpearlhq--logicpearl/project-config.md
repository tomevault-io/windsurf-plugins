---
trigger: always_on
description: LogicPearl separates deterministic policy logic from explanation metadata.
---

# Agent Guidance

LogicPearl separates deterministic policy logic from explanation metadata.

## Feature Dictionaries

Use a feature dictionary before discovery when raw feature IDs need reviewer-facing text:

```bash
logicpearl build traces.csv \
  --feature-dictionary feature_dictionary.json \
  --output-dir /tmp/pearl
```

The dictionary is embedded into the artifact as `input_schema.features[].semantics`. Generated rule labels, messages, counterfactual hints, `inspect`, and `diff` may use it. Runtime evaluation must ignore it.

Minimal dictionary shape:

```json
{
  "feature_dictionary_version": "1.0",
  "features": {
    "feature_id": {
      "label": "Readable feature label",
      "source_id": "optional-source-id",
      "source_anchor": "optional-source-anchor",
      "states": {
        "missing_or_failed": {
          "when": {
            "op": "<=",
            "value": 0.0
          },
          "label": "Readable state label",
          "message": "This rule fires when the readable condition is true.",
          "counterfactual_hint": "Describe the smallest useful change."
        }
      }
    }
  }
}
```

For simple CSV work, `label` alone is enough. Use `states` only when a specific predicate needs precise reviewer-facing text.

Do not fix unreadable output by patching `rules[].label` after discovery or by rewriting labels in a UI. Generate a dictionary from the same source that generated the traces, then pass it to `build` or `discover`.

Do not add healthcare, payer, or other domain-specific parsing to the core crates. The core should not parse prefixes like `requirement__`, suffixes like `__satisfied`, or IDs like `req-003`. Domain integrations own those meanings and should express them through feature dictionary fields.

Proof checklist before claiming feature dictionaries work:

- Build with `logicpearl build ... --feature-dictionary feature_dictionary.json`.
- Inspect the emitted `pearl.ir.json` and confirm `input_schema.features[].semantics` exists for dictionary-backed features.
- Confirm generated `rules[].label`, `rules[].message`, and `rules[].counterfactual_hint` came from LogicPearl rule generation, not a post-build patch or frontend rewrite.
- Run `logicpearl inspect <artifact> --json` and confirm readable feature metadata appears alongside raw `deny_when`.
- Run the artifact with and without dictionary metadata when relevant and confirm runtime bitmasks are identical for identical raw rules.

When reviewing diffs, keep these separate:

- `source_schema_changed`: features or source anchors changed
- `learned_rule_changed`: raw rule expressions changed
- `rule_explanation_changed`: labels, messages, hints, or dictionary text changed while raw logic stayed the same

Diff expectation:

- `logicpearl diff old_artifact new_artifact --json` should expose readable rule metadata when dictionaries are present.
- Raw expression changes should be reported separately from explanation-only changes.
- Adding or improving dictionary text should not be described as a learned policy change unless the raw `deny_when` changed.

Downstream demos and frontends may display artifact metadata, but they must not synthesize, infer, or rewrite rule meaning. The artifact should already contain the human-facing label, message, and counterfactual hint.

Raw `deny_when` expressions are the source of deterministic truth.

## Artifact Manifests

`artifact.json` is a public bundle contract, not only a CLI pointer. New gate,
action, and composed pipeline bundles should emit
`schema_version: "logicpearl.artifact_manifest.v1"`.

The v1 manifest should identify the artifact with `artifact_id`,
`artifact_kind` (`gate`, `action`, or `pipeline`), `engine_version`,
`ir_version`, `created_at`, and the deterministic `artifact_hash`. Use
`files.ir` as the canonical pointer to `pearl.ir.json` or a pipeline JSON file.
Optional sidecars belong under `files.build_report`, `files.feature_dictionary`,
`files.native`, `files.wasm`, and `files.wasm_metadata`.

Use `logicpearl artifact inspect <artifact> --json`,
`logicpearl artifact digest <artifact> --json`, and
`logicpearl artifact verify <artifact> --json` when checking bundle behavior.
`artifact verify` should validate the manifest version, referenced files,
file hashes, artifact hash, IR kind, and input schema hash. Keep old manifest
aliases readable for compatibility, but prefer the v1 field names in new code,
tests, and docs.

## Build Provenance

Build reports should carry `provenance.schema_version:
"logicpearl.build_provenance.v1"`. Provenance belongs in core build output, not
downstream demos, because it is the audit trail for where an artifact came from.

Record stable hashes and bounded metadata: `engine_version`, optional
`engine_commit`, redacted CLI command args, redacted build options plus
`build_options_hash`, trace input hashes and row counts, feature dictionary hash,
plugin run provenance, optional source manifest hash, limited environment facts,
and generated artifact file hashes.

Do not store raw environment variables, full PATH values, hostnames, usernames,
home directories, raw plugin stdout/stderr, raw source documents, or raw
free-form provenance values. Hash plugin boundaries and redact plugin option,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LogicPearlHQ/logicpearl](https://github.com/LogicPearlHQ/logicpearl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
