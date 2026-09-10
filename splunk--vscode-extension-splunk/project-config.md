---
trigger: always_on
description: This file guides Copilot cloud agent sessions that review `.spec` files against `resources/semantic_rules.json` after a spec pull. It covers two related but distinct tasks with different risk profiles.
---

# Copilot Instructions: Semantic Rules Spec Review

This file guides Copilot cloud agent sessions that review `.spec` files against `resources/semantic_rules.json` after a spec pull. It covers two related but distinct tasks with different risk profiles.

## Project Structure

| Path                                   | Purpose                                                              |
| -------------------------------------- | -------------------------------------------------------------------- |
| `spec_files/<version>/`                | Official Splunk spec files (e.g., `spec_files/10.4/props.conf.spec`) |
| `package.json`                         | Extension manifest (includes `splunk.spec.FileVersion` enum)         |
| `resources/semantic_rules.json`        | Semantic linting rules                                               |
| `resources/semantic_rules.schema.json` | JSON schema for rules validation                                     |
| `out/semanticRules.js`                 | Rule evaluation engine                                               |
| `scripts/update-spec-files.js`         | Spec file automation script (also updates package.json)              |
| `scripts/validate-rule-ids.js`         | Rule ID uniqueness validator                                         |

## Automated Workflow

When the `update-spec-files.yaml` workflow runs, it automatically:

1. **Downloads new spec files** to `spec_files/<version>/`
2. **Updates `package.json`** - adds new versions to `splunk.spec.FileVersion` enum and sets the newest as default
3. **Runs deprecation analysis** and generates semantic rules
4. **Creates a PR** with all changes
5. **Creates a Copilot review issue** for manual review of semantic rules

The `package.json` update ensures users can immediately select new Splunk versions in VS Code settings after updating the extension.

## Task A: Coverage Gap Analysis (Low Risk)

**Goal:** Identify settings in `.spec` files with no corresponding trigger in `semantic_rules.json`.

### Steps

1. For each `.spec` file relevant to a conf file in `semantic_rules.json`, extract every setting name.

2. Extract every setting referenced in `semantic_rules.json` triggers (`settings`, `settings_regex`, `settings_equal`, `setting_exists`, `setting_missing`, `with_any`, `with_all`, `without`, `value_less_than`, `value_greater_than`, `value_not_equal`). For `with_any_prefix`/`without_prefix`, treat the prefix as covering any setting starting with it.

3. Report settings with zero references, grouped by conf file. Note whether settings are newly added (diff against previous spec version) vs. pre-existing uncovered.

4. **Do not propose a rule for every gap.** Only propose where the `.spec` prose describes a real behavioral consequence (performance cost, data loss, conflict with another setting, required companion). If silent on consequences, list as "uncovered, no rule proposed".

## Task B: Deprecation Detection (Medium Risk)

**Goal:** Find settings marked deprecated in `.spec` comments and create `warning`-severity rules.

### Steps

1. Scan `.spec` comments for deprecation language: `DEPRECATED`, `deprecated in`, `no longer has any effect`, `use X instead`.

2. For each deprecated setting, determine:
   - The exact deprecated setting name
   - Replacement setting (only if explicitly stated - do not guess)
   - Version deprecated as of

3. Check if the setting appears in multiple conf files and add rules to each relevant section.

4. Author rules using the pattern below.

## Trigger Primitives

| Primitive            | Type    | Description                                           |
| -------------------- | ------- | ----------------------------------------------------- |
| `stanza_pattern`     | regex   | Match stanza name                                     |
| `settings`           | object  | Exact key-value matches (case-insensitive by default) |
| `settings_regex`     | object  | Regex matches for values                              |
| `setting_exists`     | string  | Fire if setting present                               |
| `setting_missing`    | string  | Fire if setting absent                                |
| `with_any`           | array   | Fire if any setting exists                            |
| `with_all`           | array   | Fire if all settings exist                            |
| `without`            | array   | Suppress if any setting exists                        |
| `value_less_than`    | object  | Numeric comparison                                    |
| `value_greater_than` | object  | Numeric comparison                                    |
| `value_not_equal`    | object  | Exclude specific numeric values                       |
| `settings_equal`     | array   | Fire if all named settings have equal values          |
| `with_any_prefix`    | array   | Fire if any key starts with prefix                    |
| `without_prefix`     | array   | Suppress if any key starts with prefix                |
| `case_sensitive`     | boolean | Case-sensitive comparisons (default: false)           |

## Deprecation Rule Pattern

```json
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [splunk/vscode-extension-splunk](https://github.com/splunk/vscode-extension-splunk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
