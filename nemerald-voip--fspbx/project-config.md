---
trigger: always_on
description: This repo is a Laravel, Vue/Inertia, VueForm, and FreeSWITCH application. Before changing behavior, read the nearby code and follow the existing page, service, route, and update patterns.
---

# FS PBX Agent Guide

This repo is a Laravel, Vue/Inertia, VueForm, and FreeSWITCH application. Before changing behavior, read the nearby code and follow the existing page, service, route, and update patterns.

## Working Style

- Prefer small, focused changes that match the current app structure.
- Do not revert unrelated local changes. This repo is often worked on with a dirty tree.
- Use `rg`/`rg --files` for searches.
- Use `apply_patch` for manual edits.
- Keep user-facing copy short and practical.
- For release notes, write for users first. Avoid implementation details unless they explain visible behavior.

## Laravel And Vue Pages

- Rebuild legacy `/public/app/.../*.php` admin pages as native Laravel/Vue pages when asked.
- Follow existing first-class pages, especially Devices and Conference-style pages, before inventing new patterns.
- Put the Inertia page route in `routes/web.php`.
- Put axios/data/action routes in `routes/api.php`.
- Use Spatie QueryBuilder for searchable/sortable/paginated data tables when that is the local pattern.
- For modern Inertia pages, pass page-specific permissions from the controller as a `permissions` prop and read `props.permissions` in Vue. Do not add new page-specific permission keys to `HandleInertiaRequests` unless a truly shared layout/component needs them globally.
- For date filters, follow the CDR pattern: pass `timezone` as a top-level Inertia prop for the DatePicker/UI, but do not include it in the API `filter` payload. Only send filters the QueryBuilder endpoint explicitly allows.
- Use VueForm for create/update modals.
- Keep forms clear and operational. Avoid marketing-style UI.

## Extensions And Voicemail

- Extension numbers and voicemail IDs are only unique inside a domain. Any lookup, relationship use, eager load, listener, observer, job, or response reload that connects `v_extensions.extension` to `v_voicemails.voicemail_id` must also constrain `domain_uuid`.
- `Extensions::voicemail()` is intentionally broad because Eloquent does not enforce the domain match there. Do not update, delete, sync copies, or read user-facing voicemail data from `$extension->voicemail` unless the relation was loaded with a domain filter or the voicemail was queried explicitly by `domain_uuid + voicemail_id`.
- Extension suspension and unsuspension must only toggle voicemail for the same tenant. DND updates should target the concrete extension row by `extension_uuid` and `domain_uuid`, not by extension number alone.

## Settings Pages

- Default Settings and Domain Settings are native Laravel/Vue pages. Keep page routes in `routes/web.php` and data/action routes in `routes/api.php`.
- Default Settings is the global catalog. Domain Settings is the working surface for overrides and custom domain rows.
- Domain Settings should open focused on overrides/custom rows by default; users can switch the source filter to see inherited defaults.
- Domain Settings status toggles should update the row in place and refresh silently. Do not blank the whole list for a single on/off click.
- Use VueForm for settings edit and copy modals, including copy-to-domain workflows. Keep VueForm labels non-floating on these forms.
- In settings edit forms, Category should be a searchable/createable select populated from existing categories, and Subcategory should be labeled as `Setting Name`.
- Do not invent an order value for existing or newly-created settings. Preserve `NULL`/blank order unless the user enters one or a default-backed override is intentionally copying the default row.
- Domain option labels should prefer `domain_description` and only fall back to `domain_name` when no human label exists.
- The native Domain Settings route requires a domain UUID (`/domains/{domain}/settings`). Generic navigation should go to `/domains`, where each domain row links to its settings.
- Preserve legacy settings semantics: non-array domain overrides match defaults by category/subcategory/type, while `array` settings remain distinct rows.

## Devices And Provisioning

- Device Key Templates are the preferred reusable key layout workflow. Show Key Template controls before legacy Device Profile controls, but keep legacy Profile support in place unless explicitly asked to remove it.
- Key Template and Device Profile assignment should be mutually exclusive in device create, update, bulk update, and extension device modals. If one is selected, clear/disable the other.
- Device tables and extension assigned-device mini tables should show a combined Profile / Key Template column so users can see which assignment is active.
- Device Key Templates are domain-scoped. Use `device_key_template_assign` for assignment UI/actions and pass permissions from the controller props rather than global Inertia middleware when working on the Devices page.
- Provisioning merge order for effective keys is: old assigned Device Profile keys, new assigned Device Key Template keys, old per-device legacy `v_device_keys`, then new per-device `device_keys`.
- Blank or `N/A` Key Template keys intentionally occupy their slot and can clear older profile keys.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nemerald-voip/fspbx](https://github.com/nemerald-voip/fspbx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
