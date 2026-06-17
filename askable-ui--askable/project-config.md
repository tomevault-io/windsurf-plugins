---
trigger: always_on
description: This file explains how to integrate `askable-ui` correctly. Copy it to your project root so coding agents have accurate, copy-pasteable guidance.
---

# askable-ui — Agent Instructions

This file explains how to integrate `askable-ui` correctly. Copy it to your project root so coding agents have accurate, copy-pasteable guidance.

---

## What askable-ui does

`askable-ui` tracks which annotated UI element the user is currently focused on and serialises that focus into a prompt-ready string. You feed that string to your LLM — no manual prompt engineering required.

```
"User is focused on: metric: revenue — value $2.3M, delta +12%"
```

The three moving parts:
1. **Annotate** — add `data-askable` (or wrap with `<Askable>`) to any element whose data is relevant to AI
2. **Observe** — one hook/composable/store call wires the DOM listener
3. **Inject** — pass `promptContext` (or `ctx.toContext()`) into your LLM system prompt

---

## Installation

```bash
# React
npm install @askable-ui/react

# Vue 3
npm install @askable-ui/vue

# Svelte
npm install @askable-ui/svelte

# Vanilla / framework-agnostic
npm install @askable-ui/core
```

---

## Annotating elements

### HTML attribute (vanilla / any framework)

```html
<div data-askable='{"widget":"revenue","value":"$2.3M","delta":"+12%"}'>
  <RevenueChart />
</div>
```

- The value can be a **JSON object** (preferred) or a plain string.
- Only annotate elements whose data is meaningful to an AI answer. Do not annotate every div.

### React component

```tsx
import { Askable } from '@askable-ui/react';

<Askable meta={{ widget: 'revenue', value: '$2.3M', delta: '+12%' }}>
  <RevenueChart data={data} />
</Askable>
```

`<Askable>` keeps `data-askable` in sync with reactive props. Use it whenever `meta` comes from component state or props.

### Nesting and hierarchy

Nested `[data-askable]` elements are automatically chained. Inner elements inherit outer context.

```tsx
<Askable meta={{ section: 'deals' }}>
  <TableContainer>
    <Askable meta={{ row: 3, company: 'Acme', stage: 'Closed Won' }}>
      <TableRow />
    </Askable>
  </TableContainer>
</Askable>
```

When the row is focused the serialized output includes both levels.

### Override extracted text

By default askable-ui extracts `textContent`. Override it when the DOM text is noisy or screen-reader labels are better:

```html
<div data-askable='{"metric":"churn"}' data-askable-text="Monthly churn rate 4.2%">
  <ChurnChart />
</div>

<!-- Suppress text entirely -->
<div data-askable='{"id":42}' data-askable-text="">...</div>
```

### Priority

When two annotated elements overlap, the innermost wins by default. Set `data-askable-priority` to override:

```html
<div data-askable='{"section":"header"}' data-askable-priority="1">
  <div data-askable='{"cta":"upgrade"}' data-askable-priority="10">Upgrade</div>
</div>
```

---

## Passive interaction patterns

Passive patterns fire automatically as the user clicks, hovers, or focuses annotated elements.

### React

```tsx
import { Askable, useAskable } from '@askable-ui/react';

function Dashboard() {
  const { promptContext } = useAskable(); // shared context, all events

  return (
    <>
      <Askable meta={{ widget: 'revenue', value: kpi.revenue }}>
        <RevenueCard />
      </Askable>
      <Askable meta={{ widget: 'churn', value: kpi.churn }}>
        <ChurnChart />
      </Askable>
      <button onClick={() => sendToAI(promptContext)}>Ask AI</button>
    </>
  );
}
```

### Restrict which events trigger focus

```tsx
// Click-only — no hover, no keyboard focus
const { promptContext } = useAskable({ events: ['click'] });

// Hover + focus, no click
const { promptContext } = useAskable({ events: ['hover', 'focus'] });
```

### Vue 3

```vue
<script setup>
import { Askable, useAskable } from '@askable-ui/vue';
const { promptContext } = useAskable();
</script>

<template>
  <Askable :meta="{ widget: 'revenue', value: kpi.revenue }">
    <RevenueCard :data="kpi" />
  </Askable>
</template>
```

### Svelte

```svelte
<script>
  import { Askable, createAskableStore } from '@askable-ui/svelte';
  const { promptContext, ctx } = createAskableStore();
  // call ctx.destroy() in onDestroy
</script>

<Askable meta={{ widget: 'revenue', value: kpi.revenue }}>
  <RevenueCard data={kpi} />
</Askable>
```

### SolidJS

```tsx
import { Askable, useAskable } from '@askable-ui/solid';

function Dashboard() {
  const { promptContext } = useAskable();

  return (
    <>
      <Askable meta={{ widget: 'revenue', value: kpi.revenue }}>
        <RevenueCard />
      </Askable>
      <button onClick={() => sendToAI(promptContext())}>Ask AI</button>
    </>
  );
}
```

Note: in SolidJS `promptContext` is a signal accessor — call it as `promptContext()` to read the current value.

### Angular

```ts
import { Component, inject } from '@angular/core';
import { AskableService } from '@askable-ui/angular';

@Component({
  template: `
    <div data-askable='{"widget":"revenue","value":"$2.3M"}'>
      <revenue-card />
    </div>
    <button (click)="askAI()">Ask AI</button>
  `,
})
export class DashboardComponent {
  private readonly askable = inject(AskableService);

  askAI() {
    const context = this.askable.promptContext();
    sendToLLM(context);
  }
}
```

Use `AskableDirective` for reactive annotations:

```ts
@Component({

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [askable-ui/askable](https://github.com/askable-ui/askable) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
