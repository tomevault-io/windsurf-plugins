---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Prompt Gallery is a ComfyUI custom node plugin that provides:

- **Floating gallery UI**: Draggable button (🎨) with modal interface for browsing prompt reference images
- **Storage system**: JSON-based persistence for prompts, categories, combinations, and image-prompt mappings
- **Custom nodes**: PromptGallery (UI), PromptSelector (workflow integration), SaveToGallery (saving images)
- **Combination system**: Group multiple prompts into selectable units, auto-create on save
- **Category system**: Hierarchical prompt categorization with tree navigation
- **Toast notification system**: Modern, non-blocking user feedback
- **Dialog components**: Reusable modal dialog system
- **Automatic detection**: Scans ComfyUI output directory for images matching `@prompt_name,_number.ext` pattern

## Architecture

### Backend (Python)

**`__init__.py`**: Plugin entry point

- Registers three node classes via `NODE_CLASS_MAPPINGS` and `NODE_DISPLAY_NAME_MAPPINGS`
- Sets `WEB_DIRECTORY = "./web"` for frontend assets

**`nodes.py`**: Node classes and output processing logic

- **PromptGallery**: Output node for UI (no workflow output)
- **PromptSelector**: Workflow node that provides prompt selection widget
    - Processes partitions, resolves prompts from `orderItems` (unified flat list per partition, each item has `type` + `key`)
    - Handles random/cycle mode, format templates, auto-create combination
    - Tracks `partition_used_prompts` (actual prompts after random/cycle filtering)
    - Tracks `partition_formats` (per-partition format string)
- **SaveToGallery**: Saves generated images to the gallery system
    - Supports two input sources (priority: `metadata_json` > `prompt_string`):
        - `metadata_json`: from `PromptSelector`, contains explicit prompt selections
        - `prompt_string`: auto-matches known prompt names via loop-based substring matching
    - Images are always saved even when no prompts match (prompts list will be empty)
    - `_match_prompts_from_prompt()`: Loop-based matching (`name.lower() in prompt_string.lower()`), cached at module level with frozenset fingerprint. Skips prompts in categories with `metadata.blockGallerySave = true` (including descendant categories)
    - Uses `collect_prompt()` to register prompt associations for saved images
- **`_apply_format()`**: Applies format template (e.g., `@{content}`) to prompt names

**`storage/`**: Data persistence layer (split into modules)

| Module             | Class                 | Main Storage File  | Glob Pattern          | Purpose                                        |
| ------------------ | --------------------- | ------------------ | --------------------- | ---------------------------------------------- |
| `prompt.py`        | `PromptStorage`       | `prompts.json`     | `*.prompts.json`      | Prompt CRUD, batch operations, import batch    |
| `category.py`      | `CategoryStorage`     | `categories.json`  | `*.categories.json`   | Hierarchical category tree                     |
| `combination.py`   | `CombinationStorage`  | `combinations.json`| `*.combinations.json` | Combination CRUD, duplicate, move              |
| `image_mapping.py` | `ImageMappingStorage` | `images.json`      | `*.images.json`       | Image-prompt relationships, cover image lookup |
| `migration.py`     | —                     | —                  | —                     | Data migration utilities                       |
| `_resolve.py`      | —                     | —                  | —                     | Storage directory resolution                   |

All storage classes are thread-safe with locking mechanism. Access via `get_storage()` from `storage/__init__.py`.

**Multi-file glob storage**: Each storage class reads from its main file + all glob-matched shard files (e.g., `import_20260506_120000.prompts.json`), merges items on read, and splits back by `_source_file` tag on write. This supports the "separate storage" import option — imported data writes to new shard files instead of appending to the main file.

**`routes/`**: HTTP API endpoints (split into modules)

| Module             | Endpoints                                                                                        |
| ------------------ | ------------------------------------------------------------------------------------------------ |
| `gallery.py`       | `GET /data` — returns prompts + combinations with `coverImagePath` (no full `images` array)      |
| `prompts.py`       | Prompt CRUD, batch operations, `GET /prompt_images` (lazy-load prompt images), `POST /batch_resolve` (resolve mixed prompt/category/combination keys) |
| `categories.py`    | Category CRUD, move (supports `metadata` field update)                                           |
| `combinations.py`  | Combination CRUD, duplicate, move, images (intersection of member prompts), batch delete         |
| `images.py`        | Image info, save to gallery, delete/move/copy image, restore from metadata                       |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Luo-Lotus/ComfyUI-Prompt-Gallery](https://github.com/Luo-Lotus/ComfyUI-Prompt-Gallery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
