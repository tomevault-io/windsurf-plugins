---
trigger: always_on
description: > **Documentation**: [AGENTS.md](../AGENTS.md) | [Architecture](../docs/ARCHITECTURE.md) | [Code Patterns](../docs/CODE_PATTERNS.md) | [Testing](../docs/TESTING.md) | [Build Requirements](../docs/BUILD_REQUIREMENTS.md)
---

# GitHub Copilot Instructions for MrRSS

> **Documentation**: [AGENTS.md](../AGENTS.md) | [Architecture](../docs/ARCHITECTURE.md) | [Code Patterns](../docs/CODE_PATTERNS.md) | [Testing](../docs/TESTING.md) | [Build Requirements](../docs/BUILD_REQUIREMENTS.md)

## Project Context

MrRSS is a modern, privacy-focused, cross-platform desktop RSS reader built with Wails (Go + Vue.js).

**Core Principles**: Privacy-first, cross-platform, modern UI, high performance, accessible

## Tech Stack

- **Backend**: Go 1.25+ with Wails v3 (alpha) framework, SQLite with `modernc.org/sqlite`
- **Frontend**: Vue 3.5+ Composition API, Pinia, Tailwind CSS 3.3+, Vite 5+
- **Tools**: Wails CLI v3, npm, Go modules
- **Icons**: Phosphor Icons | **I18n**: vue-i18n (English/Chinese)

## Quick Patterns Reference

### Backend (Go)

**Key Principles**:

- Always use `context.Context` for exported methods
- Error wrapping with `fmt.Errorf("operation failed: %w", err)`
- Prepared statements for all database queries
- Proper cleanup with `defer`
- Input validation before processing

📚 **Full Patterns**: See [CODE_PATTERNS.md](../docs/CODE_PATTERNS.md#backend-patterns-go)

### Frontend (Vue 3)

When writing Vue components, follow this pattern:

```vue
<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useAppStore } from '@/stores/app';
import { useI18n } from 'vue-i18n';

// Props with proper typing
interface Props {
  item: Article;
  isActive?: boolean;
}
const props = withDefaults(defineProps<Props>(), {
  isActive: false
});

// Store and i18n
const store = useAppStore();
const { t } = useI18n();

// Reactive state
const isLoading = ref(false);

// Async operations with error handling
async function loadData() {
  isLoading.value = true;
  try {
    const data = await fetch('/api/articles').then(r => r.json());
    // Process data...
  } catch (error) {
    console.error('Failed to load:', error);
    window.showToast(t('error'), 'error');
  } finally {
    isLoading.value = false;
  }
}

onMounted(() => loadData());
</script>

<template>
  <div class="component-container">
    <!-- Content with proper i18n -->
  </div>
</template>
```

📚 **Full Patterns**: See [CODE_PATTERNS.md](../docs/CODE_PATTERNS.md#frontend-patterns-vue)

## Internationalization

Always use i18n for user-facing strings:

```vue
<!-- Template -->
<h1>{{ t('welcome') }}</h1>
<button :title="t('clickToOpen')">{{ t('open') }}</button>

<!-- Script -->
window.showToast(t('successMessage'), 'success');
```

## UI Components

### Common Patterns

**Card Container**:

```html
<div class="bg-bg-primary border border-border rounded-lg p-4">
  <h3 class="text-text-primary font-semibold">{{ t('title') }}</h3>
  <p class="text-text-secondary text-sm">{{ t('description') }}</p>
</div>
```

**Modal/Dialog**:

```html
<div class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm p-4">
  <div class="bg-bg-primary w-full max-w-2xl rounded-2xl shadow-2xl border border-border">
    <div class="modal-header">
      <h2 class="text-xl font-bold">{{ t('modalTitle') }}</h2>
      <button @click="close" class="btn-icon">
        <i class="ph ph-x"></i>
      </button>
    </div>
  </div>
</div>
```

### Toast Notifications

```javascript
// Success message
window.showToast(message, 'success');

// Error message
window.showToast(t('operationFailed'), 'error');

// Info message with custom duration
window.showToast(t('updateAvailable'), 'info', 5000);
```

### Confirm Dialogs

```javascript
const confirmed = await window.showConfirm(
  t('confirmDelete'),
  t('deleteWarning'),
  true  // isDanger - shows red confirmation button
);

if (confirmed) {
  // Proceed with dangerous operation
}
```

### Context Menu Pattern

```vue
<script setup>
import { useContextMenu } from '@/composables/ui/useContextMenu';

const { contextMenu, openContextMenu, closeContextMenu } = useContextMenu();

// Define menu items
const menuItems = [
  { label: t('edit'), action: 'edit', icon: 'ph-pencil' },
  { label: t('delete'), action: 'delete', icon: 'ph-trash', danger: true },
  { type: 'divider' },
  { label: t('markAsRead'), action: 'mark-read' }
];

// Handle right-click
function handleRightClick(event: MouseEvent, item: Article) {
  event.preventDefault();
  openContextMenu(event, menuItems, item);
}

// Handle menu action
function handleMenuAction(action: string, item: Article) {
  switch (action) {
    case 'edit':
      // Handle edit
      break;
    case 'delete':
      // Handle delete
      break;
    case 'mark-read':
      // Handle mark as read
      break;
  }
}
</script>

<template>
  <div @contextmenu="handleRightClick($event, item)">
    <!-- Item content -->
  </div>
</template>
```

## Settings Management (OPTIMIZED)

✅ **The settings system has been optimized with schema-driven code generation!**

### Quick Method (3 Steps)

**Step 1**: Edit `internal/config/settings_schema.json`

```json
"new_setting_key": {
  "type": "bool",
  "default": false,
  "category": "general",
  "encrypted": false,
  "frontend_key": "new_setting_key"
}
```

**Step 2**: Generate all code


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RocM301/MrRSS](https://github.com/RocM301/MrRSS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
