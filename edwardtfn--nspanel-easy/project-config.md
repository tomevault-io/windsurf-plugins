---
trigger: always_on
description: Guidance for AI coding agents working on NSPanel Easy. Human contributors should read [CONTRIBUTING.md](CONTRIBUTING.md) first; this file complements it and does not replace it.
---

# AGENTS.md

Guidance for AI coding agents working on NSPanel Easy. Human contributors should read [CONTRIBUTING.md](CONTRIBUTING.md) first; this file complements it and does not replace it.

## Project overview

NSPanel Easy is an open-source integration of the Sonoff NSPanel with Home Assistant. It has three layers that must stay in sync:

| Layer | Location | Language |
| --- | --- | --- |
| ESPHome firmware | `nspanel_esphome*.yaml`, `esphome/`, `components/nspanel_easy/` | YAML, C++, Python (codegen) |
| Home Assistant Blueprint | `nspanel_easy_blueprint.yaml` | YAML, Jinja2 |
| Nextion display (HMI/TFT) | `hmi/` | Nextion Editor project files |

Supported display models: EU (landscape), US portrait, and US landscape.

## Repository layout

- `nspanel_esphome.yaml` - Entry point users include; pulls packages from `esphome/`.
- `esphome/` - ESPHome packages, split by concern (`hw_*`, `page_*`, `addon_*`, `api*`, `core`, `standard`, `version`).
- `components/nspanel_easy/` - External component (C++ sources and `__init__.py`).
  Page-specific code is gated by `NSPANEL_EASY_PAGE_*` build flags defined in the matching `esphome/nspanel_esphome_page_*.yaml`.
- `nspanel_easy_blueprint.yaml` - Single Blueprint file (very large; edit surgically).
- `hmi/` - `.hmi` sources and compiled `.tft` files. `hmi/dev/` holds developer tooling, fonts, images and the `nextion2text` output.
- `docs/` - User documentation. Update it whenever user-visible behavior changes.
- `.test/` - ESPHome configurations used by the CI build matrix.
- `prebuilt/` - Prebuilt firmware configuration and binaries (experimental).
- `versioning/` - Version files managed by CI. Do not edit.
- `.github/` - Workflows, issue templates, CI scripts and pinned Python tooling (`requirements.txt`).
- `.rules/` - Linter configurations (yamllint, markdownlint, markdown link check).

## Build and validation

Run the relevant checks before proposing a change:

```bash
# One-time setup (Ubuntu/Debian; markdownlint-cli2 requires Node.js/npm)
sudo apt-get update && sudo apt-get install -y clang-format
npm install --global markdownlint-cli2
pip install -r .github/requirements.txt
pip install esphome  # Intentionally not pinned in requirements.txt

# C++ formatting (config: .clang-format, ColumnLimit 120); same scope as validate_clang_format.yml
find ./components/nspanel_easy ./.test/unit \( -name '*.h' -o -name '*.c' -o -name '*.cpp' \) -print0 | xargs -0 -r clang-format --style=file -i

# YAML lint (max line length 200)
yamllint -c ./.rules/yamllint.yml .

# Python lint
flake8 --max-line-length=200 components/nspanel_easy

# Markdown lint (config: .rules/.markdownlint.jsonc)
markdownlint-cli2 --config .rules/.markdownlint.jsonc "**/*.md"

# Compile one of the CI test configurations
esphome compile .test/esphome_idf_basic.yaml
```

CI builds every configuration in `.test/` against ESPHome latest and dev.
When a change affects a feature that is only included by specific packages (climate, cover, Bluetooth, customizations, Arduino), compile the matching `.test/` file, not only the basic one.

## Versioning and compatibility

- Versioning is CalVer (`YYYY.M.seq`) and fully automated by `.github/workflows/versioning.yml` on push to `main`. Never edit `versioning/` or bump version numbers manually.
- The three layers check each other's versions at runtime.
  When a change makes one layer depend on a newer version of another, bump the relevant minimum in the same PR:
  - `min_blueprint_version`, `min_tft_version` and `min_esphome_compiler_version` in `esphome/nspanel_esphome_version.yaml`.
  - `min_version` (Home Assistant) in the Blueprint header.
- Never use `yq` for in-place writes on `nspanel_easy_blueprint.yaml` (corrupts Unicode escapes),
  `esphome/nspanel_esphome_version.yaml` or `.github/ISSUE_TEMPLATE/bug.yml` (strips blank lines, normalizes merge keys). Use `sed` for simple replacements or a dedicated Python script.

## Commits and pull requests

- Title format (enforced by `validate_pr_title.yml`): `<prefix>: <Description starting with a capital letter>`, lowercase prefix, no trailing period.
- Allowed prefixes: `fix`, `feat`, `improve`, `ci`, `docs`, `style`, `build`, `refactor`, `test`, `chore`. Use `improve` when behavior is unchanged but the implementation is better.
- PR titles become release titles verbatim; write them for end users.
- Reference related issues in the description (`Closes #N` when resolved; a plain `#N` reference otherwise).
- Do not hard-wrap text in PR descriptions, issues or release notes.
- Keep PRs scoped to one logical change. Unrelated fixes go in separate PRs.
- Target `main` from a short-lived feature branch.

## General coding rules

- Keep the project lean: do not add sensors, globals or entities when the data is already reachable through an existing path. No speculative abstractions.
- Never remove existing comments or inline documentation unless explicitly asked.
- Prefer explicit over implicit: explicit enum comparisons instead of range checks, and explicit `if`/`else if` branches instead of a catch-all `else` where it improves defense in depth.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edwardtfn/NSPanel-Easy](https://github.com/edwardtfn/NSPanel-Easy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
