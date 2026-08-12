---
trigger: always_on
description: For all tasks in this repository, follow this priority order:
---

# Repository workflow

## Rule priority

For all tasks in this repository, follow this priority order:

1. The root `AGENTS.md` and the nearest applicable nested `AGENTS.md`.
2. Existing published public APIs, compatibility requirements, and repository conventions.
3. The explicit requirements of the current task.
4. Approved Figma designs, Figma Variables, and Design Tokens.
5. The applicable files under `agent-guidelines/`.
6. Arco Design default behavior and styles.

If rules conflict, report the conflict, affected files, API or behavior impact, and a recommended resolution before making changes. Do not silently override a higher-priority rule.

---

# DesignKit architecture boundaries

## Base components

Base components provide reusable UI primitives and framework-level interaction capabilities, such as:

- Button
- Input
- Select
- MultiSelect
- Checkbox
- Tag
- DatePicker
- RangePicker
- Cascader
- TreeSelect
- Modal
- Drawer
- Table
- Pagination
- Tabs
- Tooltip
- Dropdown
- Trigger

Base component tasks may include visual optimization, token alignment, accessibility, Popup or Portal behavior, framework parity, and compatibility fixes.

Do not duplicate a base capability inside a business component or page template.

## Business components

Business components provide stable, reusable business capabilities that can be used across multiple pages.

A business component is expected to have:

- A clear business task and applicable scenarios
- Stable public Props, Events, Types, and extension boundaries
- A defined state model
- React and Vue behavior parity
- Real Docs demos
- Tests
- Public export and versioning decisions when applicable

Examples include:

- `FilterBar`
- Reusable table toolbars
- Tag management structures
- Reusable description or summary blocks

Business components should compose existing base components instead of reimplementing them.

## Page templates

Page templates are runnable page-level composition examples.

They are not public all-in-one business components by default.

A page template should demonstrate:

- How base components and business components are combined
- How page regions are arranged
- How page-level interactions are connected
- How loading, empty, error, and responsive states are handled
- How React and Vue implementations remain visually and behaviorally equivalent
- How users can copy and adapt the implementation

Examples include:

- Basic list
- Filtered list
- Tag management list
- Basic form
- Grouped form
- Step form
- Detail page
- Dashboard
- Login page
- Result page

Do not implement a page template as a large configuration-driven public component such as:

```tsx
<BasicListPage
  title="..."
  filterFields={...}
  columns={...}
  request={...}
/>
```

unless the user explicitly approves a reusable page framework and there is sufficient cross-page reuse evidence.

A page template normally belongs to Docs or a template example directory and should not be exported from React or Vue package entry points.

---

# DesignKit task routing

## Guideline registry

Component-related guideline files are located at:

- `agent-guidelines/designkit-base-component-style-optimization-guideline.md`
- `agent-guidelines/designkit-business-component-development-guideline.md`
- `agent-guidelines/designkit-filterbar-codex-master-prompt.md`

First classify the task, then read only the relevant guideline files. Do not load every guideline by default.

## Base component style optimization

Read:

`agent-guidelines/designkit-base-component-style-optimization-guideline.md`

when the task involves:

- Arco Design base component style optimization
- Figma visual alignment for base components
- Figma Variables or Design Tokens
- CSS specificity, style order, scoped style, or override issues
- Popup, Dropdown, Trigger, Tooltip, or Portal styles
- React and Vue base component visual consistency
- Input, Select, Dropdown, Tag, Checkbox, Button, Tabs, DatePicker, Cascader, TreeSelect, or similar base components

For these tasks:

- Prefer existing Figma Variables, Design Tokens, and shared component variables.
- Reuse already optimized base component capabilities.
- Do not create duplicate private styles for an existing base capability.
- Do not use broad unscoped `.arco-*` overrides.
- Do not use `!important` before identifying the root cause.
- Keep React, Vue, and Docs Preview visually equivalent.

## Business component development

Read:

`agent-guidelines/designkit-business-component-development-guideline.md`

when the task involves:

- Creating a new business component
- Extending a business component API or Schema
- Changing a business component state model or event contract
- Implementing React and Vue business components
- Creating Docs, Recipes, AI Contract, Evaluator, or business-component tests

For these tasks:

- Define the business task, applicable scenarios, non-applicable scenarios, state model, API, events, and extension boundaries before coding.
- Prefer composition of existing base components.
- Do not reimplement Input, Select, Button, Drawer, Modal, Table, or other base capabilities.
- Keep React and Vue behavior contracts aligned.
- Preserve existing public APIs unless a breaking change is explicitly approved.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nink1992/Starbucks-Design-main](https://github.com/Nink1992/Starbucks-Design-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
