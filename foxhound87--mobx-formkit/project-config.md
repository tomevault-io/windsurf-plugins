---
trigger: always_on
description: A concise guide for LLMs working with **MobX Formkit** — the reactive MobX form state management library.
---

# Guide for AI Agents

A concise guide for LLMs working with **MobX Formkit** — the reactive MobX form state management library.

---

## 1. Project Identity

| Field            | Value                                                                                  |
| ---------------- | -------------------------------------------------------------------------------------- |
| **Package**      | `mobx-formkit`                                                                       |
| **NPM**          | [npmjs.com/package/mobx-formkit](https://www.npmjs.com/package/mobx-formkit)         |
| **GitHub**       | [github.com/foxhound87/mobx-formkit](https://github.com/foxhound87/mobx-formkit)     |
| **Docs**         | [foxhound87.github.io/mobx-formkit](https://foxhound87.github.io/mobx-formkit/)  |
| **Version**      | 7.x (current: 7.1.0)                                                                  |
| **Size**         | ~8KB gzip (tree-shakeable)                                                             |
| **License**      | MIT                                                                                    |
| **Dependencies** | MobX 5, 6 & 7 (peer: `^5.15.0 || ^6.0.0 || ^7.0.0`), lodash, React (optional)                      |

---

## 2. Philosophy

MobX Formkit is built on three core principles:

1. **Reactive by default** — Every field is a MobX observable. Form state (value, error, dirty, touched, etc.) is reactive. Your UI updates automatically with `observer()` — no manual `setState`, no change handlers to write.

2. **Composable by design** — Fields nest infinitely (objects, arrays, arrays of arrays). Forms compose via `composer()` for wizards and multi-step flows. `ArrayMap` preserves insertion order for dynamic lists.

3. **Plugin-driven validation** — 8 validation drivers (DVR, VJF, AJV, YUP, JOI, ZOD, VALIBOT, VINEJS) share a uniform lifecycle. Swap or combine them without changing field definitions.

The result: you define your form structure declaratively, and reactivity, validation, change tracking, and UI bindings are automatic.

---

## 3. Architecture

```
┌──────────────────────────────────────────────────────┐
│                      Form                            │
│  entry point — creates fields, runs validation,      │
│  manages form-level computed state (isValid, error)  │
├──────────────────────────────────────────────────────┤
│  fields: ArrayMap<Field>       validator: Validator  │
│  $hooks / $handlers            options: Options      │
│  state: State (initial/current props, struct, etc.)  │
└──────────────────────┬───────────────────────────────┘
                       │ creates
┌──────────────────────┴───────────────────────────────┐
│                      Field                            │
│  wraps each form input — value, validation, events    │
├──────────────────────────────────────────────────────┤
│  $value (observable)   errorSync / errorAsync         │
│  $focused / $blurred / $touched / $changed            │
│  $label / $placeholder / $disabled / $rules           │
│  $hooks / $handlers    $observers / $interceptors     │
│  $converter / $input / $output                        │
│  fields: ArrayMap (nested sub-fields)                 │
└──────────────────────┬───────────────────────────────┘
                       │ shared via Base
┌──────────────────────┴───────────────────────────────┐
│                      Base                             │
│  $(), get(), set(), validate(), submit(), add(),      │
│  del(), update(), clear(), reset(), each(), map()     │
│  execHook(), execHandler()                            │
│  onClear(), onReset(), onSubmit(), onAdd(), onDel()   │
└──────────────────────────────────────────────────────┘
```

### Supporting Classes

| Class            | Role                                                                                                             |
| ---------------- | ---------------------------------------------------------------------------------------------------------------- |
| **`Validator`**  | Runs validation across all enabled drivers. Manages `validate()`, `validateField()`, `validateRelatedFields()`.  |
| **`Options`**    | 40+ form-level configuration flags (validation triggers, debounce, strict modes, converters, soft delete, etc.). |
| **`Bindings`**   | Maps field props → component props via rewriters (simple key mapping) or templates (function-based).             |
| **`State`**      | Holds initial, current, and struct data. Bridges field → form during initialization.                             |
| **`ArrayMap`**   | Ordered key-value collection (backed by observable array). Supports `move(from, to)` for sortable lists.         |
| **`composer()`** | Utility function for orchestrating multiple Form instances (wizards, multi-step flows).                          |

---

## 4. Key Concepts

### 4.1 Form

The entry point. Instantiated with field definitions + config:

```javascript
const form = new Form(
  { fields: [{ name: 'email', label: 'Email', rules: 'required|email' }] },
  { plugins: { dvr: dvr({ package: validatorjs }) }, hooks: { onSuccess(form) { ... } } }
);
```

**Constructor signature:**

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foxhound87/mobx-formkit](https://github.com/foxhound87/mobx-formkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
