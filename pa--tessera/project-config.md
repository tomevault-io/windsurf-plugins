---
trigger: always_on
description: A native macOS menu-bar app that brings terminal-multiplexer workflows
---

# Tessera

A native macOS menu-bar app that brings terminal-multiplexer workflows
(tmux/Zellij: tabs, splits, panes, a command palette) to arbitrary GUI apps by
puppeting their windows through the Accessibility (`AXUIElement`) APIs.

## Build & run

```sh
swift build -c release            # compile only
./scripts/build-app.sh            # compile + assemble a signed .app bundle
open .build/Tessera.app           # launch the menu-bar agent
.build/Tessera.app/Contents/MacOS/Tessera   # launch attached to a terminal (see logs)
```

Tessera is a menu-bar agent (`LSUIElement`), so there is no Dock icon or window
— look for the `▚` glyph in the menu bar after launch.

## Distribution: Homebrew, no Apple Developer ID

Tessera ships as a **single Swift binary via a Homebrew tap** — no `.app`
wrapper, no notarization, no $99/yr Developer ID, no Gatekeeper quarantine (a
source build isn't downloaded, it's compiled locally, so nothing is quarantined
— the same trick paneru uses).

```sh
brew tap pa/tessera https://github.com/pa/tessera
brew install tessera
brew services start tessera        # run now + at login
```

Two pieces make the bare binary a first-class agent that keeps its Accessibility
grant across upgrades:

1. **Embedded Info.plist.** `Package.swift` passes `-sectcreate __TEXT
   __info_plist Resources/Info.plist` linker flags, so `swift build` bakes the
   plist (LSUIElement + `CFBundleIdentifier=pramodh.ayyappan.tessera`) into the
   Mach-O's `__TEXT,__info_plist` section. `codesign` binds it on signing, so
   the bare executable has a real bundle identity with no `.app` around it.
2. **Self-sign on launch (`SelfSign.swift`).** A Homebrew source build (and each
   `brew upgrade`) is **ad-hoc** signed, whose DR is the exact code hash — so TCC
   would drop the grant on every update. On launch, if the binary isn't already
   signed with the per-user **"Tessera Code Signing"** cert, Tessera creates that
   cert once (dedicated keychain, non-interactive — same logic as
   `scripts/create-signing-cert.sh`, embedded as a string), `codesign --force
   --sign`s its own on-disk binary, and re-execs once (guarded by
   `TESSERA_SELFSIGN_DONE` against loops). Every version then shares one stable
   DR — `identifier "pramodh.ayyappan.tessera" and certificate leaf = H"<user cert>"`
   — and because TCC matches on the DR (not the path), the Accessibility grant
   **persists across upgrades**. The cert is created per-machine, so the leaf
   hash differs per user; each user grants Accessibility exactly once.
   - Gotcha: detect the cert with `security find-identity -p codesigning` **without
     `-v`**. A self-signed codesigning cert always reports
     `CSSMERR_TP_NOT_TRUSTED`, so the valid-only (`-v`) listing hides it even
     though `codesign` signs with it fine — using `-v` would make SelfSign think
     the cert is missing and recreate it, minting a new leaf and losing the grant.

`Formula/tessera.rb` builds from source (`depends_on xcode: :build`) and defines
a `brew services` launch agent. The dev flow below (`.app` bundle) is still used
for local iteration and log attachment.

## Why the .app bundle matters (do not `swift run` for real testing)

macOS keys the Accessibility (TCC) grant to an app's **code-signing identity +
bundle path**. A bare SPM executable has no stable bundle identity, so its
Accessibility access is attributed to the *parent terminal* instead — the app
can never reliably control windows. Always test through
`./scripts/build-app.sh` + the `.app`, never `swift run`.

- Bundle id: `pramodh.ayyappan.tessera` (fixed, in `Resources/Info.plist`).
- Default signing is **ad-hoc** (`-`). Ad-hoc signatures change every rebuild,
  so macOS re-prompts for Accessibility after each build. To make the grant
  stick, run `./scripts/create-signing-cert.sh` once — it creates a self-signed
  "Tessera Code Signing" identity in a dedicated keychain (non-interactive) —
  then build with `CODESIGN_IDENTITY="Tessera Code Signing" ./scripts/build-app.sh`.
  The identity gives a stable Designated Requirement (`identifier
  pramodh.ayyappan.tessera and certificate leaf = H"…"`) that TCC keys the grant to.
  Switching signing identity (ad-hoc → cert, or regenerating the cert) requires
  re-granting Accessibility once.

## Accessibility permission

The app needs System Settings → Privacy & Security → Accessibility. On first
launch it triggers the system prompt; the menu also has a one-click "grant"
row that deep-links to the pane. `AXIsProcessTrusted()` reports the live state.

## Coordinate space

AX position/size and `CGDisplayBounds` share a **top-left-origin, y-down**
global coordinate space. Tessera works entirely in that space — do NOT use
AppKit's `NSScreen.frame` (bottom-left origin) for placement math without
flipping. `ScreenLayout` centralizes this.

## Layout / structure

```
Sources/
  TesseraCore/                      Pure, UI-independent logic (CoreGraphics only)
    ScreenLayout.swift              Screen-relative named placements (prototype)
    BSPLayout.swift                 BSP tree + LayoutTree: split/remove/resize → frames
    FuzzyMatcher.swift              Subsequence fuzzy match + ranking (palette filter)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pa/tessera](https://github.com/pa/tessera) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
