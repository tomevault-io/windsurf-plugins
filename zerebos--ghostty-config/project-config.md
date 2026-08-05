---
trigger: always_on
description: Operating guide for automated agents working in this repository. Read this before making any changes.
---

# AGENTS.md: Ghostty Config

Operating guide for automated agents working in this repository. Read this before making any changes.

## Quick orientation

- **Runtime**: Bun ≥ 1.3 required. All commands run from the repo root.
- **Module system**: `"type": "module"` ES imports only, no `require`.
- **Framework**: Svelte 5 with runes. This is **not** Svelte 3/4. Do not use `writable`, `derived`, `readable`, or any legacy store primitives. See the Svelte 5 section below.

## Required checks: run before marking anything ready

```bash
bun run check   # svelte-kit sync + svelte-check (type errors)
bun run lint    # ESLint strict flat config
bun run test    # unit tests via Vitest (bun is just the script runner)
```

All three must pass cleanly. Do not open or mark a PR ready if any fail.

## Architecture: read before touching settings

These files form the core of the settings system with strict relationships between them. The triad that most changes must stay in sync is `types.ts` (shapes) → `registry.ts` (per-setting data: config key + widget) → `navigation.ts` (placement tree).

### `src/lib/settings/types.ts`

Defines `SettingInfo` — the registry entry shape — as a union `ScalarSettingInfo | RepeatableSettingInfo` discriminated on the `repeatable?: true` value-shape flag. An entry carries config-key metadata (`key`, `name`, `description`, `note`, `platform`, `since`, `default`) **plus** an optional `widget?: WidgetDef`, the data-only discriminated union for widget selection + widget metadata. The union's job is compile-time safety: `satisfies SettingsRegistry` rejects any entry whose `default` shape (`string` vs `string[]`) or widget kind (`ScalarWidgetDef` vs `RepeatableWidgetDef`) disagrees with `repeatable` — do not collapse it back to one interface. There is **no `SettingDef` union and no `TypeToValue`** — the store is flat strings (see the store-flatten note below). Widget-metadata types (`DropdownOption`, `FeatureDef`, `PillOption`, `SpecialValue`) live here too, consumed by `WidgetDef` and the renderer.

### `src/lib/settings/registry.ts`

The flat camelCase-keyed record of every setting. The export pattern is:

```ts
export const registry = { ... } satisfies SettingsRegistry;
```

**`satisfies` without `as const` is intentional and must be preserved.** It keeps `repeatable: true` as a literal so the `SettingValues` mapped type at the bottom of the file can resolve each key to `string[]` (repeatable) or `string` (everything else) — this is what replaced the old `type`/`TypeToValue` machinery. The store holds only strings; a registry `default` is a literal string (`"13"`, `"false"`, `""` for unset) or a `string[]` for repeatable settings. Initializers may still mutate `.default` (it's a plain `string`), so don't add `as const`.

Each entry also carries its `widget?: WidgetDef` (omit it for a plain `Text`/`RepeatableText` input — that's the renderer's default). Everything about a setting except its placement lives on the entry, so an upstream Ghostty sync (defaults, descriptions, enum option lists) is a single-file edit. `validateRegistry()` runs at import time in dev (and in `registry.test.ts`) to check the data invariants types can't: option-backed widgets (`dropdown`/`pill`/`theme`) must include their own non-empty `default` among their `options` — the tripwire for upstream enum drift — and `duration` widgets must satisfy `allowEmpty` ⟺ unset (`""`) default.

### `src/lib/settings/navigation.ts`

A typed tree of panels → groups → setting keys that drives the sidebar UI. **Placement only** — entries are bare camelCase registry identifiers; widget selection and widget metadata live on the registry entry, not here. (Group-level `preview` keys stay here: which preview renders above a group genuinely is placement.) `validateNavigation()` runs at import time in dev and throws if:

- any key in the nav tree doesn't exist in the registry (typo protection)
- any registry key isn't referenced anywhere in the nav tree (exhaustiveness)

**Adding a setting to `registry.ts` without placing it in `navigation.ts` will throw at dev startup.** Do not add a bypass or suppress this validation.

### `src/lib/settings/options.ts`

Option lists derived purely from build-time data (the generated `themes`/`macicons` modules): computed once at module scope and referenced directly by widget defs in `registry.ts`. No runtime population step — the registry is complete at import time. Add new static option lists here (shared or large lists belong here; a short inline enum can stay on the widget def).

### `src/lib/settings/initializers.ts`

Populates registry fields that require **genuinely runtime** data: OS detection for platform-dependent defaults, and (future) async sources like a native font list. Sync initializers run first via `runSyncInitializers()`; async ones after. Add new runtime-populated fields here, do not mutate the registry from a component — and if the data is static, use `options.ts` instead.

### `src/lib/settings/codecs.ts`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zerebos/ghostty-config](https://github.com/zerebos/ghostty-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
