---
trigger: always_on
description: Official Buefy skill for Vue.js + Bulma UI components. TRIGGER when generating any Vue template, SFC, or component code that involves UI elements, forms, modals, tables, navigation, or feedback messages.
---


# Buefy

Buefy is a lightweight UI component library for Vue 3 based on Bulma CSS. Components are Vue SFCs (`<b-button>`, `<b-modal>`, etc.) that wrap Bulma's class-based styles with reactive props, events, and slots.

## When to run this skill

Trigger whenever you are:
- Writing a Vue template or SFC that includes any UI element (buttons, forms, modals, tables, navigation, etc.)
- Asked to "use Buefy" or "add a Buefy component"
- Working in a project that has `buefy` in its `package.json` dependencies
- Generating component code without a specified UI library in a Vue project

## Install

```bash
npm install buefy
```

### Full bundle (registers all components globally)

```typescript
// main.ts
import { createApp } from 'vue'
import Buefy from 'buefy'
import 'buefy/dist/buefy.css'
import App from './App.vue'

const app = createApp(App)
app.use(Buefy)
app.mount('#app')
```

### Individual components (tree-shakeable)

```typescript
// main.ts — register as plugins
import { createApp } from 'vue'
import { Button, Field, Input } from 'buefy'
import App from './App.vue'

const app = createApp(App)
app.use(Button)
app.use(Field)
app.use(Input)
app.mount('#app')
```

```vue
<!-- Or import directly in a SFC -->
<script lang="ts">
import { BButton, BField, BInput } from 'buefy'
export default { components: { BButton, BField, BInput } }
</script>
```

## Usage rules

1. **Use the `type` prop, not `class`** — Buefy components expose Bulma modifiers as a `type` prop. Use `type="is-primary"` not `class="is-primary"` on Buefy components.
2. **Size prop values** — `"is-small"` | `"is-medium"` | `"is-large"`. No bare size strings.
3. **Always wrap form inputs in `<b-field>`** — `<b-field>` provides the label, validation message, and grouping. Direct bare inputs miss label/message wiring.
4. **v-model target** — Most components bind to `modelValue`. Named v-models on `b-table`: `v-model:selected`, `v-model:checked-rows`, `v-model:current-page`, `v-model:opened-detailed`.
5. **Icons** — Use the `icon` prop with the icon name only (no `mdi-` prefix). The icon pack (MDI by default) must be registered in your project. Example: `icon="home"` not `icon="mdi-home"`.
6. **b-table columns** — `<b-table>` requires `:data="arrayOfObjects"` and `<b-table-column field="key" label="Header">` children. Columns can also be passed via the `:columns` prop.
7. **Theming** — Customize via Bulma SCSS variable overrides in a `.scss` file, not a JS config object. The `ConfigProgrammatic` utility sets defaults (icon pack, locale, etc.).
8. **Programmatic APIs** — Components like Toast, Snackbar, Dialog, Loading, Modal, and Notification can be opened imperatively. Import the programmatic class or use the composable:
   ```typescript
   import { ToastProgrammatic as Toast } from 'buefy'
   Toast.open('Hello world')
   // or with options:
   Toast.open({ message: 'Saved!', type: 'is-success', duration: 3000 })

   // Composable-style (preferred in Composition API):
   import { useToast } from 'buefy'
   const { open: openToast } = useToast()
   openToast({ message: 'Saved!', type: 'is-success' })
   ```

## Colors & Types

Use these values for the `type` prop on components that accept it:

| Value | Meaning |
|-------|---------|
| `is-primary` | Primary brand color (default blue) |
| `is-link` | Link color |
| `is-info` | Informational (cyan) |
| `is-success` | Success (green) |
| `is-warning` | Warning (yellow) |
| `is-danger` | Error / danger (red) |
| `is-dark` | Dark (near-black) |
| `is-light` | Light (near-white) |
| `is-white` | White |
| `is-black` | Black |

Size values: `is-small` · `is-medium` · `is-large`

---

## Components

### b-autocomplete

Autocomplete text input with dropdown suggestions.

**Key props:** `v-model` (String), `data` (Array — suggestion list), `field` (String — dot path into data objects for display), `expanded`, `loading`, `icon`, `size`, `rounded`, `keep-first`, `open-on-focus`, `keep-open`, `clearable`, `placeholder`

**Slots:** default (scoped: `{ option, index }`) for custom option rendering; `header`, `footer`, `empty`, `group`

**Emits:** `typing`, `select`, `focus`, `blur`, `infinite-scroll`

```vue
<b-field label="Search">
  <b-autocomplete
    v-model="search"
    :data="filteredResults"
    field="name"
    placeholder="Type to search"
    @typing="onTyping"
    @select="onSelect"
  >
    <template #default="{ option }">
      {{ option.name }} — {{ option.category }}
    </template>
  </b-autocomplete>
</b-field>
```

---

### b-breadcrumb

Breadcrumb navigation trail.

**Key props:** `align` (`"is-left"` | `"is-centered"` | `"is-right"`), `separator` (`"has-arrow-separator"` | `"has-bullet-separator"` | `"has-dot-separator"` | `"has-succeeds-separator"`), `size`

**Slots:** default — place `<b-breadcrumb-item>` children

```vue
<b-breadcrumb>
  <b-breadcrumb-item tag="router-link" to="/">Home</b-breadcrumb-item>
  <b-breadcrumb-item tag="router-link" to="/docs">Docs</b-breadcrumb-item>
  <b-breadcrumb-item active>Current Page</b-breadcrumb-item>
</b-breadcrumb>
```

---

### b-button

Standard button with Bulma styling.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buefy/buefy](https://github.com/buefy/buefy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
