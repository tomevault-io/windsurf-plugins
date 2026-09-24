---
trigger: always_on
description: The renderer ships eight translated locales: **de** (German), **es** (Spanish), **fr** (French),
---

# Open-Science — Agent Notes

## i18n — translating new user-visible strings

The renderer ships eight translated locales: **de** (German), **es** (Spanish), **fr** (French),
**zh-Hans** (Simplified Chinese), **zh-Hant** (Traditional Chinese), **ja** (Japanese), **ko**
(Korean), and **ru** (Russian). Every
user-visible string added to the renderer must have a corresponding entry in the `renderer`
namespace for all translated locales unless the same meaning is intentionally shared with Electron
main through the `common` namespace:

```
src/shared/i18n/locales/zh-Hans.json
src/shared/i18n/locales/zh-Hant.json
src/shared/i18n/locales/ja.json
src/shared/i18n/locales/ko.json
src/shared/i18n/locales/fr.json
src/shared/i18n/locales/ru.json
src/shared/i18n/locales/de.json
src/shared/i18n/locales/es.json
```

Each locale file has exactly three top-level namespace objects: `common`, `native`, and `renderer`.
`common` is loaded by main and renderer, `native` only by main, and `renderer` only by the React
adapter. Put a key in `common` only when its UI meaning and reviewed translation are the same in both
processes; an identical English key is not enough.

The guard suite in `src/renderer/src/i18n/resources.test.ts` runs on every `npm test` and **will
fail the PR** if any of the following are violated.

### Key format

Keys are the **English source text verbatim** — there is no English catalog. `keySeparator` and
`nsSeparator` are both disabled, so dots and colons in copy are literal characters.

```tsx
// ✓ correct
t('Data folder not found')

// ✗ wrong — semantic path, not copy
t('workspace.dataRoot.missing')
```

### How to wrap strings

| Surface                                                             | How to translate                                                                       |
| ------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| JSX text node                                                       | `{t('Copy here')}`                                                                     |
| JSX attribute visible to users (`aria-label`, `placeholder`, `alt`) | `aria-label={t('Close dialog')}`                                                       |
| Sentence with an embedded link or element                           | `<Trans i18nKey="See <docs>the guide</docs>" components={{ docs: <a href="…" /> }} />` |
| Interpolated value                                                  | `t('{{count}} files', { count: n })`                                                   |

For `Trans`, **never use an HTML void-element name** (`link`, `br`, `img`, `input`, …) as a
placeholder tag — the HTML parser self-closes it and the wrapped label falls outside the anchor.
Use a descriptive name like `<docsLink>`, `<guideAnchor>`.

### Catalog entry format

Add one key to the appropriate namespace in each locale file. The key is the exact English string
(or the base English string with an i18next suffix appended). Entries are plain JSON strings and
remain flat inside each namespace; the three namespace objects are the only nesting.

```jsonc
// zh-Hans.json
{
  "common": {},
  "native": {},
  "renderer": {
    "Data folder not found": "未找到数据文件夹"
  }
}
```

Each catalog must be updated independently. **Every translated locale falls back directly to
English**, so a missing key renders in English instead of borrowing another translated locale.

### Plurals

Chinese, Japanese, and Korean have a single plural category. Use the `_other` suffix only — never
`_one`, `_few`, etc. German uses `_one` and `_other`. French and Spanish have `_one`, `_many`, and
`_other` categories, so all three entries are required; `_many` is selected for values such as
1,000,000 and can usually reuse the `_other` translation. Russian uses `_one`, `_few`, `_many`, and
`_other`; every counted Russian key must provide all four forms. The English singular is passed as
`defaultValue_one` at the call site and never needs a catalog entry.

```tsx
// Call site — English needs no catalog entry
t('{{count}} files', { count: n, defaultValue_one: '{{count}} file' })

// Catalog entries — every category selected by the locale
"{{count}} files_other": "{{count}} 个文件"   // zh-Hans
"{{count}} files_other": "{{count}} 個檔案"   // zh-Hant
"{{count}} files_other": "{{count}}個のファイル" // ja
"{{count}} files_other": "파일 {{count}}개" // ko

"{{count}} files_one": "{{count}} Datei"     // de
"{{count}} files_other": "{{count}} Dateien" // de

"{{count}} files_one": "{{count}} fichier"      // fr
"{{count}} files_many": "{{count}} fichiers"    // fr
"{{count}} files_other": "{{count}} fichiers"   // fr

"{{count}} files_one": "{{count}} archivo"     // es
"{{count}} files_many": "{{count}} archivos"   // es
"{{count}} files_other": "{{count}} archivos"  // es

// ru uses all four CLDR categories
"{{count}} files_one": "{{count}} файл",
"{{count}} files_few": "{{count}} файла",
"{{count}} files_many": "{{count}} файлов",
"{{count}} files_other": "{{count}} файла"
```

### Context suffixes

Use `_verb` when the same English word is used as both a noun and a verb and a translated locale

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aipoch/open-science](https://github.com/aipoch/open-science) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
