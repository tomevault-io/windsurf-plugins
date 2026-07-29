---
trigger: always_on
description: Read together with the repo-wide
---


# Copilot instructions — integration / script authoring tools

Read together with the repo-wide
[`copilot-instructions.md`](../copilot-instructions.md) and
[`commands.instructions.md`](commands.instructions.md). This file specialises
guidance for the cluster of commands that **generate, scaffold, split, or
diff Cortex integration and script content**:

| Command | Where |
|---|---|
| `init` (scaffold a new pack/integration/script) | [`demisto_sdk/commands/init/`](../../demisto_sdk/commands/init) |
| `split` (split a unified YAML back to a folder layout) | [`demisto_sdk/commands/split/`](../../demisto_sdk/commands/split) |
| `generate-integration` (Python → integration YAML+code) | [`demisto_sdk/commands/generate_integration/`](../../demisto_sdk/commands/generate_integration) |
| `generate-yml-from-python` | [`demisto_sdk/commands/generate_yml_from_python/`](../../demisto_sdk/commands/generate_yml_from_python) |
| `openapi-codegen` (OpenAPI spec → integration) | [`demisto_sdk/commands/openapi_codegen/`](../../demisto_sdk/commands/openapi_codegen) |
| `postman-codegen` (Postman collection → integration) | [`demisto_sdk/commands/postman_codegen/`](../../demisto_sdk/commands/postman_codegen) |
| `generate-unit-tests` | [`demisto_sdk/commands/generate_unit_tests/`](../../demisto_sdk/commands/generate_unit_tests) |
| `integration-diff` | [`demisto_sdk/commands/integration_diff/`](../../demisto_sdk/commands/integration_diff) |

Reference data models:
- [`generate_integration/XSOARIntegration.py`](../../demisto_sdk/commands/generate_integration/XSOARIntegration.py)
  — the dataclass model of an integration YAML used by the generator.
- [`generate_integration/base_code.py`](../../demisto_sdk/commands/generate_integration/base_code.py)
  — the boilerplate Python that every generated integration starts from.
- [`commands/common/schemas/`](../../demisto_sdk/commands/common/schemas)
  — Pykwalify schemas for `integration`, `script`, `playbook`, `dashboard`,
  `incidentfield`, `incidenttype`, `classifier`, `mapper`, `layout`,
  `modelingrule`, `parsingrule`, `correlationrule`, `widget`, `report`,
  `wizard`, `job`, `trigger`, `xsiamdashboard`, `xsiamreport`,
  `xdrctemplate`, `genericdefinition`, `genericfield`, `genericmodule`,
  `generictype`, `pack_metadata`, …
- Default content templates: [`commands/init/templates/`](../../demisto_sdk/commands/init/templates).

## Domain knowledge Copilot must respect

### YAML keys for `integration`

Top-level keys (subset, see schema for full list):

| Key | Meaning |
|---|---|
| `commonfields.id` | Unique integration ID. Cannot contain spaces. |
| `commonfields.version` | Schema version, almost always `-1`. |
| `name` | Display name. |
| `display` | Human-friendly name shown in the UI. |
| `category` | One of the marketplace-allowed categories (see [`pack_metadata` schema](../../demisto_sdk/commands/common/schemas/pack_metadata.json)). |
| `description` | Short description; rendered in the marketplace card. |
| `detaileddescription` | Markdown description shown on the integration page (sourced from a sibling `*_description.md` file when split). |
| `image` | Integration logo (sourced from a sibling `*_image.png` when split, must be 120×50 PNG ≤ 10 KB by content guidelines). |
| `fromversion` / `toversion` | Server version constraints (e.g. `6.10.0`). |
| `marketplaces` | List of `MarketplaceVersions` (`xsoar`, `marketplacev2`, `xpanse`, `xsoar_saas`, `platform`). |
| `defaultEnabled` / `defaultEnabled_x2` | Auto-enable on install (per marketplace). |
| `configuration` | List of integration **params** (see below). |
| `script.script` | Python source (`-` placeholder when split into `.py`). |
| `script.type` | `python` or `powershell` (almost always `python`). |
| `script.subtype` | `python2` or `python3` (always `python3` for new content). |
| `script.dockerimage` | `demisto/python3:<tag>` or `demisto/<image>:<tag>`. |
| `script.commands` | List of integration commands (see below). |
| `script.feed` / `script.isfetch` / `script.isfetchsamples` / `script.ismappable` / `script.isremotesyncin` / `script.isremotesyncout` / `script.longRunning` / `script.longRunningPort` | Capability flags. |
| `script.runonce` | Execute once on install. |
| `tests` | List of test playbook IDs, or `["No tests"]` with a justification. |
| `supportedModules` | Optional list of supported modules. |

### Integration `configuration` (params)

Each entry is a parameter shown to the customer when configuring an
instance:

```yaml
- display: Server URL
  name: url
  type: 0                # ParameterType — see commands/common/constants.py::ParameterType
  required: true
  defaultvalue: https://example.com
  additionalinfo: Optional help text shown next to the field.
  hidden: false          # bool, or list of marketplaces where it should be hidden
```

Common `type` values (defined in
[`ParameterType`](../../demisto_sdk/commands/common/constants.py)):

| Value | Meaning |
|---|---|
| `0` | Short text (single-line string). |
| `4` | Encrypted (password / API key). |
| `8` | Checkbox (boolean). |
| `9` | Authentication (username + password). |
| `12` | Long text (multi-line). |
| `13` | Single select (from `options`). |
| `15` | Multi select (from `options`). |
| `16` | Multi-instance only (single-select). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [demisto/demisto-sdk](https://github.com/demisto/demisto-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
