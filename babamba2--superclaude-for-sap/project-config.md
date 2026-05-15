---
trigger: always_on
description: This file is the **index** of development rules — NOT a payload to preload. Detailed rules live in `common/` and `exceptions/`. Consult files on demand per [`common/context-loading-protocol.md`](common/context-loading-protocol.md) — 4-tier model: **Tier 1** global mandatory safety baseline (always loaded: `data-extraction-policy`, `sap-version-reference`, `naming-conventions`, `context-loading-protocol`, `model-routing-rule`), **Tier 2** role-mandatory per agent group, **Tier 3** triggered reads
---

# SuperClaude for SAP (sc4sap) — Development Rules

This file is the **index** of development rules — NOT a payload to preload. Detailed rules live in `common/` and `exceptions/`. Consult files on demand per [`common/context-loading-protocol.md`](common/context-loading-protocol.md) — 4-tier model: **Tier 1** global mandatory safety baseline (always loaded: `data-extraction-policy`, `sap-version-reference`, `naming-conventions`, `context-loading-protocol`, `model-routing-rule`), **Tier 2** role-mandatory per agent group, **Tier 3** triggered reads, **Tier 4** per-task kit. Everything referenced below is MANDATORY for every sc4sap session (agents, skills, direct user requests, pipelines).

## Development Standards — References

| Topic | Reference |
|---|---|
| Naming conventions (Z/Y prefixes, module prefix, includes, FGs, tables, DEs, DOs) | [`common/naming-conventions.md`](common/naming-conventions.md) |
| Field typing priority (Standard DE → CBO DE → new CBO DE → Data Type+Length; never `LIFNR CHAR 10`) | [`common/field-typing-rule.md`](common/field-typing-rule.md) |
| Function Module source convention (inline IMPORTING/EXPORTING/TABLES in FUNCTION statement; no placeholder, no shadow locals) | [`common/function-module-rule.md`](common/function-module-rule.md) |
| SAP version awareness (ECC vs S/4HANA — tables, TCodes, BAPIs, patterns) | [`common/sap-version-reference.md`](common/sap-version-reference.md) |
| ABAP release awareness (syntax availability per release, never emit newer features than `ABAP_RELEASE`) | [`common/abap-release-reference.md`](common/abap-release-reference.md) |
| Clean ABAP coding standards (paradigm-split) | shared baseline [`common/clean-code.md`](common/clean-code.md) + one of [`common/clean-code-oop.md`](common/clean-code-oop.md) or [`common/clean-code-procedural.md`](common/clean-code-procedural.md) chosen from the Phase 1B Paradigm dimension; also pulls in `constant-rule.md`, `text-element-rule.md`, `abap-release-reference.md` |
| Procedural FORM naming (screen-bound suffix) | [`common/procedural-form-naming.md`](common/procedural-form-naming.md) |
| OOP two-class split pattern | [`common/oop-pattern.md`](common/oop-pattern.md) |
| Include structure (Main + conditional includes) | [`common/include-structure.md`](common/include-structure.md) |
| ALV display rules | [`common/alv-rules.md`](common/alv-rules.md) |
| OK_CODE binding pattern (TOP decl + screen NAME + PAI FORM routing; never `CASE sy-ucomm`) | [`common/ok-code-pattern.md`](common/ok-code-pattern.md) |
| SPRO lookup protocol (consultant agents, IMG/Customizing) | [`common/spro-lookup.md`](common/spro-lookup.md) |
| Customization lookup protocol (existing Z*/Y* BAdI impl, CMOD, form-exits, appends — mandatory before recommending new enhancements/extensions) | [`common/customization-lookup.md`](common/customization-lookup.md) |
| Data extraction policy (blocklist rule, `acknowledge_risk` hard rule, refusal template) | [`common/data-extraction-policy.md`](common/data-extraction-policy.md) |
| Transport client rule (every `CreateTransport` must receive explicit `client` from `.sc4sap/sap.env` SAP_CLIENT — never an implicit default) | [`common/transport-client-rule.md`](common/transport-client-rule.md) |
| Blocklist (per-category table lists) | [`exceptions/table_exception.md`](exceptions/table_exception.md) (index) + `exceptions/*.md` section files |
| Industry business-context references (14 industries) | [`industry/README.md`](industry/README.md) + `industry/*.md` |
| Country / localization references (16 countries + EU common) | [`country/README.md`](country/README.md) + `country/*.md` |
| Active modules — cross-module integration matrix (MM↔PS, SD↔CO, QM↔PP, …) | [`common/active-modules.md`](common/active-modules.md) |
| Context loading protocol (CLAUDE.md is an index; every dispatch declares its minimal context kit) | [`common/context-loading-protocol.md`](common/context-loading-protocol.md) |
| Model routing (Sonnet for reads + repetitive bulk; Opus for novel code, cross-file reasoning, ambiguity) | [`common/model-routing-rule.md`](common/model-routing-rule.md) |

Before any work: verify `.sc4sap/config.json` exists and contains `sapVersion`, `abapRelease`, `industry`. Every recommendation, piece of generated code, and tool invocation must respect those fields.

## Critical Rules (summary — full text in referenced files)

1. **Custom objects use `Z`/`Y` prefix.** Full rules: `common/naming-conventions.md`.
2. **Match SAP version and ABAP release.** Never use S/4-only tables on ECC or syntax newer than the configured `abapRelease`. Full rules: `common/sap-version-reference.md`, `common/abap-release-reference.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [babamba2/superclaude-for-sap](https://github.com/babamba2/superclaude-for-sap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
