---
trigger: always_on
description: Guidance for an AI (or human) building a terminal app **on top of** bare-tui.
---

# Building apps with bare-tui

Guidance for an AI (or human) building a terminal app **on top of** bare-tui.
This is not API reference — the [README](README.md) and [`docs/`](docs) cover
that, and each shipped component's source is short and meant to be read. This
file captures the patterns and traps that show up once an app grows past a
single screen. The [`examples/claude-code.js`](examples/claude-code.js) demo
exercises most of them in one place; reach for it as a worked reference.

## The shape of every app

bare-tui is [The Elm Architecture](https://guide.elm-lang.org/architecture/):

- **model** — a class holding all state. Mutating `this` is idiomatic.
- **`init() -> Cmd | null`** — the first command to run (timers, IO, a spinner).
- **`update(msg) -> [model, Cmd]`** — fold one message into new state. Return
  `[this, cmd]`; a bare `this` means "no command".
- **`view() -> string`** — render state to a string. **Keep it pure.** Do not
  mutate state, start work, or rebuild caches in `view()` — it can be called
  more than once per logical change, and side effects there cause flicker and
  heisenbugs. Compute in `update()`, render in `view()`.

Everything that happens is a message. Keystrokes, resizes, timer fires, async
results — all arrive through `update`. If you find yourself reaching for a
callback or an `await` inside `update`, you want a **Cmd** instead (below).

## Composing components (the thing you'll do constantly)

A component is just a model. To embed one: hold it as a field, route messages
into it, and thread its returned Cmd back up.

```js
update(msg) {
  if (msg.type === 'spinner.tick') {
    const [s, cmd] = this.spinner.update(msg)
    this.spinner = s
    return [this, cmd]          // thread the child's Cmd up to the Program
  }
  ...
}
```

In a larger app you'll have many components. Two rules keep this sane:

- **Gate input on focus.** Only the focused component should consume keys. The
  built-ins already no-op when unfocused (`textinput`, `autocomplete`, …), so
  you can broadcast a key to several and only the focused one reacts — but for
  your own components, follow the same contract.
- **Route by message type first, then broadcast the rest.** Handle the messages
  _you_ own (keys you bind, your own tagged msgs), then forward everything else
  to the active child. Don't try to demultiplex every key at the top level.

When you build your own component, copy the closest built-in and keep its
conventions: a `create()` factory, `update -> [model, cmd]` / `view -> string`,
ignore unrelated messages, gate on a `focused` flag, define keymaps with
`key.binding`, stay style-agnostic, and do animation/IO through Cmds.

## Commands, async, and the stale-result trap

A `Cmd` is `() => Msg | Promise<Msg> | null`. The runtime runs it _off_ the
update path and feeds the result back as a message. This is how you do timers,
network, disk, or worker IPC without blocking the UI. `init`/`update` return
Cmds; results come back as messages.

The trap that bites every non-trivial app: **an async result can arrive after
the state it was for is gone** — the user cancelled, navigated away, restarted
the operation, or a newer request superseded it. If you blindly apply it, you
get flicker, double-application, or a "ghost" of cancelled work.

The fix is a **generation/run id**. Stamp each operation with an id, capture
that id _when you issue the Cmd_, and ignore results whose id is stale:

```js
// issuing — capture the CURRENT id in the closure, do NOT read this.runId at fire time
_tick(ms) {
  const id = this.runId
  return tick(ms, () => ({ type: 'agent.advance', id }))
}

// handling — drop anything from a superseded run
if (msg.type === 'agent.advance') {
  if (msg.id !== this.runId) return [this, null]   // stale: cancelled or restarted
  ...
}

// cancelling/restarting — just bump the id; in-flight Cmds become stale
_interrupt() { this.runId++; this.busy = false; ... }
```

Capturing `id` at _issue_ time is the whole point — if your closure reads
`this.runId` at _fire_ time, a cancel that bumped the id will make the stale
result look current. The shipped `spinner` uses exactly this pattern
(`id` + monotonic `tag`) so a second spinner's ticks or a late/duplicate tick
can't double-drive its loop. Mirror it for your own loops.

Other Cmd notes:

- `batch(a, b)` runs Cmds concurrently; `sequence(a, b)` runs them in order.
- `tick(ms, fn)` is a relative delay; `every(ms, fn)` aligns to the wall clock
  so repeated timers don't drift. Re-issue from `update` to repeat.
- Errors thrown in a Cmd arrive as `{ type: 'error', error }` — handle them in
  `update` rather than letting the loop crash.

## Layout that survives the diff renderer

The renderer only repaints lines that changed, and components like `viewport`
emit a **fixed number of lines** every frame. This is fast, but it means
**layout stability matters**: if the number or position of lines shifts when it
doesn't need to, you get visible reflow and flicker.

Concrete consequences for a larger app:

- **Compute fixed regions from the window size.** On `resize`, derive your
  layout: e.g. `bodyHeight = totalHeight - headerHeight - footerHeight`, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [holepunchto/bare-tui](https://github.com/holepunchto/bare-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
