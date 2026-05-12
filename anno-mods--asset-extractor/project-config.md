---
trigger: always_on
description: This guide documents how to navigate and extract data from Anno 117's asset structure using the asset-extractor library.
---

# Asset Extractor Guide for AI Agents

This guide documents how to navigate and extract data from Anno 117's asset structure using the asset-extractor library.

## Project Overview

Asset Extractor is a modular Python library for reading Anno game `assets.xml` files, resolving dependencies, and converting them into legible formats. It supports localization and icon conversion for Anno 117.

## Architecture

Four modules under `assetextractor/`:

1. **Extraction** (`assetextractor/extraction/`) — opens RDA files via `RDAConsole.exe`, extracts XML/DDS/CFG to a cache directory.
   - `utils.py` — `Config` class for path management
   - `extract.py` — dynamic RDA extraction
2. **Parsing** (`assetextractor/parsing/core/`) — reads XML files and reconstructs the hierarchical structure with full inheritance resolution.
   - `templates.py` (asset structure / OOP-class analogue), `assets.py` (concrete instances), `properties.py` (nested building blocks), `attributes.py` (typed values), `common.py` (base classes), `texts.py` (localization), `uitext.py` (UI text mapping).
3. **Conversion** (`assetextractor/conversion/`) — generates excerpts (HTML, JSON) from parsed asset data.
   - `assetbrowser/` — Jinja2 HTML converter; outputs to `config.assetbrowser_dir`; requires a `Config` passed to `Converter`.
   - `statistics/` — item extraction and Google Sheets export.
4. **Versioning** (`assetextractor/versioning/`) — tracks asset changes across game versions in SQLite (`versioning/anno117/assets.db`); supports snapshot, diff, export to CSV/JSON, and history queries.

## Topic Documentation

In-depth guides live under `docs/`. Read the relevant file before working on a topic:

- Development commands, release workflow, configuration, dependencies — `docs/development.md`
- Implementation notes (key concepts, RDA quirks, parsing pitfalls, iteration) — `docs/implementation_notes.md`
- Buffs and `Asset.buff_ui` — `docs/buffs.md`
- Asset pools (RewardPool / AssetPool, flattening, reverse index) — `docs/asset_pools.md`
- UI text mapping (rarity, niche, BuffUpgradeType, etc.) — `docs/ui_text_mapping.md`
- Boost conditions (`ItemWithBoost`) — `docs/conditions.md`
- Item sources (traders, tech, quests, expeditions) — `docs/item_sources.md`
- API patterns (`find` / `find_value`, iteration, formatting) — `docs/api_patterns.md`
- Test conventions — `tests/AGENTS.md`

## Running Scripts and Modules

Always run scripts as modules from the project root (`C:\dev\asset-extractor`) using `-m`. Running by file path will fail with `ModuleNotFoundError`.

```bash
uv run python -m assetextractor.extraction.extract
uv run python -m assetextractor.versioning
uv run pytest                                    # tests handle imports automatically
uv run pytest -k "building_size"                 # tests matching keyword
```

Path → module: drop the `.py`, replace `/` with `.`. Example: `assetextractor/extraction/extract.py` → `assetextractor.extraction.extract`.

## Class Reference

### Parsing (`assetextractor/parsing/core/`)

**common.py**
- `NamedElement[CacheT]` — base class; `name`, `find()`, `find_value()`, `get()`, `full_path`, `property_path`
- `Group[CacheT]` — container, `print_tree()`
- `ElementCache[ElementT, GroupT]` — `add()`, `get()`, `find()`, `print_tree()`
- `Dataset` / `DatasetCache` — datasets.xml; `literals` property
- `AttributeMissingError` — raised for missing XML attributes

**attributes.py**
- `parse_bool(text)` — Anno boolean parser
- `Property` — nested property; `resolve_inheritance()`, `print_tree()`
- `Attribute[CacheT, ValueT]` — base; `__call__()`, `is_compound`, `is_default`
- `PrimitiveAttribute` — Boolean/Choice/Float/FloatOrPercental/Int64/Integer/String; `ui_text_id`, `ui_icon_guid`, `ui_text_variants`, `buff_ui`
- `ColorAttribute`, `TextAttribute`, `TimeAttribute`
- `UpgradeAttribute` — numeric + percental; `buff_ui`
- `FlagsAttribute` — semicolon-separated dataset literals; `buff_ui`
- `FileNameAttribute` — file path with .dds resolution; `is_image`, `get_image()`, `get_data_url()`
- `ReferenceAttribute` / `QuestAttribute` — GUID references
- `ListItem` / `ListAttribute` — ordered lists; `buff_ui`
- `GenericDictAttribute` / `DictAttribute` — dict-like; `buff_ui`
- `TemplateAttribute` — AutoCreateAsset; `set_template()`, `process_properties()`
- `AttributeFactory` — `create()`, `create_default_node()`

**properties.py**
- `ValueDefinition`, `MetaProperty`, `PropertyGroup`, `MetaPropertyCache` — properties-toolone.xml metadata; `resolve_template_attributes()`

**templates.py**
- `WeightedReference`, `Template` (`add_instance()`, `assets`), `TemplateGroup`, `TemplateCache`

**assets.py**
- `Asset` — concrete instance; `guid`, `text`, `template`, `resolve_inheritance()`, `set_referenced_by()`, `print_tree()`, `short_description`, `long_description`, `buff_ui` (returns `list[BuffUI]`)
- `AssetGroup`, `AssetCache` — `resolve_inheritance()`, `resolve_references()`, `resolve_dlc_unlocks()`, static `load(config)`

**texts.py**
- `Text` — `id`, `values`, `has_html_escapes()`, `count_format_args()`, `format(list)`, `__call__()`
- `TextCache` — loads texts_*.xml; `converter` property
- `StandardTextConverter` — `__call__(text)` for language-specific output


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anno-mods/asset-extractor](https://github.com/anno-mods/asset-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
