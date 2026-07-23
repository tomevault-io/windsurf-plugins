---
trigger: always_on
description: JS isolation engine: a Compartment-shaped facade owns the Proxy **Membrane**, classic-script evaluator, and module hooks. DOM behavior lives in independently registered **IsolationPlugin** instances. Depends only on `@qiankunjs/shared`.
---

# @qiankunjs/sandbox

JS isolation engine: a Compartment-shaped facade owns the Proxy **Membrane**, classic-script evaluator, and module hooks. DOM behavior lives in independently registered **IsolationPlugin** instances. Depends only on `@qiankunjs/shared`.

## STRUCTURE

```
sandbox/
├── core/
│   ├── sandbox/          # createSandbox() + StandardSandbox preset + container protocol
│   ├── membrane/         # Proxy wrapper for global (window/document) isolation
│   ├── compartment/      # globals + module hooks + blob classic evaluation facade
│   ├── globals.ts        # global property definitions
│   └── esm-globals.ts    # esmDestructurableGlobals — globals the ESM engine may destructure/rebind
├── patchers/             # public IsolationPlugin protocol + built-in plugin presets
│   ├── dynamicAppend/    # appendChild/insertBefore interception → redirect to app container
│   ├── windowListener.ts # event listener tracking
│   ├── interval.ts       # timer tracking
│   └── historyListener.ts
└── consts.ts             # qiankunHeadTagName / qiankunBodyTagName, nativeGlobal, nativeDocument
```

## WHERE TO LOOK

| Task | File | Notes |
| --- | --- | --- |
| Create sandbox | `core/sandbox/index.ts` | Builds a `StandardSandbox`, installs the JS-only or DOM preset, and returns mount/unmount/dispose |
| Prepare container | `core/sandbox/container.ts` | Owns `<qiankun-head>`, `data-name`, style scope, and cleanup contracts |
| Proxy logic | `core/membrane/index.ts` | Write → local target; Read → local → configured globals → host window |
| Compartment facade | `core/compartment/index.ts` | Owns the membrane, module facade, and CSP-safe blob classic evaluation |
| Plugin protocol | `patchers/types.ts` | Public `IsolationPlugin`, context, `Free`, and `Rebuild` contracts |
| Built-in plugins | `patchers/index.ts` | Data-driven Standard/Snapshot presets; user plugins append after these |
| DOM interception | `patchers/dynamicAppend/forStandardSandbox.ts` | Redirects dynamic script/style/link to app container |
| Side-effect cleanup | `patchers/*.ts` | Each patcher returns a `free()` called on unmount |
| ESM globals contract | `core/esm-globals.ts` | Consumed by `shared/esm-sandbox` engine, passed as `globalsBaseSet` |

## KEY PATTERNS

### Membrane (Proxy)

- **Writes** are trapped and stored on a local `target` object (the sandbox's own globals).
- **Reads** check local target → configured globals → fall back to the real host window.
- **Native rebinding**: `fetch`, `console`, etc. are rebound to the real receiver to avoid "Illegal invocation".
- `latestSetProp` records the last global the entry script assigned — that's how the loader recovers a classic app's exported lifecycles when no explicit export exists.
- **Non-transitive by design**: unlike full membranes (es-membrane, Salesforce near-membrane), only the global (and document view) entry point is proxied — objects crossing the boundary keep their raw identity (`===`/`instanceof` hold across it). This is qiankun's deliberate isolation stance (isolate writes, share identity, for DOM compatibility and performance), not a gap to fill. Do NOT wrap crossing objects with proxies. The Membrane is a private mechanism of `Compartment` — the spec has no membrane concept, which is exactly why it must stay behind the facade.
- `incubatorContext` (the host window the view reads through to) is a qiankun host extension: the Compartment spec and ses isolate by absence and have no read-through concept. The name borrows the ShadowRealm proposal's official "incubator realm" term for the creating context.

### Compartment surface and deliberate differences

`new Compartment({ globals, modules, resolveHook, importHook, loadHook, transforms })` owns its `globalThis`. StandardSandbox adds qiankun self-references (`window`, `self`, `globalThis`, `top`, `parent`) in a second phase with `defineUnshadowableGlobals()`.

- Deliberately absent: synchronous `evaluate`, `harden`, and `lockdown`.
- qiankun host extensions: DOM globals, `evaluateScript`, the synchronous `transformClassicScript` streaming adapter, lifecycle and entry-discovery facades, module-document registration, `defineUnshadowableGlobals`, `dispose`, `incubatorContext` / `moduleHost`, and the IsolationPlugin protocol. The executable list lives in `COMPARTMENT_HOST_EXTENSIONS`.
- `evaluateScript()` uses a native blob `<script>` path. Do not introduce `eval` or `new Function` as an implementation shortcut.
- `transformClassicScript()` exists because loader node transformation must finish synchronously before the original node enters live DOM. Loader consumes it through structural `CompartmentLoaderFacade`; never restore a concrete-class or private WeakMap bridge.
- The pre-alignment names `Endowments` and `addIntrinsics()` have been removed entirely (the package has no external users, so no compatibility aliases). Use `CompartmentGlobals` / `globals` and `defineUnshadowableGlobals()`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umijs/qiankun](https://github.com/umijs/qiankun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
