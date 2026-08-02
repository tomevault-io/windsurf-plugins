---
trigger: always_on
description: > **Repository:** https://github.com/opensalt/opensalt
---

# AGENTS.md

> **Repository:** https://github.com/opensalt/opensalt
> **Purpose:** This file tells agents how to work productively and safely in this codebase: what tasks it may perform, which commands to run, our coding and PR conventions, and the guardrails that must be respected.

## 0) TL;DR
- **Environment:** Docker + Docker Compose; Vuejs 3 application for viewing and editing frameworks
- **Build & run (local/CI containers):**
  - `npm run dev` — start development application
  - `npm install` - install dependencies
  - `npm run build` — build application
- **Testing & quality (do in this order):**
  1. `npm run test` — run all tests with Vitest
  2. `npm run test:watch` — run tests in watch mode
  3. `npm run test:coverage` — run tests with coverage report
  4. `npm run lint` — run ESLint
- **Typical PR workflow:** create branch `feature/<scope>-<short-desc>`, commit in small diffs, open PR linked to an issue, include migration notes, test results, and a brief change log (see §6).
- **Guardrails:** do **not** commit secrets, DB data, or container volumes; prefer minimal diffs; never bypass CI failures; ask for human review for schema or API changes (see §7).  Do **NOT** commit anything unless explicitly requested, let the developer manage git.

---

## 1) Architecture: Modal system (READ BEFORE TOUCHING MODALS)

The editor uses a **centralized modal system**. Understanding the event chain is essential — mistakes here cause subtle, hard-to-debug bugs.

### Components involved

| Layer | File | Responsibility |
|-------|------|----------------|
| State composable | `src/composables/useModalState.js` | Owns all modal `show*` refs + data refs + open/close functions |
| Modal renderer | `src/components/tree/ModalManager.vue` | Renders every modal; receives props from parent, forwards emits |
| Top-level editor | `src/components/tree/EnhancedDocumentTreeEditor.vue` | Wires `useModalState()`, passes props to `ModalManager`, handles events |
| Individual modals | `src/components/tree/modals/*.vue` | Self-contained modal components with `show` prop and `hidden` emit |

### Event chain (critical to get right)

```
Child panel (e.g. ItemDetails.vue)
  └─ emits event (e.g. 'view-json', 'edit-item')
     └─ ItemDetailsPanel.vue forwards the emit
        └─ RightSidePanel.vue forwards the emit
           └─ EnhancedDocumentTreeEditor.vue
              └─ calls modalState.openXxxModal(data)
                 └─ ModalManager.vue renders the modal
```

**Panels must never call `useModalState()` directly.** Each call to `useModalState()` creates **independent refs** — they do not share state. Only `EnhancedDocumentTreeEditor.vue` should call `useModalState()`. Panels emit events that bubble up; the editor calls the open/close methods.

### Modal component pattern (Bootstrap Modal JS API)

Every modal follows the `ExportModal.vue` pattern:

```vue
<script setup>
import { ref, watch, onMounted } from 'vue';
import Modal from 'bootstrap/js/dist/modal';

const props = defineProps({
  show: Boolean,
  /* other props */
});
const emit = defineEmits(['hidden', /* other emits */]);

const modalElement = ref(null);
let bsModal = null;

onMounted(() => {
  bsModal = new Modal(modalElement.value);
  modalElement.value.addEventListener('hidden.bs.modal', () => emit('hidden'));
});

watch(() => props.show, (val) => {
  if (!bsModal) return;
  val ? bsModal.show() : bsModal.hide();
});
</script>
```

- **`show` prop** (Boolean): controlled by parent; parent flips it to `true` then waits for the `hidden` emit to flip it back to `false`.
- **`hidden` emit**: fired when Bootstrap finishes hiding; the parent's handler resets `show` to `false`.

### Adding a new modal

1. Create `src/components/tree/modals/MyModal.vue` following the pattern above.
2. In `useModalState.js`: add `showMyModal` ref, a data ref if needed, `openMyModal()` / `closeMyModal()` functions, add them to the `return` statement and `resetModalData()`.
3. In `ModalManager.vue`: lazy-import the component, add it to the template with `:show`/data props/`@hidden`, add the `show*` prop to `defineProps`, add `<name>-modal-hidden` to `defineEmits`, and add an `onMyModalHidden()` handler.
4. In `EnhancedDocumentTreeEditor.vue`: destructure the new refs/functions from `modalState`, pass the `:show-*` props to `<ModalManager>`, add the `@*-modal-hidden` handler, and add an `onMyXxx()` trigger function.
5. In the child panel: **emit an event** (do not call `useModalState()`). Forward it through `ItemDetailsPanel` → `RightSidePanel` → `EnhancedDocumentTreeEditor`.

### Lazy loading

All modals in `ModalManager.vue` use `defineAsyncComponent()` so they are code-split and only loaded when first shown.

---

## 2) Architecture: API service

All HTTP requests go through the singleton `api` instance from `src/services/api.js`.

- **Base URL:** relative (same host); `this.baseUrl = ''`.
- **Auth:** `Authorization: Bearer <token>` from `localStorage.getItem('saltApiToken')` (optional for public content).
- **Usage:** `import { api } from '@/services/api.js'` then `await api.get(endpoint)`.
- **Error handling:** throws `ApiError` with `.status` and `.message`.

### Key CASE v1.1 endpoints

| Purpose | Endpoint |
|---------|----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensalt/opensalt](https://github.com/opensalt/opensalt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
