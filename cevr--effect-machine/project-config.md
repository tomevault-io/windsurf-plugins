---
trigger: always_on
description: Type-safe state machines for Effect.
---

# effect-machine

Type-safe state machines for Effect.

## Commands

```bash
bun run gate          # typecheck + lint + test + build
bun test              # Run tests
bun run typecheck     # tsgo --noEmit for v4 and v3, patched by @effect/tsgo
bun run lint          # type-aware oxlint; Effect diagnostics run through tsgo plugin
bun run fmt           # oxfmt
```

## Conventions

- Files: kebab-case (`actor.ts`, `persistent-actor.ts`)
- States/Events: schema-first with `State({...})` / `Event({...})` - they ARE schemas
- Empty structs: plain values - `State.Idle` (not callable)
- Non-empty: `State.Loading({ url })` - constructor requiring args
- Machine creation: `Machine.make({ state, event, initial })` - types inferred
- Exports: all public API via `src/index.ts`
- Namespace pattern: `import { Machine } from "effect-machine"` then `Machine.make`, etc.
- v4 services use `class X extends Context.Service<X, Shape>()("key") {}`; keep `serviceNotAsClass` enabled.
- `effect` stays peer-only at runtime; keep concrete Effect versions in dev deps for validation.

## Fluent Builder

```ts
const machine = Machine.make({ state, event, initial })
  .on(State.Idle, Event.Start, () => State.Running)
  .on([State.Draft, State.Review], Event.Cancel, () => State.Cancelled) // multi-state
  .onAny(Event.Reset, () => State.Idle) // wildcard (any state)
  .spawn(State.Running, ({ slots }) => slots.poll())
  .timeout(State.Loading, { duration: Duration.seconds(30), event: Event.Timeout })
  .postpone(State.Connecting, Event.Data)
  .final(State.Done);
```

- Builder methods mutate `this`, return `this`
- Builder chain ends naturally — no terminal method needed
- `.onAny()` fires when no specific `.on()` matches for that event

## .task()

Async work that emits an event on completion:

```ts
// Explicit onSuccess mapping
.task(State.Loading, ({ state }) => fetchData(state.url), {
  onSuccess: (data) => Event.Loaded({ data }),
  onFailure: (cause) => Event.Failed({ error: Cause.pretty(cause) }),
})

// Shorthand — when task returns Event directly, onSuccess can be omitted
.task(State.Loading, ({ state }) => fetchData(state.url).pipe(Effect.map(d => Event.Loaded({ data: d }))), {
  onFailure: (cause) => Event.Failed({ error: Cause.pretty(cause) }),
})

// Multi-state
.task([State.Loading, State.Retrying], ({ state }) => fetchData(state.url), { onSuccess: ... })
```

## State.derive()

Construct state from existing source. Per-variant and union-level:

```ts
// Per-variant: target-specific, works cross-state
State.Active.derive(state, { count: state.count + 1 });
State.Shipped.derive(processingState, { trackingId: "TRACK-123" });
State.Idle.derive(anyState); // → { _tag: "Idle" }

// Union-level: dispatches by _tag, preserves specific variant subtype
const updated = MyState.derive(state, { queue: newQueue });
// If state is Streaming, returns Streaming (not union type)
// Partial keys not in target variant are silently dropped
```

## Slots

Unified parameterized slots via `Slot.define` + `Slot.fn`. Handlers take only params (no ctx parameter):

```ts
const MySlots = Slot.define({
  canRetry: Slot.fn({ max: Schema.Number }, Schema.Boolean),
  fetch: Slot.fn({ url: Schema.String }),
});

const machine = Machine.make({ state, event, slots: MySlots, initial }).on(
  State.X,
  Event.Y,
  ({ slots }) =>
    Effect.gen(function* () {
      if (yield* slots.canRetry({ max: 3 })) {
        yield* slots.fetch({ url: "/api" });
      }
      return State.Z;
    }),
);

// Provide slot implementations at spawn time — handlers take only params
const actor =
  yield *
  Machine.spawn(machine, {
    slots: {
      canRetry: ({ max }) => attempts < max,
      fetch: ({ url }) => Http.get(url),
    },
  });
yield * actor.start;

// When a handler needs machine state, access via service
Machine.spawn(machine, {
  slots: {
    canRetry: ({ max }) => machine.Context.pipe(Effect.map((ctx) => ctx.state.attempts < max)),
  },
});
```

## Running Machines

**Simple (no registry):**

`Machine.spawn` returns an **unstarted** actor. Call `actor.start` to fork the event loop.

```ts
const actor = yield * Machine.spawn(machine);
yield * actor.start; // fork event loop, background effects, spawn effects
yield * actor.stop; // caller responsible

// Scope-aware — use Machine.scoped to bridge ActorScope from Scope:
yield *
  Effect.scoped(
    Machine.scoped(
      Effect.gen(function* () {
        const actor = yield* Machine.spawn(machine);
        yield* actor.start;
        // actor.stop called automatically when scope closes
      }),
    ),
  );
```

**With registry:**

`system.spawn` auto-starts — no `actor.start` needed.

```ts
const system = yield * ActorSystemService;
const actor = yield * system.spawn("my-id", machine);
```

**ActorScope:** `Machine.spawn` and `system.spawn` detect `ActorScope` via `Effect.serviceOption` — if present, attach stop finalizer; if absent, skip. Use `Machine.scoped(effect)` to bridge `Scope.Scope` → `ActorScope`. This is explicit opt-in — ambient `Scope.Scope` does NOT trigger auto-cleanup (prevents bugs where unrelated scopes tear down actors).

## Recovery + Durability

Lifecycle hooks for persistence. Replace the old `PersistConfig`.

```ts
const actor =
  yield *
  Machine.spawn(machine, {
    lifecycle: {
      recovery: {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cevr/effect-machine](https://github.com/cevr/effect-machine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
