---
trigger: always_on
description: A native macOS e-book library manager. SwiftUI app, single-user, local-first.
---

# CLAUDE.md

## What this is

A native macOS e-book library manager. SwiftUI app, single-user, local-first.
The library folder lives wherever the user puts it (possibly iCloud Drive);
the app's internal state never does.

The pitch: a minimal and well-designed macOS app that handles language variants 
natively and treats device delivery as a first-class workflow. Supported
devices: Kindle and Kobo, over USB.

Source plugins (JS files in `~/Library/Application Support/com.pdrbrnd.tomo/plugins/`) are a separate concept — they search external catalogues; they don't extend the app itself.

## Owner / context

Solo project. Strong
preference for simple, readable, maintainable code over clever code. KISS,
YAGNI, flat structure, functional composition where it fits.

## Code is the source of truth

Docs (this file, `docs/`, the skill) describe intent and constraints. Once
something is implemented, the **code** is the source of truth for shape
and behaviour. Read the relevant Swift files before relying on a doc's
description of how something works.

When working on this project:

- If a doc describes a struct or protocol that exists in code, trust the
  code and update the doc if it has drifted.
- If a doc describes a design that has now been built, replace the design
  description with a one-line pointer to where the code lives.
- If you finish implementing something that was previously described in
  detail in a doc (data models, classifier internals, source protocols,
  etc.), prune the detailed description and leave a stub.
- Stale design docs are worse than no design docs. They mislead.

The architectural principles, project scope, and watchouts stay in docs —
those describe *intent*, which the code can't express. Everything else
should migrate to code over time.

## Architectural principles

These are load-bearing. Don't violate them without flagging it.

1. **Library on disk is the source of truth.** Flat folder structure:
   `Library/Author/Title (Year)/<author-title-year>.epub`. The folder
   carries human-readable identity; the file is a kebab-case ASCII slug
   built by `bookFileSlug` so every book has a unique, identity-bearing
   filename — devices like Kindle dedupe by filename, so the legacy
   `book.epub` shape collided on every send. Survives the app being
   deleted. Each book carries a `metadata.json` sidecar with everything
   the index needs (title, authors, locale, collections by name, id,
   etc.) including the on-disk filename. Collection *definitions* (id,
   sortOrder, dateCreated) live in `<library>/.tomo/collections.json`
   so empty collections and sortOrder also survive a rebuild.

2. **The SQLite index is disposable.** Stored outside the library folder
   at `~/Library/Application Support/[app-bundle-id]/index.db`. Never in
   iCloud (SQLite + cloud sync = corruption). Rebuilds entirely from the
   sidecars + `.tomo/collections.json` on demand.

3. **iCloud Drive is supported, not promoted.** The library folder may live
   in iCloud. Use `NSFileCoordinator` for reads/writes. Detect `.icloud`
   placeholder files and handle eviction gracefully. Never assume eager
   access to file contents.

4. **Manual override always wins.** Anywhere we classify, detect, or guess
   (language, duplicates, metadata), the user's explicit choice is final
   and persisted.

5. **No network calls without user action.** Metadata fetches, cover lookups,
   etc. are explicit clicks, never background.

6. **No external binary dependencies.** Everything ships in the app
   bundle as Swift code or pure-Swift packages. No shelling out to
   `ebook-convert`, `pandoc`, or anything else.

## Tech stack

- Swift 6.0 / SwiftUI, macOS 26+ target (`MainActor` default isolation)
- GRDB for SQLite
- ZIPFoundation for EPUB reading (EPUB is just zip + XML)
- SwiftSoup for HTML parsing (used by source plugins' `querySelectorAll` host binding)
- [`AZW3`](https://github.com/pdrbrnd/swift-azw3) — our own SwiftPM package, the EPUB→AZW3 writer
- JavaScriptCore for source plugins (none bundled — installed from registries)
- Native `FileManager` + `NSFileCoordinator` for file ops
- No external binaries. No Python. No bundled apps.

Avoid: heavy frameworks, async libraries beyond Swift Concurrency, anything
that isn't pulling its weight.

Fast and snappy is not negotiable:
- Background everything that touches disk
- Don't trigger iCloud downloads accidentally
- Index SQLite columns you'll search on
- Use LazyVStack/List with stable IDs

## Data model

Models live in `Tomo/Models/`. Read those files for the current shapes
(`Book.swift`, `Collection.swift`, `LanguageProfile.swift`). The sidecar
`metadata.json` carries everything in `Book` *including* `id`, plus the
`collections` array (by name) and a top-level `version` field for future
migrations. See `Tomo/Core/Metadata/MetadataSidecar.swift`.

Disk truth split:

- Per-book sidecar (`metadata.json`): all `Book` fields + collection memberships by name.
- Per-library `<root>/.tomo/collections.json`: collection definitions (id, name, sortOrder, dateCreated). Pairs with sidecar memberships to fully reconstruct the index.

Intent notes the code can't express:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pdrbrnd/tomo](https://github.com/pdrbrnd/tomo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
