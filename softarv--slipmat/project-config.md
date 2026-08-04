---
trigger: always_on
description: Project instructions for Claude Code. Read this fully before writing code.
---

# CLAUDE.md

Project instructions for Claude Code. Read this fully before writing code.

## What this is

**Slipmat** — a small, native GNOME desktop app to play **Apple Music** on one
personal Linux laptop. Not a product, not multi-user, not cross-platform. One
user, one machine.

It is the sibling of **Dockyard** (a native GNOME Docker manager) and **Pitwall**
(a native GNOME GitHub Actions monitor), and shares their stack, architecture and
taste. The name is the felt disc between the platter and the record: the layer
that sits between the mechanism and the music, and lets the record turn on its
own terms.

The app should be indistinguishable from a first-party GNOME application. If a
design decision would make it look like an Electron app or a generic Qt tool, it
is the wrong decision.

**Why this project exists.** Apple ships no Linux client, and the DRM makes a
fully native one impossible — so the existing options load `music.apple.com` in
an Electron window, which is a reasonable answer to a hard constraint. Slipmat
takes a different one: draw the *interface natively* and keep the web engine
present but never rendered. Cider and Sidra solved the DRM first and this
depends on their groundwork; the difference is where the boundary sits, not who
did it right. **Never write about them with an edge** — in the README, in
commits, or in reply to the user.

**The headline feature is playback itself** — gapless transport with correct,
bidirectional **MPRIS** in the GNOME Shell applet and on the lock screen. Build
the rest in service of that. Browsing stays deliberately thin until transport is
perfect.

## Author context — read this, it changes how you should respond

The author is a senior frontend engineer (~10 years: TypeScript, React, React
Native, Node) who is **new to Rust**. Consequences:

- When you introduce ownership, borrowing, lifetimes, `Rc`/`Arc`/`RefCell`, or
  `async` pinning, **briefly explain why** in a comment or in your reply. Do not
  silently sprinkle `.clone()` to quiet the borrow checker — say what the
  ownership problem was and why the clone is the right or pragmatic fix.
- Analogies to React/Redux land well. relm4 *is* the Elm architecture; say so.
- Do not dumb down the Rust. Idiomatic code with explanation, not beginner code.
- Prefer clarity over cleverness. No macro tricks, no premature generics.
- The **sidecar is JavaScript**, which is home turf. Keep it small anyway —
  every line there is a line that isn't native.

## The constraint that shapes everything

Apple Music full-track playback is HLS + **Widevine** DRM (FairPlay only in
Safari). On Linux the only Widevine CDM that exists is the one Google ships
inside Chrome x86_64 and Chromium shells that bundle it.

| Path                            | Verdict                                                        |
| ------------------------------- | -------------------------------------------------------------- |
| WebKitGTK + MusicKit JS         | **Dead.** Ships Clear Key only; no Widevine CDM.                |
| Rust + GStreamer direct         | **Dead.** Cannot decrypt Widevine-protected HLS.                |
| Stock Electron                  | **Dead.** `navigator.requestMediaKeySystemAccess` is absent.    |
| `pywidevine` + an extracted CDM | **Rejected.** See rule 1. Out of scope, permanently.            |
| **castLabs Electron (`wvcus`)** | **Works.** *Fetches* the CDM — see below. What Sidra uses.      |

So a 100% native Apple Music player *cannot exist*. The honest ceiling — and the
whole design — is: **everything the user sees is native; the audio decoder is
invisible.**

**`wvcus` does not ship the CDM — it fetches it.** The suffix is *Widevine CDM
Update Service*: what castLabs adds is the plumbing that lets Chromium's own
component updater pull the CDM down, exactly as Chrome does for itself. There is
no Widevine binary anywhere in the 327 MB Electron dist; verify with

```bash
find sidecar/node_modules -iname "*widevine*"   # finds nothing
```

It lands at `~/.config/Slipmat/WidevineCdm/` on first run, beside the copies
Chrome and Chromium fetched for themselves.

This matters for **packaging**, and this entry used to say "bundles", which
pointed the wrong way. Nothing Slipmat would ship contains proprietary Google
code: Electron is MIT, our code is GPL-3, and the CDM arrives on the user's own
machine through their own component updater into their own config directory. So
redistribution is not the obstacle it looked like — a Flatpak or an AUR package
would carry Electron and nothing more, and the CDM download needs only network
access and a writable, persistent config directory.

**And that is now measured rather than reasoned.** Slipmat was run inside a real
`org.gnome.Platform//49` sandbox with `--nofilesystem=home`, a private `HOME`
carrying the Apple session but **no CDM**, and playback worked:

```text
widevine ready: {"status":"updated","version":"4.10.3050.0"}
…/.config/Slipmat/WidevineCdm/4.10.3050.0/_platform_specific/linux_x64/libwidevinecdm.so
```

So the component updater reaches out over `--share=network`, writes into the
app's own config directory, and the CDM decrypts — a track played. The original
plan called a sandboxed CDM "genuinely hard" and deferred Flatpak on the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SoftARV/Slipmat](https://github.com/SoftARV/Slipmat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
