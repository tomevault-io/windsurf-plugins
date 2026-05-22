---
trigger: always_on
description: This document outlines the UI component standards and usage patterns for the OpenFrame project.
---

# UI Components

This document outlines the UI component standards and usage patterns for the OpenFrame project.

## Component Library

OpenFrame uses a shared component library to ensure consistency across the application. The component library is located in `services/openframe-frontend/src/components/ui/`.

### Core Components

- Use shared components from the component library
- Follow consistent naming conventions
- Implement proper props and events
- Document component usage
- Support both light and dark modes

Example component structure:
```
components/
├── ui/                 # Shared UI components
│   ├── Button.vue      # Button component
│   ├── Card.vue        # Card component
│   ├── DataTable.vue   # Data table component
│   ├── Dialog.vue      # Dialog component
│   ├── Dropdown.vue    # Dropdown component
│   ├── Input.vue       # Input component
│   ├── Tabs.vue        # Tabs component
│   └── ...
├── shared/             # Shared feature components
│   ├── Header.vue      # Application header
│   ├── Sidebar.vue     # Application sidebar
│   ├── Footer.vue      # Application footer
│   └── ...
└── features/           # Feature-specific components
    ├── rmm/            # RMM components
    ├── mdm/            # MDM components
    └── ...
```

### Button Component

The Button component is used for all clickable actions in the application.

```vue
<template>
  <button
    :class="[
      'of-button',
      `of-button--${variant}`,
      `of-button--${size}`,
      { 'of-button--loading': loading }
    ]"
    :disabled="disabled || loading"
    @click="$emit('click', $event)"
  >
    <span v-if="loading" class="of-button__loader"></span>
    <span v-else class="of-button__content">
      <slot></slot>
    </span>
  </button>
</template>

<script setup lang="ts">
defineProps({
  variant: {
    type: String,
    default: 'primary',
    validator: (value: string) => ['primary', 'secondary', 'danger', 'ghost'].includes(value)
  },
  size: {
    type: String,
    default: 'medium',
    validator: (value: string) => ['small', 'medium', 'large'].includes(value)
  },
  loading: {
    type: Boolean,
    default: false
  },
  disabled: {
    type: Boolean,
    default: false
  }
});

defineEmits(['click']);
</script>

<style scoped>
.of-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

.of-button--primary {
  background-color: var(--primary-color);
  color: white;
  border: none;
}

.of-button--secondary {
  background-color: transparent;
  color: var(--primary-color);
  border: 1px solid var(--primary-color);
}

.of-button--danger {
  background-color: var(--danger-color);
  color: white;
  border: none;
}

.of-button--ghost {
  background-color: transparent;
  color: var(--text-color);
  border: none;
}

.of-button--small {
  padding: 6px 12px;
  font-size: 12px;
}

.of-button--medium {
  padding: 8px 16px;
  font-size: 14px;
}

.of-button--large {
  padding: 10px 20px;
  font-size: 16px;
}

.of-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.of-button__loader {
  width: 16px;
  height: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: white;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
</style>
```

Usage example:
```vue
<Button variant="primary" size="medium" @click="saveChanges">Save Changes</Button>
<Button variant="secondary" @click="cancel">Cancel</Button>
<Button variant="danger" @click="deleteItem">Delete</Button>
<Button variant="ghost" @click="showDetails">View Details</Button>
<Button loading>Processing...</Button>
```

### Data Table Component

The DataTable component is used for displaying tabular data throughout the application.

```vue
<template>
  <div class="of-data-table">
    <div v-if="loading" class="of-data-table__loading">
      <div class="of-data-table__spinner"></div>
      <span>Loading data...</span>
    </div>
    <table v-else>
      <thead>
        <tr>
          <th v-for="column in columns" :key="column.field">
            {{ column.label }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in data" :key="index">
          <td v-for="column in columns" :key="column.field">
            <slot :name="column.field" :row="row">
              {{ getColumnValue(row, column) }}
            </slot>
          </td>
        </tr>
      </tbody>
    </table>
    <div v-if="!loading && data.length === 0" class="of-data-table__empty">
      <slot name="empty">No data available</slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';

const props = defineProps({
  columns: {
    type: Array,
    required: true
  },
  data: {
    type: Array,
    default: () => []
  },
  loading: {
    type: Boolean,
    default: false
  }
});

const getColumnValue = (row: any, column: any) => {
  if (column.formatter) {
    return column.formatter(row);
  }
  
  const value = column.field.split('.').reduce((obj, key) => {
    return obj && obj[key] !== undefined ? obj[key] : null;
  }, row);
  
  return value !== null ? value : '';
};
</script>


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
