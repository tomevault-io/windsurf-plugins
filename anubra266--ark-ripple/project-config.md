---
trigger: always_on
description: Ripple is a JS-first templating framework. Its syntax looks like JSX but follows different rules.
---

# Ripple Framework Patterns for this library

Ripple is a JS-first templating framework. Its syntax looks like JSX but follows different rules.
The package lives at `packages/ripple/`.

## Critical Syntax Rules

### 1. Control flow is NOT wrapped in `{}`

Control flow statements (`if`, `else`, `for`) are native JS statements — do NOT wrap them in `{}`.

```ripple
// ✅ CORRECT
if (@open) {
  <Dialog.Content />
} else {
  <span>{'closed'}</span>
}

for (const item of items; key item.id) {
  <Item value={item.id} />
}

// ❌ WRONG — never wrap control flow in {}
{if (@open) { ... }}
{for (...) { ... }}
```

### 2. No JSX fragments — sibling elements are placed directly

Ripple does NOT support JSX fragments (`<>...</>`). Unlike React, Ripple is declarative — multiple sibling root elements can appear directly in a component body without any wrapper.

```ripple
// ✅ CORRECT — sibling elements placed directly
export component MyComponent() {
  <Menu.Root>...</Menu.Root>
  <Dialog.Root>...</Dialog.Root>
}

// ❌ WRONG — fragments are not supported
export component MyComponent() {
  <>
    <Menu.Root>...</Menu.Root>
    <Dialog.Root>...</Dialog.Root>
  </>
}
```

### 3. Ternaries are fine for string values, not for elements

```ripple
// ✅ CORRECT — ternary for strings
{@open ? 'Hide' : 'Show'}
{'Dialog is '}{@dialog.open ? 'open' : 'closed'}

// ❌ WRONG — ternary for elements (use if/else instead)
{@open ? <ElementA /> : <ElementB />}
```

### 4. No render props — use `component children({ context }) {}`

Context components pass their API to children via `component children({ context }) {}`, not via render props.

```ripple
// ✅ CORRECT
<Dialog.Context>
  component children({ context }) {
    <span>{@context.open ? 'open' : 'closed'}</span>
  }
</Dialog.Context>

// ❌ WRONG — no render prop functions
<Dialog.Context>
  {(dialog) => <span>{dialog.open}</span>}
</Dialog.Context>
```

### 5. Component children type

Use `Component<Props>` from `ripple` for children that receive context, not function signatures:

```typescript
import { type Component } from 'ripple'

// ✅ CORRECT
export interface DialogContextProps {
  children: Component<{ context: UseDialogContext }>
}

// ❌ WRONG — don't use function types
export interface DialogContextProps {
  children: (context: UseDialogContext) => any
}
```

### 6. Static strings in JSX children need `{}`

```ripple
// ✅ CORRECT
<button>{'Click me'}</button>
<Dialog.Title class={styles.Title}>{'Welcome'}</Dialog.Title>

// ❌ WRONG
<button>Click me</button>
```

### 6. Reactive signals

```ripple
let open = track(false)    // declare reactive signal
@open                       // dereference (read)
@open = true                // assign
{ @open }                   // use in JSX expression
```

**Never unbox a tracked value into a `const` and then use `@` on it.** The `@` operator only works on tracked variables — once you unbox with `@`, the result is a plain snapshot and re-applying `@` does nothing. Use the tracked variable directly with `@`.

```ripple
const [children, value, localProps] = trackSplit(props, ['children', 'value']);

// ✅ CORRECT — use tracked variable directly
let mergedProps = track(() => mergeProps(@value.getRootProps(), @localProps));

// ❌ WRONG — unboxing into const loses reactivity, @imageCropper is meaningless
const imageCropper = @value;
let mergedProps = track(() => mergeProps(@imageCropper.getRootProps(), @localProps));
```

**Never unbox tracked values when passing them as props.** Props should receive the tracked signal so the child component stays reactive. Only use `@` when you actually need to read/consume the value (e.g. in `for` loops, expressions, conditions).

```ripple
const { collection, filter } = useListCollection({ ... });

// ✅ CORRECT — pass tracked value as prop, unbox when reading
<Listbox.Root {collection}>
for (const item of @collection.items; key item.value) { ... }

// ❌ WRONG — unboxing with @ when passing as prop
<Listbox.Root collection={@collection}>
```

### 7. `effect` vs `onMount`

Use `effect` when the setup depends on reactive values or needs cleanup. The return value of an `effect` is the cleanup function (like React's `useEffect`). Use `onMount` only for one-time, non-reactive initialization with no cleanup.

```ripple
// ✅ CORRECT — effect with reactive deps and cleanup
effect(() => {
  return @api.createFileUrl(@itemProps.file, (newUrl: string) => {
    @url = newUrl;
  });
});

// ❌ WRONG — onMount + onDestroy for reactive cleanup
let cleanup: (() => void) | undefined;
onMount(() => {
  cleanup = @api.createFileUrl(@itemProps.file, (newUrl: string) => {
    @url = newUrl;
  });
});
onDestroy(() => {
  cleanup?.();
});
```

### 8. Event handlers are camelCase like React

```ripple
// ✅ CORRECT (React style)
onClick, onChange, onInput

// ❌ WRONG
onclick, onchange, oninput, onsubmit
```

### 8. Lucide icons have no `Icon` suffix

When importing from `lucide-ripple`, drop the `Icon` suffix:

```ripple
// ✅ CORRECT
import { Pencil, Check, X, ChevronDown, Play } from 'lucide-ripple'

// ❌ WRONG
import { PencilIcon, CheckIcon, XIcon, ChevronDownIcon, PlayIcon } from 'lucide-ripple'
```

### 9. Passing elements/components as props


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anubra266/ark-ripple](https://github.com/anubra266/ark-ripple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
