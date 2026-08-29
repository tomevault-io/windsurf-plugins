---
trigger: always_on
description: A scheduled task diffs this fork's `NuvioMobile` submodule (`upstream` remote = `github.com/NuvioMedia/NuvioMobile`, branch `cmp-rewrite`) against upstream daily and logs actionable ports to `docs/upstream-port-plan-YYYY-MM-DD.md`. Latest run: **2026-08-28**, see `docs/upstream-port-plan-2026-08-28.md`. Upstream has **not moved** since 2026-08-27 — still pinned at `1b84ee47`, zero new commits. This run re-verified (by reading current `shared/` file contents, not trusting prior notes) that all 6 
---

# Nuvio tvOS — Claude Code project notes

## Upstream sync status (auto-checked daily against NuvioMedia/NuvioMobile)

A scheduled task diffs this fork's `NuvioMobile` submodule (`upstream` remote = `github.com/NuvioMedia/NuvioMobile`, branch `cmp-rewrite`) against upstream daily and logs actionable ports to `docs/upstream-port-plan-YYYY-MM-DD.md`. Latest run: **2026-08-28**, see `docs/upstream-port-plan-2026-08-28.md`. Upstream has **not moved** since 2026-08-27 — still pinned at `1b84ee47`, zero new commits. This run re-verified (by reading current `shared/` file contents, not trusting prior notes) that all 6 items from the 2026-08-27 batch are **still unported**, and that the carried 2026-08-25 HIGH subtitle/player-engine batch (`StartupMode` still present in `shared/`) is also still unstarted. The A2 crash-cluster fixes (reported ported 2026-08-26) were spot-checked and confirmed still present, no regression. Same 6-item batch as 08-27: (1) **[HIGH, possible live break]** `InAppYouTubeExtractor` preferred client swap `android_vr`→`visionos` (upstream `c72a3825`) — YouTube may have blocked/deprecated `android_vr`, meaning in-app trailer playback could be degraded right now; bundled with `isDefaultAudioTrack` dub-track skip logic; (2) **[MEDIUM]** trailer dedup + language-preference tiebreaker (`HeroTrailerSelector` `distinctBy`, `MetaTrailer.iso6391`, TMDB trailer-list sort) — same upstream commit as #1, port together; (3) **[MEDIUM]** TMDB CJK/Romaji cast-crew name fallback, 3-commit chain (`c66e80e6`→`e6314ba2`→`754605cb`) — person bio+name, filmography, network-browse pages, port as one batch; (4) **[MEDIUM]** `tv/{id}/aggregate_credits` instead of `tv/{id}/credits` for full-series cast/crew (`0aac4dbe`); (5) **[LOW]** Simkl `stringValue()` JsonNull guard (`e39084d6`), mechanical; (6) **[MEDIUM]** scrobble guard for short placeholder/error-clip durations (`31f5d0e6`) — shared `isProgressComplete()` half is a clean port and `WatchProgressRules.kt` picks it up for free, but tvOS's native player completion call site (near `MPVPlayerView.swift`) needs its own guard — upstream's fix lives in Compose-only `MainAppContent.kt` with no tvOS equivalent, so this needs investigation, not just a mechanical port. All confirmed unported by reading current `shared/` file contents on `claude/beta15`, not commit messages. Full detail, batching suggestion, and verification checklist in `docs/upstream-port-plan-2026-08-27.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youngchris29-art/NuvioTV](https://github.com/youngchris29-art/NuvioTV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
