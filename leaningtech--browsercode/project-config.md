---
trigger: always_on
description: You are running inside **BrowserCode**, a demo environment that hosts AI coding agent CLIs inside **BrowserPod** sandboxes. Read this file in full before generating, installing, or running any code. The constraints below are not optional — they describe the actual runtime you are executing in.
---

# AGENTS.md — Operating Instructions for BrowserCode (BrowserPod Sandbox)

You are running inside **BrowserCode**, a demo environment that hosts AI coding agent CLIs inside **BrowserPod** sandboxes. Read this file in full before generating, installing, or running any code. The constraints below are not optional — they describe the actual runtime you are executing in.

---

## 1. What BrowserPod is

BrowserPod is a **browser-native sandbox**. The Node.js runtime, filesystem, processes, and network stack you have access to are **executing inside the user's browser tab**, compiled to WebAssembly. There is no remote VM, no container host, and no Linux kernel underneath you in the conventional sense — Pods target a Linux-compliant syscall layer implemented on top of the browser's security model.

Practical implications you must internalize:

- The CPU architecture is effectively **Wasm**, not x86_64 or arm64. Anything that assumes a host architecture will fail.
- The filesystem is a virtualized POSIX filesystem scoped to the Pod. Files persist within the browser, backed by the OPFS API or IndexedDB.
- Boot is near-instant; you do not need to wait on cloud provisioning.
- Concurrency is not metered — you can spawn additional processes freely, but you are still bound by the user's device resources.
- You have `bash`, `git`, `node`, `npm`, and standard coreutils. Use them.
- BrowserPod runs Node.js, Rust, and Python (preview); Go and Ruby are on the roadmap. This environment is set up for Node.js — assume that unless the user explicitly asks for another runtime.

---

## 2. The single most important constraint: native modules must be Wasm

Because the runtime is Wasm-based, **any npm package that ships prebuilt native binaries for a specific CPU architecture (x64, arm64, darwin, linux-gnu, etc.) will not run.** This is the failure mode you are most likely to hit, and it is the failure mode you must proactively prevent.

### The rule

For every project you scaffold or modify, check whether any direct or transitive dependency relies on native binaries. If it does, **add an `overrides` block to `package.json`** that swaps the native package for its Wasm equivalent.

### Common substitutions you should apply by default

| Native package | Wasm replacement |
|---|---|
| `esbuild` | use in `overrides`: `npm:esbuild-wasm@*` |
| `rollup` | use in `overrides`: `npm:@rollup/wasm-node@*` |
| `@parcel/watcher` | use in `overrides`: `npm:@parcel/watcher-wasm@*` |
| Next.js SWC native binary | add `@next/swc-wasm-nodejs` to `dependencies`, pinned to the same version as `next` |
| `@oxc-minify/*` (native) | `@oxc-minify/binding-wasm32-wasi` |
| `@oxc-parser/*` (native) | `@oxc-parser/binding-wasm32-wasi` |
| `@oxc-transform/*` (native) | `@oxc-transform/binding-wasm32-wasi` |

These cover the build-tool layer that virtually every modern JS framework (Vite, Svelte, Nuxt, Next, Astro, SvelteKit, SolidStart, Remix) depends on. **Add the esbuild, rollup, and `@parcel/watcher` overrides preemptively** for any Vite, Rollup, or Next.js-based project — do not wait to see an error. For Next.js, also add `@next/swc-wasm-nodejs` as a direct dependency at the matching version.

### Reference: Vite + Svelte `package.json`

```json
{
  "name": "vite-test2",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "devDependencies": {
    "@sveltejs/vite-plugin-svelte": "^5.0.3",
    "svelte": "^5.28.1",
    "vite": "^6.3.5"
  },
  "overrides": {
    "esbuild": "npm:esbuild-wasm@*",
    "rollup": "npm:@rollup/wasm-node@*"
  }
}
```

### Reference: Nuxt `package.json`

```json
{
  "name": "nuxt-app",
  "private": true,
  "type": "module",
  "scripts": {
    "build": "nuxt build",
    "dev": "nuxt dev",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "postinstall": "nuxt prepare"
  },
  "dependencies": {
    "@oxc-minify/binding-wasm32-wasi": "^0.98.0",
    "@oxc-parser/binding-wasm32-wasi": "^0.98.0",
    "@oxc-transform/binding-wasm32-wasi": "^0.98.0",
    "nuxt": "^3.17.5"
  },
  "overrides": {
    "esbuild": "npm:esbuild-wasm@*",
    "rollup": "npm:@rollup/wasm-node@*"
  }
}
```

### Reference: Next.js 13 `package.json`

Next.js is workable in BrowserPod, but its native SWC compiler must be replaced with the Wasm SWC build. Add `@next/swc-wasm-nodejs` as a direct dependency at the **same version as `next`**, and override `@parcel/watcher` (which Next pulls in for dev-mode file watching).

```json
{
  "name": "next-app",
  "scripts": {
    "dev": "next dev"
  },
  "dependencies": {
    "next": "13.5.11",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "@next/swc-wasm-nodejs": "13.5.11"
  },
  "overrides": {
    "esbuild": "npm:esbuild-wasm@*",
    "rollup": "npm:@rollup/wasm-node@*",
    "@parcel/watcher": "npm:@parcel/watcher-wasm@*"
  }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leaningtech/browsercode](https://github.com/leaningtech/browsercode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
