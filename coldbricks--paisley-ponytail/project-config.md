---
trigger: always_on
description: > 105.9 TB of deleted Webshots photos live inside 2,437 megawarc blobs
---

# CLAUDE.md — Paisley Ponytail (the Webshots Resurrector)

> 105.9 TB of deleted Webshots photos live inside 2,437 megawarc blobs
> on archive.org. The raw blobs are 401-walled and the old index is dark,
> but everything was ingested into the Wayback Machine. This tool goes in
> through the Wayback door.

---

## Prime Directives

1. **This is a data recovery project.** Given a username, find and download their Webshots photos. A false negative (missing recoverable photos) is worse than being slow.
2. **The Wayback Machine is the ONLY door.** Verified 2026-07-08: every freeze-frame collection item has `access-restricted-item: true` (raw downloads 401 for everyone), and the `webshots-freeze-frame-index` item is dark (`is_dark: true`). Do not build against raw megawarc/CDX-file access — it is gone.
3. **Never guess image URLs.** Audit-proven 2026-07-08: the image server number is NOT derivable from a thumbnail URL (thumb13 photos live on image04, image12, image20…). The photo detail page's `<img src>` is the only source of truth. Guessing is fallback-only.
4. **Be a guest at archive.org.** Global rate limiter (~1 req/s sustained), shared cooldown on 429/503, contact URL in the User-Agent. If a change could hammer their servers by default, it is wrong.
5. **Always leave the user with something.** Full-size → 800×600 → archived thumbnail. Resume must never lose work; interrupted runs must save manifests.

---

## Project Identity

| Field | Value |
|---|---|
| **Product name** | Paisley Ponytail (subtitle: the Webshots Resurrector) |
| **Repo** | github.com/coldbricks/paisley-ponytail (PUBLIC; renamed from webshots-resurrector, old URL redirects) |
| **Stack** | Python 3.10+ (developed on 3.14/Windows 11), httpx + rich; requirements.txt |
| **Brand** | Tailstrike Studios × Ash Airfoil // coldbricks; WWII nose-art mascot (assets/nose_art.jpg); tower-cab terminal UI (Zulu clock, flight strips, LANDED/MISSED APCH callouts) |
| **Audience** | People recovering lost Webshots accounts, mostly non-technical, mostly on Windows — first-run UX matters |

## File Map (real, v2.0.0)

| Path | Purpose |
|---|---|
| `resurrector.py` | CLI: recon/scan/deep, search + pull (stats/on_photo/on_phase hooks), SAY INTENTIONS, relief wire |
| `lib/engine.py` | Async engine: rate-limited transport, CDX API, extraction, download chain, fail reasons, per-photo `ts`, `cooldown_remaining` |
| `lib/truth.py` | Truth-state copy matrix — authored controller line + plain English + one action per kind of miss. Used by the scope GUI; the CLI still has its own v1.6.x copy of the same states (keep wording aligned; wiring CLI through this matrix is pending) |
| `lib/ui.py` | rich tower-cab UI (ATIS, relief, datablocks, HF palette); UTF-8 on Windows |
| `lib/gallery.py` | Scope-grade offline gallery.html (strip bay, CAT grade, per-photo provenance, empty-strip honesty, print CSS) |
| `lib/grade.py` | Wreckage CAT grade (pure) |
| `lib/relief.py` | Hangar SIA scan for position relief |
| `lib/scope_gui.py` | Scope glass. **TRAINING mode (default): simple wizard.** **PROFESSIONAL mode: the entire cascade** (CEDAR gate, relief brief, VSCS/ZWY/channels SIM panels, live ADS-B). Mission instruments wired to real engine events in both |
| `lib/nas_brief.py` | Live METAR + NAS/OIS for the Professional relief brief |
| `lib/adsb_feed.py` | Live ADS-B (adsb.lol / OpenSky) — Professional, toggleable |
| `lib/prefs.py` | `.ppty_prefs.json` — mode choice; git-ignored |
| `lib/remarks.py` | Local RMK/ store |
| `tests/test_extract.py` | Offline era-grammar fixtures — **no live archive.org in tests** |
| `ARCHITECTURE.md` | One-pager: engine sovereign, cab optional |
| `assets/door_chime.wav` | Sector-open transit door chime (MTA/LIRR-style ding-dong) |
| `assets/artcc_ringer.wav` | Same chime (legacy path; rewritten to door tone) |
| `lib/theme.py` | Scope palette, type scale `F()`, layout metrics, chime synth |
| `lib/__init__.py` | `__version__` — single source of truth |
| `LICENSE` | MIT |
| `Start_Here.bat` | Double-click → scope glass (Training mode first run) |
| `assets/nose_art.jpg` | The pony |

## Architecture — the recovery pipeline

```
username
  → CDX exact query (recon; distinguishes [] "no captures" from None "archive.org down")
  → latest profile + /user/NAME/N pagination pages (scan)
  → [--deep] CDX prefix query → every profile-page variant 2002–2013,
     sampled first/last/middles, capped at deep_probe_cap probes
  → per album: crawl pagination (?start=N crawl-era, /album/ID/N old-era),
     extract (thumb, photo-page) pairs + album title from plain <h1>;
     zero-thumb albums retry at their own CDX capture timestamps
  → per photo: fetch photo detail page → real imageNN URL + caption
     → try real _fs → real _ph → derived guess (legacy heuristic) → thumbnail
  → manifest.json (per-album + per-photo records) + gallery.html
```

Resume semantics: `_fs` on disk = final. `_ph` on disk = upgrade attempt next run, unless `<pid>.fs404` marker says full-size is definitively not archived (404), which stops future retries. Transient failures (5xx/network) never demote or mark — they stay failed and retryable.

## Domain Knowledge (verified)

### Webshots URL truths
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coldbricks/paisley-ponytail](https://github.com/coldbricks/paisley-ponytail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
