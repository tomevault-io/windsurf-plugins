---
trigger: always_on
description: Rule-authoring instructions for this directory.
---

# AGENTS.md

Rule-authoring instructions for this directory.

## Scope
- Applies to `crates/kingfisher-rules/data/rules/` and all files under it.
- This file overrides broader AGENTS guidance for rule-writing tasks in this subtree.

## Goal
- Add or update YAML detection rules with high precision, low false positives, and safe remediation support.

## Rule File Conventions
- Keep provider rules in provider-named files (for example `github.yml`, `openai.yml`).
- Prefer lowercase filenames with `.yml`.
- Keep rule IDs stable and unique. Prefer `kingfisher.<provider>.<number>` unless a descriptive suffix is already established for that provider.
- Reuse nearby provider patterns/styles instead of inventing new structure.

## Required Rule Shape
Each rule entry should define:
- `name`
- `id`
- `pattern`
- `min_entropy` (default to 3.0)
- `confidence` (default to medium)
- `examples` (at least one realistic positive example)

Strongly recommended fields:
- `pattern_requirements` (for extra filtering)
- `references`

## Pattern Quality Rules
- Prefer specific anchors/prefixes and provider context over broad generic regex.
- Keep helper/context regex narrow. Avoid patterns that match generic URLs, hostnames, query params, or assignments without strong provider-specific constraints; broad helpers can create huge match counts and cause major memory/time regressions on large repos and git history.
- When the token format is generic or common-looking (for example bare 32-hex keys), prefer contextual patterns of the form: provider keyword -> short flexible gap -> key/secret label -> short flexible gap -> token. A good default is:
  - `\b`
  - provider identifier (for example `amplitude`, `azure`, `speech`, `translator`)
  - `(?:.|[\n\r]){0,N}?`
  - common credential labels such as `(?:SECRET|PRIVATE|ACCESS|KEY|TOKEN|AUTHORIZATION|API)`
  - `(?:.|[\n\r]){0,M}?`
  - the token capture wrapped in a single unnamed capture group
- Do not add surrounding context when the token is already strongly self-identifying by prefix or structure (for example `sk-ant-api...`, `AstraCS:...`, `dvc_client_...`, `secret-test-...`). In those cases, prefer the tighter self-identifying regex.
- Use `pattern_requirements` to enforce quality constraints (`min_digits`, `min_uppercase`, `min_lowercase`, `min_special_chars`, `ignore_if_contains`, `checksum`).
- Use checksum validation in `pattern_requirements.checksum` when token formats support it. This is preferred when the provider token format includes a documented or reverse-engineered check segment, because it can sharply reduce false positives without adding brittle surrounding context.
- For checksum-based rules, prefer named captures for the main token body and checksum suffix/prefix, then compute the expected checksum in Liquid. A typical pattern is:
  - `(
      prefix_(?P<body>...)(?P<checksum>...)
    )`
  - with:
    - `actual.template: "{{ checksum }}"`
    - `actual.requires_capture: checksum`
    - `expected: "{{ body | <checksum-filter> | <encoding/filter chain> }}"`
    - `skip_if_missing: true`
- Example: GitHub PATs use a CRC32-derived base62 checksum. The rule in `github.yml` captures `body` and `checksum`, then compares `{{ checksum }}` against `{{ body | crc32 | base62: 6 }}`.
- Prefer checksum validation over extra loose context whenever the token structure itself supports it. If the checksum is only present on some token generations, keep `skip_if_missing: true` so older examples continue to load safely.
- Use `visible: false` for helper/non-secret captures used only by dependent rules.
- Use `depends_on_rule` for multi-part credential validation (for example ID + secret).

## Validation Policy (Important)
- Default: define validation logic in YAML under `validation:`.
- Do not move validation logic into Rust unless YAML cannot reliably express it.
- For new rules, first attempt `Http`/`Grpc` YAML validation before considering exception paths.
- Typed validation kinds such as `AWS`, `AzureStorage`, `Coinbase`, `GCP`, `MongoDB`, `MySQL`, `Postgres`, `Jdbc`, and `JWT` are schema-level validator families. Use them when an existing typed validator already matches the problem.
- `validation: { type: Raw, content: <name> }` is the ad-hoc exception path for provider-specific or protocol-specific flows that cannot be expressed cleanly in YAML. Raw implementations live in `crates/kingfisher-scanner/src/validation/raw.rs`.
- When Rust validation is unavoidable for a one-off provider, prefer adding a raw validator instead of inventing a new typed validator.
- Do not convert existing typed validators to `Raw` just for consistency.

## HTTP Validation Request Capabilities
The `validation.content.request` block under `type: Http` supports these fields:
- `method` (required): `GET`, `POST`, `DELETE`, `HEAD`, `PUT`, etc.
- `url` (required): target URL; supports Liquid templating (`{{ TOKEN }}`, filters, etc.)
- `headers` (optional): map of header name → value; supports Liquid templating.
- `body` (optional): request body string; supports Liquid templating. Use with `Content-Type: application/x-www-form-urlencoded` for form-encoded POST bodies or `application/json` for JSON bodies.
- `multipart` (optional): multipart form data; use for file-upload endpoints.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mongodb/kingfisher](https://github.com/mongodb/kingfisher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
