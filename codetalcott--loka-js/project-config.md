---
trigger: always_on
description: Work on loka-js (the hook-contract localization pattern for the fixiproject family — fixi, moxi, ssexi, paxi, rexi) or apply the same pattern to another minimalist library. Use when editing this repo's patched libraries, the orchestrator, locale vocab, demos, tutorials, or tests — or when a user asks how to add a new locale, add a new library, or contribute the patches upstream. Encodes the v1 coverage matrix, the bounded-surface principle that distinguishes loka from a hyperscript-style preproc
---


# loka-js: localizing the fixi-family

## Quick orientation

loka-js makes it possible for non-English-native authors to write fixiproject-family code in their own language without the libraries being aware of any language. Five libraries (fixi, moxi, ssexi, paxi, rexi); three are patched with tiny hook contracts; two need no patch and are localized from outside.

This is **not** a general i18n system — it localizes *authoring vocabulary* (attribute names, modifiers, global function names, event names), not user-visible content. The audience is beginner web developers whose native language isn't English.

The project follows the fixiproject convention: flat root, no build step, single-file sources, no bundler. Don't introduce dist/, package.json scripts beyond test+gen, or compilation.

## Coverage matrix

| Library | Patched? | Hooks | Localization surface |
|---|---|---|---|
| fixi  | Yes  | `name`, `event`, `sel`, `ignoreSel` | attribute names + trigger event values |
| moxi  | Yes  | `name`, `event`, `modifier`, `ignoreSel`, `xpath` | `live`/`on-`/`mx-ignore` attrs + dotted modifiers + globals |
| paxi  | Yes  | `isSwap` | the `morph` swap value + the `window.morph` global |
| ssexi | **No** | — | event names re-fired by orchestrator listener |
| rexi  | **No** | — | global verb names aliased via `loka.alias` |

**The decision to NOT patch ssexi and rexi is principled, not lazy.** ssexi fires events on the DOM that bubble; the orchestrator can re-fire localized variants from outside. rexi is pure JS with no DOM attributes; localization is global aliasing. Patching them would add surface for no gain. Apply the same reasoning if you extend the pattern to a new library.

## The bounded-surface principle

The translation surface for each library is **finite and flat**: a known vocabulary of attribute names, modifier names, event names, or function names. **Never** translate inside an arbitrary expression language (JS bodies in `on-*` / `live`, the q-DSL keywords like `next/closest/in`, hyperscript). That path requires a parser and inherits all the brittleness of [hyperfixi's hyperscript localization](../hyperfixi/) — which is why we don't do it.

If a new library's translation surface requires parsing, the answer is no. Document the line you didn't cross and move on.

## Load order (matters and is fragile)

```html
<script src="./orchestrator.js"></script>   <!-- pre-installs hooks on every lib namespace -->
<script src="./locales/{code}.js"></script>  <!-- one or more; calls window.loka.register -->
<script src="./moxi.js"></script>            <!-- moxi before fixi (fixiproject convention) -->
<script src="./ssexi.js"></script>           <!-- any order before fixi -->
<script src="./paxi.js"></script>            <!-- any order before fixi -->
<script src="./rexi.js"></script>            <!-- any order; rexi is standalone -->
<script src="./fixi.js"></script>            <!-- LAST among fixi-family -->
```

- **orchestrator must precede every patched library.** If a patched lib loads first, its `??=` defaults take effect and the orchestrator's hooks are silently ignored. The orchestrator emits a console warning for the fixi case (`document.__fixi_mo` already set) but not for all libs.
- **moxi before fixi** is upstream's rule — moxi needs to register `fx:init`/`fx:process` listeners before fixi dispatches them on DOMContentLoaded.
- **rexi is standalone** — no DOM lifecycle coupling.

## Per-element language resolution

[orchestrator.js](orchestrator.js) `langOf(elt)` checks in order:
1. `data-loka-lang` on element or ancestor (explicit override)
2. `lang` on element or ancestor (HTML standard — `<html lang>`, `<section lang>`)
3. Falls back to `"en"`

This is the capability that distinguishes loka from preprocessor-style approaches generally: loka resolves at attribute-read time per element, so a single page can mix `<section lang="es">` and `<section lang="ja">` without rewriting markup.

## Vocab pipeline

Single source of truth: [scripts/fx-vocab.mjs](scripts/fx-vocab.mjs) exports `LOCALES`, a table keyed by locale code. Each entry has per-library sub-objects:

```js
es: {
  profile: 'spanish',           // basename of @lokascript/semantic profile (sibling-repo)
  name: 'Spanish',
  reviewed: true,                // fixi attrs reviewed by native speaker
  globalsOptIn: true,            // emit window.loka.alias calls (off by default)
  fixi:  { attrs, events },
  moxi:  { attrs, modifiers, globals, reviewed },
  ssexi: { events, reviewed },
  paxi:  { swaps, globals, reviewed },
  rexi:  { globals, reviewed },
  props,                          // for psatina-modular, not fixi
}
```

[scripts/gen-locales.mjs](scripts/gen-locales.mjs) reads this plus the `@lokascript/semantic` profile from `~/projects/hyperfixi` (sibling checkout) and emits two files per locale:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [codetalcott/loka-js](https://github.com/codetalcott/loka-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
