---
trigger: always_on
description: Note that "sunpeak", except where required in URLs or code, is always lowercase.
---

# sunpeak

Note that "sunpeak", except where required in URLs or code, is always lowercase.

sunpeak is a framework for building MCP Apps with interactive UIs that run inside AI chat hosts (ChatGPT, Claude, and future major hosts). Built on top of the MCP Apps SDK (`@modelcontextprotocol/ext-apps`).

ChatGPT apps are now submitted and published as plugins. The app portion is still based on MCP Apps or the OpenAI Apps SDK, so sunpeak's protocol and runtime architecture do not change. Adding a custom app requires ChatGPT Developer mode, enabled from the bottom-left user menu under `Settings > Security and login > Developer mode`. Local developer-mode apps are then created from the ChatGPT homepage under `Plugins > +`, while public apps are submitted as MCP-backed plugins through the OpenAI Platform.

The value proposition of the sunpeak framework is to help developers and their agents:

1. Test MCP Apps locally and automatically (in CI/CD) using a replica of the ChatGPT and Claude runtimes.
   1. Save time manually testing all possible host, server, app, ui, and backend states.
   2. Protect developers from 4-click manual refreshes on every code change in each host.
   3. Cancel all the $20 per person per host per month testing accounts.
   4. Avoid burning host credits on every test and code change.
2. Verify MCP tools work across multiple LLM models via the eval framework. Evals connect to the MCP server, discover tools, and send prompts to multiple models (GPT-4o, GPT-4o-mini, o4-mini, Claude Sonnet, Gemini 2.0 Flash) via the Vercel AI SDK. Cases can include App Context for follow-up turns that depend on model-visible UI state. Each case runs N times per model and reports pass/fail counts, so developers can confirm that tool descriptions, schemas, and App Context work reliably on smaller and cheaper models. Opt-in via `sunpeak test --eval` because evals cost money.
3. Automate the real-host testing loop with **live tests** (`sunpeak test --live`): scripts that drive a real browser into ChatGPT (and other hosts as they're supported), send prompts that trigger MCP tool calls against the developer's server, and assert against the actually-rendered app via Playwright. Live tests catch what the inspector can't (real MCP connection behavior, real LLM tool invocation, host-specific iframe rendering, production resource loading) and replace the manual prompt-and-click loop. Opt-in because they hit real accounts.
4. Build multi-platform MCP Apps in a structured way that's easy to understand and get started.
5. Test their MCPs in ChatGPT with HMR and Claude with automatic rebuilds and refresh notifications.
6. **Embed the Inspector as a React component** inside any third-party React app to render arbitrary MCP Apps. The `<Inspector app={...} onCallTool={...} />` shape exposes the same double-iframe runtime, host shells, and conversation chrome as the CLI inspector, but without any sunpeak-owned servers or `/__sunpeak/*` runtime dependencies. Tool calls flow through a callback the embedder owns; resource HTML is passed as a string. Embedders host the static sandbox proxy (`dist/sandbox-proxy.html`) on a separate origin from their own app — or fall back to the same-origin `srcdoc` variant for zero deployment. This is a first-class use case; treat it as load-bearing when designing changes.

## Embedded Inspector — design constraints

Maintain these properties when changing the Inspector or its supporting code:

- **Public hierarchical API**: the `app` prop on `<Inspector />` is the public input shape for embedders (`InspectorApp` → resources + tools → simulations). Flatten internally; never require embedders to learn the legacy flat `simulations` map.
- **No `/__sunpeak/*` runtime dependencies in the React component**: the bundled Inspector must not assume any sunpeak-owned HTTP endpoints exist in the embedder's origin. Health checks, tool calls, simulation discovery, and OAuth all flow through props (`onCallTool`, `app.resources[].html`) or are gated off in embedded mode.
- **Two CSS entries — keep them aligned**: `sunpeak/style.css` ships full Tailwind preflight (used by resources loading into iframe documents — `pnpm dev` and the CLI / template path). `sunpeak/embed.css` ships the same Tailwind utilities + theme, but with preflight scoped under `.sunpeak-inspector-root` (used by embedders hosting the Inspector inside another React app). When changing the scoped preflight rules, update both files — `src/inspector/globals.css` (for style.css, full preflight via `@import "tailwindcss"`) and `src/embed.css` (scoped preflight, theme + utilities only). Embed mode invariant: the host page's `<button>`, `<input>`, typography, and `data-theme` attribute must be untouched. The one acknowledged document-level injection is host `@font-face` (inert to host pages).
- **Theme + host-context application stays root-scoped**: variables, color-scheme, `data-theme` go on the Inspector's root ref, not `document.documentElement`. The `ThemeProvider` default still targets the document for callers using it outside the bundled Inspector — the Inspector overrides with a no-op.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alignbase/sunpeak](https://github.com/Alignbase/sunpeak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
