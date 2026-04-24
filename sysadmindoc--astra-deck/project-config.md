---
trigger: always_on
description: Chrome + Firefox MV3 extension and Tampermonkey userscript for comprehensive YouTube enhancement. Theater mode, ChapterForge AI chapters, DeArrow, filler skip, transcript extraction, video/channel hiding.
---

# YouTube-Kit

## Overview
Chrome + Firefox MV3 extension and Tampermonkey userscript for comprehensive YouTube enhancement. Theater mode, ChapterForge AI chapters, DeArrow, filler skip, transcript extraction, video/channel hiding.

## Architecture
- ISOLATED world content script (`ytkit.js`) at `document_idle` — all DOM manipulation and feature logic
- MAIN world bridge script (`ytkit-main.js`) at `document_start` — handles `canPlayType` patching for codec/format filtering via data attribute bridge (`data-ytkit-codec`)
- Early CSS injection (`early.css`) at `document_start` for anti-FOUC (hide end cards, info cards, autoplay toggle, etc.)
- GM_* compatibility shim for userscript mode
- YTYT-Downloader consolidated into this repo (separate repo was deleted)
- Settings panel cleanup registry (`_panelCleanups`) prevents memory leaks from intervals/observers
- Options page (`options.html`/`options.js`) uses `chrome.storage.local` directly — no dependency on gm-compat or ytkit.js

## Build System
- `node build-extension.js [--bump patch|minor|major]` — produces all artifacts in `build/`
- Outputs: Chrome ZIP + CRX3, Firefox ZIP + XPI, userscript copy
- CRX3 signing via `crx3` npm package with persistent `ytkit.pem` key (gitignored)
- Firefox manifest auto-patched: `background.scripts` array instead of `service_worker`, `browser_specific_settings.gecko` added
- XPI is a renamed ZIP — Firefox's native extension format
- `--bump` updates version in `manifest.json`, `ytkit.js` (YTKIT_VERSION), and `ytkit.user.js` header

## Firefox Differences
- Manifest uses `background.scripts` array instead of `service_worker` (broader Firefox MV3 compat)
- `browser_specific_settings.gecko.id` set to `ytkit@sysadmindoc.github.io`
- `strict_min_version: "128.0"` — requires Firefox 128+
- `chrome.*` APIs work in Firefox as a compat alias — no code changes needed

## v3.2.0 Features (9 waves)

### Wave 1 — Quick Wins + Medium
**All off by default:** autoDismissStillWatching, remainingTimeDisplay, showPlaylistDuration, showTimeInTabTitle, reversePlaylist, rssFeedLink, preciseViewCounts, videoScreenshot, compactUnfixedHeader, returnYoutubeDislike, perChannelSpeed, hideWatchedVideos (select: dim/hide), antiTranslate, pauseOtherTabs
**Theme:** customProgressBarColor (color picker, default #ff0000)

### Wave 2 — Complex & Differentiating
**All off by default:** abLoop, fineSpeedControl, showChannelVideoCount, redirectHomeToSubs, notInterestedButton, timestampBookmarks, blueLightFilter (intensity range 10-80), disableInfiniteScroll, popOutPlayer (Document PiP API + fallback)

### Wave 3 — Player Polish
**All off by default:** watchTimeTracker (90-day retention), alwaysShowProgressBar, sortCommentsNewest, autoSkipChapters (textarea patterns), chapterNavButtons, videoLoopButton, persistentSpeed (select), codecSelector (H.264/VP9/AV1), ageRestrictionBypass, autoLikeSubscribed, thumbnailPreviewSize

### Wave 4 — Polish & Deep Enhancement
**All off by default:** cinemaAmbientGlow (canvas color sampling), transcriptViewer (sidebar with clickable timestamps), searchFilterDefaults (select: upload_date/view_count/rating), forceStandardFps (block 60fps), stickyChat, autoExpandDescription, scrollToPlayer, hideEndCards (CSS), hideInfoCards (CSS), keyMoments (chapter highlight CSS)

### Wave 5 — Power User & QoL
**All off by default:** autoTheaterMode, resumePlayback (StorageManager, 500 entry cap, 15s save interval), miniPlayerBar (floating bar with progress/play/pause on scroll-past), playbackStatsOverlay (codec/resolution/dropped frames/bandwidth), hideNotificationBadge (CSS), autoPauseOnSwitch (visibilitychange API), creatorCommentHighlight (CSS border+badge), copyVideoTitle (button next to title), channelAgeDisplay (calculated age badge), speedIndicatorOverlay (monospace overlay at 1x+ speeds), hideAutoplayToggle (CSS), fullscreenOnDoubleClick (dblclick capture handler)

### Wave 6 — Interaction & Media Control
**All off by default:** rememberVolume (persist volume level), pipButton (one-click PiP in controls), autoSubtitles (auto-enable CC), focusedMode (hide everything except video+comments), thumbnailQualityUpgrade (maxresdefault replacement with fallback), watchLaterQuickAdd (clock overlay on thumbnails), playlistEnhancer (shuffle + copy all URLs), commentSearch (filter bar above comments), videoZoom (Ctrl+scroll to zoom, drag to pan up to 5x), forceDarkEverywhere (dark theme on all YT pages)

### Wave 7 — Customization & Utilities
**All off by default:** customCssInjection (textarea for user CSS), shareMenuCleaner (hide social share buttons), autoClosePopups (cookie/survey/premium popup auto-dismiss), videoResolutionBadge (4K/HD badges on thumbnails), likeViewRatio (like:view % badge), downloadThumbnail (maxres download button), grayscaleThumbnails (grayscale until hover), disableAutoplayNext (clicks autoplay toggle off), channelSubCount (prominent sub count badge), customSpeedButtons (0.5x-3x preset bar), openInNewTab (video links open new tabs)

### Wave 8 — Restored Archive Features

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SysAdminDoc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
