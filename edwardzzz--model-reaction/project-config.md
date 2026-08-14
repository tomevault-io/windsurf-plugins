---
trigger: always_on
description: > A high-density guide for **coding agents / LLMs** working on or with the
---

# AGENTS.md

> A high-density guide for **coding agents / LLMs** working on or with the
> `model-reaction` library. Read this **before** README when you only have
> a few KB of context budget.

---

## 1. Mental Model

```
schema  ─►  ModelManager  ─►  data            (validated, source of truth)
                          ├─►  dirtyData      (unvalidated user input)
                          └─►  reactions      (auto-derived fields)
```

Three layers, nothing else:

1. `data` — only fields that **passed** validation live here.
2. `dirtyData` — last user input that **failed** validation, indexed by field.
3. `reactions` — derived values recomputed when their declared `fields`
   change. They write back into `data`.

---

## 2. The 5 APIs You Use 90% of the Time

```ts
const m = createModel(schema, options?);     // factory
await m.setField('name', 'Ada');              // set + validate; returns boolean
m.getField('name');                           // read
await m.validateAll();                        // re-validate every field
m.dispose();                                  // ALWAYS call this in cleanup
```

**Discriminating rules:**

| Need | API |
| --- | --- |
| Set one field with validation | `setField(name, value)` |
| Set many fields in one validation+reaction pass | `setFields({ a, b })` |
| Read current value | `getField(name)` or `m.data.name` |
| Get failed input back | `getDirtyData()` |
| React to change outside React | `subscribeField` / `subscribe` |
| Inspect schema for tooling | iterate the schema literal directly (it's a plain object) |

---

## 3. The 7 Pitfalls You'll Hit

1. **Forgetting `await`** — `setField` returns `Promise<boolean>` (true = passed validation). If you don't await, validation may still be in-flight.

2. **Reading `data` after a failed `setField`** — failed values go to `dirtyData`, not `data`. Use `getDirtyData()` to retrieve them.

3. **Side effects inside `reaction.computed`** — `computed` MUST be pure. Put side effects in `reaction.action` instead.

4. **Not calling `dispose()`** — leaks reactions, event listeners and pending validation timers. Always wire it to your cleanup path (React effect, test `afterEach`, server shutdown).

5. **Sharing one model across React trees without dispose** — see [docs/REACT.md](docs/REACT.md). Use `useEffect` cleanup or instantiate per route.

6. **Assuming `default` values are validated** — `default`s are written straight into `data` at construction and **bypass validation entirely**. A model can start in an invalid state with an empty `validationErrors`. If initial validity matters (e.g. before reading `data` to submit), call `await validateAll()` first.

7. **Assuming a reaction's `computed` output skips validation** — a reaction writes its result back through the same validate-then-commit path. If the derived field has a `validator` and the computed value fails it, `data` keeps the old value and the computed value lands silently in `dirtyData`. Keep derived-field validators loose, or watch `dirtyData` / `reaction:error`.

---

## 4. Code Skeletons (copy-paste these)

### 4.1 Form model

```ts
import { createModel, ValidationRules } from 'model-reaction';

export const userModel = createModel({
  name: {
    type: 'string',
    default: '',
    validator: [
      ValidationRules.required.withMessage('Name required'),
      ValidationRules.minLength(2).withMessage('Min 2 chars'),
    ],
  },
  email: {
    type: 'string',
    default: '',
    validator: [
      ValidationRules.required,
      ValidationRules.email.withMessage('Bad email'),
    ],
  },
});
```

### 4.2 Cross-field reaction

```ts
fullName: {
  type: 'string',
  default: '',
  reaction: {
    fields: ['firstName', 'lastName'],
    computed: ({ firstName, lastName }) =>
      `${firstName} ${lastName}`.trim(),
  },
},
```

### 4.3 React form field

```tsx
import { useModelFieldState } from 'model-reaction/react';

function Input({ model, field, label }) {
  const [value, setValue, meta] = useModelFieldState(model, field);
  const [touched, setTouched] = useState(false);
  const showError = touched && meta.error;

  return (
    <label>
      <span>{label}</span>
      <input
        value={value ?? ''}
        onChange={(e) => setValue(e.target.value)}
        onBlur={() => setTouched(true)}
        aria-invalid={Boolean(showError)}
        disabled={meta.validating}
      />
      {showError && <small role="alert">{meta.error}</small>}
    </label>
  );
}
```

### 4.4 Selector subscription

```ts
const unsub = model.subscribe(
  (data) => data.cart.totalCents,
  (next, prev) => console.log('total changed', prev, '→', next)
);
// later: unsub()
```

### 4.5 Iterate the schema literal

```ts
// schema is just a plain object — no helper API needed.
for (const [name, field] of Object.entries(schema)) {
  console.log(name, field.type, field.validator?.length ?? 0);
}
```

---

## 5. What's NOT in this Library

These are **deliberate omissions**. Don't add them; don't fake them.

| Missing | Why |
| --- | --- |
| Per-field `touched` state in the model | Belongs to UI lifecycle, not to data model. Use component-local `useState` (see §4.3). |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EdwardZZZ/model-reaction](https://github.com/EdwardZZZ/model-reaction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
