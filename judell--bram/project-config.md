---
trigger: always_on
description: You are running in the **terminal** of a Tauri desktop shell. The shell
---

# Bram

You are running in the **terminal** of a Tauri desktop shell. The shell
puts a real terminal (where you run) next to an agent pane the user can
SEE while talking to you. It can *optionally* also show a right-pane
target-app iframe, but that pane is **off by default** and often absent —
most users run their app in their own browser. Detect before you assume an
iframe is there; when one is present, use it.

Keep two distinct surfaces straight — they are not the same, and the rules
differ:

- **The agent pane** — Bram's own UI (the Worklist / Transcript / Sessions /
  Context / Status tabs). **Always XMLUI** — that's how Bram
  is built. Editing it means `app/tools/Main.xmlui`,
  `app/tools/components/*.xmlui`, `app/__shell/helpers.js`,
  `app/tools/Globals.xs`.
- **The target app** — whatever project the user is developing with Bram's
  help, shown in the **optional** target-app iframe *when that pane is
  enabled* (off by default, often absent). Per `app/__shell/conventions.md`,
  Bram "works with any project that serves a web UI (vanilla HTML/JS, a
  React or other Node app, a Python web app, an XMLUI app, etc.)." It **may
  or may not be XMLUI** — detect before you assume, and don't assume the
  pane is even present.

The XMLUI-specific guidance below is **unconditional when working on Bram
itself**, and applies to the target app **only when the target is XMLUI**.

## Working on Bram itself (the agent pane is XMLUI)

Most edits to Bram land in `.xmlui` files. Rules the xmlui-standalone
evaluator enforces hard:

- **No raw browser JS in event handlers** — `setTimeout`, `setInterval`,
  `fetch` outside DataSource, `async` / `await`, etc. are rejected at
  evaluation time with an unhandled rejection. Stay within App
  abstractions: `delay(ms)`, `debounce(ms, fn, ...args)`, the `Timer`
  component, `DataSource` for HTTP, `ChangeListener` for derived
  reactivity.
- **Lead with the xmlui-mcp tools** before reaching for a JS solution.
  The `xmlui_search_howto` tool is the fastest way to find the
  XMLUI-native pattern for a feature (e.g. "delay function", "debounce
  input", "wrap text in table cell"); `xmlui_component_docs` is for
  component-prop lookups; `xmlui_get_prompt` re-injects the server's
  framing guidance mid-session when you suspect you've drifted.
- **Cite a doc URL** for any non-obvious markup decision —
  `https://www.xmlui.org/docs/reference/components/<Name>` or
  `https://www.xmlui.org/docs/howto/<slug>`. If you can't cite one,
  search again.

The `xmlui-mcp` server is loaded for this conversation. Use it.

The helpers.js / Globals.xs / window code-organization discipline (where
each kind of code lives, when delegators are warranted, the `__bram*`
prefix, the xs failure modes, and the post-edit error grep) is in
`@docs/developing-bram.md`, `@`-imported below. That file is
source-repo-only; `@app/__shell/conventions.md` (also imported below) is
the cross-target half that Setup seeds into every managed project.

### Files you'll edit most (Bram)

- `app/tools/Main.xmlui` — the agent-pane surface
- `app/tools/components/*.xmlui` — Worklist (the primary gate, route
  `/worklist2`), Sessions, Toolbar, Architecture, etc. (the legacy
  `Workspace.xmlui` tab was retired in the 0.5.3 run)
- `app/tools/config.json` — XMLUI app config (resources, appGlobals)
- `app/tools/resources/*.svg` — custom icons; register in `config.json`
  under `resources` with the `icon.<name>` prefix
- `app/__shell/helpers.js` — window helpers loaded by `index.html` via
  `xmlui://localhost/__shell/helpers.js`

Reload boundary (settled 2026-08-26): under the documented `./bram`
symlink launch, ALL of `app/**` is served from disk per request —
`tools/**`, `helpers.js`, and `vendor/**` go live on a pane reload
(no rebuild); parent-shell files (`main.js`, `index.html`,
`styles.css`) need an app relaunch to re-execute but no rebuild.
Only `src-tauri/**` (Rust) is rebuild + relaunch territory. Launched
any other way (raw binary, installed bundle), everything serves from
the embedded tree and the old rebuild-everything rule applies — full
table, proofs, and launch discipline in `@docs/developing-bram.md`.

## Working on the target app

The embedded target app is **optional and off by default** — most sessions
won't have one (the user previews their app in their own browser). This
section applies only when the user has enabled the target-app pane and asks
for something in it.

When the user asks for something in the target app, **first detect what the
target is**, then render output its native way:

- **Vanilla HTML/JS** — `index.html` + plain `.js`, no framework manifest.
  Edit the HTML/JS directly.
- **React / other Node** — `package.json` (look for `react`, `vue`, `next`,
  etc.). Edit components in the project's own framework.
- **Python web app** — `requirements.txt` / `pyproject.toml` / `*.py`
  serving templates. Edit templates / handlers.
- **XMLUI** — `config.json` + `.xmlui` files. See *When the target app is
  XMLUI* below.

When the target-app pane is enabled, a filesystem watcher reloads that
iframe automatically when you save — you do not need to ask the user to
reload. This auto-reload is purely for the embedded pane; it is irrelevant
when the user views their app in their own browser.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [judell/bram](https://github.com/judell/bram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
