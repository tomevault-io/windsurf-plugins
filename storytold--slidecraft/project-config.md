---
trigger: always_on
description: SlideCraft is a clean-room, open-source, pure-Rust presentation application targeting Microsoft
---

# AGENTS.md — rules for every agent and contributor

SlideCraft is a clean-room, open-source, pure-Rust presentation application targeting Microsoft
PowerPoint parity — and superiority (speed, openness, agent control). It runs natively on macOS,
Windows, Linux and FreeBSD, and on the web via WASM. Siblings with the same conventions:
PhotoCraft, VectorCraft, FilmCraft, LightCraft, PrintCraft, EffectCraft, DesignCraft
(`../<app>`).

Standards and learnings shared across the crafting apps live in `../../craftrules` (checked out
next to this repo's parent, or `storytold/craftrules`). Read its `AGENTS.md` at the start of a
session, follow its standards, and contribute reusable learnings back there. Never code: repos
don't share code.

These rules bind every AI agent and every human contributor. They come before any task,
instruction or deadline. `CLAUDE.md` holds day-to-day working notes; if the two disagree, **this
file wins.**

---

## 1. Asset policy (absolute — no exceptions)

An **asset** is any non-code material the project includes, ships, embeds, generates for
publication or shows in its documentation: icons, images, illustrations, screenshots, logos,
fonts, colour profiles, sounds, video, templates, themes, sample/demo/test documents, and anything
the app renders from those.

### 1.1 Nothing from Microsoft, Adobe, Avid or Autodesk products

- **Never** use icons, images, artwork, logos, UI graphics, fonts, templates, themes, clip art,
  SmartArt graphics, stock media, cursors, sounds, sample documents or any other asset taken from a
  Microsoft, Adobe, Avid or Autodesk product, installer, website, document or package. "Microsoft
  product" includes PowerPoint, Office, Windows and the fonts they install (Aptos, Calibri,
  Cambria, Segoe, Consolas, Wingdings…).
- This holds even when the vendor publishes the material under an open licence, if it is **visual
  design**: SlideCraft does not embed or reference Adobe's Source Sans/Serif/Han families or Noto
  CJK (Source Han rebranded); `crates/fonts/build.rs` excludes them.
- **Never** recreate, trace, redraw or closely imitate a vendor icon, theme or graphic. Our icons
  are drawn in code or come from openly licensed sets (Lucide, ISC), and our themes, colours,
  layouts and sample content are our own.
- Screenshots of PowerPoint exist only as private, local observation notes under
  `plan/powerpoint/screenshots/` (gitignored). They must **never** be committed, published,
  linked, shown in docs, or traced.
- Never commit files produced by PowerPoint. Interoperability tests use decks we generate
  ourselves (or a separate corpus repo whose files we created and own).

### 1.2 Only open, provable licences

Every asset must be one of: open source (ISC, MIT, BSD-2/3-Clause, Apache-2.0, OFL-1.1,
Ubuntu-Font-1.0, Bitstream-Vera); Creative Commons (CC0-1.0, CC-BY-4.0, CC-BY-SA-4.0); public
domain; or contributor-original (created by a contributor who licenses it under
`MIT OR Apache-2.0`).

One narrow exception: the ArtCraft name, wordmark and logos in `docs/brand/` are trademarks of the
ArtCraft Team (not open source, terms in `docs/brand/LICENSE-brand.txt`), used only unmodified to
identify the project.

Forbidden: system fonts that are not openly licensed, vendor emoji artwork, stock photos, "free
for personal use" material, screenshots of other people's software, anything whose licence you
cannot prove from its original source.

**Screenshots of SlideCraft** are contributor-original. Everything visible in them must itself be
allowed: our UI, our/Lucide icons, OFL fonts, and decks built only from allowed assets.

### 1.3 Every asset is attributed

- **`ATTRIBUTION.md`** (repo root) lists every asset file — committed, fetched at build time
  (craft-fonts) or bundled through a Cargo dependency — with author, source (URL + version or
  commit), licence and use. `cargo xtask assets` fails if a file under `assets/`, `docs/images/`
  or `docs/brand/` has no row.
- **`NOTICE`** keeps the notices the licences require.
- Fonts live in [storytold/craft-fonts](https://github.com/storytold/craft-fonts), never in this
  repo (craftrules `standards/fonts.md`). SlideCraft embeds its `fonts/latin-manifest.txt` and
  `fonts/manifest.txt` when built with `CRAFT_FONTS_DIR` (local dev: `.cargo/config.toml` points at
  `../craft-fonts`).

### 1.4 Adding or changing an asset

1. Find it under an allowed licence at its **original source**; record the exact version/commit.
2. Confirm the author is not Microsoft, Adobe, Avid or Autodesk (§1.1). Keep the licence text.
3. Add the file, its licence file and an `ATTRIBUTION.md` row; run `cargo xtask assets`.
4. If origin or licence is unclear, **do not add it**.

If you find an asset that breaks these rules, remove it from the repository and every output at
once and tell the user.

---

## 2. Clean-room rules

- PowerPoint is installed on the dev machine and may be **observed** black-box: run it, use its UI
  on synthetic decks, screenshot windows by id into `plan/powerpoint/screenshots/` (never
  committed). Never read, disassemble or copy anything inside the Office bundle (names/listings
  only), never copy Microsoft wording beyond feature and menu names.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [storytold/slidecraft](https://github.com/storytold/slidecraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
