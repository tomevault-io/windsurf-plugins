---
trigger: always_on
description: BeatPrints is a music-poster creation tool, not a generic image generator or analytics
---

# BeatPrints Agent Guidelines

## Product context

BeatPrints is a music-poster creation tool, not a generic image generator or analytics
dashboard. Its primary track-poster journey is:

1. Search for a track and select the exact catalog result.
2. Review the matched cover, title, artists, album, release information, and duration.
3. Optionally select up to four lyric lines.
4. Optionally choose one music platform to add as the poster's QR destination.
5. Choose poster appearance options and generate the final PNG.

Album posters are a related path: select an album, optionally configure track indexing and
shuffle, optionally choose one QR platform, choose appearance options, and generate the PNG.

Use `docs/frontend-product-brief.md` as the source of truth for the frontend journey, API
mapping, current backend capabilities, and known integration gaps. Preserve these distinctions:

- Metadata source (`provider`: Deezer or Spotify) is not the same as the optional poster QR
  destination (`qr_platform`: Spotify, Apple Music, QQ Music, or NetEase Music).
- After a user selects a search result, generation should use the result's unchanged
  `provider + id` as `provider + catalog_id`; do not fall back to a query that silently picks
  the first result.
- No QR platform means no platform mark or QR code on the poster.
- The generated poster PNG is the primary outcome. Cover artwork and music metadata should
  drive the interface's visual identity; avoid generic AI-product and dashboard conventions.
- BeatPrints is licensed CC BY-NC-SA 4.0 for non-commercial use and requires attribution.

### Pluggable integration architecture

All integrations with external music capabilities are plug-ins, not branches in a central
provider, region, or product-name switch. This requirement applies to QR destinations, catalog
search sources, lyrics sources, artwork/code renderers, and any future third-party music service.

1. Give each integration its own module and a small explicit contract. An integration may own
   its transport, normalization, public-link parsing, source-specific capabilities, and visual
   code/mark behavior; it must not share a region-based catch-all module with unrelated platforms.
2. Keep all enabled integrations in one registry whose imports are the complete enablement list.
   Temporarily disabling an integration must require commenting/removing its one registry import,
   not editing central conditionals, route enums, request fields, or renderer branches. Avoid
   indirect imports that would re-register a disabled plug-in.
3. Core journeys consume the contract and registry lookup only. They must not branch on a
   platform's name, market, or geography. Platform-specific exceptions belong inside that
   platform's plug-in; generic fallback behavior belongs in shared infrastructure.
4. Separate source roles from destination roles. For example, disabling a Spotify QR destination
   must not disable Spotify catalog metadata. A source item's identity and user-selected output
   destination remain separate throughout the flow.
5. Keep public request payloads and route dispatch extensible: use destination-keyed maps and
   registry validation rather than fixed per-platform object fields or hard-coded route literals.
   UI availability must come from the same enabled-integration configuration or a backend-exposed
   registry, so a disabled integration is not offered to users.
6. Preserve a consistent user contract across enabled integrations: explicit source selection,
   conservative matching, ranked/manual fallback, current-metadata resolution, disabled states,
   and equivalent track/album behavior where applicable. Do not silently choose a weak result.
7. Add regression coverage for the registry, every enabled integration, and the disabled/unknown
   path whenever this architecture changes. Test an integration's normalization independently
   with recorded fixtures when practical.

Apply these rules to future search-provider optimizations and multi-source lyrics work: source
selection, priority/fallback, normalization, provenance, and failure handling belong behind
independent source adapters and a shared orchestration contract, never in provider-name conditionals.

### Cross-platform link matching

When implementing or changing a QR platform destination, preserve the established matching
journey for both **tracks and albums**:

1. Start from the user's exact selected `provider + id` and pass it unchanged as
   `provider + catalog_id` plus `type=track|album`. Do not replace this with a text query that
   silently selects a first result.
2. Match conservatively. Prefer stable identifiers such as ISRC; only use title, artist, release,
   and duration fallbacks when they meet a strict confidence threshold. Return an explicit no-match
   result rather than linking a plausibly named but unconfirmed release.
3. Display a successful match in the same `Item`-style hierarchy as a source search result: cover,
   title, artists, and contextual metadata. Tracks show their album; albums show release year and
   track count. The right-side confirmation action is labelled only “Open”.
4. When an automatic result is absent or the user rejects it, offer ranked candidates for every

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sdrpsps/beatprints-web](https://github.com/sdrpsps/beatprints-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
