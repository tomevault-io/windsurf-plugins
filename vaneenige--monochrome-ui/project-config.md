---
trigger: always_on
description: Accessible, headless UI components. Four components — Accordion, Collapsible, Menu, Tabs — powered by a ~2KB core with zero dependencies.
---

# Monochrome

Accessible, headless UI components. Four components — Accordion, Collapsible, Menu, Tabs — powered by a ~2KB core with zero dependencies.

Framework-agnostic core with React and Vue wrappers included. The core uses event delegation and ARIA attributes as the source of truth. Import it once and every component on the page works automatically.

---

# Part 1: Using Monochrome

> For AI agents helping developers build with monochrome.

## Installation

```bash
npm install monochrome    # Core + React + Vue wrappers included
```

```ts
import "monochrome"                        // Core (auto-activates)
import { Accordion, Tabs } from "monochrome/react"  // React
import { Accordion, Tabs } from "monochrome/vue"    // Vue
```

## Components

### Accordion

Collapsible content panels. `type="single"` allows one open at a time. `type="multiple"` allows any combination.

<table>
<tr><th>React</th><th>Vue</th></tr>
<tr><td>

```tsx
import { Accordion } from "monochrome/react"

<Accordion.Root type="single">
  <Accordion.Item open>
    <Accordion.Header as="h3">
      <Accordion.Trigger>
        Section Title
      </Accordion.Trigger>
    </Accordion.Header>
    <Accordion.Panel>
      Content here
    </Accordion.Panel>
  </Accordion.Item>
  <Accordion.Item disabled>
    <Accordion.Header>
      <Accordion.Trigger>
        Disabled
      </Accordion.Trigger>
    </Accordion.Header>
    <Accordion.Panel>
      Content here
    </Accordion.Panel>
  </Accordion.Item>
</Accordion.Root>
```

</td><td>

```vue
<script setup lang="ts">
import { Accordion } from "monochrome/vue"
</script>

<template>
  <Accordion.Root type="single">
    <Accordion.Item :open="true">
      <Accordion.Header as="h3">
        <Accordion.Trigger>
          Section Title
        </Accordion.Trigger>
      </Accordion.Header>
      <Accordion.Panel>
        Content here
      </Accordion.Panel>
    </Accordion.Item>
    <Accordion.Item :disabled="true">
      <Accordion.Header>
        <Accordion.Trigger>
          Disabled
        </Accordion.Trigger>
      </Accordion.Header>
      <Accordion.Panel>
        Content here
      </Accordion.Panel>
    </Accordion.Item>
  </Accordion.Root>
</template>
```

</td></tr>
</table>

**Props:**

| Component | Prop | Type | Default | Description |
|-----------|------|------|---------|-------------|
| `Root` | `type` | `"single" \| "multiple"` | `"single"` | Whether one or many panels can be open |
| `Item` | `open` | `boolean` | `false` | Start open |
| `Item` | `disabled` | `boolean` | `false` | Cannot toggle, skipped by keyboard |
| `Header` | `as` | `"h1"` – `"h6"` | `"h3"` | Heading level |

**Keyboard:** ArrowDown/Up navigate items, Home/End jump to first/last, Enter/Space toggle.

### Collapsible

A single trigger that shows/hides content.

<table>
<tr><th>React</th><th>Vue</th></tr>
<tr><td>

```tsx
import { Collapsible } from "monochrome/react"

<Collapsible.Root open>
  <Collapsible.Trigger>Toggle</Collapsible.Trigger>
  <Collapsible.Panel>Content</Collapsible.Panel>
</Collapsible.Root>
```

</td><td>

```vue
<script setup lang="ts">
import { Collapsible } from "monochrome/vue"
</script>

<template>
  <Collapsible.Root :open="true">
    <Collapsible.Trigger>Toggle</Collapsible.Trigger>
    <Collapsible.Panel>Content</Collapsible.Panel>
  </Collapsible.Root>
</template>
```

</td></tr>
</table>

**Props:**

| Component | Prop | Type | Default | Description |
|-----------|------|------|---------|-------------|
| `Root` | `open` | `boolean` | `false` | Start open |

**Keyboard:** Enter/Space toggle. No arrow key navigation (WAI-ARIA disclosure pattern).

### Tabs

Tabbed interface with manual activation and roving tabindex.

<table>
<tr><th>React</th><th>Vue</th></tr>
<tr><td>

```tsx
import { Tabs } from "monochrome/react"

<Tabs.Root
  defaultValue="tab1"
  orientation="horizontal"
>
  <Tabs.List>
    <Tabs.Tab value="tab1">Tab 1</Tabs.Tab>
    <Tabs.Tab value="tab2" disabled>
      Tab 2
    </Tabs.Tab>
    <Tabs.Tab value="tab3">Tab 3</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel value="tab1">Content 1</Tabs.Panel>
  <Tabs.Panel value="tab2">Content 2</Tabs.Panel>
  <Tabs.Panel value="tab3">Content 3</Tabs.Panel>
</Tabs.Root>
```

</td><td>

```vue
<script setup lang="ts">
import { Tabs } from "monochrome/vue"
</script>

<template>
  <Tabs.Root
    default-value="tab1"
    orientation="horizontal"
  >
    <Tabs.List>
      <Tabs.Tab value="tab1">Tab 1</Tabs.Tab>
      <Tabs.Tab value="tab2" :disabled="true">
        Tab 2
      </Tabs.Tab>
      <Tabs.Tab value="tab3">Tab 3</Tabs.Tab>
    </Tabs.List>
    <Tabs.Panel value="tab1">Content 1</Tabs.Panel>
    <Tabs.Panel value="tab2">Content 2</Tabs.Panel>
    <Tabs.Panel value="tab3">Content 3</Tabs.Panel>
  </Tabs.Root>
</template>
```

</td></tr>
</table>

**Props:**

| Component | Prop | Type | Default | Description |
|-----------|------|------|---------|-------------|
| `Root` | `defaultValue` | `string` | — | Initially selected tab |
| `Root` | `orientation` | `"horizontal" \| "vertical"` | `"horizontal"` | Arrow key direction |
| `Tab` | `value` | `string` | — | Unique tab identifier |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vaneenige/monochrome-ui](https://github.com/vaneenige/monochrome-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
