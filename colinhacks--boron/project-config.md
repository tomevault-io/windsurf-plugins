---
trigger: always_on
description: **Read [wiki/architecture.md](wiki/architecture.md) first.** It is the map of the system — the four layers and which way they depend, how a paste becomes pixels, the invariants that are load-bearing (above all: everything Boron draws has to be expressible as an ANSI escape sequence), the three renderers that have to agree, and a "where do I change X?" table. This file covers *process*; that one covers *the code*. Share links are **not shipped** — [wiki/share-links.md](wiki/share-links.md) explai
---

# Working in this repo

**Read [wiki/architecture.md](wiki/architecture.md) first.** It is the map of the system — the four layers and which way they depend, how a paste becomes pixels, the invariants that are load-bearing (above all: everything Boron draws has to be expressible as an ANSI escape sequence), the three renderers that have to agree, and a "where do I change X?" table. This file covers *process*; that one covers *the code*. Share links are **not shipped** — [wiki/share-links.md](wiki/share-links.md) explains why the first attempt was pulled, and [wiki/og-images.md](wiki/og-images.md) sketches the server-side card that depended on them. The latter documents a measured gap worth knowing about either way: the bundled font is latin-only, so box-drawing and arrows lay out against the reader’s system font.

## Git

**Commit straight to `main`.** No branches, no pull requests, no staging your work for review. When something is done, commit it and push it — a push deploys to [boron.sh](https://boron.sh) via Vercel, and that is fine.

**Don't spend effort on git hygiene.** Atomic commits, tidy history, branch strategy — none of it matters here. Do not stop to ask whether a commit is scoped cleanly, and do not leave finished work uncommitted because the diff also touches something else.

**Several agents may be working at once**, so the tree you are in probably contains edits that are not yours. That is normal and expected:

- Leave other agents' changes alone. Don't revert them, don't "fix" them, and don't wait for them to finish.
- `git add` the paths you actually worked on and commit. If someone else's in-flight change rides along, that is acceptable — it is not worth a round trip to avoid.
- Your own files may get swept into someone else's commit before you get to them. Check `git log -- <path>` before assuming work was lost.
- The one thing that does bite: never rewrite shared history. No `rebase`, `reset --hard`, or force-push on `main` — other agents are working from it, and rewriting it destroys their work rather than just untidying yours.

## Toolchain

The package manager is **`nub`**, not npm or pnpm.

| Command | What it does |
| --- | --- |
| `nub run dev` | Vite dev server |
| `nub run build` | `tsc --noEmit`, then a production build to `dist/` |
| `nub run test` | Vitest |
| `nub run typecheck` | `tsc --noEmit` alone |

`nub run build` is the gate — it typechecks before bundling, so a green build covers both.

This is a **Vite + React** app, not Next.js. There is a root `index.html` carrying all the page metadata, the entry is `src/main.tsx`, and static files are served from `public/` verbatim.

## Copy

Emojis are fine here — in the README, on the site, and in commit messages. The general prose guide bans them; this project overrides that.

## Persisted state

The whole workspace — document, theme, backdrop and frame settings — is saved to `localStorage`. A changed default therefore reaches nobody who has already opened the app; their stored copy wins. When you change a default that everyone should get, bump `STORAGE_KEY` in `src/App.tsx`.

## Clipboard fixtures

`src/core/clipboard-fixtures.ts` holds rich-text clipboard payloads captured **verbatim from a real copy**. Terminals disagree enough about their `text/html` flavour that invented markup only tests your imagination — Ghostty marks every styled run as a `<div style="display: inline">`, VS Code and Konsole use one block element per row, and Terminal.app states its colours only in class-based CSS in a `<style>` block, with each row's dominant colour on the `<p>` and the runs that differ overriding it.

**Capture HTML the way the app receives it — off a real paste event, not off the pasteboard.** On macOS those are not the same thing. Terminal.app and iTerm2 write no HTML flavour at all, only `public.rtf`, and inspecting the pasteboard makes them look unsupportable. They aren't: Chrome converts RTF through `NSAttributedString` on the way in, so the page is handed a full `text/html` that the pasteboard never held. That conversion lives in Chromium's `clipboard_mac.mm` and exists **only on macOS**, which is why we parse `text/rtf` ourselves too — see `rtf-paste.ts`. RTF has no such subtlety and can be read straight off the pasteboard.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colinhacks/boron](https://github.com/colinhacks/boron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
