---
trigger: always_on
description: - Support Linux and macOS; keep paths, terminal input, and shell documentation
---

# Agent Instructions

## Engineering and Documentation

- Support Linux and macOS; keep paths, terminal input, and shell documentation
  portable.
- Keep cyclomatic complexity at or below 10. Ruff `C901` runs through
  `scripts.analyze`; prefer focused helpers over suppressions.
- Before finishing, review the complete diff for clarity, duplication, and
  maintainability, then rerun relevant analysis and tests.
- Keep each document authoritative for one audience: `README.md` and `site/`
  share the product slogan and capability summary, `USAGE.md` owns user-facing
  commands and behavior, and `DEVELOPMENT.md` owns contributor workflows and the
  sandbox. This file records implementation invariants; link to the canonical
  document instead of repeating its examples or reference material.
- Keep affected documentation concise and current. Remove obsolete or
  contradicted guidance rather than recording implementation history.
- Importing `kaskade` must not create directories, open files, or modify the root
  logger. Configure the named logger lazily and tolerate an unavailable log
  destination.

## CLI and Configuration

- Share admin and consumer client configuration and Kafka connection
  declarations. Help groups are `Configuration options`, `Kafka connection
  options`, `AWS options`, and `Application options`; consumer also separates
  `Consumption options` and `Deserialization options`.
- Keep `--earliest` and `--partition` declaratively mutually exclusive. Render
  the theme argument as `name` without weakening choice validation.
- `--config-file client.ini` loads optional `[kafka]`, `[registry]`, and `[aws]`
  sections. Merge file values before matching repeatable CLI properties, then
  apply `-b/--bootstrap-servers`; require a non-empty resolved
  `bootstrap.servers`. Forward client properties for downstream validation and
  keep `examples/client.ini` synchronized.
- Keep `-k` as the short form of consumer `--key`; `--kafka` has no short form.
- Keep settings loading in `settings.py`, key binding parsing in `keymaps.py`,
  and supported-theme resolution in `themes.py`. Settings failures retain valid
  values and produce an in-app warning; keep `examples/settings.yaml`
  synchronized. Theme precedence is CLI, settings, then `eva01-berserk`, with
  the original `eva01` and all Textual built-in themes available.
- `--aws region=<region>` enables Amazon MSK IAM in admin, consumer, and sandbox
  population. Validate repeatable `--aws property=value` settings before client
  construction. Do not raise the `aws-msk-iam-sasl-signer-python>=1.0` baseline
  without using newer functionality.
- Local JSON, Avro, and Protobuf use raw framing by default. Select Confluent
  framing explicitly through global or field-scoped properties; never infer it
  from payload bytes.
- Registry provider selection is explicit and defaults to Confluent. Native
  Apicurio uses supported official deserializer properties and its v3 API; keep
  provider-specific validation, framing, and metadata behavior isolated.

## Data Loading and Consumer Records

- Render topic metadata before record and consumer-group metrics. Batch
  partition-offset requests and bound consumer-group offset concurrency; do not
  create temporary consumers for admin metrics.
- Preserve the last complete metrics during refresh. Never overlap automatic,
  manual, resumed, or post-mutation refreshes; coalesce non-periodic requests in
  the shared refresh coordinator.
- Admin auto-refresh defaults to 30 seconds, pauses outside the topic list, and
  is configured by `admin.refresh-interval` or
  `admin --refresh-interval`; `0` disables it.
- `consumer --earliest` subscribes to all partitions with
  `auto.offset.reset=earliest`. Repeatable
  `--partition PARTITION[:OFFSET|earliest]` uses manual assignment and must not
  fetch unlisted partitions. Numeric offsets, including `0`, are absolute.
- Handle recognized key and value deserialization failures independently per
  record. Preserve the configured deserializer for later records, expose BYTES
  fallback diagnostics in details, tooltips, copy, and export, and keep broker
  failures and unexpected exceptions fatal.
- Details, copy, and export share the versionless contract in
  `schemas/consumer-record.schema.json`; treat the schema as authoritative and
  keep its examples and conformance tests synchronized. Preserve ordered
  headers, independent key/value Registry metadata, sibling error and BYTES
  fallback metadata, and null-content semantics without restating the full
  contract here.

## TUI Interaction

- Preserve arrows, `h`/`j`/`k`/`l`, and `g`/`G` navigation. Keep shortcuts safe
  for tmux and Zellij. Quit is `ctrl+c`; never add `ctrl+q` or a copy alias for
  `ctrl+c`.
- Use stable binding IDs represented in `KNOWN_BINDING_IDS`. Visible bindings
  need concise Title Case descriptions and useful tooltips; update examples and
  tests when adding or renaming them.
- Plain-character shortcuts must not intercept input. `?` opens Help normally;
  `f1` remains available from focused text inputs.
- Contextual entity copy uses `y` and Textual's OSC 52 API. Keep it out of the
  Footer but available in Help and Commands. `USAGE.md` owns the compatibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sauljabin/kaskade](https://github.com/sauljabin/kaskade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
