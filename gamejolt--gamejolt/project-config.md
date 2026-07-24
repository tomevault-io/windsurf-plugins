---
trigger: always_on
description: This guide defines the conventions for all Vue components in this project. It is the authoritative reference for the ongoing migration away from `vue-class-component` / `vue-property-decorator` to Vue 3 Composition API.
---

# Vue Component Style Guide

This guide defines the conventions for all Vue components in this project. It is the authoritative reference for the ongoing migration away from `vue-class-component` / `vue-property-decorator` to Vue 3 Composition API.

---

## Post-Work Checks

After completing work, run lint and format checks **only on the files you modified** — never on the full `src/` tree. Then run a typecheck across the whole project. Do this once at the end, not between iterations.

```sh
# Replace with the actual modified paths, space-separated
yarn eslint path/to/File1.vue path/to/File2.ts
yarn oxfmt --check path/to/File1.vue path/to/File2.ts

# Typecheck runs project-wide (vue-tsc --noEmit) — no per-file mode.
yarn typecheck
```

Fix any issues reported (`--fix` for eslint, `--write` for oxfmt) before reporting the task as complete.

---

## Script

- Always `<script lang="ts" setup>` with TypeScript.
- `vue-class-component` and `vue-property-decorator` are **banned** — do not add new usages.
- Don't annotate function return types unless required (e.g. to break recursion, or to tighten a type that would otherwise infer too broadly). Let TypeScript infer.

---

## Props

```ts
type Props = {
	myProp: string;
	optional?: boolean;
};
const { myProp, optional = false } = defineProps<Props>();
```

- Use `type Props = { ... }` (not `interface`)
- Destructure immediately with defaults inline
- Never access `props.x` — always use the destructured name
- Do **not** use `withDefaults(defineProps<Props>(), { ... })` — defaults belong in the destructure

---

## Emits

```ts
const emit = defineEmits<{
	done: [];
	seek: [pos: number];
}>();
```

---

## v-model

Use `defineModel()` — never declare a `modelValue` prop + `update:modelValue` emit by hand.

```ts
// Default v-model
const modelValue = defineModel<string>({ required: true });

// Named v-model (v-model:packs)
const packs = defineModel<StickerPackModel[]>('packs', { required: true });
```

- The returned ref is two-way bound: reading `modelValue.value` gets the prop, assigning `modelValue.value = x` emits `update:modelValue`.
- Do **not** also declare the model name in `defineProps` — `defineModel` registers the prop and emit internally.
- In templates, refs auto-unwrap at the top level of expressions, so `@click="modelValue = foo"` works and compiles to a `.value` assignment that triggers the update emit.

---

## Reactive State

- `ref()` for reactive values, `computed()` for derived values
- Plain `let` for non-reactive helpers (timers, callbacks, etc.)
- `toRef(() => expr)` to create a reactive ref from an expression

---

## Watchers

```ts
watch(() => myProp, val => { ... });
watch([() => a, () => b], () => { ... });
watch(someRef, val => { ... }, { immediate: true });
```

Use the callback form `() => prop` for destructured props — reactive destructure requires this for watchers to track correctly.

---

## Lifecycle

```ts
import { onMounted, onBeforeUnmount } from 'vue';
onMounted(() => { ... });
onBeforeUnmount(() => { ... });
```

---

## Template Refs

```ts
const elRef = useTemplateRef('el');
```

Always use `useTemplateRef` when possible. Do not specify the generic type — Vue infers it from the template.

---

## Directives

Always import explicitly — do not rely on auto-injection:

```ts
import { vAppTooltip } from '../../_common/tooltip/tooltip-directive';
```

---

## Stores

Stores are creator functions returning a bag of refs, not classes. Follow the pattern from [src/\_common/sidebar/sidebar.store.ts](src/_common/sidebar/sidebar.store.ts), [src/\_common/sticker/sticker-store.ts](src/_common/sticker/sticker-store.ts), [src/\_common/sticker/layer/layer-controller.ts](src/_common/sticker/layer/layer-controller.ts).

```ts
import { inject, InjectionKey, ref, shallowReadonly } from 'vue';

export type FooStore = ReturnType<typeof createFooStore>;

export const FooStoreKey: InjectionKey<FooStore> = Symbol('foo-store');

export function useFooStore() {
	return inject(FooStoreKey)!;
}

export function createFooStore() {
	const isOpen = ref(false);
	const items = ref<Item[]>([]);

	function open() {
		isOpen.value = true;
	}

	function addItem(item: Item) {
		items.value.push(item);
	}

	return shallowReadonly({ isOpen, items, open, addItem });
}
```

- Never `class Foo { ... }` with method bodies. Never `reactive(new Foo())`.
- Each piece of state is its own `const x = ref(...)` or `computed(...)`. Collections are `ref<T[]>([])`, not `reactive`/`shallowReactive` arrays.
- Functions close over the refs in the same scope; no `this`.
- Return `shallowReadonly({ ...refs, ...functions })`. `shallowReadonly` freezes the bag's shape — `store.foo = newRef` is blocked — while leaving the refs inside fully mutable via `.value`. Refs are _not_ auto-unwrapped, so consumers keep using `.value` explicitly.
- Pair with `XxxStoreKey: InjectionKey<XxxStore>`, `type XxxStore = ReturnType<typeof createXxxStore>`, and a `useXxxStore()` composable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gamejolt/gamejolt](https://github.com/gamejolt/gamejolt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
