---
trigger: always_on
description: Read and follow this file before changing Tuva Core, including from a linked
---

# Tuva Core Agent Operating Manual

Read and follow this file before changing Tuva Core, including from a linked
worktree. It is the repository-local source of truth for ownership, safety,
validation, CI, and release rules.

## Tuva Core 1.0 Contract

- The repository is `tuva-health/tuva-core`.
- The dbt project name remains `the_tuva_project` for package compatibility.
- `dbt_project.yml` declares version `1.0.0` and requires dbt 1.10.5 through
  2.x. The complete 1.0 package ecosystem is validated against dbt Core 2.0
  and dbt Fusion on DuckDB.
- Repository `main` is the active integration line. A version on `main` is not
  a formal release by itself. A release also requires an immutable `v<version>`
  tag and a GitHub release.
- Treat published model schemas, grains, identifiers, variables, selectors,
  and stable Data Quality interfaces as public contracts. Assume a change is
  breaking unless compatibility is demonstrated.
- Use `v0.18.0` only as the frozen pre-1.0 parity baseline. Never use a mutable
  branch as a release comparison baseline.

## Repository Boundaries

Tuva Core owns the common transformation path:

- Input Layer contracts in `models/input_layer`.
- Standardization and terminology normalization in `models/normalized_layer`.
- Service categories, encounters, member months, claims enrollment, and
  provider attribution in `models/claims_preprocessing`.
- Public common-model outputs in `models/core`.
- Opt-in Structural Data Quality and Logical Data Quality models in
  `models/data_quality`.
- Package metadata in `models/metadata`.
- The opt-in parity metric producer in `models/parity`.
- Core-owned seed headers and loader macros in `seeds` and `macros`.

Tuva Core does not own optional marts or extensions. The accepted 1.0
standalone packages are:

- `ahrq_quality_indicators`
- `ccsr`
- `cms_chronic_conditions`
- `cms_hcc`
- `fhir_preprocessing`
- `nyu_ed_classification`
- `quality_measures`
- `semantic_layer` in repository `semantic-layer`

Installing a standalone package is its package-level enablement. Each package
owns its models, tests, data assets, documentation, compatibility, and release
lifecycle. Do not move optional-package code back into Core.

Related repositories have distinct ownership:

- `docs` owns the public documentation site, data dictionaries, and migration
  guidance. During local development it reads this checkout through
  `TUVA_CORE_PATH`.
- `dag-viewer` owns the lineage application and also reads the selected Core
  checkout through `TUVA_CORE_PATH`.
- `tuva-maintenance` owns data-asset source recipes, deterministic staging,
  publication, mirroring, verification, synthetic-data generation, and other
  maintainer-only utilities.
- Each connector owns its source-to-Input-Layer mappings.

Core still owns checked-in seed headers, runtime loader macros, and changes to
those package contracts. Cloud manifests own payload inventory and source
provenance. Route publisher, recipe, mirror, and maintenance-tool changes to
the maintenance tooling; do not put those concerns in the dbt package.

## Architecture And Public-Contract Rules

- Input Layer models are refs supplied by the parent connector project. The
  package-owned wrappers carry the Tuva contract metadata.
- The Normalized Layer is the standardization boundary before Claims
  Preprocessing and Core. Do not duplicate normalization downstream without a
  demonstrated need.
- Claims are source-scoped. Preserve `data_source` in claim-line identities,
  joins, partitions, grouping, attribution, and deduplication.
- Retain populated medical-claim diagnoses independently of claim
  classification. Put classification-dependent filters at the grouping
  algorithm that requires them, not in the canonical diagnosis relation.
- Claims-derived encounter IDs and public condition IDs use the established
  deterministic, collision-safe 32-character lowercase hash contracts. Do not
  reintroduce dataset-relative, delimiter-ambiguous, or literal-null-ambiguous
  identifiers.
- Clinical encounter groups come from the canonical encounter-type
  terminology. Do not restore a separate hard-coded `clinical` group.
- Open eligibility spans use a nullable `enrollment_end_date`; do not cap them
  at `tuva_last_run`.
- Appointment `type`, `status`, `reason`, and `cancellation_reason` remain
  source-native descriptive fields. Do not recreate unsupported normalized
  appointment vocabularies.
- Core `location` has public grain `(location_id, data_source)`, and Core
  `practitioner` has public grain `(practitioner_id, data_source)`. Claims rows
  retain their source. Clinical rows win only when the exact composite key
  matches; never collapse provider identifiers globally across sources.

Public Input Layer, Normalized Layer, Claims Preprocessing, Core, and
standalone-package fields ending in `_flag` are nullable binary integers:

- `1` means true or present.
- `0` means false or absent.
- null means unknown or not applicable according to the field description.

Reserve `_flag` for binary fields and use `_code` or `_status` for categorical
values. Cast public flags with `{{ dbt.type_int() }}`. Internal working flags,
Logical Data Quality result flags, and data-asset attributes are outside this

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tuva-health/tuva](https://github.com/tuva-health/tuva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
