---
trigger: always_on
description: Usage and architecture are in README.md. This file holds what the code does not say.
---

# AGENTS.md

Usage and architecture are in README.md. This file holds what the code does not say.

- **`web/vendor/JIZURA` is upstream JIZURA as a submodule — never edit it.** The page loads its `src/*.js` directly as classic scripts (global `J`); `tools/update-jizura-scripts.sh` writes the `<script>` list into `web/index.html` between the `jizura:begin` / `jizura:end` markers, in byte order (`LC_ALL=C`, matching upstream's `sorted(glob)`), minus `12_ui.js`. Change what JIZURA does through the project / plan values `app.js` passes in, not by patching it.
- **JIZURA's renderer is a pure function of time.** Sync is `frame(ctx, plan, positionMs + offsetMs)` every animation frame; there is no sync state to keep.
- **JIZURA settings that matter here:** `unify` defaults off upstream and its random look (`J.omakase`) never sets it, which makes every cut an independent draw; it is on by default here (`U`). `fx.koma` (frame stepping) is chosen per mood by `omakase`; `K` overrides it.
- **Section breaks feed `unify`.** JIZURA ends a part at an empty lyric row or a `[間奏]` line; `lyrics.js` `partBreaks()` inserts empty rows from LRC blanks, long pauses and repeated blocks, then merges parts under 3 lines and splits parts over 8. It was tuned on LRCLIB data for four songs — re-check part sizes (`lines … in … parts` in the console) when changing it.
- **Spotify:** PKCE with a full-page redirect; storage keys `jizura-sync.clientId` / `jizura-sync.tokens` (localStorage) and `jizura-sync.pkce` (sessionStorage). The redirect URI is `origin + pathname`, which is why `index.html` moves `localhost` to `127.0.0.1` before anything loads. A development-mode app answers `403 … not registered` for accounts missing from its User Management list — terminal, polling stops.
- **LRCLIB** asks clients to identify themselves (`Lrclib-Client` header, allowed by its CORS preflight) and to honour `Retry-After` on 429. It also has short outages: `/get` and `/search` answered 503 several times on 2026-09-29, so 5xx and network failures are retried after 2 s and 5 s. A hit must be within `DURATION_TOLERANCE_S` (4 s) of the player's length; re-releases such as "Blue Flame (2023 Ver.)" often have synced lyrics only for a version 5–12 s longer or shorter, and show "No lyrics found". That is a matching limit, not a bug.
- **Why this is not a public service** (README "Disclaimer"): Spotify Developer Policy III forbids synchronizing sound recordings with visual media, and Apple's MusicKit terms forbid synchronizing MusicKit Content with other content — so Apple Music is not a way out. LRCLIB lyrics are unlicensed community data. Keep the web page bring-your-own-Client-ID; do not commit a Client ID to the repo or host a deployment for others. The Mac app uses neither API and its notarized builds are published on GitHub Releases (the user's decision, 2026-09-30); the unlicensed-lyrics caveat still applies to it. This repo must also never contain LSE-Core / COTODAMA code or names: the Spotify code here was written from scratch for that reason.
- **Lyrics offset sign:** the renderer draws `positionMs + offsetMs`, so a positive offset shows lyrics *earlier*. `]` is +50 ms (earlier), `[` is −50 ms (later).
- **Play/pause follows `SpotifyPlayer.isPlaying()`, not the track state.** The poll asks for `additional_types=track`, so a podcast episode arrives with no track item and `getState()` returns null or a stale track; `is_playing` is recorded for every response. With a null state the old `paused` check sent pause for a Play icon.
- **Seek is optimistic.** `seek()` moves the local clock at once and sets `_seekAt`; a poll whose request started before that carries the old position and is ignored, otherwise the slider jumps back for up to a second.
- **Now-playing bar:** hidden state is `jizura-sync.barHidden` (localStorage) and the bar is `inert` while hidden, so its controls leave the tab order. The status line lives inside the bar, so errors go through `showError()` → a toast outside it, once per distinct message while there is no playback state — "Nothing is playing" repeats every 3 s. Why a song has no lyrics goes to `#notice` in the middle of the window instead, and gets no toast (the two said the same thing); the notice is centred with `inset: 0; margin: auto`, because `translate(-50%, -50%)` lands on half pixels and blurred its text in WKWebView. Popping the whole bar up on every status was tried and reverted: with no device it stayed up permanently, and a click during the pop-up un-hid it for good. A click toggles the bar after 250 ms so a double click (fullscreen) does not flash it. The global keydown guard skips text inputs only; a focused range input (the seek slider) must not swallow shortcuts.

- **Tesla / deploy:** `tools/deploy-cloudflare.sh` writes the Client ID only into a staged copy of `index.html` (seeds `jizura-sync.clientId` when empty) and uploads just `vendor/JIZURA/src`; the repo still never holds a Client ID. Tap buttons in the help are created by `app.js` only when the user agent contains `Tesla/` (or `?tesla`); elsewhere the help HTML is untouched — the user asked that non-Tesla browsers stay exactly as before. Workers assets are served `max-age=0, must-revalidate`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Saqoosha/jizura-sync](https://github.com/Saqoosha/jizura-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
