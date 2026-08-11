---
trigger: always_on
description: > This guide targets developers who are already familiar with the web front-end stack. It explains Lynx's core principles, how it differs from the web, and how to build cross-platform applications with Lynx.
---


# Read This Before Building with Lynx

> This guide targets developers who are already familiar with the web front-end stack. It explains Lynx's core principles, how it differs from the web, and how to build cross-platform applications with Lynx.

---

> \[!IMPORTANT] > **Critical information:**
>
> Reference links are provided after certain statements, pointing to the corresponding sections of the official Lynx documentation. Please read them **proactively** for a complete understanding.

---

## 1. Positioning and Capability Boundaries

* **What**: Lynx is a cross-platform rendering engine that treats the web as its semantic baseline. It targets iOS, Android, HarmonyOS, and the web with a single codebase. A unified element abstraction maps to native views or custom web elements on different hosts, avoiding the performance bottlenecks of traditional WebViews. (See [Composing Elements](https://lynxjs.org/guide/ui/elements-components.md))
* **Why**: Mobile users are extremely sensitive to first-screen time and interaction latency. Lynx combines a dual-thread JavaScript runtime, Instant First-Frame Rendering (IFR), and a native rendering pipeline to deliver the React developer experience alongside near-native performance. (See [Instant First-Frame Rendering](https://lynxjs.org/guide/interaction/ifr.md))
* **How**: On the tooling side, use Rspeedy (an Rspack-powered build tool) to generate a Lynx bundle. On the front end, ReactLynx (a React implementation with a Preact core) describes the UI and communicates with the host through Native Modules or Custom Elements. (See [ReactLynx](https://lynxjs.org/react/introduction.md), [Native Modules](https://lynxjs.org/guide/use-native-modules.md))

## 2. Mental Model: Align Lynx with the Web

| Web Mental Model             | Lynx Counterpart                                                                | Key Differences                                                                                                                                                                                                               |
| ---------------------------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `index.html` + assets        | Lynx bundle (binary that contains JS bytecode + styles) or a `template.js` file | Bundles must be compatible with the Lynx engine version; configure `engineVersion` (previously `targetSdkVersion`). (See [Compatibility](https://lynxjs.org/guide/compatibility.md))                                                            |
| DOM + CSSOM                  | Element tree + styling system                                                   | Every element behaves like a block-level node. Custom tags such as `view`/`text` map to native controls. (See [Composing Elements](https://lynxjs.org/guide/ui/elements-components.md))                                                         |
| Browser main thread          | Lynx main thread                                                                | Handles first-screen rendering, layout, and main-thread scripts, executing PrimJS bytecode. (See [Main Thread Runtime](https://lynxjs.org/guide/scripting-runtime/main-thread-runtime.md))                                                      |
| Browser rendering-task queue | Lynx background thread                                                          | Runs ReactLynx scheduling, lifecycle, and most side effects. Executes PrimJS/JavaScriptCore with syntax support up to ES2015 (SWC transpiles during build). (See [JavaScript Runtime](https://lynxjs.org/guide/scripting-runtime/index.md))     |
| `window` / `document`        | `lynx` global object + API set                                                  | No DOM APIs. Access nodes via `lynx.getElementById`, SelectorQuery, `main-thread:ref`, etc. (See [ReactLynx](https://lynxjs.org/react/introduction.md), [Direct Manipulation](https://lynxjs.org/guide/interaction/event-handling/manipulating-element.react.md)) |

## 3. Runtime Architecture: How Dual-Thread React Works

* **Dual-thread parallel rendering**: The main thread renders ReactLynx output immediately for the first screen, while the background thread constructs the full node tree and syncs state back to the main thread to avoid a white screen. (See [Rendering Process and Lifecycle](https://lynxjs.org/react/lifecycle.md), [IFR](https://lynxjs.org/guide/interaction/ifr.md))
* **“Your code runs on two threads”**: Dual-thread React means your logic can run on both threads. However, not all code can execute in both environments—some APIs are only available on the background thread, and ReactLynx only executes events, lifecycle hooks, and `useEffect`-style side effects from the background thread.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tamer4lynx/tamer4lynx](https://github.com/tamer4lynx/tamer4lynx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
