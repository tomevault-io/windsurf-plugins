---
trigger: always_on
description: Portman is a macOS menu bar app that lists everything listening locally, identifies what it
---

# Agent Notes

Portman is a macOS menu bar app that lists everything listening locally, identifies what it
is, and lets you act on it. Swift 6, SwiftUI hosted in an AppKit panel, SwiftPM only — there
is no Xcode project.

Read this before changing anything. The **Traps** section in particular: each entry is a bug
that was found the hard way, and undoing one reintroduces it.

## Product Direction

The category is crowded with apps that list ports and kill them. That part is a commodity.

Portman's edge is **knowing what a server actually is** — the project, the framework and
version, the git branch, whether it's still answering, what page it's serving. Detection is
the moat. When choosing between two implementations, favour the one that identifies things
more precisely, even if it costs more work.

Second principle: **honesty over convenience**. A reading we can't take renders as `—`, never
`0`. A kill that failed says so rather than animating the row away. A tunnel that isn't
resolvable yet isn't handed over as if it were.

Third: **the panel is a glance, not a dashboard**. Anything not scannable in a second belongs
in the expanded row, not on it. Rows default to Simple density for this reason.

## Repo Map

```
Sources/Portman/
  Core/        scanning, project + framework detection, Docker, metrics, git, ancestry
  Services/    health probing, page previews, tunnels, terminal attach, app launching
  Store/       ServerStore (@Observable), filtering/sorting/grouping, preferences
  UI/          status item, panel, list, detail card, settings, hotkey, theme
  main.swift   CLI entry (--list, --login-status, --enable-login, --disable-login)

Tests/PortmanTests/     pure-logic tests only
Resources/AppIcon.icon  Icon Composer document, compiled by actool at build time
scripts/                build-app.sh, build-dmg.sh, install-local.sh, release.sh, icon/
```

`Core` never imports SwiftUI. The UI never shells out — everything that spawns a process
lives in `Core` or `Services`.

## Architecture

`PortScanner` runs `lsof -nP -iTCP -sTCP:LISTEN -F pcLn` and produces `ServerEntry` values.
Enrichers then fill in optional fields independently — metrics, git, health — so a row
renders before every enricher has finished.

`ServerStore` owns all state and rebuilds derived values (`sections`, `rows`,
`conflictPorts`) only when an input changes, via `inputsChanged()`. They were computed
properties once, and `conflictPorts` was read per row inside the list body, so the whole
folding pipeline ran n times per frame.

`ListShaper` is pure: search, sort, fold, group. Everything the list does is reproducible
from its inputs, which is why it's the part with tests.

## Traps

**The panel must not resize while open.** `NSWindow` origins are bottom-left, so a height
change moves the *top* edge and walks the panel away from the menu bar. `PanelController`
picks a size once in `sizeOnOpen` and leaves it; content scrolls. Never set
`NSHostingView.sizingOptions = .intrinsicContentSize` here.

**Anything that changes on refresh needs a reserved slot.** CPU and uptime are
monospaced-digit in fixed frames, and the row's second line is a single composed `Text`
rather than separate views, so nothing can shove anything else sideways. The sparkline uses
a floored y-axis and fixed x-slots — scaling it to its own window maximum made an idle
server redraw as a full-height cliff every two seconds.

**Only animate the list when it changes shape.** `commit()` compares row identities. A
refresh that merely moved a number must not animate, or the list shimmers while you read it.

**Health is checked once per port, not per scan.** Results live in `healthByPort` across
scans; re-probing every cycle rewrote the list continuously. Probing also runs off the scan
cycle — awaiting it made a scan outlast the poll interval whenever a few servers were wedged.

**No `confirmationDialog` from the panel.** It's a separate window, so presenting it takes
key focus and dismisses the panel underneath. Confirmations are an in-panel footer bar.
Relatedly, `windowDidResignKey` must not close the panel — menus opened from inside it
resign key too.

**Status item images must be templates, and never tinted.** Set `isTemplate = true` on the
image handed to the button, *after* any `withSymbolConfiguration` (which returns a copy with
the flag cleared). Setting `contentTintColor` at all opts the button out of automatic
menu-bar adaptation, so it stops following light/dark.

**The alert dot is the one exception, and it has to pay for the adaptation itself.** A template
image is tinted wholesale to the menu bar's colour, so a badged image can't be one — the dot
would come out black. `PanelController.badged` therefore draws the plug itself, in
`labelColor`, and the image must be built with `NSImage(size:flipped:drawingHandler:)` rather
than composited once: the handler runs inside the button's own appearance, so `labelColor`
resolves to white on a dark menu bar and black on a light one. Compositing eagerly bakes in
whichever appearance was current when that scan finished, and the icon then stays that colour
through a light/dark switch. Verified both ways by switching appearance with the app running.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iannuttall/portman](https://github.com/iannuttall/portman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
