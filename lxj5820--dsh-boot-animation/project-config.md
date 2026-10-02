---
trigger: always_on
description: Maintainer briefing. Written for whoever (or whatever) picks this up next: what the
---

# AGENTS.md — dsh-boot-animation

Maintainer briefing. Written for whoever (or whatever) picks this up next: what the
package is, which two facts about DSH it depends on, the invariants that break it
silently, and how to prove a change is safe.

> **Porting note (DSH 0.2.0-rc.2).** The Host settings contract changed: a plugin
> no longer *registers* a settings namespace, it *declares* `Config`. Everything
> below reflects that. The `verify-*` suites this file names belong to the
> author's working copy and do **not** ship with this package — see
> [Verifying a change](#verifying-a-change) for what can be run here.

Human-facing usage lives in [MANUAL.md](MANUAL.md). The long forensic record —
every bug, why it happened, what the evidence was — is [README.md](README.md).
Read this file first; go to README only for the history of a specific symptom.

## What it is

A DSH plugin that replaces the kernel's boot page with a full-window video clip,
then dissolves into the app. No DSH source is modified: it injects a script into
`<head>` ahead of the shell, and contributes a card to Settings → Plugins.

| Half | File | Job |
|---|---|---|
| Host (Node) | `entry.js` | Serves clips over HTTP with Range, declares the `Config` schema the settings namespace is derived from, injects the pre-boot screen into the index |
| Browser (pre-boot) | `src/boot-screen.js` | The overlay itself: framework-free, injected as text, must run before the shell |
| Browser (plugin) | `src/client.js` → `lib/client.js` | Reports activation to the overlay, and renders the settings card |

### The two DSH facts everything rests on

1. **`webserver/index-inject` is the only moment early enough.** Rows pushed into
   that table render into `<head>`, ahead of the shell module. Anything later
   cannot pre-empt the boot page. `entry.js` subscribes to it.
2. **A settings namespace is a plugin entry, and a field is served only if it is
   volatile.** The Host declares `Config` on the module namespace object; the
   entry id in the profile patch (`- insert: id: boot-animation`) *is* the
   namespace; the browser card reaches that same id through the client
   `configForms` service. `@deepseek-ai/dsh-settings` describes only an entry
   whose `Config` projects a non-empty form, and that projection keeps only
   fields marked `.volatile()` — so a schema with no volatile field yields no
   namespace, no card, and no error anywhere. `src/client.js` holds the other end
   of the pairing in `SETTINGS_NAMESPACE`, and nothing in the harness checks that
   the two strings agree.

### What `.volatile()` costs on the Host side

A volatile field does not arrive in `apply(ctx, config)` as a value: it arrives as
a stable reference (`{ get() }`) whose snapshot the loader commits in place, so
`entry.js` reads every field through `readField()` rather than directly. That is
also why a settings edit needs no restart — when only volatile values change,
`@deepseek-ai/cordis-plugin-loader` calls `updateVolatile` on the existing
reference and emits `loader/volatile-update` instead of restarting the fiber, so
the next index render already sees the stored value.

## Rules that were each learned the hard way

Every line here cost a real debugging session. They are not style preferences.

| Rule | What happens if it is broken |
|---|---|
| **Never read/write a CJK-bearing source file with PowerShell.** Use the file tools; verify with Node `readFileSync(..., 'utf8')`. | PowerShell 5.1 decodes BOM-less UTF-8 as GBK, and a `Set-Content` round-trip destroys every Chinese character — once collapsed `'…'` into `鈥?` and broke the script. |
| **No backtick anywhere inside a CSS blob** (the screen's `style()` string, the card's `CSS` string). | The string ends early and the build keeps the *previous* artifact. It looks like the change had no effect. |
| **Read optional services with `ctx.inject([...], (owner) => owner.get(name))`.** Never `ctx.get(name)`. | `ctx.get` is a one-shot read that races activation and silently answers `undefined`; the card then never registers and there is no error anywhere. Every other client plugin in the deployment uses the `ctx.inject` form. |
| **The client half declares no `inject`.** | A required service leaves the fiber PENDING in a profile that lacks it, `apply` never runs, `clientReady` is never sent, and the animation refuses to hand over. |
| **Every settings field the card edits must be marked `.volatile()`** — and the schema library that resolves beside this package must actually have the method. | `@deepseek-ai/dsh-settings` projects volatile fields and nothing else: a `Config` with none is not described at all, so the namespace is never served, the card's `whileServed` never fires, and the settings row is simply absent with no error. The 3.18.1 build linked next to a workspace checkout has no `.volatile()` at all — calling it would throw while `entry.js` is being evaluated; `LIVE_CAPABLE` probes for it and warns instead. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lxj5820/dsh-boot-animation](https://github.com/lxj5820/dsh-boot-animation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
