---
trigger: always_on
description: When analyzing this repository, focus on only these files:
---

# Instructions For AI Agents

## Primary Scope

When analyzing this repository, focus on only these files:

- [fedramp-consolidated-rules.json](fedramp-consolidated-rules.json)
- [schemas/fedramp-consolidated-rules.schema.json](schemas/fedramp-consolidated-rules.schema.json)

The rest of the repository is supporting infrastructure. The `tools` directory,
tests, and READMEs can help with validation and orientation, but they are not
the rules and should not be treated as authoritative rule content.

## Source Of Truth

[fedramp-consolidated-rules.json](fedramp-consolidated-rules.json) is the
source of truth for the FedRAMP Consolidated Rules for 2026 Public Preview.

[schemas/fedramp-consolidated-rules.schema.json](schemas/fedramp-consolidated-rules.schema.json)
is the source of truth for the expected data shape.

## Rules JSON Edit Guardrail

Do not modify
[fedramp-consolidated-rules.json](fedramp-consolidated-rules.json) unless the
user specifically instructs you to edit that file.

If a task appears to require changing
[fedramp-consolidated-rules.json](fedramp-consolidated-rules.json), stop before
editing it. Propose a concise plan that identifies the specific rule,
definition, indicator, metadata, or structural paths you intend to change, then
wait for the user's explicit confirmation before making those edits.

Analysis, validation, structured reads, and reports may use
[fedramp-consolidated-rules.json](fedramp-consolidated-rules.json) without
additional permission. The guardrail applies to file modifications.

## Test Creation Guardrail

When the user asks to add or update tests for the tooling, test harness, or
validation behavior, assume the requested test may expose existing rules data
issues, warnings, or intentionally failing cases. That is often the reason the
test is being added.

Do not fix test failures or warnings by editing
[fedramp-consolidated-rules.json](fedramp-consolidated-rules.json) unless the
user explicitly asks for rule-content changes. If a newly added test reports
errors or warnings against the current rules file, report the result and keep
the change scoped to test or tooling support.

If a test cannot be made meaningful without changing the rules JSON, stop before
editing it. Explain the specific rule, definition, indicator, metadata, or
structural path that would need to change and wait for explicit confirmation.

## Dataset Structure

The JSON file has four top-level sections:

- `info`
  Dataset metadata, including title, description, version, `last_updated`, and
  default artifact expectations.
- `FRD`
  FedRAMP Definitions. Use these definitions to resolve terms used in rules and
  indicators.
- `FRR`
  FedRAMP Rules process documents. These contain process-oriented requirements
  and recommendations.
- `KSI`
  Key Security Indicators. These describe security capabilities and evidence
  expectations.

### FRD

`FRD` entries are controlled definitions. Definition IDs follow `FRD-XXX`.
Important fields include `term`, `definition`, `alts`, references, notes, and
`updated` history.

The FRD data container uses applicability buckets. Shared definitions live under
`FRD.data.all`; framework-specific definitions, if present, live under
`FRD.data.20x` or `FRD.data.rev5`.

FRD effective metadata may be common (`FRD.info.effective`) or split into
paired framework-specific blocks (`FRD.info.20x.effective` and
`FRD.info.rev5.effective`).

When a defined term appears in an FRR rule or KSI indicator, use the FRD
definition instead of assuming the plain-language meaning.

### FRR

`FRR` is keyed by process short names such as `VDR`, `FRC`, `CCM`, and `SCN`.
Each process contains:

- `info`
  Rule metadata, purpose, status, effective metadata, subset definitions, and
  optional flow descriptions. Effective metadata may be common
  (`info.effective`) or split into paired framework-specific blocks
  (`info.20x.effective` and `info.rev5.effective`). Subsets and flows may also
  be common or framework-specific.
- `data`
  The rule tree.

The rule tree is organized as:

```text
FRR -> process -> data -> applicability -> subset -> requirement ID
```

Applicability keys are `all`, `20x`, and `rev5`. Subsets identify actors,
scopes, or process buckets. Requirement IDs follow the
`PROCESS-SUBSET-KEY` pattern, such as `VDR-CSO-123`.

Each requirement contains either:

- a single `statement` and `force`, or
- a `varies_by_class` object with class-specific statements and force values.

Class-specific variants may also include `following_information`, `artifacts`,
notes, effective dates, simple timeframes, and `pain_timeframes`.

Other useful top-level fields include `affects`, `controls`, `artifacts`,
`following_information`, `following_information_bullets`, `examples`,
`notification`, simple timeframes, terms, related rule references, references,
corrective actions, effective dates, and `updated` history.

### KSI

`KSI` is keyed by security theme short names such as `IAM`, `CNA`, `MLA`, and
`SCR`. Indicator IDs follow `KSI-THEME-KEY`.

Indicators describe security capabilities. They include statements or
class-specific variants, mapped controls, optional artifact expectations,
terms, references, and update history.

## Analysis Best Practices


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FedRAMP/rules](https://github.com/FedRAMP/rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
