---
trigger: always_on
description: Coding patterns specific to Nuxt UI.
---

# Conventions

Coding patterns specific to Nuxt UI.

## Auto-registered modules

Nuxt UI automatically registers `@nuxt/icon`, `@nuxt/fonts`, and `@nuxtjs/color-mode`. Do **not** add them to your `modules` array. Configure them via root-level keys in `nuxt.config.ts`:

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/ui'],
  icon: { /* @nuxt/icon options */ },
  fonts: { /* @nuxt/fonts options */ },
  colorMode: { /* @nuxtjs/color-mode options */ }
})
```

Disable any of them: `ui: { fonts: false }`, `ui: { colorMode: false }`.

## Content module integration

When using `@nuxt/content`, it **must** come after `@nuxt/ui` in the `modules` array — otherwise prose components won't be available:

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/ui', '@nuxt/content']
})
```

Add `@source` in your CSS so Tailwind generates classes used in markdown/MDC:

```css
/* app/assets/css/main.css */
@import "tailwindcss";
@import "@nuxt/ui";

@source "../../../content/**/*";
```

Use `mapContentNavigation` to transform content navigation for components like `UBreadcrumb`:

```ts
import { mapContentNavigation } from '@nuxt/ui/utils/content'
import { findPageBreadcrumb } from '@nuxt/content/utils'

const breadcrumb = computed(() =>
  mapContentNavigation(findPageBreadcrumb(navigation.value, page.value?.path))
)
```

## IDE setup

Recommended `.vscode/settings.json` for Tailwind IntelliSense autocomplete with Nuxt UI:

```json
{
  "files.associations": { "*.css": "tailwindcss" },
  "editor.quickSuggestions": { "strings": "on" },
  "tailwindCSS.classAttributes": ["class", "ui"],
  "tailwindCSS.classFunctions": ["defineAppConfig"]
}
```

## UApp wrapper

Always wrap your app in `UApp` — it provides:
- Toast container (`useToast`)
- Tooltip provider
- Programmatic overlay context (`useOverlay`)
- i18n locale support

```vue
<UApp :locale="fr">
  <NuxtPage /> <!-- or <RouterView /> for Vue -->
</UApp>
```

## Icons

Nuxt UI registers `@nuxt/icon` automatically. Format: `i-{collection}-{name}`. Prefer `lucide` collection.

```vue
<UIcon name="i-lucide-sun" class="size-5" />
<UButton icon="i-lucide-plus" label="Add" />
<UAlert icon="i-lucide-info" title="Heads up" />
```

Install collections locally for reliable SSR and best performance:

```bash
pnpm i @iconify-json/lucide
pnpm i @iconify-json/simple-icons
```

Custom local collections (Nuxt only):

```ts
// nuxt.config.ts
export default defineNuxtConfig({
  icon: {
    customCollections: [{
      prefix: 'custom',
      dir: './app/assets/icons'
    }]
  }
})
```

### Default icon overrides

Components like `Modal`, `Select`, `Accordion`, etc. use default icons from `appConfig.ui.icons`. Override them globally:

```ts
// app.config.ts
export default defineAppConfig({
  ui: {
    icons: {
      loading: 'i-lucide-refresh-cw',
      close: 'i-lucide-x',
      check: 'i-lucide-check',
      chevronDown: 'i-lucide-chevron-down',
      chevronRight: 'i-lucide-chevron-right',
      arrowLeft: 'i-lucide-arrow-left',
      arrowRight: 'i-lucide-arrow-right'
    }
  }
})
```

## Slot patterns

Most components follow consistent slot naming:

| Slot | Used by | Purpose |
|---|---|---|
| `#header` | Card, Modal, Slideover, DashboardPanel | Top section |
| `#body` | DashboardPanel | Scrollable content area |
| `#footer` | Card, Modal, Slideover, DashboardPanel | Bottom section |
| `#left` | Page, DashboardNavbar | Left sidebar or content |
| `#right` | Page, DashboardNavbar, Header | Right sidebar or content |
| `#leading` | Input, Button, Alert | Before main content (icon area) |
| `#trailing` | Input, Button | After main content (icon area) |
| `#content` | Modal, Slideover, Popover, Tooltip | Full content override |
| `#default` | Most components | Main content area |

## Items arrays

Many components accept an `items` prop. Two patterns:

**Flat array** — plain list:

```ts
const items = [
  { label: 'Edit', icon: 'i-lucide-pencil' },
  { label: 'Delete', icon: 'i-lucide-trash', color: 'error' }
]
```

**Nested array** — groups with automatic separators between them:

```ts
const items = [
  [
    { label: 'Edit', icon: 'i-lucide-pencil' },
    { label: 'Duplicate', icon: 'i-lucide-copy' }
  ],
  [
    { label: 'Delete', icon: 'i-lucide-trash', color: 'error' }
  ]
]
```

Components supporting nested arrays: `UDropdownMenu`, `UContextMenu`, `UCommandPalette`, `UNavigationMenu`.

## Composables

### useToast

```ts
const toast = useToast()

toast.add({
  title: 'Success',
  description: 'Item saved',
  color: 'success',
  icon: 'i-lucide-check-circle',
  duration: 5000,
  actions: [{ label: 'Undo', onClick: () => {} }]
})

toast.remove('toast-id')
toast.clear()
```

### useOverlay

Programmatic modals, slideovers, drawers — no template `v-model` needed. See [overlays recipe](../recipes/overlays.md) for full patterns.

```ts
const overlay = useOverlay()
const modal = overlay.create(MyComponent)
const instance = modal.open({ title: 'Confirm?' })
if (await instance.result) { /* confirmed */ }
```

### defineShortcuts

```ts
defineShortcuts({
  meta_k: () => openSearch(),
  escape: () => close(),
  meta_enter: {
    handler: () => submit(),
    whenever: [isFormValid]
  }
})
```

Keys: `meta` (Cmd/Ctrl), `ctrl`, `alt`, `shift`. Separator: `_`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ocyss/boss-helper](https://github.com/Ocyss/boss-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
