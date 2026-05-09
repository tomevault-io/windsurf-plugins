---
trigger: always_on
description: - **Django monolith** in `amplifier/` with feature apps in `apps/` (catalog, web, users, api, support, media, homepage).
---

# Copilot instructions for AMPER-B2C

## Architecture & Data Flow

- **Django monolith** in `amplifier/` with feature apps in `apps/` (catalog, web, users, api, support, media, homepage).
- **URL composition**: Routes assembled in [amplifier/urls.py](../amplifier/urls.py); each app exposes its own `urls.py`.
- **Templates**: Server-rendered Jinja-style templates in `templates/`; base layout at [templates/web/base.html](../templates/web/base.html).
- **Frontend pipeline**: Vite builds `assets/` → `static/`; templates load via `{% vite_asset %}` tags. Tailwind + Flowbite for styling.
- **HTMX enabled**: Middleware + `hx-headers` in body; use HTMX patterns for partial updates.
- **API layer**: DRF in `apps/api/`; hybrid permission `IsAuthenticatedOrHasUserAPIKey` for session/API key auth (see [apps/api/permissions.py](../apps/api/permissions.py)).

## Safety & Preservation Rules (CRITICAL)

- **NEVER use `git restore`** (or `git checkout --`) on files that were modified in the working tree but were NOT changed by your current task. These files may contain the user's intentional, uncommitted work. If you see "unrelated" modified files in `git status`, **IGNORE THEM** and do not attempt to clean them up.
- **NEVER use `open_simple_browser`** for any testing, debugging, or visual verification. It is deprecated and unreliable for this environment. **ALWAYS use Chrome MCP tools** (`mcp_chrome-devtoo_...`) for all browser-based interactions and inspections.

## Model Patterns & Base Classes

- **Always extend `BaseModel`** from [apps/utils/models.py](../apps/utils/models.py) – provides `created_at`/`updated_at` timestamps.
- **Singleton models** use `SingletonModel` base class with `get_settings()` classmethod (e.g., `SiteSettings`, `Footer`, `BottomBar`).
- **File storage**: Use `DynamicMediaStorage()` for ImageField/FileField to support local/S3 switching (see [apps/media/storage.py](../apps/media/storage.py)).
- **CKEditor rich text**: Use `CKEditor5Field(config_name="extends")` for product descriptions.
- **Auto-slugs**: Use `AutoSlugField(populate_from="name", unique=True, always_update=False)` from django-autoslug.

## Admin Patterns (Django Unfold)

- Admin uses **django-unfold** – import from `unfold.admin` (ModelAdmin, TabularInline, StackedInline).
- **Image previews**: Use `make_image_preview_html()` from [apps/utils/admin_utils.py](../apps/utils/admin_utils.py) for consistent thumbnails.
- **Media Library Source Links**: Models using `DynamicMediaStorage` are automatically tracked. To enable "Source" links in the Media Library, the model MUST be registered in `admin.py`. For inlines, use a hidden admin: `has_module_permission = lambda self, r: False`.
- **Price formatting**: Emit `<span data-price="..." data-currency="...">` for JS-based Intl.NumberFormat (see [static/js/admin_custom.js](../static/js/admin_custom.js)).
- **Import/Export**: Use `ImportExportModelAdmin` mixin with `ImportForm`/`ExportForm` from unfold.contrib.
- For file inputs needing custom preview, add `data-product-image-upload="true"` attribute.

## Draft Preview System

- **Automatic autosave**: Admin forms auto-save drafts to session via JS in [static/js/admin_custom.js](../static/js/admin_custom.js).
- **Zero config required**: Works for all admin-registered models including inlines (BottomBarLink, FooterSection, etc.).
- **Preview middleware**: `DraftPreviewMiddleware` applies drafts when `?preview_token=` is in URL.
- **Key utilities** in [apps/support/draft_utils.py](../apps/support/draft_utils.py):
  - `apply_draft_to_instance(instance, form_data, temp_files)` – mutates model instance
  - `apply_drafts_to_context(context, drafts_map)` – applies drafts to template context
  - `get_new_draft_instance(request, model_class)` – creates unsaved instance with draft data
- **Custom preview routes**: Add before slug routes if model has custom preview needs.
- **New forms/modules**: Ensure draft preview is supported for new (unsaved) records; templates should receive model-specific context keys (e.g., `page`, `section`, `banner`) so previews render correctly.
- **Existing records**: Ensure draft preview applies to saved objects too (not just new records), so edited content renders on the live detail templates.
- **Inline lists in preview**: When draft changes affect inline items, avoid filtering by `is_active` before applying drafts. Apply drafts first, then filter and set `_draft_inline_applied` on remaining items so `apply_drafts_to_context` does not rebuild and reintroduce filtered items.

## Dev Workflows

| Task                | Command                                                                                                                                                                                                                                                                                                                                         |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AMPLIFIER-sp-z-o-o/amper-b2c](https://github.com/AMPLIFIER-sp-z-o-o/amper-b2c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
