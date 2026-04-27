---
trigger: always_on
description: puru is a zero-dependency thread pool library for JavaScript with Go-style concurrency primitives such as channels, `WaitGroup`, `ErrGroup`, `select`, `context`, `Mutex`, `RWMutex`, `Cond`, and `Timer`. It runs functions off the main thread with no worker files and no boilerplate.
---

# puru — Guide for AI Assistants

puru is a zero-dependency thread pool library for JavaScript with Go-style concurrency primitives such as channels, `WaitGroup`, `ErrGroup`, `select`, `context`, `Mutex`, `RWMutex`, `Cond`, and `Timer`. It runs functions off the main thread with no worker files and no boilerplate.

Full AI-facing reference: https://raw.githubusercontent.com/dmop/puru/main/llms-full.txt

## Install

```bash
npm install @dmop/puru
# or
bun add @dmop/puru
```

## The Most Important Rule

Functions passed to `spawn()` are serialized via `.toString()` and sent to a worker thread. **They cannot access variables from the outer scope.**

```typescript
// WRONG — closes over `data`, will throw ReferenceError at runtime
const data = { id: 1 }
spawn(() => processData(data))

// WRONG — closes over `processData` imported in the outer file
import { processData } from './utils'
spawn(() => processData({ id: 1 }))

// RIGHT — inline everything the function needs
spawn(() => {
  function processData(d: { id: number }) {
    return d.id * 2
  }
  return processData({ id: 1 })
})

// RIGHT — inline the data as a literal
spawn(() => {
  const data = { id: 1 }
  return data.id * 2
})
```

Helper functions used inside `spawn()` must also be defined inside the function body:

```typescript
// WRONG
function fibonacci(n: number): number {
  if (n <= 1) return n
  return fibonacci(n - 1) + fibonacci(n - 2)
}
spawn(() => fibonacci(40)) // ReferenceError: fibonacci is not defined

// RIGHT
spawn(() => {
  function fibonacci(n: number): number {
    if (n <= 1) return n
    return fibonacci(n - 1) + fibonacci(n - 2)
  }
  return fibonacci(40)
})
```

## Common Patterns

### CPU-bound work (exclusive mode)

```typescript
import { spawn } from '@dmop/puru'

const { result } = spawn(() => {
  // define everything you need inside
  function crunch(n: number) {
    let sum = 0
    for (let i = 0; i < n; i++) sum += i
    return sum
  }
  return crunch(1_000_000)
})

console.log(await result)
```

### Multiple tasks in parallel (`task()`)

```typescript
import { task } from '@dmop/puru'

const items = [1, 2, 3, 4]
const processItem = task((item: number) => item * 2)
const results = await Promise.all(items.map((item) => processItem(item)))
```

### Concurrent I/O (concurrent mode)

```typescript
import { spawn } from '@dmop/puru'

const user = spawn(
  () => fetch('https://api.example.com/users/1').then((r) => r.json()),
  { concurrent: true },
)

const orders = spawn(
  () => fetch('https://api.example.com/users/1/orders').then((r) => r.json()),
  { concurrent: true },
)

const results = await Promise.all([user.result, orders.result])
```

### Cancel on first error (ErrGroup)

```typescript
import { ErrGroup } from '@dmop/puru'

const eg = new ErrGroup()
eg.spawn(() => fetch('https://api.example.com/users/1').then((r) => r.json()), { concurrent: true })
eg.spawn(() => fetch('https://api.example.com/users/1/orders').then((r) => r.json()), { concurrent: true })

const [user, orders] = await eg.wait() // throws on first error, terminates remaining workers
```

### ErrGroup with concurrency limit

```typescript
import { ErrGroup } from '@dmop/puru'

const eg = new ErrGroup()
eg.setLimit(4) // max 4 tasks in flight at once

eg.spawn(() => fetch('https://api.example.com/a').then(r => r.json()), { concurrent: true })
eg.spawn(() => fetch('https://api.example.com/b').then(r => r.json()), { concurrent: true })
eg.spawn(() => fetch('https://api.example.com/c').then(r => r.json()), { concurrent: true })
eg.spawn(() => fetch('https://api.example.com/d').then(r => r.json()), { concurrent: true })

const results = await eg.wait()
```

### Context-integrated spawn (auto-cancel)

```typescript
import { spawn, background, withTimeout } from '@dmop/puru'

// Task auto-cancels when context expires — no manual wiring needed
const [ctx, cancel] = withTimeout(background(), 5000)
const { result } = spawn(() => {
  let total = 0
  for (let i = 0; i < 1_000_000; i++) total += i
  return total
}, { ctx })

try {
  console.log(await result)
} finally {
  cancel()
}
```

### Context with WaitGroup / ErrGroup

```typescript
import { background, withTimeout, WaitGroup } from '@dmop/puru'

const [ctx, cancel] = withTimeout(background(), 5000)

// Pass context to WaitGroup — all tasks auto-cancel when ctx expires
const wg = new WaitGroup(ctx)
wg.spawn(() => { /* CPU work */ return 42 })
wg.spawn(() => fetch('https://api.example.com/data').then(r => r.json()), { concurrent: true })

try {
  const results = await wg.wait()
} catch {
  console.log('timed out or cancelled')
} finally {
  cancel()
}
```

### RWMutex (read-write lock)

```typescript
import { RWMutex } from '@dmop/puru'

const rw = new RWMutex()

// Many readers can run concurrently
const data = await rw.withRLock(() => cache.get('config'))

// Writers get exclusive access
await rw.withLock(() => cache.set('config', newValue))
```

### Timer (resettable one-shot)

```typescript
import { Timer, select } from '@dmop/puru'

const t = new Timer(5000)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmop/puru](https://github.com/dmop/puru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
