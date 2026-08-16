---
trigger: always_on
description: Fetches and parses Open Graph / Twitter Card / JSON-LD / oEmbed metadata for link previews.
---

# metadata/ — URL Preview Metadata Pipeline

Fetches and parses Open Graph / Twitter Card / JSON-LD / oEmbed metadata for link previews.
Replaces the former `metadata_fetch` package dependency.

Public entry point is `MetadataHelper` (`../metadata_helper.dart`), re-exported by
`helpers/helpers.dart`. Import the barrel `metadata/metadata.dart`, not individual files.

## Layout

| Directory | Responsibility |
|-----------|----------------|
| `models/` | `UrlMetadata`, `MetadataFetchResult`, `MetadataSource`, `MetadataFetchStatus` |
| `parsing/` | Document parsers + `MetadataDocumentPipeline` that merges them |
| `network/` | HTTP client, SSRF guard, charset decoding, oEmbed, image downloader |
| `sites/` | Per-site knowledge, registered in `SiteParserRegistry` |
| `cache/` | In-memory single-flight cache + `Message.metadata` persistence |
| `util/` | Text cleanup and URL resolution/normalisation |

## Flow

```
MetadataHelper.shouldAutoFetch()            policy + sender gate
MetadataHelper.fetchForMessage()
  └─ UrlMetadataFetcher.fetch()            url_metadata_fetcher.dart
       ├─ MetadataUrls.parse                   scheme defaulting
       ├─ SiteParserRegistry.forUrl → prepare  canonicalise, strip trackers
       ├─ MetadataHttpClient.fetch             own Dio; concurrency-limited
       │    └─ per hop: UrlSafetyGuard → request → follow Location manually
       ├─ HtmlStructureGuard.isSafe            reject pathological nesting
       ├─ MetadataDocumentPipeline.run         OG → Twitter → JSON-LD → HTML
       │                                        → Microdata → BodyImage → Icon
       ├─ OEmbedResolver.resolve               only if a gap remains
       └─ SiteMetadataParser.refine            site-specific DOM knowledge
```

Concurrent calls for the same URL collapse onto one request (`MetadataMemoryCache`),
keyed by normalised URL — **not** by message GUID. `PreviewImageDownloader` single-flights the
same way for image and icon downloads.

Two widgets asking for the same preview at once is the normal case, not an edge case:
`MessagePopup` renders a **second copy** of the bubble against the same `MessageState`, so both
copies' `previewRefreshKey` workers fire on "Refresh Preview" (it is dispatched before the popup
closes). Anything in this pipeline that hits the network must therefore be single-flighted, or
every long-press and every refresh does the work twice.

## Parser Precedence

`UrlMetadata.fillMissingFrom` never overwrites a field that is already set, so precedence is
expressed purely by the order parsers run in. To change precedence, reorder
`MetadataDocumentPipeline._primary`.

`MicrodataParser`, `BodyImageParser` and `IconParser` are conditional — they walk the DOM, so
the pipeline only runs them when the cheap strategies left the relevant gap.

## Adding a Site Parser

1. Create `sites/<site>_site_parser.dart` extending `SiteMetadataParser`.
2. Override only the hooks the site needs:
   - `prepare(url)` — canonicalise **before** the fetch (expand short links, drop site-specific
     tracking params). The user's original URL is preserved separately, so this never changes
     what tapping the preview opens.
   - `refine(base, context)` — fill gaps from the site's own DOM.
   - `fallback(url)` — facts derivable from the URL alone, applied when the fetch fails.
     Never invent a title or description here.
3. Register it in `SiteParserRegistry._parsers` (order is match order).

Currently registered: Apple Maps, YouTube, Amazon, Reddit.

## Site Names

`util/site_display_names.dart` maps a host to the name people call that site —
`chat.whatsapp.com` → WhatsApp, `music.apple.com` → Apple Music. `SiteDisplayNames.names` is a
`const` map maintained in code; add entries there. It is deliberately **not** a user setting.

It is applied at **display time** — `UrlPreviewController.siteText` and `linkPreviewDomain`, the
only two places that render a site line. Nothing is written onto the message row, so a new entry
applies to cards that are already cached, without a refetch. Don't move this into `_finalize` or a
site parser: that would bake one label into persisted metadata, so an entry added later would only
apply to links seen after the change.

Lookup walks up the host one label at a time (`m.youtube.com` → `youtube.com`), stopping at two
labels. Labels are only ever dropped at a dot, which is what keeps `evil-apple.com` and
`apple.com.evil.com` from inheriting `apple.com`'s label. The key is always the **real host**, so
this does not weaken the "never `og:site_name`" rule below — an unmapped lookalike still renders
itself.

## Persistence

Never read or write `message.metadata` keys directly — go through `MessageMetadataStore`.
`MetadataCacheSlot` owns the key names for each preview a message can carry
(`urlPreview`, `photoSlideshow`).

A completed attempt is stamped with a timestamp and retried after
`MessageMetadataStore.retryAfter` (24h). Retryable failures (timeout, 5xx, 429, socket error)
are deliberately **not** stamped, so they retry on the next build — see
`MetadataFetchResult.isRetryable`.

Legacy `metadata_fetch` keys are handled asymmetrically, on purpose:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BlueBubblesApp/bluebubbles-app](https://github.com/BlueBubblesApp/bluebubbles-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
