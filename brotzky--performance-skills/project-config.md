---
trigger: always_on
description: Agent-loadable instructions distilled from each article on performance.dev. Claude Code reads this file as project-level guidance when it lives at the project root.
---

# performance.dev skills

Agent-loadable instructions distilled from each article on performance.dev. Claude Code reads this file as project-level guidance when it lives at the project root.

Canonical source: https://performance.dev/skills
Plugin install: `/plugin marketplace add brotzky/performance-skills && /plugin install perf@skills`

---

## linear-local-first-architecture

> Use when building a web app that must feel instant, when users complain about spinners or perceived slowness despite acceptable latency, or when designing a local-first sync architecture with optimistic updates.

## When to use this skill

- You're building a productivity tool where perceived speed is critical to user experience
- Users report the app "feels slow" despite reasonable network latency
- You need to eliminate loading spinners and skeleton states from user workflows
- You're architecting a local-first application with offline capabilities
- You want to implement optimistic updates that feel instant
- You're designing a keyboard-first interface for power users

## Core principles

1. **The network is the bottleneck—eliminate it wherever possible.** Every network request costs hundreds of milliseconds; the best optimization is to avoid the request entirely by reading from local state.

2. **Treat the browser as the database for each user.** Store the full workspace in IndexedDB and hydrate into an in-memory observable graph; the UI reads from local state, not the server.

3. **Mutations apply locally first, sync asynchronously.** Update the local observable immediately so the UI re-renders synchronously, then queue the transaction for background sync to the server.

4. **Render first, authenticate second.** If local data exists, render it immediately and verify the session in the background; only redirect to login if the server rejects.

5. **Ship less code in more pieces.** Aggressive code splitting, modern-only targets, and per-package vendor chunks reduce initial payload and improve cache granularity.

6. **Animate only composited properties.** Restrict animations to `transform` and `opacity` to keep work on the GPU; never animate layout-triggering properties like `width`, `height`, or `margin`.

## Tactics

### Set up local-first data architecture

Store the workspace in IndexedDB and hydrate into an in-memory observable store (MobX in Linear's case). The UI queries the local store, not the server.

```typescript
// A traditional web app updating the server
async function updateIssue({ issue }) {
  showSpinner();
  const response = await fetch(`/api/issues/${issue.id}`, {
    method: "PATCH",
    body: JSON.stringify({ title: issue.title }),
  });
  const updated = await response.json();
  setIssue(updated)
  hideSpinner();
}

// vs Linear
issue.title = "Faster app launch";
issue.save();
```

The first line updates an in-memory datastore (MobX observable). The second line queues a transaction that the sync engine batches and flushes to the server. The UI re-renders synchronously off the local update—no spinners, no waiting.

### Implement optimistic updates with standard libraries

If you're not building a custom sync engine, use libraries like Tanstack Query or SWR with optimistic updates:

```typescript
// optimistic mutation with SWR
mutate(
  `/api/issues/${issue.id}`,
  { ...issue, title: "Faster app launch" },
  false
);

// vs Linear
issue.title = "Faster app launch";
issue.save();
```

The key: UI responsiveness should not depend on network latency. Users perceive speed based on how quickly the interface reacts, not how quickly the server responds.

### Reduce bundle size with modern-only builds

Target only modern browsers, drop legacy polyfills, and use aggressive code splitting:

```typescript
// vite.config.ts (reconstruction; matches observed chunk graph)
export default defineConfig({
  plugins: [react()],
  build: {
    target: "esnext",            // no legacy syntax, no polyfills
    cssMinify: "lightningcss",
    modulePreload: { polyfill: false },
    rollupOptions: {
      output: {
        // One chunk per npm package > ~3 KB. Cache invalidation
        // becomes per-library instead of per-app-revision.
        manualChunks(id) {
          if (id.includes("node_modules")) {
            const pkg = id.match(/node_modules\/([^/]+)/)?.[1];
            if (pkg) return `vendor-${pkg}`;
          }
        },
      },
    },
  },
});
```

Split every npm package into its own chunk. A traditional `vendor.js` invalidates the entire dependency graph on any bump; per-package chunking makes cache invalidation fine-grained.

### Preload critical chunks in parallel

Eliminate the waterfall of fetch → parse → fetch imports by declaring all critical chunks in `<head>`:

```html
<script type=module crossorigin
  src="https://static.linear.app/client/assets/html.2_JBQs3Q.js"></script>
<link rel=modulepreload crossorigin
  href="https://static.linear.app/client/assets/vendor-mobx.Crhy2qQc.js">
<link rel=modulepreload crossorigin
  href="https://static.linear.app/client/assets/SyncWebSocket.Djw6l_Op.js">
<link rel=modulepreload crossorigin

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brotzky/performance-skills](https://github.com/brotzky/performance-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-25 -->
