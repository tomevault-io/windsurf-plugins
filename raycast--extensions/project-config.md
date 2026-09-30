---
trigger: always_on
description: A Raycast extension listing the internal commands of Chromium-based browsers (`chrome://`, `brave://`,
---

# Secret Browser Commands

A Raycast extension listing the internal commands of Chromium-based browsers (`chrome://`, `brave://`,
`edge://`, …) and opening them in a chosen browser.

## Commands

- `npm run dev` — run in development mode
- `npm run build` — build the extension
- `npm run lint` / `npm run fix-lint` — lint
- `npm run check-paths` — invariant check for the generated command data (see below); also runs as part of `build` and `lint`
- `npm run publish` — publish to the Raycast Store

## Layout

| Path                               | What it holds                                                  |
| ---------------------------------- | -------------------------------------------------------------- |
| `src/listCommands.tsx`             | The single command: list, filters, detail pane, ActionPanel    |
| `src/components/OpenInActions.tsx` | The "Open in…" submenu                                         |
| `src/data/paths.ts`                | **Generated.** 307 command entries — see _The command data_ below |
| `src/types/types.ts`               | `BrowserCommand`, the shape of every entry in `paths.ts`       |
| `src/types/browsers.ts`            | `SUPPORTED_BROWSERS` — key, title, scheme, app name, bundle id |
| `src/utils/browserApps.ts`         | Which browsers are installed; which icon each gets             |
| `src/utils/browserUrl.ts`          | `buildBrowserUrl` — the only place a full URL is assembled     |
| `src/utils/openUrlInBrowser.ts`    | Launching, macOS only                                          |
| `src/utils/check-paths.mjs`        | Invariant check over the generated data                        |
| `docs/paths.md`                    | The census this data came from, per browser                    |
| `docs/solutions/`                  | Durable learnings from solved problems — read before re-solving |
| `CONCEPTS.md`                      | Glossary of the terms that mean something specific here        |

## The command data

`src/data/paths.ts` is **generated from a census**, not hand-maintained. Each browser's own
`chrome://chrome-urls` page was read over the DevTools Protocol (2026-09-09), with _Internal
debugging pages_ enabled, and that browser's list became its `supportedBrowsers` membership.
`docs/paths.md` records the result and is the receipt for every provenance claim in this section. Vendor pages — `brave://wallet`, `opera://mods`,
`comet://perplexity-spotlight` — appear in no Chromium source file, so the live census is the only
source for them.

Four flags model distinctions Chromium itself draws, and they are not interchangeable:

- `isInternalDebugging` — in the _Internal Debugging Page URLs_ section. Harmless; needs
  "Enable internal debugging pages" turned on at `chrome://chrome-urls` first.
- `isDebugCommand` — in the _Command URLs for Debug_ section. Deliberately crashes, hangs, or quits
  the browser. Hidden by preference, and `openUrlInBrowser` confirms before running one.
- `isDeprecated` — advertised by no browser in the census and verified dead by navigation. Kept so
  the answer to "what happened to `chrome://appcache-internals`?" is still in the list.
- `notDirectlyReachable` — advertised but returns a network error when navigated to. Most are panels
  drawn inside the browser's own interface; for some the reason was not identified, which is why the
  UI labels these **Don't Use** rather than trying to name what they are. Tagged that way _except_
  where `requiresFeatureFlag` is also set — that entry is tagged **Flag** instead and is exempt from
  the ⌘⇧H filter, because naming the flag makes it actionable rather than a dead end. With the
  detail pane open the tag collapses to an orange warning glyph, since the pane already explains it
  and the list column is narrow.

`npm run build` and `npm run lint` both run this check, so invalid generated data fails the normal
workflow rather than waiting for someone to remember it. Run `npm run check-paths` directly for the
fast loop while regenerating. It catches what generators produce: duplicate ids (which silently
break starring, since stars key on id), browser keys that match no browser, contradictory flag
combinations, and preferences read in code but absent from the manifest.

## Conventions that are load-bearing

- **One URL builder.** Everything that displays, copies, or opens a URL goes through
  `buildBrowserUrl`. Some paths are already absolute (`chrome-untrusted://compose`) and must not be
  prefixed; two copies of this logic once let the opened URL diverge from the copied one.
- **Launch the discovered bundle path, not the app name.** `openUrlInBrowser` takes
  `{ app, name }`; `app` is the absolute path found by `findInstalledBrowsers` when available, so
  the browser that opens is the one whose icon was shown. A bare name is resolved by Launch
  Services and can pick a different copy.
- **`execFile`, never `exec`.** No shell, so no quoting boundary. And no `open -F` — that means
  _fresh_ and discards the browser's restored windows.
- **The destructive-URL confirmation lives in `openUrlInBrowser`, not at the call sites.** It was at
  a call site once and the "Open in…" submenu did not have it, so a crash command launched from that

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raycast/extensions](https://github.com/raycast/extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
