---
trigger: always_on
description: Framework-agnostic React Native renderer. Extract the entire native stack of
---

# SymbioteNative

Framework-agnostic React Native renderer. Extract the entire native stack of
React Native (Fabric C++, JSI, Yoga, the iOS/Android host) and let renderers for
**any** UI framework — Vue, Svelte, Solid, Angular, React — drive it through the
framework-agnostic seam that Fabric already exposes. One native core, N thin
adapters. The proven shape from [wolf-tui](./wolf-tui) (shared retained-tree +
a thin per-framework reconciler), retargeted from ANSI to native views.

> Local knowledge and decisions live in project Claude skills (`.claude/skills/`,
> authored with SkillForge), not in `.docs/` ADRs. Read the matching skill
> before proposing architectural changes — that's where the rationale and the
> explicit list of what it rules out live now.

## The one architectural fact everything rests on

React is **not** privileged inside React Native's renderer. Fabric exposes a
framework-agnostic, JSI-bound mutation API — `global.nativeFabricUIManager`:

```
createNode / cloneNodeWithNewProps / cloneNodeWithNewChildren
createChildSet / appendChildToSet / completeRoot
```

React's renderer is just **one client** of it, and all of its React-specific glue
lives in a single file: `react/packages/react-native-renderer/src/ReactFiberConfigFabric.js`
(`supportsPersistence = true`, `createInstance`, `cloneInstance`,
`createContainerChildSet`, `completeRoot`). "Removing React" means: stop calling
that host config, call the slot directly from your own renderer. **The native
core is never touched.**

## Architecture (locked — see project skills in `.claude/skills/`)

```
@symbiote-native/components : framework-agnostic state machines + render functions (→ Descriptor)
        │  every adapter wires state→render with ITS lifecycle (hooks / reactivity)
        ▼
Vue · Svelte · Solid · Angular · React        thin reconciler + descriptor→element bridge
        │  insert / remove / setProp / commit
        ▼
@symbiote-native/engine : retained shadow-tree + diff→childSet + event normalization
        │  ALL clone-on-write lives HERE, in one place
        ▼
nativeFabricUIManager  (createNode / cloneNodeWithNewProps / appendChildToSet / completeRoot)
        ▼
stock react-native : Fabric C++ · JSI · Yoga · RCTFabricSurface     ← never forked
```

## Invariants (do not violate without recording the change in a project skill)

<native_core_is_untouched>
We consume `react-native` as an ordinary dependency. We never fork, patch, or
vendor its native (C++/Obj-C++/JNI) sources. The only thing we replace is the
**JS renderer**: instead of React's Fabric host config, our own renderer drives
`nativeFabricUIManager`. If a task seems to require editing ReactCommon, Yoga, or
any native file — stop. That is a signal the design has drifted; raise it as a
new decision, do not patch native.
</native_core_is_untouched>

<react_native_is_an_explicit_top_level_peer>
`react-native` (and `react`) is a **peerDependency** of `@symbiote-native/engine` and every
adapter — never a regular or bundled dependency — and an **explicit top-level
dependency of the consuming app**. It is a runtime singleton and the Metro version
anchor (same class as `react` / `expo`): exactly one copy, declared at the app root.
The adapter facade hides only **imports** — app _code_ names only `@symbiote-native/*`, but
the app _manifest_ still pins `react-native`. Do NOT try to make RN a hidden /
transitive dependency to keep it out of the app's `package.json`: modern Expo
autolinking would resolve it, but it forfeits version-pinning and breaks pnpm
isolated installs — an ecosystem anti-pattern. The future `create-symbiote`
scaffolder owns the top-level pin so the developer never writes it by hand.
</react_native_is_an_explicit_top_level_peer>

<third_party_rn_packages_are_react_only>
A third-party React Native component package (`@react-native-community/slider`,
`react-native-*`, any library shipping a JS component) runs ONLY under the **React
adapter**. Its component body is React internally — it calls `useState` / hooks off
the React dispatcher — so when a non-React adapter (Vue, Svelte, Solid, Angular)
renders it the dispatcher is null and it throws (`Cannot read property 'useState' of
null`). SymbioteNative only makes the _native view_ framework-agnostic (it derives the RN
ViewConfig — events + prop processors — at runtime); it does NOT make the library's
React _component_ framework-agnostic. So: examples and canaries for a non-React
adapter must NOT import RN component packages. A native third-party view is reachable
from a non-React adapter ONLY through that adapter's own thin wrapper over the engine
(the same `createNode`-by-ViewConfig path SymbioteNative uses for its own primitives) — never
by importing the library's React component. Until such a wrapper exists, the component
is React-adapter-only.
</third_party_rn_packages_are_react_only>

<clone_on_write_lives_in_engine>
Fabric is persistent / clone-on-write: you never mutate a node, you clone it with
new props and atomically commit a new child set. Mutation-oriented frameworks
(Vue, Svelte, Solid, Angular) must NOT each reimplement this dance. The entire
mutation→clone-on-write translation lives once in `@symbiote-native/engine`. Adapters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OneEyed1366/symbiote-native](https://github.com/OneEyed1366/symbiote-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
