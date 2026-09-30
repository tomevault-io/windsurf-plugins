---
trigger: always_on
description: AniZen is an Android anime/movie client (min SDK 26, Jetpack Compose + Material3, Voyager
---

# AniZen — Agent & Contributor Guide

AniZen is an Android anime/movie client (min SDK 26, Jetpack Compose + Material3, Voyager
navigation, SQLDelight, Injekt DI). `namespace = eu.kanade.tachiyomi`, `applicationId = app.anizen`.

AniZen descends from the Anikku/Aniyomi lineage, but **its git history was squashed** — the root
commit is a snapshot and `refs/original/*` is left over from a `git filter-branch`. AniZen therefore
shares **zero commits** with Anikku, and `git merge anikku/master` cannot find a merge base.

Everything that makes upstream syncing possible therefore lives in this repo's **conventions**, not
in git ancestry. The recorded content ancestor and all measurements live in
[`.anizen/upstream.json`](.anizen/upstream.json).

---

## Non-negotiables

### Git

| Rule | Required behaviour |
|------|--------------------|
| Branch | Work on a feature branch. `preview` is the active integration branch (CI: *Preview Build*). |
| Never | Commit or push directly to `master`. |
| Never | Rewrite history — it invalidates every published release tag and the `r<commitCount>` preview tag scheme. |
| Before push | Confirm `git branch --show-current` is not `master`. |

### Provenance markers

Wrap every line you **add or edit** for AniZen (imports excluded) in:

```kotlin
// ANZ -->
... your code ...
// ANZ <--
```

This is the AniZen equivalent of Anikku's `// ANK` convention. It is what turns an upstream sync
from guesswork into a mechanical review: during a merge you keep marked blocks and take upstream
everywhere else.

Preserve inherited markers — they map to other upstreams and must survive a merge:

| Marker | Origin | Merge rule |
|--------|--------|------------|
| `// ANZ` | AniZen-specific | yours; keep and update freely |
| `// ANK` | Anikku | keep intact during upstream merges |
| `// KMK` | Komikku | keep intact |
| `// SY` | TachiyomiSY | keep intact |
| `// AY` | Aniyomi | keep intact |
| `// EXH` | E-Hentai (legacy) | do not add new ones; use `// ANZ` |

**Backlog reality:** ~1,058 files have changed since the root snapshot and only 84 carry any marker.
`R3` in the provenance guard *reports* this backlog; it does not block it. New work is expected to
be marked.

### Strings / i18n

| String kind | Module | Resource class | Status |
|-------------|--------|----------------|--------|
| AniZen-only | `i18n-ank/` | `AMR` | **add all new strings here** |
| Mihon / Aniyomi base | `i18n/` | `MR` | frozen upstream |
| TachiyomiSY | `i18n-sy/` | `SYMR` | frozen upstream |
| Komikku | `i18n-kmk/` | `KMR` | frozen upstream |

- Never edit non-`base` locale files — Weblate owns translations
  (`salmanbappi/AniZen` on hosted.weblate.org).
- Never add AniZen strings to a frozen upstream module.
- Do not add `include(":i18n-aniyomi")` back to `settings.gradle.kts`; that module was removed and
  a missing project directory is an **error** in Gradle 9.

### Build verification

```bash
./gradlew spotlessApply           # fix formatting
./gradlew spotlessCheck           # must pass
./gradlew detekt                  # static analysis (CI gate)
./gradlew :app:compileDebugKotlin # compile-only, faster than assembleDebug
```

A local Android SDK is not always available; when it is not, CI is the compile check, but
`spotlessApply` and the provenance guard are still expected to pass before pushing.

---

## Upstream merging

Anikku rewrote its own `master` after 2025-04-19, so no ancestor exists to merge from. Use
`--merge-base` with the **recorded** content ancestor instead of relying on git ancestry.

### 1. Inspect what upstream changed

```bash
scripts/anz-upstream.sh report --mirror --fetch
```

Builds a blobless bare mirror at `.tmp/upstream-mirror` (a few hundred KB — safe on tight disks)
and writes [`provenance-report.md`](.anizen/provenance-report.md): identical / differing /
AniZen-only / upstream-only counts, plus the conflict surface.

### 2. See the merge without touching your tree

```bash
scripts/anz-upstream.sh merge            # local upstream ref -> upstream-sync branch
```

Runs `git merge-tree --write-tree --merge-base <recorded-base> preview <upstream>`. A clean result
becomes the `upstream-sync` branch; conflicts are listed and the command exits `2` **without**
creating a branch. It never touches your working tree.

### 3. Resolve

1. Keep every `// ANZ` block; take upstream outside the markers.
2. Preserve `// ANK`, `// KMK`, `// SY`, `// AY` blocks unless the review says otherwise.
3. Prefer upstream's structure and re-apply AniZen behaviour inside the markers.
4. Merge on a sync branch, then PR into `preview` — never merge straight into `preview`.

### 4. Guard before pushing

```bash
scripts/check-provenance.sh
```

### What to expect

| Target | Plain `git merge` | Base-corrected |
|--------|------------------:|---------------:|
| Fork-era ref (2026-02-05) | 663 conflicts | **2 conflicts** |
| Live upstream (2026-09-12) | — | 619 overlapping paths (upper bound) |

The live-upstream number is a tree-level upper bound: a base-corrected merge auto-resolves any path
whose edits do not overlap.

### Do not do this

```bash
git merge anikku/master                     # ✗ 663 conflicts: no merge base

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salmanbappi/AniZen](https://github.com/salmanbappi/AniZen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
