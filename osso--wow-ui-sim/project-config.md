---
trigger: always_on
description: - **NEVER modify files in `Interface/AddOns/Wowless/`** — this is an external test suite, not our code.
---

# WoW UI Simulator

## Important Rules

- **NEVER modify files in `Interface/AddOns/Wowless/`** — this is an external test suite, not our code.
- **NEVER modify files in `Interface/AddOns/WowlessData/`** — regenerate with `python3 tools/gen_wowless_data.py` (reads from `~/Repos/wowless/data/`). Update the source repo first: `cd ~/Repos/wowless && git pull`.
- Blizzard UI runtime files live in `~/.cache/wow-ui-sim/blizzard-ui`, populated from the committed manifest with `wow-cli casc sync-blizzard-ui`. Do not rely on `Interface/BlizzardUI` or `vendor/wow-ui-source` for runtime loading.
- **NEVER override, monkey-patch, or otherwise change Blizzard/vendor Lua behavior as a performance optimization.** Blizzard Lua is the compatibility target. For perf work, optimize simulator-side primitive/method/dirty/dispatch costs (`SetAlpha`, `SetFormattedText`, `SetPoint`, `SetFontObject`, etc.) instead. Only patch Blizzard/vendor behavior when matching real WoW semantics/correctness, never as a performance shortcut.

## Wiki

LLM-maintained knowledge base at `docs/wiki/`. See `docs/wiki/SCHEMA.md` for conventions and workflows. Read `docs/wiki/index.md` first when answering questions about the project.

### Wiki Maintenance

After completing work that produces knowledge worth preserving, update the wiki:

- **Investigations/debugging**: Create or update a page in `investigations/` with root cause, symptoms, and fix.
- **New systems or major changes**: Create or update a page in `systems/` describing how the system works.
- **Architecture decisions**: Record rationale in `design/`.

Follow the workflow in `SCHEMA.md`: check existing pages first (update > create), maintain cross-references, update `index.md` and `log.md`. Skip wiki updates for routine bug fixes, config tweaks, and cosmetic changes.

## Architecture Docs

- `docs/layout-system.md` - Anchor/layout system: AnchorPoint, single vs multi-anchor resolution, cycle detection, coordinate systems
- `docs/texture-atlas-system.md` - Texture loading, atlas lookup (~50K entries), nine-slice kits, tiling, UV remapping
- `docs/addon-loading-pipeline.md` - TOC parsing, XML/Lua loading, template registry, SavedVariables, Blizzard addon load order
- `docs/rendering-pipeline.md` - QuadBatch, WGSL shaders, tiered GPU atlas, glyph atlas, strata sorting, hit testing, alpha propagation
- `docs/widget-system.md` - Frame struct (~140 fields), WidgetType enum (18 types), WidgetRegistry, default children, visibility
- `docs/lua-api.md` - WowLuaEnv, FrameHandle, 200+ globals, 300+ frame methods, C_* namespaces, animations, implementation status
- `docs/event-system.md` - Event queue, script handler types (36 handlers), __scripts table, dispatch flow, OnUpdate tick, startup sequence
- `docs/xml-template-system.md` - XML parsing, template registry, inheritance chains, XML-to-widget conversion, inline scripts
- `docs/frame-data-flow.md` - Mixin, events, metamethods, script dispatch
- `docs/button-text-rendering.md` - Button text draw order, three-slice text-behind-background bug
- `docs/glow-plan.md` - Glow/shine effect plan
- `docs/anchor-resolution.md` - Anchor resolution walkthrough: data structures, core functions, single/multi-anchor, SetPoint API, coordinate inversion

## C API Boundary

- Treat the WoW `C_*` API surface as a first-class simulator subsystem, not as "Lua globals" or miscellaneous glue.
- `c_api` belongs at the same architectural level as `lua_api`, because it is a compatibility contract exposed to Lua, not a subcategory of Lua itself.
- When a `C_*` function is missing or wrong, default to implementing the backing system or state model. Do not reach for shims just to satisfy a failing test.
- Use **temporary shims** only as explicit stopgaps with a clear retirement path. Keep them isolated and named as temporary.
- Use **permanent shims** only for intentionally unsupported domains with a documented rationale. The existing 3D/model gap is the baseline example, not the default pattern.
- Do not hide the importance of the C API behind file placement, wrappers, or "just move it under globals" refactors. Preserve the architectural boundary in both module layout and language.

## Docker

CI image for running `run-tests` in addon CI pipelines. Published to `ghcr.io/osso/wow-ui-sim`.

### GitHub Action

```yaml
- uses: osso/wow-ui-sim@12.0.5
  with:
    addon: MyAddon
```

Action refs match the WoW interface version (e.g. `@12.0.5`). The
action resolves `github.action_ref` (passed through `env:` because
composite actions don't expose it to `run:` directly) and pulls the
matching image tag, so `@12.0.5` runs `ghcr.io/osso/wow-ui-sim:12.0.5`.

When pinning the action to a branch or SHA, pass `image-tag`
explicitly to choose which image to run:

```yaml
- uses: osso/wow-ui-sim@master
  with:
    addon: MyAddon
    image-tag: 12.0.5
```

Without a resolvable version ref and no `image-tag`, the action
fails fast rather than silently pulling `:latest`.

End-to-end example: [Osso/test-wow-addon](https://github.com/Osso/test-wow-addon)
— a minimal addon (TOC + one Lua file + `tests/`) and a workflow that
calls this action. Use it as a template or as a smoke test when
changing the action / Docker image.

### Docker

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Osso/wow-ui-sim](https://github.com/Osso/wow-ui-sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
