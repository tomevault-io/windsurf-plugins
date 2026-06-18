---
trigger: always_on
description: >
---


# ApexTree AI Skill

> **Framework wrapper detection — check `package.json` before generating code.**
> - `react` → use **`react-apextree`** instead of the core API.
> - `vue` → use **`vue-apextree`**.
> - `@angular/core` → use **`ngx-apextree`**.
>
> Wrappers handle `destroy()` automatically on unmount, accept reactive props, and forward events as idiomatic framework events. Use the core API directly only when no framework is detected, or when the user explicitly asks for vanilla. See `references/framework-wrappers.md`.

## 1. Critical Rules

1. **Data is a single `NestedNode`** (the root) passed to `render(data)`, **not** to the constructor. Constructor takes `(element, options)`; `render(rootNode)` paints.
2. **Every node needs `id`, `name`, and `children`.** Leaves have `children: []` — not `undefined` and not omitted.
3. **`id` must be unique across the entire tree.** Selection, edge highlighting, search, and breadcrumb all key off `id`.
4. **`render(data)` returns a `Graph` instance.** Save it — it exposes `collapse`, `expand`, `fitScreen`, `centerOnNode`, search APIs, and selection APIs.
5. **`enableExpandCollapseZoom: true` (default)** re-fits the viewBox on collapse / expand. Set to `false` to keep the camera locked.
6. **`enableSelection` is `'single' | 'multi' | false`**, not a boolean. `false` means selection is off; `'single'` and `'multi'` are the active modes.
7. **`onSelectionChange(listener)` is on the `Graph`** returned by `render`, not on `tree` and not in `options`.
8. **Use `contentKey: 'data'`** to switch the built-in template into org-card mode (avatar, name, title, subtitle, badge, accent stripe). Don't write a custom `nodeTemplate` for that — the built-in one is already there.
9. **For custom rendering use `nodeTemplate(content)`** where `content` is the value at `contentKey`. Set `contentKey: 'data'` for structured payloads.
10. **`direction` is `'top' | 'bottom' | 'left' | 'right'`** — controls where the root sits and which way the tree grows.
11. **Per-node options** live on `node.options` and can override font, border, tooltip, and node visuals for a single node.
12. **Call `destroy()`** before unmounting in React / Vue / Angular — it frees `ResizeObserver`s and tooltip DOM.
13. **Set the license key once** at app startup with `ApexTree.setLicense('KEY')` to remove the watermark.

---

## 2. Data Format — `NestedNode`

```ts
interface NestedNode<T = undefined> {
  id: string;                                          // REQUIRED, unique across the whole tree
  name: string;                                        // REQUIRED, display label (when contentKey is 'name')
  children: NestedNode<T>[];                           // REQUIRED — empty array for leaves
  data?: T;                                            // arbitrary payload, used when contentKey: 'data'
  options?: Partial<FontOptions & NodeOptions & TooltipOptions>;   // per-node overrides
}
```

Minimal example:

```js
import { ApexTree } from 'apextree';

const data = {
  id: '1', name: 'CEO',
  children: [
    { id: '2', name: 'CTO', children: [
      { id: '4', name: 'Eng Lead', children: [] },
      { id: '5', name: 'QA Lead',  children: [] },
    ]},
    { id: '3', name: 'CFO', children: [] },
  ],
};

const tree = new ApexTree(document.getElementById('chart'), {
  width: 800, height: 600,
  nodeWidth: 120, nodeHeight: 40,
  childrenSpacing: 70, siblingSpacing: 30,
  direction: 'top',
});

const graph = tree.render(data);
```

### Org-card mode (built-in `'data'` template)

Set `contentKey: 'data'` and put structured fields into `node.data`. The built-in template renders avatar / name / title / subtitle / badge / accent stripe automatically:

```js
const data = {
  id: 'ceo', name: 'Alice',
  data: {
    name: 'Alice Johnson',
    title: 'Chief Executive Officer',
    subtitle: 'Executive',
    imageURL: 'https://example.com/alice.jpg',
    accentColor: '#6366f1',
    badge: { text: 'Active', color: '#EEF2FF' },
  },
  children: [],
};

new ApexTree(el, {
  contentKey: 'data',
  nodeWidth: 220, nodeHeight: 80,
}).render(data);
```

### Custom `nodeTemplate`

```js
new ApexTree(el, {
  contentKey: 'data',
  nodeWidth: 200, nodeHeight: 80,
  nodeTemplate: (c) => `
    <div style="display:flex;align-items:center;gap:8px;padding:8px;">
      <img src="${c.img}" style="width:32px;height:32px;border-radius:50%;" />
      <div>
        <div style="font-weight:600;">${c.name}</div>
        <div style="font-size:11px;color:#666;">${c.role}</div>
      </div>
    </div>`,
}).render(data);
```

### Per-node style overrides

```js
const data = {
  id: 'ceo', name: 'CEO',
  options: { nodeBGColor: '#EEF2FF', borderColor: '#A5B4FC' },
  children: [
    { id: 'cto', name: 'CTO',
      options: { nodeBGColor: '#ECFDF5', borderColor: '#6EE7B7' },
      children: [] },
  ],
};
```

---

## 3. Top-Level Options

| Option | Type | Default | Notes |
|---|---|---|---|
| `width` / `height` | `number \| string` | `'100%'` / `'auto'` | Canvas size. `'auto'` sizes to content. |
| `viewPortWidth` / `viewPortHeight` | `number` | `800` / `600` | Internal SVG viewport. |
| `direction` | `'top'\|'bottom'\|'left'\|'right'` | `'top'` | Where the root sits. |
| `contentKey` | `string` | `'name'` | Key on the data object used as the label. Set to `'data'` for structured payloads. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apexcharts/apextree-skill](https://github.com/apexcharts/apextree-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
