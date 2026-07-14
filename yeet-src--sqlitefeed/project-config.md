---
trigger: always_on
description: This is a **yeet** script: a reactive JSX TUI that runs in the daemon's V8
---

# Building yeet dashboards

This is a **yeet** script: a reactive JSX TUI that runs in the daemon's V8
isolate, fed by live kernel data (eBPF + a process/system graph). This file
is the API contract and gotcha list for editing it. For build/run mechanics,
layout, and the `@/`/`#/` aliases, see `README.md` — don't duplicate that here.

## Mental model

It reads like React but it is **signals, not a vdom**. No hooks, no
reconciliation, no `useState`. A node re-renders exactly when a signal it
*read* changes — and the only way to "read inside a node" is to pass a
**thunk** (`() => …`) as a prop or child. A plain value is static forever; a
thunk is reactive.

```jsx
<Text>{() => `load ${load.get().toFixed(2)}`}</Text>   // re-renders on load change
<Text>{`load ${load.get()}`}</Text>                    // snapshot, never updates
```

Three layers, composed:

```
probes/  (BPF-aware)  →  signals  →  components/ (pure UI, read signals)
                          ↑
                   graph queries / timers
```

`probes/` is the *only* code that touches `yeet:bpf`; it exposes plain
signals. Components never see BPF — they read signals. `lib/` is pure helpers.

## Build bottom-up: data → component → layout

Build a dashboard from the inside out. Each layer is verifiable on its own, so
mistakes surface where they're cheap — at the data, not three layers up where a
blank panel could mean anything.

### 1. Get the data right first, in isolation

Before any JSX, confirm the kernel actually gives you the fields and types you
think it does. Guard a self-test with `import.meta.main` — it's `true` **only**
when this module is the run entry, so the block runs when you point `yeet run`
at the module and stays dormant once `main.jsx` imports it.

Verify the **raw source**, not a `from()` signal (a `from()` producer doesn't
run until something watches it — there's no UI here):

```js
// probes/conns.js
import { BpfObject, RingBuf } from "yeet:bpf";
import { from } from "yeet:tui";

const ctl = await new BpfObject({ exe: "../bin/probe.bpf.o", base: import.meta.dirname })
  .bind("events", { kind: "ringbuf", btf_struct: "conn_event" })
  .start();
const events = new RingBuf(ctl, "events");

export const conns = from((state) => { /* …wrap events into a signal… */ }, []);

// Standalone correctness probe — dumps real records so you can eyeball field
// names, the btf_struct envelope, and which numbers came back as BigInt.
if (import.meta.main) {
  await events.subscribe((w) => console.log(JSON.stringify(w, (_k, v) =>
    typeof v === "bigint" ? `${v}n` : v)));        // JSON.stringify chokes on BigInt
}
```

For a graph probe the self-test is a one-shot dump:

```js
if (import.meta.main) {
  const { data } = await yeet.graph.query(QUERY);
  console.log(JSON.stringify(data, null, 2));
  yeet.exit();
}
```

Run it directly — `yeet run src/probes/conns.js`. **Caveat:** `@/` and `#/` are
bundle-time aliases, so a standalone module must reach its siblings by relative
path (`./probe.js`), or be bundled as its own entry. Switching `JSON.stringify`
to flag BigInt up front saves you the "why does math give NaN" detour — wrap
64-bit values with `Number(...)` once you've seen them.

### 2. Build each component against a fake signal

A component is a pure function of signals, so prove it in isolation with a
hand-fed signal before any real data exists. Mount just the one:

```jsx
// scratch entry while developing components/gauge.jsx
import { mount, signal } from "yeet:tui";
import Gauge from "@/components/gauge.jsx";

const fake = signal(0.3);
setInterval(() => fake.set(Math.random()), 700);   // exercise the reactive path
mount(() => <Gauge frac={fake} label="cpu" />);
await new Promise(() => {});
```

You're checking one thing: does it repaint when the signal changes, and does it
fit its box? Get sizing and the thunk wiring right here, with data you control,
before it has to share the screen.

### 3. Layout and routing last

Only once the pieces work do you compose them. The layout is a single thunk
that reads the size signal (reflow on resize) and a view signal (which panel is
showing) — responsive breakpoints and "routing" are the same branch:

```jsx
const view = signal("cpu");
tty.on("keydown", (e) => {
  if (e.key === "1") view.set("cpu");
  else if (e.key === "2") view.set("net");
});

const Root = (size) => (
  <Box>
    <TitleBar view={view} />
    <Box height="1fr" overflow="hidden">
      {() => {
        const { cols } = size.get();
        if (cols < 80) return <Stacked view={view} />;     // responsive
        switch (view.get()) {                               // routing
          case "cpu": return <CpuPanel cpu={cpu} />;
          case "net": return <NetPanel conns={conns} />;
        }
      }}
    </Box>
    <Footer />
  </Box>
);
```

By now each panel is already known-good, so if the screen looks wrong it's the
layout math — `1fr`/`fit`/fixed and `overflow`, nothing deeper.

## Entry shape

JSX is the **automatic runtime** (`jsxImportSource: yeet:tui` in tsconfig +
esbuild) — write JSX directly, no pragma import. The entry mounts a root that
receives the terminal's reactive size signal, then parks forever:

```jsx
import { Box, Text, mount, signal } from "yeet:tui";

const Root = (size) => (

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yeet-src/sqlitefeed](https://github.com/yeet-src/sqlitefeed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
