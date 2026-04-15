---
trigger: always_on
description: Svelte 5 introduces runes, a set of advanced primitives for controlling reactivity. The runes replace certain non-runes features and provide more explicit control over state and effects.
---

# Svelte 5 Cursor Rules

## Overview

Svelte 5 introduces runes, a set of advanced primitives for controlling reactivity. The runes replace certain non-runes features and provide more explicit control over state and effects.

### $state

- **Purpose:** Declare reactive state.
- **Usage:**

```javascript
<script>let count = $state(0);</script>
```

- **Replaces:** Top-level `let` declarations in non-runes mode.
- **Class Fields:**

```javascript
class Todo {
    done = $state(false);
    text = $state();
    constructor(text) {
        this.text = text;
    }
}
```

- **Deep Reactivity:** Only plain objects and arrays become deeply reactive.

### $state.raw

- **Purpose:** Declare state that cannot be mutated, only reassigned.
- **Usage:**

```javascript
<script>let numbers = $state.raw([1, 2, 3]);</script>
```

- **Performance:** Improves with large arrays and objects.

### $state.snapshot

- **Purpose:** Take a static snapshot of $state.
- **Usage:**

```javascript
<script>
    let counter = $state({ count: 0 });

    function onClick() {
        console.log($state.snapshot(counter));
    }
</script>
```

### $derived

- **Purpose:** Declare derived state.
- **Usage:**

```javascript
<script>let count = $state(0); let doubled = $derived(count * 2);</script>
```

- **Replaces:** Reactive variables computed using `$:` in non-runes mode.

### $derived.by

- **Purpose:** Create complex derivations with a function.
- **Usage:**

```javascript
<script>
    let numbers = $state([1, 2, 3]); let total = $derived.by(() => numbers.reduce((a, b) => a + b,
    0));
</script>
```

### $effect

- **Purpose:** Run side-effects when values change.
- **Usage:**

```javascript
<script>
    let size = $state(50);
    let color = $state('#ff3e00');

    $effect(() => {
        const context = canvas.getContext('2d');
        context.clearRect(0, 0, canvas.width, canvas.height);
        context.fillStyle = color;
        context.fillRect(0, 0, size, size);
    });
</script>
```

- **Replacements:** $effect replaces a substantial part of `$: {}` blocks triggering side-effects.

### $effect.pre

- **Purpose:** Run code before the DOM updates.
- **Usage:**

```javascript
<script>
    $effect.pre(() =>{' '}
    {
        // logic here
    }
    );
</script>
```

- **Replaces:** beforeUpdate.

### $effect.tracking

- **Purpose:** Check if code is running inside a tracking context.
- **Usage:**

```javascript
<script>console.log('tracking:', $effect.tracking());</script>
```

### $props

- **Purpose:** Declare component props.
- **Usage:**

```javascript
<script>let {(prop1, prop2)} = $props();</script>
```

- **Replaces:** export let syntax for declaring props.

### $bindable

- **Purpose:** Declare bindable props.
- **Usage:**

```javascript
<script>let {(bindableProp = $bindable('fallback'))} = $props();</script>
```

### $inspect

- **Purpose:** Equivalent to `console.log` but re-runs when its argument changes.
- **Usage:**

```javascript
<script>let count = $state(0); $inspect(count);</script>
```

### $host

- **Purpose:** Retrieve the this reference of the custom element.
- **Usage:**

```javascript
<script>
    function greet(greeting) {
        $host().dispatchEvent(new CustomEvent('greeting', { detail: greeting }));
    }
</script>
```

- **Note:** Only available inside custom element components on the client-side.

### Shareable Runes

Using classes with Runes can actually be faster, because they don't have to compile $state variables with get and set or with value.

```javascript
import { getContext, hasContext, setContext } from "svelte";

type RCurrent<TValue> = { current: TValue };

export class Rune<TRune> {

    readonly #key: symbol;

    constructor(name: string) {
        this.#key = Symbol(name);
    }

    exists(): boolean {
        return hasContext(this.#key);
    }

    get(): RCurrent<TRune> {
        return getContext(this.#key);
    }

    init(value: TRune): RCurrent<TRune> {
        const _value = $state({ current: value });
        return setContext(this.#key, _value);
    }

    // NOT NEEDED!
    update(getter: () => TRune): void {
        const context = this.get();
        $effect(() => {
            context.current = getter();
        });
    }
}
```

#### Initialize Shareable Rune

We must initialize our $state just like anywhere else, only once, in the parent component.

Example below:

```javascript
<script lang="ts">
    import {counter} from '$lib/counter.svelte'; const count = counter.init(0); // `count.current`
    is available here
</script>
```

#### Read Shareable Rune Anywhere

We can use it safely in a child component and read the current method.

Example below:

```javascript
<script lang="ts">
    import { counter } from '$lib/counter.svelte';

    const count = counter.get();
</script>

<h1>Hello from Child: {count.current}</h1>

<button type="button" onclick={() => count.current++}>
  Increment From Child
</button>
```

#### Update Shareable Rune Anywhere

You can update the rune using the current method as normal.

Example below:

```javascript
<script lang="ts">

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SimonPhumin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
